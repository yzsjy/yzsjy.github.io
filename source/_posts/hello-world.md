---
title: Hello World
tags: hexo
---
# Hexo入门以及如何部署到Github
这篇文章主要介绍了如何在本地快速搭建Hexo框架下的博客以及如何将自己的博客部署到Github上。

## 快速开始

### 安装Hexo
前提是本地已经安装了npm，然后最好在本地先新建一个文件夹，然后运行下面的命令安装Hexo
``` bash
$ npm install hexo-cli -g
```
个人是在ubuntu系统下进行的操作，可能直接运行会安装失败，可以尝试在命令前添加sudo

### 安装你的博客
``` bash
$ hexo init blog
$ cd blog
```

### 生成静态文件

``` bash
$ hexo generate 或者 hexo g
```

### 启动服务

``` bash
$ hexo server 或者 hexo s
```

在启动服务后你就可以在你的浏览器中输入https://hexo.io/docs/one-command-deployment.html 来预览你的博客了

### 清除文件

``` bash
$ hexo clean
```

### 创建新的博文

``` bash
$ hexo new "BlogName"
```

## 进行自定义操作
### 更换Hexo默认主题
#### 下载主题

首先在Github上找到你所喜欢的主题的仓库，git clone主题仓库到你所创建博客的blog/themes文件夹，在这里我选用的是star数最高的next主题

``` bash
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
```

然后你可以进入到next文件夹下的_config.yml文件下修改参数以达到你想要的主题效果

#### 更换主题

打开博客创建文件夹blog，找到_config.yml文件并打开，修改theme参数

``` bash
theme: next
```

## 部署博客到Github
### 创建Github仓库

在这里就不展开叙述了，在自己的github账号下创建一个用户名.github.io仓库，网上有很多教程。

### 安装hexo-deployer-git

``` bash
$ npm install --save hexo-deployer-git
```

### 修改配置文件

打开博客创建文件夹blog，找到_config.yml文件并打开，添加参数

```
deploy:
	type: git
	repository: https://github.com/用户名/用户名.github.io.git
	branch: master
```

### 配置git用户名和邮箱

如果已经配置过，忽略这一步。打开git bash

``` bash
$ git config --global user.name "用户名"
$ git config --global user.email "邮箱"
```

### 部署博客到github.io仓库

``` bash
$ hexo d
```

期间会要求输入你的Github用户名和密码