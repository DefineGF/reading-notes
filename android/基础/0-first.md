#### app-build.gradle

初始内容：

```yaml
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
android {
 compileSdkVersion 29
 buildToolsVersion "29.0.2"
 
 defaultConfig {
 applicationId "com.example.helloworld"
 minSdkVersion 21
 targetSdkVersion 29
 versionCode 1
 versionName "1.0"
 testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
 }
 buildTypes {
 release {
 minifyEnabled false
 proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
 'proguard-rules.pro'
 }
 }
}
dependencies {
 implementation fileTree(dir: 'libs', include: ['*.jar'])
 implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
 implementation 'androidx.appcompat:appcompat:1.1.0'
 implementation 'androidx.core:core-ktx:1.1.0'
 implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
 testImplementation 'junit:junit:4.12'
 androidTestImplementation 'androidx.test.ext:junit:1.1.1'
 androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
}
```



apply plugin：

- com.android.application 表示这是一个应用程序模块；
  - com.android.library表示这是一个库模块；
- kotlin-android：kotlin开发必须
- kotlin-android-extensions：kotlin 扩展

android：

- compileSdkVersion用于指定项目的编译版本；29 表示 android 10；
- buildToolsVersion用于指定项目构建工具的版本；

defaultConfig：

- applicationId：应用的唯一标识符；

- minSdkVersion用于指定项目最低兼容的Android系统版本；21 对应 android 5

- targetSdkVersion 29：表示你在该目标版本上已经做过了充分的测试，系统将会为你的应用程序启用一些最新的功能和特性；

  > 比如Android 6.0系统中引入了运行时权限这个功能，如果你将targetSdkVersion指定成23或者更高，那么系统就会为你的程序启用运行时权限功能，
  >
  > 而如果你将targetSdkVersion指定成22，那么就说明你的程序最高只在Android 5.1系统上做过充分的测试，Android 6.0系统中引入的新功能自然就不会启用了
  >
  > 