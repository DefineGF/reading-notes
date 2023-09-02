### BOM （Browser Object Models） 浏览器对象

#### Window

一旦页面加载，就会自动创建window对象；

常用方法：

- 文档显示区域的宽高：window.innerWidth;  window.innerHeight；
- 浏览器的宽高：window.outerWidth； window.outerHeight

##### 应用

浏览器上常见的弹出框：警告框（alert）、确认框（confirm）、输入框（prompt）等都是通过调用window方法实现的；



#### Navigator

即浏览器对象，提供浏览器相关的信息；

常用方法：

- 浏览器名称：navigator.appName						-> Netscape
- 浏览器版本号：navigator.appVersion                 -> 5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36
- 浏览器内部代码：navigator.appCodeName       -> Mozilla
- 操作系统：navigator.platform                              -> Win32
- 是否开启Cookie：navigator.cookieEnabled       -> true
- 浏览器用户代理报头：navigator.userAgent        -> Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36

#### Screen：表示屏幕的相关信息

- screen.width； screen.height
- 可用区域大小：screen.availWidth； screen.availHeight



#### History：用于记录访问历史

```js
function goBack() {   // 返回上一次的访问
	history.back();
}
function goBackMore() { // 返回上上次
    history.go(-2);     // -1 表示上次，-2表示上上次
}
```

#### Location：表示浏览器中的地址栏

```js
function refresh() {   // 刷新
	location.reload();  
}
function jump() {     // 跳转至首页
    // location = "/"; 
    location.assign("/");
}
```

- 协议：location.protocol               -> https:
- 主机号：location.hostname        -> how2j.cn
- 端口号：location.port                   -> 80 (没有即表示80端口)
- 主机&端口号：location.host        -> how2j.cn
- 访问路径：location.pathname    ->  /k/javascript/javascript-bom-location/451.html
- 锚点：location.hash                      
- 参数列表：location.search