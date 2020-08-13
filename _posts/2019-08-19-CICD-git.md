---
layout: post
title: "git使用"
categories:
  - CI&CD
---
## git status引起的命令
### git add
使用git status命令可以看到被修改的文件状态为modified，新增文件未Untracked，此时
可以通过git add <文件名>进行commit前的存储(staged)。同时，可通过git checkou -- <文件名>放弃修改。
### git reset HEAD
通过git add命令添加到暂存区的修改，可以需要通过git reset从暂存区撤回到工作区
