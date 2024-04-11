`@Configuration` 是 Spring 框架中的一个核心注解，用于标记一个类作为bean定义的源。在基于Java的Spring配置中，`@Configuration`类起着类似于传统XML配置文件的作用，它允许你使用方法级别的注解来声明bean。

### 特点

- **容器配置类**：标记有 `@Configuration` 的类通常定义一个或多个 `@Bean` 注解的方法，这些方法生成bean并交给Spring容器管理。
- **全功能配置**：与普通的 `@Component` 类不同，`@Configuration` 类允许定义 inter-bean dependencies。Spring会特别处理这些类，确保通过调用这些方法返回的bean是Spring容器中的单例。
- **CGLIB增强**：Spring 底层使用CGLIB库生成 `@Configuration` 类的子类，在运行时拦截对 `@Bean` 方法的调用，以确保每个bean只被创建一次。

### 使用示例

下面是一个简单的示例，展示了如何使用 `@Configuration` 和 `@Bean` 注解来配置Spring容器。

```java
@Configuration
public class AppConfig {

    // 定义一个bean，方法名即为bean的名称
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }

    // 定义另一个bean，注入上面定义的myService
    @Bean
    public MyController myController(MyService myService) {
        return new MyController(myService);
    }
}
```

在上面的例子中，`AppConfig` 是一个配置类，它定义了两个beans：`MyService` 和 `MyController`。

- `myService()` 方法定义了 `MyService` 类型的bean。这个方法创建并返回了 `MyService` 的一个实例。
- `myController()` 方法定义了 `MyController` 类型的bean。这个方法需要一个 `MyService` 类型的参数，Spring会自动注入之前通过 `myService()` 方法创建的 `MyService` bean。

当Spring容器启动时，它会扫描 `@Configuration` 标记的类，执行 `@Bean` 注解的方法，创建并注册相应的bean。

### CGLIB增强的细节

当 `AppConfig` 类中的 `@Bean` 方法在代码中被调用时（比如在另一个 `@Bean` 方法内部），Spring通过CGLIB创建的代理确保不会创建新的实例，而是返回已经创建的单例bean。这种增强机制保证了你在配置类内部相互调用 `@Bean` 方法时，始终只有一个bean实例被创建和管理。

可以简化为以下形式：

```java
// 原始的@Configuration类
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}

// CGLIB动态生成的子类
public class AppConfig$$EnhancerByCGLIB extends AppConfig {
    private MyService myServiceInstance;

    @Override
    public MyService myService() {
        if (myServiceInstance == null) {
            myServiceInstance = super.myService();
        }
        return myServiceInstance;
    }
}
```

实际的CGLIB子类会更复杂，并涉及到与Spring容器的交互，但上面的伪代码提供了一个基本的理解模型。这个增强机制是Spring框架中实现高级功能（如依赖注入和声明式事务管理）的基础。

### 结论

`@Configuration` 注解是Spring中实现Java-based配置的关键，它通过结合CGLIB代理和 `@Bean` 方法，提供了一种强大且灵活的方式来定义和管理bean的配置。这种方法比传统的XML配置更加类型安全，更易于重构，并且可以很好地利用现代Java语言的特性。