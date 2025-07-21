+++
date = '2025-02-27T13:28:22+08:00'
draft = false
title = 'WPF上实现任务栏闪烁'
tags = ["WPF"]
categories = ["技术博客"]
author = "火箭"
+++


最近做了客户端的IM联系窗口的功能，有一个需求是如果收到新的消息时，任务栏需要闪烁提醒用户。
效果类似于微信和飞书收到新消息。
</br>
<center>
<img src="https://tuchuang-1258410772.cos.ap-guangzhou.myqcloud.com/feishu_hl.png" width=300px height="110px">
</center>
</br>
</br>

Windows有提供任务栏闪烁的API,直接调API即可。我这里把相关的功能都封装了一下，作为一个工具类，分享出来。</br>
知识点：
1. 从C#侧调用win32的 api
2. 把win32 api需要的参数用struct的的形式声明出来，并且标注StructLayout.Sequntial

```cSharp
public static class WpfHelper
{
    // 停止闪烁
    public const uint FLASHW_STOP = 0;
    // 闪烁窗口标题
    public const uint FLASHW_CAPTION = 1;
    // 闪烁任务栏按钮
    public const uint FLASHW_TRAY = 2;
    // 同时闪烁窗口标题和任务栏按钮
    public const uint FLASHW_ALL = 3;
    //持续闪烁，直到手动停止
    public const uint FLASHW_TIMER = 4;
    // 闪烁直到窗口获得焦点
    public const uint FLASHW_TIMERNOFG = 12;

    [DllImport("user32.dll")]
    [return: MarshalAs(UnmanagedType.Bool)]
    private static extern bool FlashWindowEx(ref FLASHWINFO pwfi);

    [StructLayout(LayoutKind.Sequential)]
    private struct FLASHWINFO
    {
        public uint cbSize;
        public IntPtr hwnd;
        public uint dwFlags;
        public uint uCount;
        public uint dwTimeout;
    }

    /// <summary>
    /// 任务栏和指定窗口闪烁，直到该窗口获得焦点
    /// </summary>
    /// <param name="window"></param>
    public static bool FlashTaskbarAndWindow(Window window)
    {
        FLASHWINFO fwi = new FLASHWINFO();
        fwi.cbSize = Convert.ToUInt32(Marshal.SizeOf(fwi));
        fwi.dwFlags = FLASHW_ALL | FLASHW_TIMER;
        fwi.uCount = uint.MaxValue;
        fwi.dwTimeout = 0;

        if (window == null)
        {
            window = Application.Current.MainWindow;
        }
        var wih = new WindowInteropHelper(window);
        fwi.hwnd = wih.Handle;

        return FlashWindowEx(ref fwi);
    }

    /// <summary>
    /// 暂停任务栏和指定窗口的闪烁
    /// </summary>
    /// <param name="window"></param>
    public static bool StopFlashing(Window window)
    {
        FLASHWINFO fwi = new FLASHWINFO();
        fwi.cbSize = Convert.ToUInt32(Marshal.SizeOf(fwi));
        fwi.dwFlags = FLASHW_STOP;
        fwi.uCount = uint.MaxValue;
        fwi.dwTimeout = 0;
        if (window == null)
        {
            window = Application.Current.MainWindow;
        }
        var wih = new WindowInteropHelper(window);
        fwi.hwnd = wih.Handle;
        return FlashWindowEx(ref fwi);
    }
}
```