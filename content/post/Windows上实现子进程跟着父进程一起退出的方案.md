+++
date = '2025-01-26T19:33:52+08:00'
draft = false
title = 'Windows上实现子进程随父进程退出的解决方案'
tags = ["Windows API"]
categories = ["技术博客"]
author = "火箭"
+++

Windows系统本身并不维护进程间的父子关系，但子进程会记录父进程的PID。在实际开发中，我们经常需要实现子进程随父进程退出的功能。本文将介绍几种成熟的实现方案。

## 环境准备

首先准备一个简单的子进程程序hello.exe，用于后续测试：

```c++
//hello.cpp
#include<iostream>
using namespace std;
int main()
{
    cout << "hello" << endl<<"please input your age:"<<std::flush;
    int a;
    cin >> a;
    cout<<"a="<<a<<endl;
    return 0;
}
```

## 方案一：标准输入输出重定向

### 场景1：父进程为控制台程序

创建parent.exe：

```c++
#include<Windows.h>
#include<iostream>

using namespace std;
int main()
{
    cout << "from parent" << endl;

    SECURITY_ATTRIBUTES secAttr;
    secAttr.lpSecurityDescriptor = nullptr;
    secAttr.nLength = sizeof(SECURITY_ATTRIBUTES);
    PROCESS_INFORMATION ps = { 0 };
    STARTUPINFO sInfo = { 0 };
    sInfo.cb = sizeof(STARTUPINFO);

    BOOL success = CreateProcess("hello.exe",
        nullptr,
        nullptr,
        nullptr,
        FALSE,
        0, // 共享父进程控制台
        nullptr,
        nullptr,
        &sInfo,
        &ps);

    if (!success)
    {
        cerr << "CreateProcess failed:" << GetLastError() << endl;
        return 1;
    }
    WaitForSingleObject(ps.hProcess, INFINITE);

    CloseHandle(ps.hProcess);
    CloseHandle(ps.hThread);

    return 0;
}
```

### 场景2：父进程为GUI程序

创建parent_gui.exe（WPF实现）：

```c++
using System.Diagnostics;
using System.Windows;

namespace parent_gui
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            this.Loaded += MainWindow_Loaded;
        }

        Process process;
        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var ps = new ProcessStartInfo()
            {
                FileName = "hello.exe",
                RedirectStandardOutput = true,
                RedirectStandardError = true,
                RedirectStandardInput = true,
                CreateNoWindow = true,
            };
            process = new Process();
            process.StartInfo = ps;
            process.EnableRaisingEvents = true;

            process.OutputDataReceived += Process_OutputDataReceived;
            process.ErrorDataReceived += Process_ErrorDataReceived;
            process.Start();

            process.BeginOutputReadLine();
            process.BeginErrorReadLine();
        }

        string str;
        private void Process_ErrorDataReceived(object sender, DataReceivedEventArgs e)
        {
            str +="[error:]"+ e.Data?.ToString()+"\n";
            Application.Current.Dispatcher.Invoke(() =>
            {
                this.Text.Text = str;
            });
        }

        private void Process_OutputDataReceived(object sender, DataReceivedEventArgs e)
        {
            str +="[info:]"+ e.Data?.ToString()+"\n";
            Application.Current.Dispatcher.Invoke(() =>
            {
                this.Text.Text = str;
            });
        }

        private void Button_Click(object sender, RoutedEventArgs e)
        {
            // 向子进程标准输入写入数据
            process.StandardInput.WriteLine(250);
        }
    }
}
```

**注意**：C++的cout带有缓冲区，需要使用`std::endl`或`std::flush`显式刷新缓冲区才能看到多行输出。

**实现原理**：通过将子进程的标准输入输出句柄重定向到父进程，当父进程退出时，系统会自动终止子进程。

## 方案二：使用Job对象关联进程

Windows系统提供了Job对象机制，可以将父进程和子进程关联到同一个Job中，实现进程组的统一管理。

继续使用hello.exe作为子进程，父进程实现如下：

```C++
#include<Windows.h>
#include<iostream>

using namespace std;
int main()
{
    cout << "from parent" << endl;

    HANDLE hJob=CreateJobObject(NULL, NULL);
    if (hJob == NULL)
    {
        cerr << "failed to create job" << endl;
        return -1;
    }

    // 设置Job属性：当所有句柄关闭时终止所有关联进程
    JOBOBJECT_EXTENDED_LIMIT_INFORMATION jobInfo = { 0 };
    jobInfo.BasicLimitInformation.LimitFlags = JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE;

    if (!SetInformationJobObject(hJob, JobObjectExtendedLimitInformation, &jobInfo,sizeof(jobInfo)))
    {
        cerr << "failed to set jobobject information:" << GetLastError() << endl;
        return 1;
    }

    SECURITY_ATTRIBUTES secAttr;
    secAttr.lpSecurityDescriptor = nullptr;
    secAttr.nLength = sizeof(SECURITY_ATTRIBUTES);
    PROCESS_INFORMATION ps = { 0 };
    STARTUPINFO sInfo = { 0 };
    sInfo.cb = sizeof(STARTUPINFO);

    BOOL success = CreateProcess("hello.exe",
        nullptr,
        nullptr,
        nullptr,
        FALSE,
        CREATE_NEW_CONSOLE,
        nullptr,
        nullptr,
        &sInfo,
        &ps);

    if (!success)
    {
        cerr << "CreateProcess failed:" << GetLastError() << endl;
        return 1;
    }

    // 将子进程分配给Job对象
    if (!AssignProcessToJobObject(hJob, ps.hProcess))
    {
        cerr << "Failed to assign process to job:" << GetLastError() << endl;
        CloseHandle(ps.hProcess);
        CloseHandle(ps.hThread);
        return 1;
    }

    WaitForSingleObject(ps.hProcess, INFINITE);

    CloseHandle(ps.hProcess);
    CloseHandle(ps.hThread);

    return 0;
}
```

**补充说明**：如需C#版本的Job实现，可参考StackOverflow上的[这个讨论](https://stackoverflow.com/questions/3342941/kill-child-process-when-parent-process-is-killed)。


## 方案三：发送Ctrl_C信号来优雅退出
在终端启动后端代码，经常能看到输出：**Press Ctrl+C to stop**的字样，我们也可以利用这种方式来终止一个进程。Pycharm中也是用了这种方式来终止进程的，安装Pycharm之后，用[Everything](https://en.wikipedia.org/wiki/Everything_(software))来搜索Ctrlc就能看到，PyCharm的安装包自带了一个发送Ctrl_C信号的程序。命令行启动该程序 把需要终止的进程Pid传给程序，就可以做到终止这个进程。

```bash
sendctrlc.x64.B02191CB70385B094C410A8C27775ABA.exe 123
```

**注意**：部分程序可能会对Ctrl_C信号做了拦截，这种情况下发送Ctrl_C可能无法终止该进程。可以加一个兜底的逻辑，如果Ctrl_C没有生效的情况下，可以补一刀强杀逻辑。

**发送信号的方式有一个好处是，程序不会立刻退出，他又机会执行一些善后逻辑。比如，Python程序中注册的atexit的钩子函数都有机会执行完。**

