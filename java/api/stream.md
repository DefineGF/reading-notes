#### stream



##### 输出集合

转换为list：

```java
Stream<String> stream = Stream.of("Apple", "", null, "Pear", "  ", "Orange");
List<String> list = stream.filter(s -> s != null && !s.isBlank())
    .collect(Collectors.toList());
```

转换为map：

```java
Stream<String> stream = Stream.of("APPL:Apple", "MSFT:Microsoft");
Map<String, String> map = stream.collect(Collectors.toMap(
                        // 把元素s映射为key:
                        s -> s.substring(0, s.indexOf(':')),
                        // 把元素s映射为value:
                        s -> s.substring(s.indexOf(':') + 1)));
```

