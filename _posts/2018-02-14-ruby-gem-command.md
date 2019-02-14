---
layout: post
title:  Ruby gem 的使用
date:   2018-02-14 18:08:48 +0800
categories: technology
tags: [Ruby gem]
---
偶有一天，在使用Markdown写完博客文档，jekyll编译报错，提示某个文件找不到，查看路径，发现安装的某个包有问题，于是了解了一下基于ruby程序的gem软件包管理工具。

1.列出gem安装源

``` shell
 gem sources -l
```
2.添加安装源

``` shell
 gem sources -a https://gems.ruby-china.org
```
3.删除安装源并更新安装源缓存
``` shell
 gem sources -r http://rubygems.org/
 gem sources -u
```
4.更新gem软件包管理工具

```shell
 gem update --system
```
5.更新某一个软件包

```shell
 gem update addressable
```
6.更新所有软件包

```shell
 gem update
```
7.安装某一个软件包

```shell
 gem install addressable
```
8.卸载某一个软件包

```shell
 gem uninstall addressable 
```
9.清除所有包旧版本，保留最新版本

```shell
 gem cleanup
```
>参考来源

[gem 安装与使用](https://blog.csdn.net/wangshuminjava/article/details/80285336)