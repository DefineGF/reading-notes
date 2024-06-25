#### 三个服务

##### 服务提供者 service-module

```java 
@RestController
@Slf4j
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        log.info("HelloController.hello 执行方法");
        return "Hello, World!";
    }
}

// 启动项
@SpringBootApplication
public class ServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceApplication.class, args);
    }
}
```

配置：

```properties
server.port=9001
```



##### 服务消费者 feign-module

```java
// 定义feign 注册名以及访问地址
@FeignClient(name = "hello-service", url = "http://localhost:9001")
public interface HelloClient {
    @GetMapping("/hello")
    String hello();
}
```

配置：

```properties
feign.client.config.default.connect-timeout=5000
feign.client.config.default.read-timeout=5000
hello-service.ribbon.listOfServers=localhost:9001
```



##### 测试 test-module

```java
package com.cjm.feign;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.cloud.openfeign.EnableFeignClients;

// 表示测试期间不启动 Web 服务器
// 这意味着测试类运行在一个非 Web 的应用程序上下文中,不会监听任何端口,也不会发布任何 Web 相关的 Bean。
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)
// 指定 feignClient 所在ba
@EnableFeignClients(basePackages = "com.cjm.feign")
@Slf4j
public class HelloClientTest {

    @Autowired
    private HelloClient helloClient;

    @Test
    void testHello() {
        // 调用 HelloClient 的 hello 方法
        String result = helloClient.hello();

        // 验证结果
        System.out.println("获取调用结果: " + result);
    }
}
```

