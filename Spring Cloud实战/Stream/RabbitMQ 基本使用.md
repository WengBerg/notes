# RabbitMQ 基本使用
## 引入依赖
**pom.xml** 中添加：
```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId> <!-- or '*-stream-kafka' --> 
            <!-- 采用 spring cloud 大版本 -->
        </dependency>
```

---
## 配置RabbitMQ
在 **application.yml** 或 **bootstrap.yml** 亦或是在 **config** 配置中心库中添加配置：

```
spring:
  rabbitmq:
    host: 127.0.0.1
    username: guest
    password: guest
    port: 5672
```

---

## 消息生产/消费
### 生产者

```
@Component
public class MySender extends OrderApplicationTests {  // 这里继承的测试类

    @Autowired
    private AmqpTemplate amqpTemplate;

    @Test
    public void send() {
        amqpTemplate.convertAndSend("myQueue",new Date().toString());
    }

    /*
    测试服务1.发送
     */
    @Test
    public void send1() {
        amqpTemplate.convertAndSend("server","server1",new Date().toString());
    }

    /*
    测试服务2.发送
     */
    @Test
    public void send2() {
        amqpTemplate.convertAndSend("server","server2",new Date().toString());
    }
}
```

---
### 消费者

```
@Component
public class MyReceiver {
    private final Logger logger = LoggerFactory.getLogger(MyReceiver.class);

    // @RabbitListener(queues = "myQueue")  这里需要在 rabbitmq 中手动添加 myQueue 队列，不然就会报错
    // @RabbitListener(queuesToDeclare = @Queue("myQueue"))  // 自动声明 myQueue 队列
    @RabbitListener(bindings = @QueueBinding( // 自动创建，Exchange和Queue绑定，具体绑定还应该添加 key 参数。消息分组
            value = @Queue("myQueue"),
            exchange = @Exchange("myExchange")
    )
    )
    public void receive(String msg) {
        logger.info("receive:{}",msg);
    }

    /**
     * 服务1，Exchange 绑定测试
     * @param msg
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("server1"),
            key = "server1",
            exchange = @Exchange("server")
    ))
    public void recevice1(String msg) {
        logger.info("receive server1:{}",msg);
    }

    /**
     * 服务2，Exchange 绑定测试
     * @param msg
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("server2"),
            key = "server2",
            exchange = @Exchange("server")
    ))
    public void recevice2(String msg) {
        logger.info("receive server2:{}",msg);
    }
}
```
> 这里的服务1和服务2的不同绑定，主要是为了实现业务上面多个服务发送消息到一个服务时，对不同服务消息的区分



