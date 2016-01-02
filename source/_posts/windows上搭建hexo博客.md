title: "Windows上使用hexo搭建博客"
date: 2016-01-12 17:26:19
categories: 收藏品
tags: hexo

---

# 环境准备
- git
- node.js
- github
- githubpage
<!-- more -->

## 安装git
>Git是一款免费、开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

下载最新版[git](http://git-scm.com/download/)的安装到本地

![Alt text](http://7xicmj.com1.z0.glb.clouddn.com/git1.png)
![Alt text](http://7xicmj.com1.z0.glb.clouddn.com/git2.png)

安装完成后配置git的用户名和邮箱，打开git bash

![Alt text](http://7xicmj.com1.z0.glb.clouddn.com/QQ截图20160102162309.png)

输入如下命令：
```bash
git config --global user.name "xxxx"
git config --global user.email "xxxx@163.com"
```
配置好后就可以开始使用git

## 安装node.js
> Node.js是一个基于Chrome JavaScript运行时建立的平台， 用于方便地搭建响应速度快、易于扩展的网络应用。Node.js 使用事件驱动， 非阻塞I/O 模型而得以轻量和高效，非常适合在分布式设备上运行的数据密集型的实时应用。

下载安装[node.js v4.2.4 LTS](https://nodejs.org/en/)，LTS指的是Long Time Support长期支持版本，安装保持默认配置

## 安装hexo
>
hexo是一款基于node.js的静态博客框架，支持多线程，支持GitHub Flavored Markdown和所有Octopress的插件。

安装好了node.js后，打开命令行工具输入：
```bash
npm install hexo --save
```
仅需一步就把 Hexo 本体和所有相依套件安装完毕

**初始化**
```bash
hexo init <folder> 
```
如果指定`<folder>`，会在当前目录下建立一个名为`<folder>`的新资料夹，否则会在当前目录直接初始化，初始化后你就拥有了一个hexo博客

**安装依赖**
```bash
npm install
```
在你初始化的博客目录下执行上面的命令，会自动安装好hexo需要的所有依赖，这时就可以使用hexo写博客了

**创建新博客**
```bash
hexo new "my first article"
```
执行这条命令后你会在`/source/_post`目录下得到一个名为`my first article.md`的文件，在这个文件中使用markdown语法编写内容，会被hexo解析成对应格式的HTML文件

**本地环境**
在完成第一个博客文件的编写后，执行如下命令
```bash
hexo server
```
会得到提示
```bash
INFO Hexo is running at http://0.0.0.0:4000/. Press CTRL+C to stop
```
这时打开浏览器，进入[http://localhost:4000](http://localhost:4000)，就可以看到你的博客了，新初始化的博客有一个默认主题，也可以用一些自定义的主题
![Alt text](http://7xicmj.com1.z0.glb.clouddn.com/QQ截图20160102165437.png)

## github
>github是一个开源协作社区，也是一个源代码托管服务商，它提供很便捷的代码托管服务，和基本的社区功能，号称程序员的Facebook，有着极高的人气，许多有名的重要项目都托管在上面

github希望用户可以看到一个简明易懂的网页，说明每个项目应该如何使用，因此设计了一个pages功能，允许用户自定义项目首页，因此我们可以利用这个功能，来作为自己的博客托管服务器

首先你需要注册你的[github](https://github.com/)，注册完成后你需要认证你的邮箱，邮箱认证通过后github将为你提供pages服务

**建立仓库**

首先需要建立一个仓库来存放你经过hexo解析好的HTML代码，这个仓库的名称是固定的，`<your github name>.github.io`，你的github用户名加上github.io结尾，当github检测到你有这个仓库后会自动为你分配一个域名，域名就是`<your github name>.github.io`
当你将代码通过git提交到这个仓库之后，就可以通过github提供的域名看到自己的博客了

**在hexo中配置自己的github地址**
在刚刚建立的hexo根目录找到一个叫做`_config.yml`的配置文件，在最下面的配置项中配置你创建的的github仓库和分支：
```yml
deploy:
  type: git
  repository: https://github.com/lybc/lybc.github.io.git
  branch: master
```

配置好后在博客根目录执行：
```bash
hexo g # 生成博客文件
hexo d # 部署到github
```
部署完成后等待一段时间就可以在github为你分配的域名上看到你的博客了


