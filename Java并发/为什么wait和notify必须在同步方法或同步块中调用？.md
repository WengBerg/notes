# 为什么wait和notify必须在同步方法或同步块中调用？
1. 不使用会导致IllegalMonitorStateException异常
2.  wait(),notify(),notifyAll() 这三个方法主要时用于实现线程之间的通信
    1. 其是这里的wait()方法是让线程等待并将锁释放出来，让给其他线程使用。
    2. notify(),notifyAll()是该线程在使用完锁后，通知其他线程可以获取锁继续执行下去。notify()是唤醒其中一个线程，notifyAll()是唤醒全部线程使其争抢。
3. 最后附上一个比较细致讲解的 [链接](https://leokongwq.github.io/2017/02/24/java-why-wait-notify-called-in-synchronized-block.html)