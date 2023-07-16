---
title: hexo+githubPage创建博客
date: 2022/8/23 14:14:12
categories: practical-experience
excerpt: 记录作者用hexo+githubPage+travis创建博客的全部过程，官方虽有教程，但有些粗略，尤其是部署上，并且有几个坑没有说..
hide: true
tag: markdown
---
想要拥有和本网站类似的博客吗？本篇博客记录作者用hexo+githubPage+travis创建博客的全部过程，部分步骤官方教程比较详细，因此本博客将结合官方教程和个人经历进行说明，主要是讲讲几个踩过的坑，
{% note warning %}
**环境：windows10**
适用于对github有了解的人（至少有github账号）
{% endnote %} 

## 安装
这里官网教程比较详细，按照[官网教程](https://hexo.io/zh-cn/docs/)先安装[Node.js](https://nodejs.org/en/)和[Git](https://git-scm.com/)即可。

**注意**：Node.js需要添加到系统环境变量中（安装时有Add to PATH 选项需要勾选上，如果未勾选可在编辑系统环境变量中手动添加，即在系统Path中添加bin文件夹目录，具体过程可自行百度，比较简单），如果添加成功在cmd中输入指令`node -v`可以显示node.js的版本信息（后面有用）

在cmd中使用以下指令安装hexo即可
```bash
npm install -g hexo-cli
```

## 建站（其实就是创建范例）
建站的前几步和[官网教程](https://hexo.io/zh-cn/docs/)一致，这里简写了（建议参考官网教程）

依次输入以下指令，创建本地项目，folder是文件夹名称可自定义
```bash
# 创建文件夹
$ hexo init <folder>
# 进入文件夹
$ cd <folder>
$ npm install
```
之后该文件夹下的目录结构为（每个文件夹作用可见[hexo教程](https://hexo.io/zh-cn/docs/)）
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

到这里项目已经可以运行了（具体运行方法参见[hexo教程](https://hexo.io/zh-cn/docs/)，但运行不是本博客的重点，后面将介绍如何添加主题(theme)，作者使用的主题是fluid，对应[fluid教程](https://hexo.fluid-dev.com/docs/start/#%E4%B8%BB%E9%A2%98%E7%AE%80%E4%BB%8B)

在博客目录下执行以下指令
```bash
npm install --save hexo-theme-fluid
```

然后在博客目录下创建 _config.fluid.yml，将主题的[_config.yml](https://github.com/fluid-dev/hexo-theme-fluid/blob/master/_config.yml)内容复制过去。

修改目录下的_config.yml文件，原文件若无theme则添加，若有theme只需要修改对应主题，language同
```yml
theme: fluid  # 指定主题
language: zh-CN  # 指定语言，会影响主题显示的语言，按需修改
```

至于更详细的favicon、背景图等修改参见[fluid配置指南](https://hexo.fluid-dev.com/docs/guide/)

## 部署
部署主要参考[hexo教程](https://hexo.io/zh-cn/docs/github-pages)，但教程非常粗略，导致作者踩了几个坑，这也是本博客的重点，前面的安装、建站都可以跟着教程走。

1. 在github中新建一个命名为<你的GitHub用户名>.github.io的repository
2. 利用git将博客目录全部推送到这个仓库，参考指令（具体过程有点记不清了，若有问题可联系我）
```bash
# 关联一个远程仓库，origin是对远程仓库的命名，可修改，git@后的内容在github里也会有提示，需要自定义
git remote add origin git@user-name:path/repo-name.git

# 如果本地仓库还没有分支，可以先创建分支，以下指令即创建dev分支并转入到dev分支
git checkout -b dev

# 在本地仓库完成commit
git add --all
git commit -m "Description of this submission"

# 上传给远程仓库（在git中使用这个指令应该会有一段报错，因为远程仓库还没有分支，可按照给出的提示完成push）
git push 
```
3. 创建免费的[Travis CI](https://github.com/marketplace/travis-ci)账户，用于在github上运行项目。只需要使用0元的plan即可满足本项目需求，需要完成创建账户，可直接使用github创建账号
4. 在GitHub的Applications settings(点击右上角菜单->下拉框中选择setting->找到Applications->点击Travis Cl的Configure)，配置 Travis CI 权限，使其能够访问你的 repository
![Github图示](/img/github-travis.png)
5. 上一步点击save后会自动跳转到Travis中，若没有则[手动前往](https://travis-ci.com/)
6. 点击对应项目，如图所示（图示是已经正常运行，读者应该项目是灰色），选择More option->在下拉框中选择Setting（或者也有其他方法进入setting）->找到Environment Variables一栏，准备添加Environment Variable
![](/img/github-travis2.png)
7. 在github的[新建 Personal Access Token](https://github.com/settings/tokens)中，点击Generate new token，只勾选repo的权限并生成一个新的Token，Token内容复制好（网页关闭无法再查看，只能重新创建）
8. 在Travis CI中，新建一个环境变量，Name为GH_TOKEN（关系后面的配置文件），Value为刚才在GitHub生成的Token。确保 DISPLAY VALUE IN BUILD LOG 保持不被勾选避免你的Token泄漏，点击Add保存。
9. 在Hexo站点文件夹（即博客目录下）中新建一个.travis.yml文件，
```yml
sudo: false
language: node_js
node_js:
  - 10 # 需要自定义
cache: npm
branches:
  only:
    - master # 需要自定义
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN # 需要自定义
  keep-history: true
  on:
    branch: master # 需要自定义
  local-dir: public
```
**注意**：部分内容需要自行修改node_js为本地所使用的node版本，可在cmd中用`node -v`查看，github-token与上一步的Name一致，branch和branches需要与github中的分支一致
修改完成后将该文件推送到远程仓库
10. 在Travis Cl中选择该项目，与第六步类似，选择More option->在下拉框中选择Trigger build运行项目（如果该按钮为灰色，说明账户plan没有设置好，需要根据提示将账户配置好后再build，作者当时账号中不仅填写了个人信息还绑定了农行卡，才得以运行），如果运行成功与第六步中图示类似，如果错误可看运行log修改.travis.yml（如果是网站出现红色弹窗则是网站的一些配置没有做好，根据提示自行搜索解决方案）
11. Travis Cl正常运行后进入Github中的对应项目（可以发现会多一个gh-pages分支），点击Setting->Pages->Branch设置页面的分支为gh-pages，等待一段时间（github说是最长等待20分钟），可以看到Pages中出现`Your site is live at https://danmoliuhen.github.io/`即可去访问网站
![](/img/github-pages.png)

## 作者猜想
Travis Cl在线运行后的gh-pages分支是一堆html,css,js等文件，与本地执行`hexo generate`后生成的public文件夹内容似乎一致，是否可以直接在本地博客目录下执行`hexo generate`后将public内容提交给githubPage，即可跳过Travis Cl（并且Travis Cl中也是执行了`hexo generate`指令）


## 参考资料
[^1]: [hexo官网教程](https://hexo.io/zh-cn/docs/)
[^2]:[fluid主题](https://hexo.fluid-dev.com/docs/start/#%E8%8E%B7%E5%8F%96%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC)
[^3]: [githubpage教程](https://pages.github.com/)
