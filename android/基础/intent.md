#### Intent

##### 操作

- setAction() 或者 Intent中设置：

  -  **ACTION_VIEW**：倘若要向Activity传递可以显示的内容时，与startActivity()配合使用   
  -  **ACTION_SEND：**拥有其他应用可以共享的数据，与startActivity()配合使用

- setData()

  - ACTION_EDIT：包含可编辑文档的uri；若为MIME类型，可通过setType()  如有必要，可通过setDataAndType()同时设置两者

- addCategory()

  - **CATEGORY_BROWSABLE:**    目标activity允许通过浏览器访问链接 
  - **CATEGORY_LAUNCHER:**     初始化Activity，在系统的启动器中列出

- Extra：

      附加请求操作的信息键值对，可通过putExtra添加键值对，也可以通过putExtras添加Bundle

- **标志**：setFlag()

  - FLAG_ACTIVITY_NEW_TASK： 
  - FLAG_ACTIVITY_SINGLE_TOP：与launchMode 值为 singleTop 产生的行为相同；   
  - FLAG_ACTIVITY_CLEAR_TOP

##### 显式Intent

```java
Intent downloadIntent = new Intent(this, DownloadService.class);
downloadIntent.setData(Uri.parse(fileUrl));
startService(downloadIntent);
```

指明类名启动activity或者服务（启动服务务必使用显式Intent，且指定组件名称；隐式存在安全隐患，无法确定启动哪个服务，且无法确定服务是否启动。因此显示也避免无意中启动其他应用的service）



##### 隐式Intent

声明要执行的常规操作；  

系统通过将Intent内容与设备上其他应用的清单文件声明的Intent过滤内容比较；相同则启动相应的组件；  

可以调用系统上任何应用;

```java
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType("text/plain");

//由于应用或者管理员的限制，可能无法调用访问应用，也就会导致当前应用崩溃；
//因此调用 resolveActivity() 来查看是否有接收Intent的应用
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(sendIntent);
}
```

接收隐式 Intent：

```xml
<activity android:name="ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
```

**若想确保只有自己的应用才能启动自己的某一组件，请勿在清单文件中设置intent-filter，而是将activity中的exported设置成false**



##### 进阶

使用Parcelable 和 Bundle封装向另一进程传递数据；

（当向另一应用使用Intent发送数据时，请勿使用Parcelable或者Serializable，对方若无序列化类，则会抛出RuntimeException）

````java
public class LoginInfo implements Parcelable {
    private String name;
    private int    age;
    private String psw;

    private LoginInfo(Parcel in) {
        this.name = in.readString();
        this.age  = in.readInt();
        this.psw  = in.readString();
    }

    public LoginInfo(String name,int age,String psw){
        this.name = name;
        this.age  = age;
        this.psw  = psw;
    }

    @NonNull
    @Override
    public String toString() {
        return "name --> " + name +" ;age --> " + age +" ;psw --> "+ psw;
    }

    @Override
    public int describeContents() {//返回默认值即可
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(age);
        dest.writeString(psw);
    }
    
    //底层调用
    public static final Creator<LoginInfo> CREATOR = new Creator<LoginInfo>() {
        @Override
        public LoginInfo createFromParcel(Parcel in) {
            return new LoginInfo(in);
        }

        @Override
        public LoginInfo[] newArray(int size) {
            return new LoginInfo[size];
        }
    };
}
````

使用：

```java
//启动
LoginInfo info = new LoginInfo("cheng",22,"1234");
Bundle bundle = new Bundle();
bundle.putParcelable("login_info",info);
Intent intent = new Intent(this,IntentReceiverActivity.class);
intent.putExtras(bundle);                   //可简写为：intent.putExtra("login_info",info)；put Bundle记得extra加s
startActivity(intent);

//接收
Intent intent = getIntent();
LoginInfo info = intent.getParcelableExtra("login_info");
```