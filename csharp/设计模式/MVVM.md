##### Model

用作数据处理源: 保存所有添加过的数据

```java
public class HelloWorldModel
{
    private readonly List<string> _data;

    public HelloWorldModel()
    {
        _data = new List<string>
        {
            "cheng",
            "铭"
        };

    }
    
    // 添加数据
    public void AddData(string data)
    {
        _data.Add(data);
    }

    // 获取数据
    public string ImportMessage()
    {
        return string.Join("-", _data);
    }
}
```

##### ViewModel

持有Model, 用以控制数据的操作流程;

```java
public class HelloWorldViewModel : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    private readonly HelloWorldModel _model;

    public string Data
    {
        get => _model.ImportMessage();
        set 
        {
            _model.AddData(value);
            OnPropertyChanged();
        }
    }

    public HelloWorldViewModel()
    {
        _model = new HelloWorldModel();
    }

    protected void OnPropertyChanged([CallerMemberName]string name = null)
    {
        PropertyChanged?.Invoke(this, 
                new PropertyChangedEventArgs(name));
    }
}
```

解释一下:

- INotifyPropertyChanged: 需要添加 PropertyChangedEventHandler;
- Data: 用以view的属性绑定, 当Data属性修改时, View做出相应显示变化;但是要调用相应的方法OnPropertyChanged()
- 方法中CallerMemberName类似于java中的注解, 即传入方法调用者名称, 比如当前就是 "Data", 用以区别那个属性变化了.

##### view

```xml
    <!-- 也可以在后置代码设置: this.DataContext = _viewModel; 
    <Window.DataContext>
        <vm:HelloWorldViewModel/>
    </Window.DataContext>-->
<Grid>
    <TextBlock x:Name="label" FontSize="28" Text="{Binding Data}"
           HorizontalAlignment="Center" VerticalAlignment="Center"/>
    <StackPanel>
        <TextBox Name="TBInput" FontSize="25" Text="输入内容"/>
        <Button Content="commit" FontSize="18" Click="BtnCommit_Click"/>

    </StackPanel>
</Grid>
```

后置代码:

```java
public partial class HelloWorldView : Window
{
    private readonly HelloWorldViewModel _viewModel;
    public HelloWorldView()
    {
        InitializeComponent();
   
        _viewModel = new HelloWorldViewModel();
        this.DataContext = _viewModel; // 可以在xaml文件中配置
    }

    private void BtnCommit_Click(object sender, RoutedEventArgs e)
    {
        string input = TBInput.Text.ToString();
        _viewModel.Data = input;
    }
}
```