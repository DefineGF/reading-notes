本地有两个项目`FacadeDemo` 和 `FeignDemo`

- FacadeDemo 有两个模块：
  - demo-facade：用于定义向外暴露的接口 interface，比如xx方法；
  - demo-facade-service：主要实现这些接口，并提供 restful api；
- FeignDemo：直接引入FacadeDemo的demo-facade，直接继承其接口（并不实现），然后使用 Feign 远程调用restful api 即可；

这样做有以下好处：

- 解耦合：接口和实现分离，你可以降低模块之间的依赖性。其他模块或服务只需要依赖于接口模块，而不是具体的实现。这意味着实现可以独立于接口进行变更或优化，只要它们遵守相同的接口契约。
- 可替换性：如果你决定用一个更高效的算法重新实现 AFacade，只要新实现遵循了原有的接口，其他依赖于该接口的代码无需修改即可继续工作。这对于维护和升级系统是非常有用的。
- 可测试性：接口的存在使得进行单元测试和集成测试变得更容易。你可以使用模拟（mocking）技术来模拟接口的实现，这样可以在不依赖具体实现细节的情况下测试系统的其他部分。

#### FacadeDemo 实现

根 pom.xml：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

##### 子模块 demo-facade

只定义了一个访问接口：
```java
public interface SimpleFacade {
    @GetMapping("/details/{id}")
    String getDetails(@PathVariable("id") Long id);
}
```



##### 子模块 demo-facade-service

首先引入模块 demo-facade（实现其接口）：
```xml
<artifactId>demo-facade-service</artifactId>
<dependencies>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>demo-facade</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

接口实现：

```java
@RestController
@RequestMapping("/api")
public class SimpleFacadeImpl implements SimpleFacade {

    @Override
    public String getDetails(Long id) {
        return "DATA from SimpleFacadeImpl";
    }
}
// 访问 localhost:8080/api//details/1 即可测试当前 getDetails 方法
```



#### FeignDemo 实现

首先需要将第一个项目编译安装一下：

```bash
mvn clean install
```

在当前项目引入依赖：
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version> <!-- 使用与Spring Boot版本兼容的Spring Cloud版本 -->
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>demo-facade</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>    <dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version> <!-- 使用与Spring Boot版本兼容的Spring Cloud版本 -->
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <!--引入Facade 模块依赖-->
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>demo-facade</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

注意：

> `<dependencyManagement>`：这是一个特殊的Maven块，用于管理依赖版本，但不会实际引入依赖项。它只是声明了一组依赖项及其版本，供项目中的其他模块引用。
>
> 例如，如果你使用的是 Spring Boot `2.2.2.RELEASE`，那么一个兼容的 Spring Cloud 版本可能是 `Hoxton.SR1` 或 `Hoxton.RELEASE`。



##### FeignClient 实现

```java
@FeignClient(name = "project-simple-service", url = "http://localhost:8080/api")
public interface SimpleFeignClient extends SimpleFacade {
}
// 间接调用另一项目服务：http://localhost:8080/api
```

实现原理：

1. **接口声明：** 在你的代码中，你声明了一个接口`SimpleFeignClient`，并在其上方添加了`@FeignClient`注解，指定了服务的名称和URL。
2. **动态代理：** 当应用程序启动时，Spring容器会扫描带有`@FeignClient`注解的接口，并为其创建一个动态代理对象。这个代理对象实现了被注解的接口（在这种情况下是`SimpleFeignClient`接口）。
3. **请求映射：** 在运行时，当你在代码中调用`SimpleFeignClient`接口的方法时，Feign会将这个方法调用转换为一个HTTP请求，并发送到指定的URL上。Feign会根据方法的注解（例如`@GetMapping`、`@PostMapping`等）来确定HTTP请求的类型和路径，并将方法参数转换为HTTP请求的参数。
4. **编码和解码：** Feign还负责将方法的返回值从HTTP响应中解码，并将其转换为你期望的Java对象。它也负责将方法的参数编码为HTTP请求的格式（例如JSON或表单数据），并将其发送到远程服务。
5. **负载均衡和服务发现：** 如果你的应用程序使用了服务发现机制（例如Eureka、Consul等），Feign会与服务注册中心交互，以确定要发送请求的目标服务的实际位置。如果服务配置了负载均衡，Feign还会负责选择一个实际的服务实例来处理请求。



##### 使用

```java
@RestController
public class BusinessController {
    private final SimpleFeignClient simpleFeignClient;

    @Autowired
    public BusinessController(SimpleFeignClient simpleFeignClient) {
        this.simpleFeignClient = simpleFeignClient;
    }

    @GetMapping("/fetch/{id}")
    public String getDetails(@PathVariable Long id) {
        return this.simpleFeignClient.getDetails(id);
    }
}
```

启动项设置：

```java
@SpringBootApplication
@EnableFeignClients
public class FeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignApplication.class, args);
    }
}
```



**最终测试：**

```bash
http://localhost:8081/fetch/1
```

