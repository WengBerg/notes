# Spring Cloud Stream 简单使用
## 引入依赖

```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId> <!-- or '*-stream-kafka' -->
        </dependency>
```
## 消息生产/消费
### 生产者
首先定义一个接口：
```java
/**
 * 使用 stream 首先定义一个接口
 */
public interface StreamClient {

    String MSG = "myStreamMsg"; //发送消息的队列名称

    String RETURN_MSG = "returnMsg"; //消费消息成功返回消息的队列名称
    @Output(MSG)
    MessageChannel output();

    @Input(RETURN_MSG)
    SubscribableChannel input();
    
    // MessageChannel 发送消息类型
    // SubscribableChannel 订阅消息类型
}
```
生产者：

```java
/*
这里采用的 controoller 的方式来测试发送消息
 */
@RestController
@EnableBinding(StreamClient.class)  // 这里必须要绑定接口，否者就会启动报错，查找不到 streamClient
public class StreamSenderController {

    @Autowired
    private StreamClient streamClient;

     // 测试 stream 发送方法

/*    @GetMapping("sendMsg")
    public void send() {
        streamClient.output().send(MessageBuilder.withPayload(new Date().toString()).build());
    }

    @GetMapping("sendDate")
    public void send1() {
        streamClient.output().send(MessageBuilder.withPayload(new Date()).build());
    }*/
    @GetMapping("sendPerson")
    public void send2() {
        Person person = new Person();
        person.setName("Berg");
        person.setSex("man");
        streamClient.output().send(MessageBuilder.withPayload(person).build()); // 发送消息
    }

    @StreamListener(value = StreamClient.RETURN_MSG)
    public void returnMsg(String msg) {
        System.out.println("reurn message:"+msg);
    } // 消费消息后的返回
}
```

---
### 消费者
首先定义一个接口：
```
/**java
 * 使用 stream 首先定义一个接口
 */
public interface StreamClient {

    String MSG = "myStreamMsg";  // 消费消息队列

    String RETURN_MSG = "returnMsg"; // 返回队列
    @Input(MSG)
    SubscribableChannel input();

    @Output(RETURN_MSG)
    MessageChannel output();

}
```
消费者：

```java
@Component
@EnableBinding(StreamClient.class)  // 这里同样需要绑定接口
public class StreamReceiver {
    private final Logger logger = LoggerFactory.getLogger(StreamReceiver.class);

/*    @StreamListener(value = "myStreamMsg")
    public void receive(String msg) {
        logger.info("receive:{}",  msg);
    }

    @StreamListener(value = "myStreamMsg")
    public void receive1(Date date) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSXXX");
        logger.info("receive:{}",  dateFormat.format(date));
    }*/

    @StreamListener(StreamClient.MSG)  // 监听消费消息队列
    @SendTo(StreamClient.RETURN_MSG) // 返回的消息队列
    public String  receive2(Person p) {
        logger.info("name:{}",p.getName());
        logger.info("sex:{}",p.getSex());
        return "received message";
    }
}
```

---
## 问题 
### 重复订阅问题：
> 通常在生产环境，我们的每个服务都不会以单节点的方式运行在生产环境，当同一个服务启动多个实例的时候，这些实例都会绑定到同一个消息通道的目标主题（Topic）上。默认情况下，当生产者发出一条消息到绑定通道上，这条消息会产生多个副本被每个消费者实例接收和处理（出现上述重复消费问题）。但是有些业务场景之下，我们希望生产者产生的消息只被其中一个实例消费，这个时候我们需要为这些消费者设置消费组来实现这样的功能。

**解决方案:** 在消费者配置文件中添加如下配置
```
spring:
  cloud:
    stream:
      bindings:
        myStreamMsg: # 消息队列名称
          group: one # 如果这里不定义分组，就会出现多个实例重复消费的问题
```
### 消息类型问题：
> 在发送消息时有可能时对象，也有可能时文本。因此不同类型之间的转换可能会出现问题。默认为 json 类型
**解决方案：** 在生产者配置文件中配置消息类型

```
spring:
    stream:
      bindings:
        myStreamMsg:
          Content-Type: application/json  // 配置类型
```

---

## 代码
[github地址](https://github.com/WengBerg/studyspringcloud/tree/master)






