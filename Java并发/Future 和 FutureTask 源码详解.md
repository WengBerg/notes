# Future 和 FutureTask 源码详解
## Future 概述 
Future 就像它字面上的意思差不多就是“未来”的意思。在实际开发场景中有时需要异步调用一些接口或函数，但是这些接口或函数都不会立即返回结果且该结果不会影响后续程序的执行，这个时候使用Future就非常的适合，举例来说：
> 如查一个数据集合，第一页至第一百页，返回总页数的总结集，然后导出。一次需要limit 0 10000,这样，一个SQL查询出非常慢。但用100个线程，一个线程只查limit0 10 就非常快了，
利用多线程的特性，返回多个集合，在顺序合并成总集合。
---
## 常用方法源码分析
### 构造方法
> Future 是一个接口而 FutureTask 是它的常用实现类。

FutureTask(Callable<V> callable)
```
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable 设置任务的初始化状态
    }
```
FutureTask(Runnable runnable, V result)
```
    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result); // 调用该方法生成一个callable
        this.state = NEW;       // ensure visibility of callable
    }
```
> 该方法最终是通过 RunnableAdapter 来生成的 callable
```
    static final class RunnableAdapter<T> implements Callable<T> {
        final Runnable task;
        final T result;
        RunnableAdapter(Runnable task, T result) {
            this.task = task;
            this.result = result;
        }
        public T call() {
            task.run();
            return result;
        }
    }
```
> 这里实现了 Callable 接口
---
### run 方法
```
FutureTask<HashMap<String,String>> future = new FutureTask<>(new Callable<HashMap<String,String>>() 
{   这里实现call方法 }
new Thread(future).start();
```
> 这里的 start 方法会去调用 FutureTask 中 run 方法，这里就不讲这里如何要调用 run 方法了，想了解细节的话可以去看一下 Thread 的 start 方法里面讲的很清楚。在讲 run 方法之前先讲一下线程任务的运行状态：
```
     /** 
	 * 状态之间的转换如下：
	 * Possible state transitions:
     * NEW -> COMPLETING -> NORMAL
     * NEW -> COMPLETING -> EXCEPTIONAL
     * NEW -> CANCELLED
     * NEW -> INTERRUPTING -> INTERRUPTED
     */
    private volatile int state;
    private static final int NEW          = 0; // 初始化状态
    private static final int COMPLETING   = 1; // 完成中
    private static final int NORMAL       = 2; // 正常
    private static final int EXCEPTIONAL  = 3; // 异常
    private static final int CANCELLED    = 4; // 取消
    private static final int INTERRUPTING = 5; // 中断中
    private static final int INTERRUPTED  = 6; // 已中断
```
下面开始详细讲解 run 方法：
```
    public void run() {
        if (state != NEW || // 这里的 NEW 代表的是该线程任务的运行状态，此时代表的意思是初始化
            !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                         null, Thread.currentThread())) // compareAndSwapObject 设置当前线程为 runner 线程
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex); // 调用失败就讲异常设置到返回结果中
                }
                if (ran)
                    set(result);  // 调用成功则执行 set 讲执行结果设置到返回结果中
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
```
> 这里需要讲一下 set(result) 这个方法

```
    protected void set(V v) {
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {  // 设置线程任务的状态为执行中状态
            outcome = v; // 设置返回值
            UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state 设置线程任务的最终状态为正常状态
            finishCompletion(); // 移除等待的线程并对等待的线程进行唤醒操作及调用 done 并置空callable
        }
    }
```
> 这里就不讲 setException(ex) 这个方法了，因为它和 set 方法几乎一致。但是在这里我们看到了一个 finishCompletion 方法，那这个方法具体是怎么执行的呢？下面就来对它进行一个细致的分析：
```
    private void finishCompletion() {
        // assert state > COMPLETING;
        for (WaitNode q; (q = waiters) != null;) {
            if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) { // 设置等待线程的头节点置空
                for (;;) { // 自旋唤醒所有等待线程
                    Thread t = q.thread;
                    if (t != null) {
                        q.thread = null;
                        LockSupport.unpark(t);
                    }
                    WaitNode next = q.next;
                    if (next == null)
                        break;
                    q.next = null; // unlink to help gc
                    q = next;
                }
                break;
            }
        }

        done();

        callable = null;        // to reduce footprint
    }
```
---
### get 方法
```
    public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING) // 判断是否需要等待
            s = awaitDone(false, 0L);
        return report(s);
    }
``` 
> 等待方法 awaitDone
```
    private int awaitDone(boolean timed, long nanos)
        throws InterruptedException {
        final long deadline = timed ? System.nanoTime() + nanos : 0L;
        WaitNode q = null;
        boolean queued = false;
        for (;;) {
            if (Thread.interrupted()) { // 判断是否中断
                removeWaiter(q); // 移除等待节点
                throw new InterruptedException();
            }

            int s = state;
            if (s > COMPLETING) { // 这里大于 COMPLETING 表示已经执行完毕或者已经抛出异常
                if (q != null)
                    q.thread = null;
                return s;
            }
            else if (s == COMPLETING) // cannot time out yet
                Thread.yield(); // 使当前线程从执行状态（运行状态）变为可执行态（就绪状态）。说人话就是隔一会执行。
            else if (q == null)
                q = new WaitNode();
            else if (!queued)
                queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                     q.next = waiters, q); // 设置等待节点
            else if (timed) { // 这里是设置了超时时间才会调用
                nanos = deadline - System.nanoTime();
                if (nanos <= 0L) {
                    removeWaiter(q);
                    return state;
                }
                LockSupport.parkNanos(this, nanos);
            }
            else
                LockSupport.park(this);
        }
    }
```
> 设置返回值 report 方法
```
    private V report(int s) throws ExecutionException {
        Object x = outcome;  // 拿到返回值
        if (s == NORMAL)
            return (V)x;
        if (s >= CANCELLED)
            throw new CancellationException();
        throw new ExecutionException((Throwable)x);  // 其他的异常处理办法
    }
```
---
## 总结
这里主要讲了 run 方法和 get 方法。在 run 方法中是通过拿取和改变 state 的状态来判断和执行具体代码。在 run 方法中只要不出现异常，一开始 state 都是 NEW 。在是 NEW 的情况下才会去 runner 设置为当前线程。后续才会去调用具体的 call 方法，不管是抛出异常或者是拿到正确的执行结果都会设置到返回值当中。在设置返回值，又会去更改 state 的状态，之后就会对等待的线程依次进行唤醒（这里可能也没有等待线程，因为等待线程的出现是因为在异步执行的代码还未执行完拿到结果，这边就调用了 get 方法。此时就会出现等待线程），唤醒的线程就能拿到结果了。这里的 get 方法中的整体逻辑就相对来说要简单一些了，在 get 方法中首先通过 state 判断是否需要等待，若要等待就调用 awaitDone 方法去加入等待（这里会有逻辑去判断是否取消、中断、超时）。最后在调用 report 方法返回结果值（这里有逻辑会对异常进行一个处理）。其实这里都是通过改变和获取 state 的状态来对后续的操作进行一个判断。

