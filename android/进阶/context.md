### context

在 Android 中，Context 是一个抽象类，它的实例代表了当前应用程序的环境。Context 类本身并不是一个具体的实现类，而是一个抽象类，它的具体实现类包括 `Activity`、`Service`、`Application` 等。

底层原理上，Context 是通过 Android 系统为应用程序提供的一个接口，它是一个连接应用程序与 Android 系统的桥梁。它提供了一系列方法和属性，用于访问应用程序的资源、启动组件、获取系统服务等。

当应用程序启动时，系统会为每个应用程序创建一个 `Application` 对象，而 `Application` 类是 `Context` 类的子类。`Application` 对象是一个**全局的单例**对象，在应用程序的整个生命周期中都存在。

在不同的组件中，通过不同的方式获取到的 Context 对象可能不同：

1. Activity Context：在 Activity 中，可以通过 `this` 或 `Activity.this` 来获取当前 Activity 的 Context。Activity Context 是一个特殊的 Context 对象，它具有与 Activity 相关的特性和生命周期。
2. Application Context：在需要获取全局的 Context 或者对 UI 生命周期无依赖的情况下，可以使用 Application Context。可以通过 `getApplicationContext()` 方法获取到 Application Context。
3. Service Context：在 Service 中，可以通过 `this` 或 `Service.this` 来获取当前 Service 的 Context。Service Context 是一个特殊的 Context 对象，它具有与 Service 相关的特性和生命周期。
4. BroadcastReceiver Context：在 BroadcastReceiver 中，可以通过 `Context` 参数获取到当前的 Context。

总之，Context 是一个抽象类，它是 Android 应用程序与系统之间的桥梁，提供了访问资源、启动组件、获取系统服务等功能。具体的 Context 对象通过不同的组件类型和获取方式来获取，以满足不同的应用场景和需求。



#### 主要作用

1. 访问应用程序资源：通过 Context，你可以获取应用程序的资源，如字符串、图像、布局等。这是因为 Context 包含了应用程序的资源管理器，可以通过调用 `context.getResources()` 方法获取资源。
2. 启动组件：通过 Context，你可以启动其他组件，如启动 Activity、启动 Service 或发送广播。通过调用 `context.startActivity()、context.startService()、context.sendBroadcast()` 等方法，可以启动相应的组件。
3. 获取应用程序的包信息：通过 Context，你可以获取应用程序的包名、版本号等信息，如调用 `context.getPackageName()、context.getPackageManager().getPackageInfo()` 等方法。
4. 访问系统服务：通过 Context，你可以获取系统服务，如获取传感器服务、网络服务、定位服务等。通过调用 `context.getSystemService()` 方法，可以获取相应的系统服务。
5. 创建 View：在创建自定义 View 或者动态添加布局时，需要传递一个有效的 Context 对象。



##### 创建view为何需要 context

1. 访问应用程序的资源：View 组件通常需要访问应用程序的资源，如布局文件、字符串、图像等。通过传递 Context，View 可以使用 `context.getResources()` 方法获取应用程序的资源。
2. 获取系统服务：有时 View 可能需要与系统服务进行交互，如获取传感器、网络状态等。通过传递 Context，View 可以使用 `context.getSystemService()` 方法获取系统服务。
3. 主题和样式：Context 包含应用程序的主题和样式信息，这些信息可以在 View 的创建过程中使用和应用。
4. 资源解析：在 View 的创建过程中，需要进行布局文件的解析和处理。Context 提供了与解析相关的方法，如 `context.getAssets().open()` 可以用于打开布局文件。
5. 上下文环境：View 的创建和显示需要一个合适的上下文环境，以便正确地与应用程序进行交互。通过传递 Context，View 可以了解当前应用程序的环境和上下文



