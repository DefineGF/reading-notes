### stream

##### stream.generate()

```JAVA
Stream<Integer> randomStream = Stream.generate(() -> new Random().nextInt());
// 限制流的大小为10，并打印每个元素
randomStream.limit(10).forEach(System.out::println);
```

封装写法：

```java
public class RandomCharGenerator implements Supplier<String> {
    Random random = new Random(47);
    char[] letters = "ABCDEFGHIJKLMNOPQRSTUVWX".toCharArray();

    @Override
    public String get() {
        return "" + letters[random.nextInt(letters.length)];
    }

    public static void main(String[] args) {
        // Stream.generate 每次调用 RandomCharGenerator 中的 get 函数获取一个结果
        String word = Stream.generate(new RandomCharGenerator())
                .limit(10)
                .collect(Collectors.joining());
        System.out.println("result: " + word);
    }
}
```



#### Optional

常见用法：

```java
String str = null;
Optional<String> optionalStr = Optional.ofNullable(str);

// 默认值
sout(optionalStr.orElse("Default Value"));

// 检查是否有值
optionalStr.isPresent();

// 执行后续
optionalStr.isPresent(val -> sout("val: " + val));
```

