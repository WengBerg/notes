# Zuul
### 引入的依赖
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
```
### 启动类

```java
@SpringBootApplication
@EnableZuulProxy
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```
> 这里需要添加 @EnableZuulProxy 或 @EnableZuulServer 这两个注解当中的一个，这样才能表示该服务是一个 Zuul 网关。@EnableZuulProxy简单理解为@EnableZuulServer的增强版。事实上，当Zuul与Eureka、Ribbon等组件配合使用时，@EnableZuulProxy是我们常用的注解。

### yml文件配置
这里我们需要配置 eureka 服务发现注册中心，用来找到对应的服务。
如果需要配置自定义路由路径，需要像如下这样：

```yml
zuul:
  routes:
#    自定义路由
#    test:
#      path: /test/**
#      serviceId: product
#    简介写法
    product: /test/**
#    忽略的url（通过正则去匹配）
  ignored-patterns:
    - /test1/**
```
- 如何做到 zuul 可以转发cookie？
> 修改敏感头过滤（sensitiveHeaders）为空


