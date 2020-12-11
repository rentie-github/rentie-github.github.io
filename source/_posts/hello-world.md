title: Hexo 更换电脑重新部署，更新博客
tags:
  - Hexo
categories: []
author: rentie
date: 2020-06-20 10:47:00
---
一台电脑上已有一个在用的博客，又新用了一台电脑，实现原电脑和新电脑都可以提交更新博客，实现同步或者说博客的版本管理，本文主要讲述新电脑怎么重新部署一个Hexo的博客编写、发部环境，博客源码备份可以参照[Hexo博客同步管理及迁移](https://www.jianshu.com/p/fceaf373d797)

<!--more-->

## 环境部署

### 安装Git
从[Git官网](https://git-scm.com/)下载git，在新电脑上安装，因为https速度慢，而且每次都要输入口令，常用的是使用ssh。使用下面方法创建：

1、打开git bash，设置用户名称和邮件地址

``` bash
$ git config --global user.name "username"
$ git config --global user.email "username@example.com"
```

2、打开git bash，运行：ssh-keygen -t rsa -C “youremail@example.com” 把其中的邮件地址换成自己的邮件地址，然后一路回车

3、最后完成后，会在用户主目录下生成.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH key密钥对，id_rsa是私钥，千万不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。

4、登陆GitHub，打开「Settings」->「SSH and GPG keys」，然后点击「new SSH key」，填上任意Title，在Key文本框里粘贴公钥id_rsa.pub文件的内容（千万不要粘贴成私钥了！），最后点击「Add SSH Key」，你就应该看到已经添加的Key。

### 安装TortoiseGit
TortoiseGit是一个Git的桌面操作工具，从[TortoiseGit官网](https://tortoisegit.org/download/)选择对应操作系统下载，安装过程一路无脑下一步即可

### 安装Node.Js
从[Node.Js官网](https://nodejs.org/zh-cn/)选择对应操作系统下载，安装过程一路无脑下一步即可

### 若博客本地目录已存在package.json文件
执行下述命令即可安装插件，然后跳过后面的安装环节，直接进入使用环节
    
``` bash
$ npm install
```

### 安装 Hexo
进入博客本地目录，执行下述命令即可安装hexo

``` bash
$ npm install hexo-cli -g 
```

### 安装 Hexo发布插件
进入博客本地目录，执行下述命令即可安装hexo 发布插件

``` bash
$ npm install npm install hexo-deployer-git --save
```
### 安装 Hexo-Admin插件
进入博客本地目录，执行下述命令即可安装hexo 发布插件

``` bash
$ npm install --save hexo-admin
```
### 使用
进入博客本地目录，执行下述命令

``` bash
$ hexo clean  --清理项目
$ hexo g --本地编译
$ hexo s --本地部署
```
打开浏览器输入[http://localhost:4000/admin]
现在就可以正常使用hexo-admin插件了；hexo-admin插件的具体使用自行百度

### 发布博客
进入博客本地目录，执行下述命令

``` bash
$ hexo clean  --清理项目
$ hexo g --本地编译
$ hexo d --发布到GitHub
```