+++
date = '2024-12-15T19:33:52+08:00'
draft = false
title = '我影刀RPA订阅了1000+的独立博客的RSS'
tags = ["RPA"]
categories = ["技术博客"]
author = "火箭"
+++


很早之前我在 Github 上发现了一个[Repo](https://github.com/timqian/chinese-independent-blogs)，里面整理了大量独立博客网站。通过这个项目，我发现了许多计算机界的大佬博客，比如[王垠的博客](https://www.yinwang.org/)、[王登科-DK博客](https://greatdk.com)、[MacTalk-池建强的随想录](http://macshuo.com)等。我把这些博客都保存在浏览器的书签文件夹里，每当有空就会去看看这些大佬是否有更新。

   偶然间，我了解到了RSS，并发现了一个神奇的项目[RSSHub](https://docs.rsshub.app/)。于是我开始探索博客订阅的新天地。最终选择了InoReader作为订阅工具，主要是因为它支持账号登录，可以在不同设备间同步阅读进度。虽然由于网络限制，IOS移动端App无法正常获取信息，但这影响不大，因为我很少用移动设备阅读。

   我欣喜若狂地打开了[博客列表的Repo](https://github.com/timqian/chinese-independent-blogs)，开始逐个点开博客网站，等待InoReader识别RSS，点击订阅，等待完成，然后继续下一个……这个过程既快乐又痛苦，因为订阅一个博客就需要一分多钟，而我要关注1000多个博客，手动订阅实在太耗时了。我准备自动化订阅RSS的过程，作为一个RPA产品的研发，我当然要用我们的软件——[影刀RPA](https://www.yingdao.com/)。

软件的安装和注册过程我就不写了，我们社区版支持最多30行指令的运行，完成订阅博客这件事绰绰有余。接下来我介绍一下开发思路。

1. 安装InoReader插件，并把插件Pin到工具栏
2. 用[打开网页](https://www.yingdao.com/yddoc/rpa/711591442174164992?source=%E6%89%93%E5%BC%80%E7%BD%91%E9%A1%B5)指令，打开[博客列表](https://github.com/timqian/chinese-independent-blogs)
3. 使用[获取相似元素列表](https://www.yingdao.com/yddoc/rpa/716549639521247232?source=%E8%8E%B7%E5%8F%96%E7%9B%B8%E4%BC%BC%E5%85%83%E7%B4%A0)指令提取所有博客链接
4. 将博客地址保存到本地文件或数据表格中
5. 处理单个博客的具体步骤：
    1. 打开博客网页
    2. 将鼠标移至InoReader按钮上
    3. 点击InoReader按钮并等待"+"按钮出现
    4. 点击"+"按钮并等待"√"出现（建议添加try-catch处理等待超时的情况）
    5. 关闭当前网页
    6. 继续处理下一个博客

> Beauty lies in the harmony of form and function
>
