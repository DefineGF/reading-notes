##### http 常见媒体类型

-   text/html ： HTML格式
-   text/plain ：纯文本格式   
-   text/xml ： XML格式
-   image/gif ：gif图片格式  
-   image/jpeg ：jpg图片格式 
-   image/png：png图片格式
-   application/xhtml+xml ：XHTML格式
-   application/xml   ： XML数据格式
-   application/atom+xml ：Atom XML聚合格式  
-   application/json  ： JSON数据格式
-   application/pdf    ：pdf格式 
-   application/msword ： Word文档格式
-   application/octet-stream ： 二进制流数据（如常见的文件下载）
-   application/x-www-form-urlencoded ： <form encType="">中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）



#### RequestMapping 不常用字段

##### content-type

在Http协议消息头中，使用Content-Type来表示具体请求中的媒体类型信息；

`consumes`属性指定了控制器方法可以接受的请求的`Content-Type`。如果HTTP请求的`Content-Type`与`consumes`属性定义的类型不匹配，请求将不会被映射到该方法。

```java
@RequestMapping(
        value = "/post/data",
        method = RequestMethod.POST,
        consumes = "application/json"
)
public ResponseEntity<String> postData(@RequestBody MyData data) {
    // 处理data
    return ResponseEntity.ok("Processed data");
}
```



##### Accept

如果HTTP请求的`Accept`头部字段与`produces`属性定义的类型不匹配，请求将不会被映射到该方法。

```java
@RequestMapping(
        value = "/get/data",
        method = RequestMethod.GET,
        produces = "application/json"
)

public ResponseEntity<MyData> getData() {
    MyData data = new MyData();
    // 填充data
    return ResponseEntity.ok(data);
}
```

