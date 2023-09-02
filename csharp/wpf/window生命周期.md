1. SourceInitialized：获取窗口的HwndSource（窗口句柄）属性时（窗口可见之前）发生；
2. App-Activated
3. Activated：用户切换到该窗口时发生（第一次加载也会引发）；相当于控件的GotFoucs事件；
4. loaded
5. ContentRendered：表明窗口完全可见并且准备好接收输入；

此时，最小化窗口再打开：

1. Deactivated
2. App-Activated
3. Activated

关闭窗口：

1. Closing
   - 关闭窗口、代码调用 Window.Close 或者 Application.Shutdown()方法关闭窗口时触发；
   - 提供了取消操作并保持打开状态的机会；
2. Deactivated：切换到其他窗口；关闭时也会触发
3. Closed：
   - 窗口已经关闭后；
   - 执行一些清理工作：向永久存储（配置文件 或者 windows 注册表）写入设置信息；
4. App-Exit
5. unload

 ![img](./C:/Users/cheng/Nutstore/1/JGWorkspace/C%23/WPF/基础/041228235951394.png)

##### 生命周期事件

Initialized：

- 元素被实例化，并已根据XAML标记设置了元素属性；
- 窗口的其他部分可能尚未初始化；

loaded：

- 整个窗口初始化并应用了样式和数据绑定；
- 此时 IsLoaded 为 true；

Unloaded：

- 元素被释放时；
- 原因是包含元素的窗口关闭或者特定的元素被删除；

##### 相关事件