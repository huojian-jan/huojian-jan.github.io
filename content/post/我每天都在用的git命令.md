+++
date = '2025-02-16T10:50:41+08:00'
draft = false
title = '我每天都在用的git命令'
tags = ["git"]
categories = ["技术博客"]
author = "火箭"
+++

在入职影刀后，我的版本控制工作流逐渐转向以命令行操作为主。除查看合并历史等特定场景会使用TortoiseGit外，90%的Git操作都通过命令行完成。本文将分享我日常高频使用的Git命令及其应用场景。

## 功能开发与问题修复

创建特性分支和修复分支是日常开发的基础操作：
```bash
git checkout -b feat-xxx  # 功能开发分支
git checkout -b fix-xxx   # bug修复分支
```

## 代码暂存管理

当编码过程中需要临时切换上下文时（如紧急问题排查），推荐使用Git stash机制：

| 命令 | 用途 |
|------|------|
| `git stash list` | 查看所有暂存栈 |
| `git stash save "temp_code"` | 将工作区改动暂存并命名 |
| `git stash apply stash@{n}` | 恢复指定索引的暂存 |
| `git stash pop` | 恢复最近一次暂存并出栈 |

最佳实践：建议为每个stash添加语义化名称，便于后续检索。

## Commit Message修改

### 最近一次Commit修改
```bash
git commit --amend
```
该命令会打开Vim编辑器，`:wq`保存修改后的message。

### 历史Commit修改
对于需要修改历史记录中第N个commit message的情况：
```bash
git rebase -i HEAD~N
```
在交互界面中将目标commit的`pick`改为`reword`后保存退出。

## Commit合并策略

对于需要整理提交历史的场景（如功能开发产生多个零散commit）：
```bash
git rebase -i HEAD~N
```
在rebase交互界面中：
1. 保留首个commit为`pick`
2. 后续需要合并的commit改为`squash`
3. 保存退出后编辑最终合并的message

## 历史Commit删除

典型场景：当开发分支包含其他未合并到master的依赖commit时：
```bash
git rebase -i HEAD~N
```
在编辑界面中：
```git
pick c1  # 保留
pick c2  # 保留
drop c3  # 删除依赖commit
drop c4
drop c5
pick c6  # 保留
pick c7  # 保留
```
最后执行`git rebase master`与主分支同步。
