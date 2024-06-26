### Class

#### 获取Class

```java
public interface Calculator {
    int add(int a, int b);
    int subtract(int a, int b);
}

public class CalculatorImpl implements Calculator {
    @Override
    public int add(int a, int b) {
        return a + b;
    }
    @Override
    public int subtract(int a, int b) {
        return a - b;
    }
}

Calculator calculator = new CalculatorImpl();
```

获取Class方法：

- calculator.getClass()
- CalculatorImpl.class
- Class.forName("com.cjm.onjavabasic._proxy.CalculatorImpl")



##### calculator.getClass()

必须先创建类的实例对象,然后通过该对象调用getClass()方法；

> 尽管 `calculator` 也是由 `CalculatorImpl` 类实例化的，但是它被声明为 `Calculator` 接口类型。不过，它仍然是 `CalculatorImpl.class`。这是因为 `getClass()` 方法返回的是实际运行时对象的具体类，而不是变量的声明类型。
>
> **运行时类型信息：**
>
> - Java 对象在内存中有一个类元数据的引用，这部分数据包含了关于类的详细信息，包括类名、方法、属性等。
> - 当调用 getClass() 方法时，它实际上是直接访问这个内部的类元数据来返回对象的实际运行时类。
>
> 如：
>
> ```java
> Object obj = new String("Hello");
> Class<?> cls = obj.getClass();
> System.out.println(cls.getName());  // 输出 java.lang.String
> ```

getClass() 底层实现：

- **获取对象头**：每次对象的 `getClass()` 方法被调用时，JVM 首先访问该对象的对象头；
- **访问类元数据**：从对象头中读取指向类元数据的指针；
- **返回Class对象**：使用从对象头中获取的指针，找到对应的 `Class` 对象，并将其返回。

特殊情况：动态代理

```java
Calculator calculator = (Calculator) Proxy.newProxyInstance(
    Calculator.class.getClassLoader(),
    new Class<?>[] { Calculator.class },
    new InvocationHandler() {
        private CalculatorImpl calculatorImpl = new CalculatorImpl();
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            // 可以在这里添加额外的逻辑
            return method.invoke(calculatorImpl, args);
        }
    });
Class<? extends Calculator> cls = calculator.getClass();
// calculator.getClass() 将不会返回 CalculatorImpl.class，而是返回一个由 Proxy 类动态生成的类: com.sun.proxy.$Proxy0
```



##### CalculatorImpl.class

每个类都有一个名为class的静态final字段,它表示该类的Class对象：

> 在Java中,当一个类被加载到Java虚拟机(JVM)中时,JVM会为该类创建一个唯一的Class对象,用于表示该类的元信息,如类名、包名、父类、接口、构造方法、普通方法、字段等。每个类都对应一个独一无二的Class对象。
>
> 当一个类被加载后,JVM会在堆内存中创建一个Class对象,并将其地址赋值给该类的一个特殊的静态final字段,这个字段就是class。因此,每个类都有一个名为class的静态final字段,它的类型是Class,且是在类加载时由JVM自动赋值的。





##### Class.forName()

运行时动态加载类,并返回该类的Class对象；如果指定的类不存在,或者类名错误,会抛出ClassNotFoundException异常。

应用场景：

- 当你在编写代码时无法确定要使用的类,类名需要在运行时动态指定或从配置文件中读取时,可以使用这种方式获取Class对象。
- 常用于框架、插件化开发、反射等场景中,需要根据配置或用户输入动态加载类。



##### 具体区分

类的生命周期：

1. 加载：JVM通过类加载器读取二进制数据（通常是.class文件）并将这些数据转换成一个在内存中的类。这个过程包括通过系统类加载器、扩展类加载器或应用程序类加载器将类的全限定名转换成对应的字节码。
2. 链接：
   - 验证：确保加载的类符合JVM规范，没有安全问题；
   - 准备：为类变量分配内存，并设置类变量的默认初始值；
   - 解析：JVM将所有符号引用转换成直接引用
3. 初始化：初始化是执行类构造器`<clinit>()`方法的过程，这个方法由编译器自动收集类变量的赋值动作和静态代码块中的语句合成。这个阶段是执行静态初始化代码和静态变量初始化。

因此针对以下类：

```java
public class HelloWorld {
    static int number = 10; // 静态变量初始化
    static {
        System.out.println("Static block initialized.");
        number = 15;
    }
}
```

- Class.forName("HelloWorld")：加载、链接和初始化；
- HelloWorld.class：加载、链接；
- 访问静态变量：加载、链接和初始化；

