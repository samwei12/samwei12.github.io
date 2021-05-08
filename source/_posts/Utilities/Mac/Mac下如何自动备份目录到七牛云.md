---
title: 'Mac下如何自动备份目录到七牛云?'
categories:
  - Utilities
date: 2016-01-19 22:54:58
tags:
  - 图床
  - 七牛云
---

> [原文链接](http://blog.samwei12.cn/2016/01/19/Utilities/Mac/Mac%E4%B8%8B%E5%A6%82%E4%BD%95%E8%87%AA%E5%8A%A8%E5%A4%87%E4%BB%BD%E7%9B%AE%E5%BD%95%E5%88%B0%E4%B8%83%E7%89%9B%E4%BA%91/)

七牛云是个很好用的图床,但是 Mac 下并没有什么很好用的客户端,每次上传都需要在网页上手动一个个传文件,十分麻烦,于是仔细看了下七牛云的部分文档,打算使用QRSBox来自动上传图片.

<!--more-->

## QRSBox简介

这里为什么要使用QRSBox呢,主要有两方面原因:

1. QRSBox 支持增量同步, 这样上传过后的文件就可以立刻删掉了,对于笔记本来说,硬盘空间毕竟还是很宝贵的.
2. QRSBox 不会同步文件的删除操作,因为我通常是使用七牛来作为图床使用,基本都是上传博客图片在里面,所以不太需要删除操作的同步.

## 下载安装

下载链接:[http://devtools.qiniu.com/qiniu-devtools-darwin_amd64-current.tar.gz](http://devtools.qiniu.com/qiniu-devtools-darwin_amd64-current.tar.gz), 这里我只裂了 Mac 版本,其他版本都可以在官网进行下载.

## 使用方法

下载解压后,得到一堆Unix可执行程序, 将其中的`qrsboxcli`拷贝到`$PATH`里任意路径即可,在这里,我拷贝到了`/usr/local/bin`路径下,之后重启终端,即可全局使用` qrsboxcli`命令.

接下来,进入[开发者自主平台](https://portal.qiniu.com/setting/key), 查看自己的 AccessKey 和 SecretKey, 命令行输入如下:

```bash
<!--注意这里没有尖括号,使用自己的AccessKey, SecretKey替换下方尖括号里的内容-->
qrsboxcli init <AccessKey> <SecretKey> <同步文件夹> <空间名称>
```

完整的命令列表如下:

```bash
qrsboxcli

Usage:
  qrsboxcli init <AccessKey> <SecretKey> <SyncDir> <Bucket>  - Init qrsbox conf
  qrsboxcli sync &                                           - Watch <SyncDir> and sync files
  qrsboxcli status [<Path>]                                  - View path status
  qrsboxcli log                                              - View sync log
  qrsboxcli stop                                             - Kill qrsboxcli sync process

BuildVersion:
  qrsboxcli v3.0.20150529
```

至此,就可以开心的使用七牛图床啦.

## 更改截图路径

Mac 上自带的截图就十分好用, 我们可以通过变更系统快捷键来让它更加实用. 打开系统设置->键盘->快捷键, 之后选中屏幕快照,更改快捷键即可.

![](http://7xlmda.com1.z0.glb.clouddn.com/2016-01-20_21-47-47.png)

有些同学可能觉得这样仍然不够方便,毕竟,系统默认是保存截图到桌面的,还需要我们把它拷贝到自动上传目录下才行,那么我们还可以变更系统默认截屏路径. 打开终端,并输入下面的命令:

```bash
`defaults write com.apple.screencapture location /path/`
```

这里其中的` path`部分就是你想要变更的截图路径,需要注意的是, 输入命令时候记得前后各有一个\`符号,我当时输错了好多次才注意到这点.

这里我们将截图路径设置为七牛云自动上传路径, 以后就可以快速截图加上传,只要打开图床网站获取外部链接即可直接使用在博客之类的地方,是不是很方便呢?

## 参考文档:

* [http://developer.qiniu.com/docs/v6/tools/qrsbox.html](http://developer.qiniu.com/docs/v6/tools/qrsbox.html)
* [如何修改Mac截屏保存路径](http://www.cnblogs.com/ygm900/p/4237593.html)


