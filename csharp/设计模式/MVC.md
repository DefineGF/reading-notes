##### 静态结构

```shell
Model <------- Controller ----------> View
  ^                                     |
  |_____________________________________|
```

*   Controller 依赖Model和View;
*   View依赖Model;
*   Model本身独立;

代码地址: <https://github.com/DefineGF/bookmark/tree/master/csharp/DesignPattern/MVC>

#### 被动模式

 之所以称被动是因为 View被告知Model变化才发起更新行为.

- View需要在Controller更新了Model信息之后,接到更新自身的通知; 
- View重新从Model中读取的信息并展示.

过程：

1.  用户调用方法调用Controller方法, 对Model中数据进行变更;
2.  Controller变更Model数据的同时通知View: model数据更新了;
3.  View调用Model接口获取更新之后的数据;

##### 代码实现

接口:

```java
internal interface IView
{
    void Print(string data);
}

internal interface IModel
{
    int[] Data { get; }
}
```

实现:

```c
// 模拟控制台View
internal class ConsoleView : IView
{
    public void Print(string data)
    {
        Console.WriteLine("输出控制台:" + data);
    }
}
// 模拟日志View
internal class LogView : IView
{
    public void Print(string data)
    {
        Console.WriteLine("写入日志: " + data);
    }
}

// 实现随机数Model
internal class RandomModel : IModel
{
    public int[] Data
    {
        get
        {
            Random random = new Random();
            int[] result = new int[10];
            for (int i = 0; i < result.Length; i++)
            {
                result[i] = random.Next() % 1024;
            }
            return result;
        }
    }
}
```

Controller:

```c
internal class Controller
{
    private List<IView> _views = new List<IView>();
    private IModel _model;

    public virtual IModel Model
    {
        get { return _model; } 
        set { _model = value; }
    }
    // 开放接口: 更新Model数据之后, 通知View做出相应的改变
    public void Process()
    {
        if (_views.Count == 0) return;
        string results = string.Join(",", _model.Data); // Controller 更新 Model 信息

        foreach (var view in _views)
        {
            view.Print(results); // Controller 调用 View 更新显示
        }
    }
    public static Controller operator+(Controller controller, IView view)
    {
        if (view == null)
        {
            throw new ArgumentNullException("view");   
        }
        controller._views.Add(view);
        return controller;
     }

    public static Controller operator-(Controller controller, IView view)
    {
        if (view == null)
        {
            throw new ArgumentNullException("view");
        }
        controller._views.Remove(view);
        return controller;
    }
}
```

测试:

```c
Passive.Controller controller = new Passive.Controller();
controller.Model = new Passive.RandomModel();
controller += new Passive.ConsoleView();
controller += new Passive.LogView();
controller.Process();
```

#### 主动模式

被动模式无法满足当Model修改时,View做出响应.
主动模式: 为相应的Model注册相应的观察者, 数据变化时主动通知View, 而无需通过Controller参与.

##### 代码实现

ModelEventArgs: 使用EventHandler, 则必须实现EventArgs;

```c
internal class ModelEventArgs : EventArgs
{
    private readonly string content;
    public string Content { get { return content; } }
    public ModelEventArgs(int[] data)
    {
        this.content = string.Join(",", data);
    }
}
```

Model实现:

```c
// 接口
internal interface IModel
{
    event EventHandler<ModelEventArgs> DataChanged;
    int this[int index] { get; set; }
}
// 实现
internal class Model : IModel
{
    public event EventHandler<ModelEventArgs> DataChanged;
    private readonly int[] data;
    public Model()
    {
        Random rnd = new Random();
        data = new int[10];
        for (int i = 0; i < data.Length; i++)
        {
            data[i] = rnd.Next() % 1024;
        }
    }
    public int this[int index] { 
        get => data[index]; 
        set
        {
            this.data[index] = value;
            DataChanged?.Invoke(this, new ModelEventArgs(data));
        }
    }
}
```

这里需要解释一下:
子类无法调用父类的event; 某个类的事件只能在该类中触发, 在其他类中只能进行相关的 += 或者 -= 操作;
那为什么父类还要添加` event EventHandler<ModelEventArgs> DataChanged;` 呢?
因为接口中添加这个event, 那么子类在实现这个接口的时候, 如果没有添加 ` event EventHandler<ModelEventArgs> DataChanged;`, 则编译器会报错!!! 强制要求实现该接口的类添加相应的event.

View相关:

```c
internal interface IView
{
    EventHandler<ModelEventArgs> Handler { get; }
    void Print(string data);
}

internal abstract class ViewBase : IView
{
    public virtual EventHandler<ModelEventArgs> Handler
    {
        get { return this.OnDataChanged; }
    }
    public virtual void OnDataChanged(object sender, ModelEventArgs args)
    {
        Print(args.Content);
    }
    public abstract void Print(string data);
}

internal class ConsoleView : ViewBase
{
    public override void Print(string data)
    {
        Console.WriteLine("控制台输出: " + data);
    }
}
// LogView 同上
```

Controller相关:

```c
internal class Controller
{
    private IModel _model;
    public virtual IModel Model
    {
        get { return _model; }
        set { _model = value; }
    }
    
    // 与 被动模式相区别的是: 
    // 添加View的时候, 只是将 View中具体执行的Handler赋值给了Model中的event,
    // 而不是Controller持有View的指针
    public static Controller operator+ (Controller controller, IView view)
    {
        if (view == null)
        {
            throw new ArgumentNullException("view");
        }
        controller.Model.DataChanged += view.Handler;
        return controller;
    }

    public static Controller operator- (Controller controller, IView view)
    {
        if (view == null)
        {
            throw new ArgumentNullException ("view");
        }
        controller.Model.DataChanged -= view.Handler;
        return controller;
    }
}
```

测试:

```c
private static void ActiveTest()
{
    Active.Model model = new Active.Model();

    Active.Controller controller = new Active.Controller();
    controller.Model = model;
    controller += new Active.ConsoleView();
    controller += new Active.LogView();

    model[1] = -1; // 更新Model中数据, 通知View做出相应操作
    model[2] = -2;
}
```

