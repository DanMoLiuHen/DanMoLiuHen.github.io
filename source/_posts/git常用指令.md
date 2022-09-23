---
title: git常用指令
date: 2022/8/23 14:14:12
updated: 2022/9/23 15:34:00
categories: learning-notes
excerpt: 记录在git中常用的指令
hide: false
tag: markdown
---

主要记录在git中经常使用的指令，笔记主要是给作者自己看或保存，访客大概率看不懂，持续更新中...
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

# 查看历史提交信息
git log 
git reflog
```