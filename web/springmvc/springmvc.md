

#### 关键注解

##### @RequestMapping

```java
@Controller
public class MyController {
    // 映射 GET 请求到 /hello 路径
    @RequestMapping(path = "/hello", method = RequestMethod.GET)
    @ResponseBody
    public String hello() {
        return "Hello World!";
    }

    // 映射 POST 请求到 /submit 路径
    @RequestMapping(path = "/submit", method = RequestMethod.POST)
    @ResponseBody
    public String submit() {
        return "Form submitted!";
    }
}
```

- GET请求到 `http://localhost:8080/hello`
- POST请求到 `http://localhost:8080/submit`

其中：@RestController = @Controller + @ResponseBody



##### @RequestParam

```java
@RestController
public class MyController {
    // 使用 @GetMapping 来处理 GET 请求
    @GetMapping("/greet")
    public String greet(@RequestParam(name = "name", required = false, defaultValue = "World") String name) {
        return "Hello, " + name + "!";
    }
}
```

> 当访问：http://localhost:8080/greet?name=John
>
> Hello, John!
>
> 当访问：http://localhost:8080/greet
>
> Hello, World!



##### @PathVariable

```java
@RestController
public class UserController {
    
    @GetMapping("/users/{userId}")
    public String getUserById(@PathVariable("userId") String userId) {
        return "User Info for User ID: " + userId;
    }
}
```

当访问：http://localhost:8080/users/42 时：`42` 会被映射到 `getUserById` 方法的 `userId` 参数中。







