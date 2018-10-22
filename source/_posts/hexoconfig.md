---
title: Hexo 迁移总结 
catalog: true
date: 2018-10-12 21:10:40
subtitle:
header-img: ""
tags: hexo
---


> <font color="#C0C0C0" >前言：之前重装了系统，发现无法找回以前的hexo，只能重新搭建一套新的，看了看官网的迁移教程，感觉太过繁琐，以下是参考网上的方法总结</font>

工具：hexo + github

1.clone gihub上的hexo项目到本地。
```php
cd /home/hexo
git clone git@github.com:zhaozepu/zhaozepu.github.io.git 
```
2.移动.git文件到本地hexo目录
```php
mv zhaozepu.github.io/.git zhaozepu.com/
```
3.创建新的分支，存储开发文件（注意：需要建立.gitignore来忽略node_modules编译文件）
```php
git co -b feature-back
git commit -m '备份版本'
git ps origin feature-back
git co master
```
注意：每次修改完master的文件时，需要merge到feature-back分支中

当换电脑时候只需要clone feature-back分支到本地，npm install 就可以了







