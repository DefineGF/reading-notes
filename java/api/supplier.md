### supplier

#### 应用场景

##### 延迟计算

`supplier` 可以用于延迟计算和构造值，直到这个值真正需要时才进行计算

```java
Supplier<Double> lazyValue = () -> {
    // 复杂计算
    return Math.random();
};

double value = lazyValue.get(); // 计算只发生在这里
```



##### 工厂方法

```java
Supplier<String> stringSupplier = () -> new String("New String");
System.out.println(stringSupplier.get());
```

使用 `Supplier` 可以延迟对象的创建直到真正需要这个对象时。虽然在这个特定示例中，延迟创建的好处不是很明显，但这种模式可以用于资源消耗较大的对象创建，或者在条件判断后才需要创建对象的情况。



##### 结合 optional

```java
Optional<String> optionalString = Optional.empty();
Supplier<String> stringSupplier = () -> "Default String";
// Supplier 提供了一个默认值，当 Optional 对象为空时使用
String result = optionalString.orElseGet(stringSupplier);
System.out.println(result); // 输出 "Default String"
```



##### 结合 stream

```java
Supplier<List<String>> listSupplier = ArrayList::new;
List<String> result = Stream.of("apple", "banana", "cherry")
    					    .collect(Collectors.toCollection(listSupplier));

// 上述写法的简化版
List<String> result = Stream.of("apple", "banana", "cherry")
                            .collect(Collectors.toList());
```

