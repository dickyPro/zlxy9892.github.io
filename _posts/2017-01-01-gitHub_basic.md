---
layout:     post
title:      "GitHub 基础操作"
subtitle:   " \"GitHub 使用小指南\""
date:       2017-01-01 22:00:00
author:     "Nova"
header-img: "img/post/post-bg-github.jpg"
catalog: true
tags:
    - github
---

## 1 GitHub 基本思想

最基本的需要区分好两个概念：git 和 github。

git 是一款免费、开源的分布式版本控制系统；github 是用 git 做版本控制的代码托管平台。

具体的解释如下：

- git 是一个软件
- git 可以在 .git 文件夹里面维护你的历史代码
- 指定了 remote 链接和用户信息（git 靠用户名+邮箱识别用户）之后，git 可以帮你将提交过的代码 push 到远程的仓库（任意提供了 git 托管服务的服务器上都可以，包括你自己建一个或者 GitHub/BitBucket 等网站提供的服务器）或者将远程仓库的代码 fetch 到本地。

## 2 基础操作

### 2.1 在 GitHub 中已有项目的基础上操作

- clone
```shell
git clone url
```
- add
```shell
git add -A
```
- commit
```shell
git commit -m "changes log"
```
- push
```shell
git push origin <branch>
```
- pull

```shell
git pull origin <branch>
```

--- 基本上述的几个操作可以完成在已有项目基础上的任务（当然，是不包含多人协作，只自己一人维护开发的情况）。

### 2.2 在本地初始化项目并放置 GitHub 上管理

首先在 GitHub 上创建一个新的空 repository，注意这里可以不添加 readme 文件，例如添加一个名为 rainbow-poem 的新仓库。

然后在本地选择一个目录位置，依次输入下列命令，即可完成从本地开始代码管理：

```bash
mkdir rainbow-poem
cd rainbow-poem
git init
# 开始创建并编辑你的代码...
git status		# 查看当前版本的相关信息，每次commit之前最好查看一次
git add .
git commit -m "first commit"
git remote add origin 创建的新repository的url
git remote		# 查看remote名称列表，现在只有origin
git remote -v 	# 查看remote的详细地址信息
git push origin master	# 完成上传同步

# 如果想撤销本次更改：
git checkout -- file	 # 指定撤销某个文件
git checkout -- .		 # 撤销所有当前修改的内容
```

### 2.3 分支操作

如果从已有的项目中进行新功能的增加或更改，通常我们不会直接在master主分支上进行操作，而是创建一个新分支，例如 dev 分支，在该分支上进行相应的开发，然后提交，最终由项目组审查，若通过审核则将该更新的 dev 分支内容合并到 master 主分支中去。

具体的分支操作如下：

```shell
# 开始在同步好的主分支基础上进行自己的开发与代码更新...
git branch			# 查看当前所有的分支
git branch dev		# 创建dev分支
git checkout dev	# 切换到dev分支状态下，可以再次用 git branch 命令查看所有分支的情况
git add .
git commit -m "update from branch dev"
git push origin dev
# 此时，dev分支已经更新同步完毕，接下来可在浏览器中打开项目页面来管理是否合并到master主分支中
# 或者也可以在控制台中管理
git chechout master
git merge dev
git push origin master
```

**有关分支操作小结：**

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`

创建+切换分支：`git checkout -b <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`

查看远程分支：`git branch -r`

在本地新建分支，建立的本地分支会和远程分支建立映射关系：`git checkout -b 本地分支名 origin/远程分支名`

## 3 本地 push 免密码设置

进入用户名目录，windows一般是在C:\users\Administrator，然后进行如下操作

```shell
touch .git-credentials
vim .git-credentials
https://{username}:{password}@github.com
```

进入 git bash，键入：

```shell
git config --global credential.helper store
```

此时重启git bash，再使用push就不需要输入用户名和密码了。

## 4 参考资料

1. [国外极为简明的git与github视频教程](https://www.bilibili.com/video/av4857819/)
2. [国内较为全面的GitHub博客教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

