### cglib

##### 基础类

```java
public interface UserService {
    String getName();
    void setName(String name);
}

public class UserServiceImpl implements UserService{
    private String name = "原始数据!";

    @Override
    public String getName() {
        return name;
    }

    @Override
    public void setName(String name) {
        this.name = name;
    }
}
```



#### jdk实现动态代理

```java
/**
 * 指定调用时操作
 */
public class MyInvocationHandler implements InvocationHandler {
    private Object target;

    public MyInvocationHandler() {
        super();
    }

    public MyInvocationHandler(Object target) {
        super();
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if ("getName".equals(method.getName())) {
            System.out.println("before invoke: " + method.getName());
            Object result = method.invoke(target, args);
            System.out.println("after invoke: " + method.getName());
            return result;
        } else {
            return method.invoke(target, args);
        }
    }
}
```

使用：

```java
UserService userService = new UserServiceImpl();
InvocationHandler invocationHandler = new MyInvocationHandler(userService);

UserService userServiceProxy = (UserService) Proxy.newProxyInstance(
        userService.getClass().getClassLoader(),
        userService.getClass().getInterfaces(),
        invocationHandler);
System.out.println(userServiceProxy.getName());
userServiceProxy.setName("cheng");              // 修改name属性
System.out.println(userServiceProxy.getName()); 
```





#### 核心类

##### net.sf.cglib.proxy.Enhancer 

主要的增强类

##### net.sf.cglib.proxy.MethodInterceptor

主要的方法拦截类;
最通用的回调（callback）类型，它经常被基于代理的AOP用来实现拦截（intercept）方法的调用;
只定义了一个方法:

```java
public Object intercept(Object object, 
    java.lang.reflect.Method method,
    Object[] args, 
    MethodProxy proxy) throws Throwable;
```

##### net.sf.cglib.proxy.MethodProxy

JDK的java.lang.reflect.Method类的代理类，可以方便的实现对源对象方法的调用