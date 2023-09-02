##### 默认存放地址

data/data/<package>/shared_prefs



##### 获取SharePreferences 对象

- Context.get**Shared**Preferences(String name,int mode)

  直接被保存在/data/data/<package name>/shared_prefs路径下，名字为 name.xml

  - name: 文件名
  - mode: 操作模式
    - MODE_PRIVATE：只有当前的应用程序才可以对这个SharedPreferences文件进行读写（默认）
    - MODE_MULTI_PROCESS：一般是用于会有多个进程中对同一个 SharedPreferences文件进行读写的情况。



- Activity类中的 getPreferences(int mode)

  使用这个方法时会自动将**当前活动的类名**作为 SharedPreferences的文件名；

  比如当前 MainAactivity 中 使用 getPreference， 那么shared_pref中创建的文件为：MainActivity.xml



- PreferenceManager类中的 getDefaultSharedPreferences(Context context)方法

  静态方法。它接收一个 Context 参数，并自动使用当前**应用程序**的包名作为前缀来命名 SharedPreferences文件。



##### 使用步骤

1. 调用 SharedPreferences对象的 edit()方法来获取一个 SharedPreferences.Editor 对象。
2. 向 SharedPreferences.Editor 对象中添加数据，添加布尔类型putBoolean(),字符串putString()
3. 调用 commit()方法将添加的数据提交，从而完成数据存储操作。

```java
SharedPreferences.Editor editor = getSharedPreferences("data", MODE_PRIVATE).edit();
editor.putString("name", "Tom");
editor.putInt("age", 28);
editor.putBoolean("married", false);
editor.apply();
```

获取数据：

```java
SharedPreferences pref = getSharedPreferences("data",MODE_PRIVATE);
String name = pref.getString("name", "");
int age = pref.getInt("age", 0);
boolean married = pref.getBoolean("married", false);//获取无结果设置为false
```

