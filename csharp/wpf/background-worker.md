#### BackgroundWorker



##### 示例

在lbResults（ListBox）中显示打印能被42整除的数，并且实时更新 pbCalculationProgress (ProgressBar)

```c#
void Click(object sender, RoutedEventsArgs e) {
    pbCalculationProgress.Value = 0;
    lbResults.Items.Clear();

    BackgroundWorker worker = new BackgroundWorker();
    worker.WorkerReportsProgress = true;
    worker.DoWork += Worker_DoWork;
    worker.ProgressChanged += Worker_ProgressChanged;
    worker.RunWorkerCompleted += Worker_Completed;
    worker.RunWorkerAsync(10000);
}

void Worker_DoWork(object sender, DoWorkEventArgs e)
{
    int max = (int)e.Argument;
    int result = 0;
    for (int i = 0; i < max; i++)
    {
        int progressPercentage = Convert.ToInt32((double)i / max * 100);
        if (i % 42 == 0)
        {
            result++;
            (sender as BackgroundWorker).ReportProgress(progressPercentage, i); // (int percentProgress, object UserState)
        }
        else
        {
            (sender as BackgroundWorker).ReportProgress(progressPercentage);
        }
        System.Threading.Thread.Sleep(1);
    
    }
    e.Result = result;
}

void Worker_ProgressChanged(object sender, ProgressChangedEventArgs e)
{
    pbCalculationProgress.Value = e.ProgressPercentage;
    if (e.UserState != null)
    {
        lbResults.Items.Add(e.UserState);
    }
}
void Worker_Completed(object sender, RunWorkerCompletedEventArgs e)
{
    MessageBox.Show("counts is: " + e.Result);
}
```

##### 解析

主要包含三个事件：ProgressChanged、RunWorkerCompleted和DoWorker.

- 通过RunWorkerAsync() 方法传入参数，通过DoWorkEventArgs.Argument获取；

- DoWorker事件负责所有冗长的工作，在另一个线程中执行（因此不能涉及UI）；
- ProgressChanged和RunWorkerCompleted事件在创建 BackgroundWorker线程中执行（可操作UI）；
- 后台任务和UI的唯一通道 BackgroundWorker.ReportProgress(object) 实现；



##### 源码

```c#
namespace System.ComponentModel {
	public class BackgroundWorker : Component {
		public event DoWorkEventHandler DoWork;
		public event ProgressChangedEventHandler ProgressChanged;
		public event RunWorkerCompletedEventHandler RunWorkerCompleted;
	}
    
	// DoWork
   	// DoWorkEventArgs 用于捕获 RunWorkerAsync(object) 传入的参数
    public delegate void DoWorkEventHandler(object sender, DoWorkEventArgs e);
    public class DoWorkEventArgs : CancelEventArgs  // CancelEventArgs -> bool Cance
    {
        public DoWorkEventArgs(object argument);
        public object Argument {get; }
        public object Result {get; set; }
    }
    
    
    // ProgresseChanged
    // ProgressChangedEventArgs 用于捕获 ReportProgress(int, object) 传入的参数
    public event ProgressChangedEventHandler ProgressChanged;
    public delegate void ProgressChangedEventHandler(object sender, ProgressChangedEventArgs e);
    public class ProgressChangedEventArgs : EventArgs
    {
        public ProgressChangedEventArgs(int progressPercentage, object userState);
        public int ProgressPercentage { get; }
        public object UserState { get; }
    }
    
    // RunWorkerCompleted
    public event RunWorkerCompletedEventHandler RunWorkerCompleted;
    public delegate void RunWorkerCompletedEventHandler(object sender, RunWorkerCompletedEventArgs e);
    public class RunWorkerCompletedEventArgs : AsyncCompletedEventArgs
    {
        public RunWorkerCompletedEventArgs(object result, Exception error, bool cancelled);
        public object Result { get; }
        public object UserState { get; }
    }
    
}
```



##### 取消事件

点击取消按钮：

```C#
worker.CancelAsync();
```

Worker_DoWork 中：

```c#
void worker_DoWork(object sender, DoWorkEventArgs e)
{
	for(int i = 0; i <= 100; i++)
	{
		if(worker.CancellationPending == true)
		{
			e.Cancel = true;
			return;
		}
		worker.ReportProgress(i);
		System.Threading.Thread.Sleep(250);
	}
	e.Result = 42;
}
```

Worker_RunWorkerCompleted中：

```c#
void worker_RunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)
{
	if(e.Cancelled)
	{
		lblStatus.Foreground = Brushes.Red;
		lblStatus.Text = "Cancelled by user...";
	}
	else
	{
		lblStatus.Foreground = Brushes.Green;
		lblStatus.Text = "Done... Calc result: " + e.Result;
	}
}
```

