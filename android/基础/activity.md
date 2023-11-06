### Activity

#### 生命周期

##### 1. onCreate():

在 Activity 第一次创建时调用，用于进行初始化操作，如设置布局、绑定视图等

此时**Activity还在后台，不可见**。



##### 2. onStart():

Activity 可见但不在前台时调用，即 Activity 进入前台之前的状态。

动画的初始化在onStart中做比较好



##### 3. onResume(): 

在 Activity 进入前台并获取焦点时调用，此时用户可以与 Activity 进行交互。



##### 4. onPause(): 

在 Activity 失去焦点但仍然可见时调用，通常用于保存持久性数据或释放资源。



##### 5. onStop(): 

在 Activity 不可见时调用，可以执行一些清理工作。

在系统内存不足的时候**可能不会**执行onStop方法，因此程序状态的保存、独占设备和动画的关闭、以及一些数据的保存最好在onPause中进行



##### 6. onRestart():

在 Activity 从停止状态重新启动时调用，即重新进入前台之前的状态。



##### 7. onDestroy():

 在 Activity 被销毁之前调用，通常用于释放资源和取消注册的监听器。



##### 详情如下

![img](./activity_lifecycle.png)

##### 其他

- onSaveInstanceState(): 在 Activity 即将被销毁前调用，用于保存 Activity 的状态数据，以便在重建时恢复。
  - 活动被暂停或停止：当 Activity 失去焦点并即将进入暂停状态或停止状态时，系统会调用 onSaveInstanceState() 方法。这可以发生在用户按下 Home 按钮、切换到其他应用程序、屏幕旋转或配置更改等情况下。
  - 活动可能被销毁：当系统需要释放内存并决定销毁后台的 Activity 时，也会调用 onSaveInstanceState() 方法。这通常发生在系统内存不足时，为了回收资源而销毁 Activity。
- onRestoreInstanceState(): 在 Activity 重新创建时调用，用于恢复之前保存的状态数据。
  - `onCreate(Bundle savedInstanceState)`: Activity 被创建时调用。在这个方法中，你可以通过检查 `savedInstanceState` 参数是否为 null 来确定是否有之前保存的状态需要恢复。
  - 如果 `savedInstanceState` 不为 null，即存在之前保存的状态数据，则系统会调用 `onRestoreInstanceState(Bundle savedInstanceState)` 方法，并将之前保存的 Bundle 作为参数传递给该方法。你可以在这个方法中恢复之前保存的状态数据。
  - `onStart()`: 在 `onCreate()` 方法之后调用，表示 Activity 进入前台但不可见。
  - `onResume()`: 在 `onStart()` 方法之后调用，表示 Activity 可见并获取焦点。在这个方法中，你可以开始执行与界面交互的操作。

onSaveInstanceState 一定会被调用吗？

> `onSaveInstanceState()` 方法并不是一定会被调用的。它的调用取决于系统在特定情况下是否需要保存 Activity 的状态。因此，不能依赖于 `onSaveInstanceState()` 来保存重要的状态信息，尤其是那些持久性数据。
>
> 对于重要的状态信息，你可以考虑使用其他方式来保存，以确保其持久性和可靠性。以下是一些常见的方法：
>
> 1. 持久化到数据库：如果状态信息是需要长期保存的，例如用户的首选项设置或应用程序的配置信息，你可以将其保存到数据库中。这样可以确保数据在应用程序关闭和重新启动后仍然可用。
> 2. 使用 SharedPreferences：SharedPreferences 是一种轻量级的键值存储机制，用于存储简单的数据类型。你可以使用 SharedPreferences 将重要的状态信息保存到设备的持久存储中，以便在应用程序重新启动时进行恢复。
> 3. 使用文件存储：如果状态信息较大或复杂，你可以将其保存到文件中。可以使用内部存储或外部存储（如SD卡）来存储文件，并在需要时读取和写入数据。
> 4. 使用 ViewModel：ViewModel 是一种在应用程序组件之间共享数据的架构组件。你可以使用 ViewModel 来保存和管理重要的状态信息，并确保在配置更改（如屏幕旋转）时保持数据的一致性。

#### 其他状况

##### 点击两次返回桌面 (或者直接点击 home 键)

> 当用户点击两次返回按钮或导航栏上的返回按钮时，应用程序会进入后台并返回到设备的主屏幕（桌面）。在这种情况下，Activity 的生命周期方法的调用顺序如下：
>
> 1. onPause(): 当用户点击返回按钮时，当前的 Activity 将失去焦点并进入暂停状态，该方法会被调用。
> 2. onStop(): 当 Activity 不再可见时，即应用程序进入后台时，该方法会被调用。
>
> 请注意，当用户点击返回按钮并不是关闭应用程序，而是将应用程序放入后台运行。因此，系统会保留 Activity 的状态，并在用户再次返回应用程序时恢复。
>
> 当用户再次点击应用程序的图标或从最近任务列表中选择应用程序时，应用程序将重新进入前台并显示当前活动的 Activity。此时，以下方法将被调用：
>
> 1. onRestart(): 如果 Activity 是从停止状态重新启动，则会调用该方法。
> 2. onStart(): 在 Activity 进入前台但不可见时，该方法会被调用。
> 3. onResume(): 在 Activity 重新获取焦点并进入前台时，该方法会被调用。

