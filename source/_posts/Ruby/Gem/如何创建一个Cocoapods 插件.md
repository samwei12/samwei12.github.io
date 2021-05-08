---
title: 如何创建一个 Cocoapods 插件
date: 2019-04-22  11:07:35
categories:
  - Ruby
tags: 
  - Cocoapods
---

> [原文链接](http://blog.samwei12.cn/2019/04/22/Ruby/Gem/%E5%A6%82%E4%BD%95%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AACocoapods%20%E6%8F%92%E4%BB%B6/)

## 前言

* 我们在使用 Cocoapods 过程中,如果发现它未能满足我们的要求该怎么办呢? 
* 最简单的粗暴的办法就是 fork 一份 Cocoapods 源码,然后自己公司内部或者个人直接针对源码进行部分修改或者新增功能,但这样做完全没有兼容性和扩展性,如果后续 Cocoapods 升级版本,你是无法兼容的,还需要重新进行一次修改,费力不讨好.
* 其实 Cocoapods 提供了一套很方便的插件机制,只需要符合插件规则,即可定制各种自定义需求,接下来我们就看下应该如何自定义一个自己的插件.

<!--more-->

## How

1. 确保 Gem 中安装了 `cocoapods-plugins` 模块,这个按理讲是安装 Cocoapods 时自带的,可以使用命令 `gem install cocoapods-plugins`进行安装
2. 新增插件 `pod plugins create NAME`
3. 针对 gemspec 进行部分修改
  ```ruby
  # 这里是对外暴露的文件
  s.files = Dir["lib/**/*.rb"] + %w{ bin/pod bin/sandbox-pod README.md LICENSE CHANGELOG.md }
  s.files = Dir["lib/**/*.rb"] + %w{ README.md LICENSE.txt }

  # 可执行文件, 没有的话可以使用默认生成的
  s.executables   = %w{ pod sandbox-pod }
  # 查找路径
  s.require_paths = %w{ lib }
  ```
4. 开发过程直接在 `lib/cocoapods_plugin` 文件中加载你 hook 的文件即可
5. 最终对外发布可以通过提 PR 给 `cocoapods-plugins` ,也可以放到自己公司的内部 Gem 源中

## Relations

### 书籍

### 其他

* https://github.com/CocoaPods/cocoapods-plugins