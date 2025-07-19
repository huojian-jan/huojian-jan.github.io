+++
title = '异常打Error日志,Review的时候被喷了'
date = '2025-02-11T13:20:19+08:00'
draft = false
tags = ["Code Review"]
categories = ["技术博客"]
author = "火箭"
+++

前几天在提交一个MR的Code Review时，遇到了一个关于日志级别的有趣讨论。我的原始代码是一个获取标签列表的方法，实现如下：

```csharp
public async Task<TagInfo[]> GetTagListAsync()
{
    try
    {
        return await _apiClient.GetTagListAsync();
    }
    catch (Exception ex)
    {
        Logging.Error(ex.Message, ex);
    }
}
```
Reviewer提了一个令我没想到的问题："为什么要在这里使用Error级别的日志？"

### 问题探讨

我最初的反应是："捕获了异常，难道不应该用Error级别吗？"我们就日志级别适用场景 「讨论」了约5分钟。

Reviewer的观点：
1. 谁会去看这个日志？
2. 这个接口异常是否会阻塞主流程继续执行？
3. 如果真的出现了异常，我们如何主动监控这类异常？

他随后分享了一个Stack Overflow上关于日志级别的讨论：[When to use the different log levels](https://stackoverflow.com/questions/2031163/when-to-use-the-different-log-levels)

### 解决方案

经过讨论，我被说服了，在这种情况下使用Warning级别更为合适。修改后的代码如下：

```csharp
public async Task<TagInfo[]> GetTagListAsync()
{
    try
    {
        return await _apiClient.GetTagListAsync();
    }
    catch (Exception ex)
    {
        // remoteLog参数会将日志上传到ELK，便于后续通过关键字进行分析
        Logging.Warn($"Failed to get tag list, msg:{ex.Message}", ex, remoteLog: true);
    }
}
```

### 日志级别的最佳实践

Stack Overflow上的高票回答提供了一个清晰的日志级别使用指南图表，非常值得参考。

- **Error**：用于确实影响程序正常执行的严重错误
- **Warning**：适用于非关键路径上的异常或可恢复的错误
- **Info**：记录程序正常运行的重要信息
- **Debug**：开发调试期间的详细信息
- **Trace**：最详细的日志信息，通常用于问题诊断

在这个案例中，获取标签列表失败不会影响核心业务流程，因此使用Warning级别更为恰当。这种区分有助于运维人员快速定位真正严重的问题，避免在大量Error日志中遗漏关键错误。

下面这个图是高赞回答的博主贴的，很清晰的说明了何时使用什么级别的日志。

![image.png](https://tuchuang-1258410772.cos.ap-guangzhou.myqcloud.com/log_leve.png)