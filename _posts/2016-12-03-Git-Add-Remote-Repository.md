---
layout: post
title:  "Git 添加远程库"
date:   2016-12-03 23:45:00 +0800
categories:
  - tools
  - git
---

## 远程库的使用

远程库的具体使用方法可以查看[2.5 Git 基础 - 远程仓库的使用][git remote]

[git remote]: https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes

## 使用GitHub作为远程库

### 情景

我在本地使用`Git`管理了一个项目，现在想将它托管到`GitHub`上。  
所以只要在`GitHub`上新建一个库，然后将其作为远程库提交就可以了。  

这里需要注意的是，在`GitHub`上新建的库必须是一个完全空的库，即不能有任何内容
(不能在建库的时候让其自动生成`Readme.md`等文件)  

如果新建的库不是一个完全空的库，则无法将本地的库和远程库进行`git merge`操作，
也就是说无法将本地库同步到`GitHub`上

假定添加的`GitHub`上的远程库地址为`github_url.git`

```bash
Spike (master) ProjectCode $ git remote add origin github_url.git
Spike (master) ProjectCode $ git push -u origin master
To github_url.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'github_url.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

直接`git push`操作失败，所以先`git fetch`操作

```bash
Spike (master) ProjectCode $ git fetch origin master
warning: no common commits
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (4/4), done.
From github_url
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
```

再手动`git merge`

```bash
Spike (master) ProjectCode $ git merge master origin/master
fatal: refusing to merge unrelated histories
```

从上面的输出结果可以看到，本地库和远程库被认为是完全不同的库，所以无法`git merge`

### 解决

删除远程库，在`GitHub`上重新建一个完全空的库，按正常流程重新添加即可
