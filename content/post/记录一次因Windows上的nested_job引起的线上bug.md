+++
title = '记录一次因Windows上的nested_job引起的线上bug'
date = '2024-12-15T19:33:52+08:00'
draft = true
tags = ["Windows技术"]
categories = ["技术博客"]
author = "火箭"
+++

## Job介绍
Windows有个job的内核对象，可以把多个进程关联到这个内核对象上实现N个进程的统一管理。比如，我们用job，把子进程"绑定"到了父进程上，实现了父进程退出时，子进程也会随着退出。

## bug的背景。
