+++
date = '2025-01-12T16:35:02+08:00'
draft = false
title = 'WPF中的Application类'
tags = ["WPF"]
categories = ["技术博客"]
author = "火箭"
+++
## 定义
Application类在WPF中程序中代表了当前的应用程序对象，等应用程序对象创建出来以后，可以在程序的任何位置用Application.Current的方式获取该对象。下面先来罗列一下Application提供的属性、方法和事件。

## 主要属性
### Current
类型:System.Windows.Application
在当前的应用域(AppDomain唯一的应用对象),从**触发OnStartup事件之后**程序的全局位置都可以获取。

### MainWindow
类型:System.Windows.Window
一个窗口对象，代表着应用程序的主窗口，没有显式设置的情况，MainWindow属性等于第一个弹出来的窗口对象，后续可以按需设置其他的窗口。

### Properties
类型:IDictionary
应用程序的属性集合。可以存储一些全局的<k,v>属性信息
```csharp
//类似于这种全局的信息
Application.Current.Properties.Add("lang", "en-US");
Application.Current.Properties.Add("version", "1.0.0");
```

### Resources
类型:ResourceDictionary
可以获取或设置应用程序的资源字典。
```xml
<Application
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    StartupUri="MainWindow.xaml">
  <Application.Resources>
    <SolidColorBrush x:Key="BackgroundColor" Color="Yellow"></SolidColorBrush>
  </Application.Resources>
</Application>
```
#### ShutdownMode
类型:是枚举类型(默认值是0,即OnLastWindowClose——)
* OnLastWindowClose——最后一个窗口对象关闭时，退出程序
* OnMainWindowClose——主窗口关闭时，退出程序
* OnExplicitShutdown——只有显式的关闭时，才退出程序  

获取或设置应用程序的关闭模式。
### Windows
类型:WindowCollection
获取应用程序的窗口集合。当一个window对象在UI线程上初始化之后会添加到这个集合中，在处理完Closing事件之后会移除。

## 主要方法
### OnActivated
* 应用程序启动的时
* 从最小化回来的时候也会被调用


### OnDeactivated
* 关闭程序的时
* 程序最小化

### OnExit
最后程序退出之前调用,可以做一些资源回收的事情。类似于python中的[atexit模块](https://docs.python.org/3/library/atexit.html)

### Shutdown
显式的关闭当前进程


## WPF程序的生命周期
![WPF程序的生命周期](https://tuchuang-1258410772.cos.ap-guangzhou.myqcloud.com/WPF%E7%A8%8B%E5%BA%8F%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)