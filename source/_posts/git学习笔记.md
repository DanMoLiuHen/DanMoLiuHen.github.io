---
title: git学习笔记
date: 2022/8/23 14:14:12
updated: 2022/9/23 15:34:00
categories: learning-notes
excerpt: git，github，gitee，个人开发和多人协作的一点笔记，围绕git展开
hide: false
tag: markdown
---
# 常用指令

```bash
# 将文件add到（本地）仓库，可一次add多个
git add readme.txt

# 提交全部修改
git add --all

# commit到（本地）仓库，双引号内为注释
git commit -m "wrote a readme file"

# 将当前分支上传到远程仓库
git push

# 查看远程仓库的信息，包括命名
git remote -v

# 本地仓库和远程仓库解绑,origin是对远程仓库的命名，查看远程仓库信息以获取
git remote rm origin

# 关联一个远程仓库，origin是对远程仓库的命名，可修改
git remote add origin git@server-name:path/repo-name.git

# 查看当前分支
git branch

# 创建分支dev，并切换到dev分支
git checkout -b dev
# 相当于一下两条指令
git branch dev # 创建分支
git checkout dev # 切换分支

# 设置用户名和邮箱(--global 为全局参数，表明本地所有Git仓库都会使用这个配置)
git config --global user.name "yourname"
git config --global user.email "your_email@youremail.com"

# 查看工作区状态
git status
```
# 理解

# 基础操作
## 为常用指令配置别名
在用户目录下创建.bashrc文件，在其中输入想要执行的指令（比较难输入的
```bash
# 输出git提交日志
alias git-log='git log --pretty=oneline --all --graph --abbrev-commit'
# 输出当前目录所有文件信息
alias ll='ls -al'
```

## 忽略列表
创建`.gitignore`文件，在其中输入不需要管理的文件名*.a
```bash
touch .gitignore
vim .gitignore
```

## 分支
查看当前分支`git branch`
### 切换与创建分支
创建分支dev，并切换到dev分支`git checkout -b <dev>`，相当于以下两条指令
- `git branch dev` # 创建分支
- `git checkout dev` # 切换分支
对于切换`checkout`可以用`switch`代替
创建并切换分支`git switch -c <dev>`

### 删除分支
不能删除当前所在的分支
删除dev01分支`git branch -d <dev01>`
强制删除dev01分支`git branch -D <dev01>`

### 合并分支
将`<dev01>`分支合并到当前所在的分支上`git merge <dev01> `

合并冲突之后，将保留冲突部分，手动消除冲突后，再add，commit

## 查看日志
查看历史提交信息
`git log [options]`
  options可以重复添加:
  - --all 显示所有分支
  - --pretty=online 将提交信息显示为一行
  - --abbrev-commit 使得输出的commitid变简短
  - --graph 以图的形式显示

查看之前的所有操作记录
`git reflog`

选中即复制，点击鼠标滚轮粘贴
## 版本回退
`git reset --hard <commitid>`
`<commitid>`根据`git log`查看

## 远程仓库
添加远程仓库`git remote add <origin> <仓库地址>`
查看关联的仓库`git remote -v`

### 推送到远程仓库
`git push [-f] [--set-upstream] [远程仓库名称] [本地分支名][:远端分支名]`
- 远程仓库和本地仓库分支名称相同`git push <origin> <master>`
- --set-upstream 推送到远程的同时并建立起和远端分支的关联
- -f表示强制覆盖
当前分支和远程分支已建立联系可以直接`git push`，第一次push时需要添加`--set-upstream`参数或者直接使用`-u`

### 抓取与拉取
抓取指令`git fetch [remote name] [branch name]`
- 需要在在抓取之后`git merge origin/master`才能查看到代码的内容
- 将远程仓库里的更新都抓取到本地，不会进行合并
- 不指定远程仓库名和分支名则抓取所有分支

拉取指令`git pull [remote name] [branch name]`
- 将远程仓库的修改拉到本地自动合并，等于fetch+merge
- 若不指定remote name 和 branch name 抓取所有并更新当前分支

# 开发流程
## 个人开发
&emsp;&emsp; 配合远程仓库使用，一般从远程仓库拉取下代码`git clone xxx`，在本地新建一个分支`git checkout -b <new_branch>`进行开发测试，开发完成后切换回到主分支，与测试新开的分支合并`git merge <new_branch>`，最后`push`到远程仓库即可

## 团队开发
&emsp;&emsp;与个人开发流程比较类似，但在本地修改完成，上传远程仓库时（此时已处于主分支上）可能会出现冲突（原因：他人先一步提交）导致上传失败，此时需要先`git pull`拉取远程主分支的内容，在本地修改冲突内容（会有提示），修改完成再依次`add commit push`即可