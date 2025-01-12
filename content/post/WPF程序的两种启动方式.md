+++
date = '2024-12-29T20:03:57+08:00'
draft = false
title = 'WPF程序的两种启动方式'
tags = ["WPF"]
categories = ["技术博客"]
author = "火箭"
+++
## 默认启动方式

新建一个名为 WpfApp1 的 WPF 程序，该程序默认包含两个类：

1. App.xaml
    1. App.cs
2. MainWindow.xaml
    1. MainWindow.cs

创建完成后，直接按 F5 启动可打开主窗口。那么，此程序是如何启动的呢？

通过 dotPeek 反编译生成的编译文件 WpfApp1.exe，反编译结果如下：

```csharp
namespace WpfApp1 {
    /// <summary>
    /// App 类
    /// </summary>
    public partial class App : System.Windows.Application {
        /// <summary>
        /// InitializeComponent 方法
        /// </summary>
        [System.Diagnostics.DebuggerNonUserCodeAttribute()]
        [System.CodeDom.Compiler.GeneratedCodeAttribute("PresentationBuildTasks", "8.0.5.0")]
        public void InitializeComponent() {
            #line 5 "..\..\..\App.xaml"
            this.StartupUri = new System.Uri("MainWindow.xaml", System.UriKind.Relative);
            #line default
            #line hidden
        }
        /// <summary>
        /// 应用程序入口点
        /// </summary>
        [System.STAThreadAttribute()]
        [System.Diagnostics.DebuggerNonUserCodeAttribute()]
        [System.CodeDom.Compiler.GeneratedCodeAttribute("PresentationBuildTasks", "8.0.5.0")]
        public static void Main() {
            WpfApp1.App app = new WpfApp1.App();
            app.InitializeComponent();
            app.Run();
        }
    }
}
```

可见编译后生成的 Main 方法，该方法执行了以下操作：

1. 创建了 App 类型的对象 app
2. 调用了 app 的 InitializeComponent 方法，从反编译结果可知，该方法中设置了 StartupUri
3. 调用了 app 的 Run 方法

那么 app.Run 内部究竟做了什么呢？

1. 检查 UI 线程
2. this.RunDispatcher((object) null);

```
public int Run()
{
    EventTrace.EasyTraceEvent((EventTrace.Keyword) 3, (EventTrace.Event) 3);
    return this.Run((Window) null);
}
public int Run(Window window)
{
    this.VerifyAccess();
    return this.RunInternal(window);
}
public void VerifyAccess()
{
    if (this.Thread!= Thread.CurrentThread;)
        throw new InvalidOperationException(MS.Internal.WindowsBase.SR.Get(nameof (VerifyAccess)));
}
internal int RunInternal(Window window)
{
    this.VerifyAccess();
    EventTrace.EasyTraceEvent((EventTrace.Keyword) 3, (EventTrace.Event) 3);
    if (this._appIsShutdown)
        throw new InvalidOperationException(SR.Format(SR.CannotCallRunMultipleTimes, (object) ((object) this).GetType().FullName));
    if (window!= null)
    {
        if (!((DispatcherObject) window).CheckAccess())
            throw new ArgumentException(SR.Format(SR.WindowPassedShouldBeOnApplicationThread, (object) ((object) window).GetType().FullName, (object) ((object) this).GetType().FullName));
        if (!this.WindowsInternal.HasItem(window))
            this.WindowsInternal.Add(window);
        if (this.MainWindow == null)
            this.MainWindow = window;
        if (window.Visibility!= Visibility.Visible)
            this.Dispatcher.BeginInvoke((DispatcherPriority) 10, (Delegate) (obj =>
            {
                (obj as Window).Show();
                return (object) null;
            }), (object) window);
    }
    this.EnsureHwndSource();
    this.RunDispatcher((object) null);
    return this._exitCode;
}

```

## 带 Main 函数的方式

新建一个名为 WpfApp2 的 WPF 程序，此程序默认包含两个类：

1. App.xaml
    1. App.cs
2. MainWindow.xaml
    1. MainWindow.cs

接着，删除 App.xaml 和 App.cs 类，并添加 Program.cs 文件，添加 Main 函数，如下：

```
    public class Program
    {
        [STAThread]
        public static void Main(string[] args)
        {
            var app = new Application();
            app.StartupUri = new System.Uri("MainWindow.xaml", System.UriKind.Relative);
            app.Run();
        }
    }

```

其中最为关键的是 STAThread 这个特性，它声明以单线程的方式启动。

## 总结

这两种方式在本质上实则相同，因为第一种方式在编译完成后，也会转化为第二种写法。相较而言，第二种方式更为灵活，在构造 app 对象之后，能够注册 app 各种生命周期变化的事件，进而开展诸多操作，例如：加载资源文件、确保程序的单例运行、请求接口等等。

> Logic will get you from A to B. Imagination will take you everywhere.
>