##### 用户杀死当前应用

> 当用户从最近使用列表中删除应用程序的 Activity 时，系统会销毁该 Activity，并调用以下生命周期方法：
>
> 1. onPause(): 当用户从最近使用列表中删除应用程序时，当前的 Activity 将失去焦点并进入暂停状态，该方法会被调用。
> 2. onStop(): 当 Activity 不再可见时，即应用程序进入后台时，该方法会被调用。
> 3. onDestroy(): 当 Activity 被销毁时，该方法会被调用。

##### 一个 activity -> 另一activity

(A)onPause→(B)onCreate→(B)onStart→(B)onResume→(A)onStop

> 当一个 Activity 跳转到另一个 Activity 时，涉及到两个 Activity 的生命周期。以下是跳转前后两个 Activity 的典型生命周期调用顺序：
>
> 1. 跳转前的 Activity（调用顺序）：
>    - onPause(): 当前 Activity 失去焦点并进入暂停状态。
>    - onStop(): 当前 Activity 不再可见，即进入停止状态。
> 2. 跳转后的 Activity（调用顺序）：
>    - onCreate(): 新的 Activity 被创建。
>    - onStart(): 新的 Activity 进入前台但不可见。
>    - onResume(): 新的 Activity 获取焦点并进入前台。
>
> 在 Activity 跳转时，跳转前的 Activity 的生命周期方法会依次被调用，以暂停和停止当前 Activity 的运行。然后，跳转后的新的 Activity 会经历创建、开始和恢复的过程，最终成为前台可见状态。
>
> 需要注意的是，如果跳转后的 Activity 是通过 `startActivityForResult()` 方法启动的，那么在新的 Activity 完成后，会返回到跳转前的 Activity，并调用以下方法：
>
> - onActivityResult(): 跳转前的 Activity 在新的 Activity 完成后，会调用该方法以接收返回的结果。

##### 返回跳转前的activity

当用户从跳转后的 Activity 返回到跳转前的 Activity，或者通过系统的返回按钮或调用 `finish()` 方法关闭跳转后的 Activity 时，跳转前的 Activity 的生命周期方法会按照相反的顺序被调用：

- onRestart(): 跳转前的 Activity 重新启动。
- onStart(): 跳转前的 Activity 进入前台但不可见。
- onResume(): 跳转前的 Activity 获取焦点并进入前台。



##### 其他

当更高优先级的程序需要内存时，优先级低的activity会执行 onDestory，还是只执行onStop？

> 当系统需要释放内存时，优先级低的 Activity 可能会执行 `onStop()` 方法，也可能执行 `onDestroy()` 方法，具体取决于系统的内存管理策略和当前 Activity 的状态。
>
> 一般情况下，当一个 Activity 处于后台不可见状态（即已经调用了 `onStop()`），而系统需要释放内存时，该 Activity 可能会被销毁，调用 `onDestroy()` 方法。这是因为后台不可见的 Activity 对用户体验的影响较小。
>
> 然而，如果一个 Activity 仍然处于前台可见状态，但系统仍然需要更多内存，那么**系统可能会选择直接终止该 Activity，而不调用 `onStop()` 或 `onDestroy()` 方法**。这种情况下，该 Activity 将被系统直接杀死，以释放所需的内存。
>
> 需要注意的是，无论是执行 `onStop()` 还是 `onDestroy()`，它们都表示 Activity 的生命周期结束。在 `onStop()` 方法中，Activity 进入了停止状态，不再可见；而在 `onDestroy()` 方法中，Activity 被销毁，释放了所有资源。因此，你应该确保在 `onStop()` 或 `onDestroy()` 方法中适当地处理和释放资源，以避免潜在的内存泄漏和其他问题。



#### 启动模式

##### standard - 默认模式

每次启动一个Activity都会重新创建一个新的实例入栈，不管这个实例是否存在。



##### singleTop：栈顶复用

需要创建的Activity已经处于栈顶时，此时会直接复用栈顶的Activity，不会再创建新的Activity；若需要创建的Activity不处于栈顶，此时会重新创建一个新的Activity入栈。

栈顶的Activity被直接复用时，它的onCreate、onStart不会被系统调用，因为它并没有发生改变，但是一个新的方法 onNewIntent会被回调（Activity被正常创建时不会回调此方法）。

> 在 `onNewIntent(Intent intent)` 方法中，你可以根据新的 Intent 进行相应的逻辑处理，例如更新界面、处理数据等。你可以通过调用 `getIntent()` 方法来获取传递给该 Activity 的新 Intent。

**应用场景：**

 如果你在当前的Activity中又要启动同类型的Activity，此时建议将此类型Activity的启动模式指定为SingleTop，可以减少Activity的创建，节省内存。



##### singleTask：栈内复用

若需要创建的Activity已经处于栈中时，此时不会创建新的Activity，而是将存在栈中的Activity上面的其它Activity全部销毁，使它成为栈顶。

**应用场景：**

应用中展示的主页（Home页）：假设用户在主页跳转到其它页面，执行多次操作后想返回到主页，如果不使用SingleTask模式，在点击返回的过程中会多次看到主页。



##### singleInstance：单实例模式

具有此模式的Activity只能单独位于一个任务栈中。这个常用于系统中的应用，例如Launch、锁屏键的应用等等，整个系统中只有一个