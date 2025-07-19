+++
date = '2025-06-29T10:00:43+08:00'
draft = true
title = 'Windows上实现应用单开的几种方案'

+++

客户端程序出于各种原因会实现程序单开，即同一个session同一时刻只能启动该程序的一个实例。有些程序甚至不允许在多个session中启动多个程序实例，下面来介绍几种实现进程单开的技术方案，这些方案没有语言限制，理论上适用于运行在Windows上所有程序。

## 使用命名的Mutex锁

[Mutex锁](https://learn.microsoft.com/en-us/windows/win32/sync/mutex-objects)是由Windows提供的用于同步的内核对象，操作系统会保证统一时刻只能由一个线程可以获得该对象。Windows是一个多用户的操作系统，每有一个用户登录就会生成一个会话(session)。Windows提供的很多内核对象至少可以设置两个名称空间:

1. Local——仅在当前用户session中可见，**创建无需管理员权限**。

2. Global——在整个操作系统中可见，创建需要管理员权限。

   下面创建两种测试的内存映射文件，可以用[winobj工具](https://learn.microsoft.com/en-us/sysinternals/downloads/winobj)查看文件的位置。

   ```c#
   //不带Global前缀的，默认是Local,会创建在用户session下面
   var session_file= MemoryMappedFile.CreateNew("Test_For_Test", 1,MemoryMappedFileAccess.ReadWrite, MemoryMappedFileOpttions.None, HandleInheritability.None);
   
   //带Global前缀的，会创建Session0中，即系统的全局名称空间中。
   var gloabal_file= MemoryMappedFile.CreateNew("Global\\Test_For_Test", 1,MemoryMappedFileAccess.ReadWrite, MemoryMappedFileOpttions.None, HandleInheritability.None);
   ```

## 使用MomoryFileMapping(内存映射文件)



## 管道通信



## 硬编码(不推荐)

