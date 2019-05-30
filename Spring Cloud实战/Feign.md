# Feign 简单使用
## 依赖添加
在 pom.xml 中添加如下依赖：
```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency> <!-- 这里少了版本号是因为默认为spring cloud的版本 -->
```

---
## 添加调用接口
在客户端中添加要调用的接口
- 接口：
```
@FeignClient(name = "product") // product 是服务名
public interface OrderClient {

    // 此处应该是一个除域名和端口外完整的请求路径
    @GetMapping("/product/msg")
    public String msg();

}
```
- controller 调用：

```
@RestController
@RequestMapping("order")
public class OrderController {

    @Autowired
    private OrderClient orderClient;

    @GetMapping("productMsg")
    public String productMsg() {
        String msg = orderClient.msg();
        return msg;

    }
}
```
最后注意在客户端启动类中添加@EnableFeignClients注解



