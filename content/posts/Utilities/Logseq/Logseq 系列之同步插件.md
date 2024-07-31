---
title: Logseq 系列之同步插件
date: 2022-09-11 22:37:13
categories:
  - Utilities
tags: 
  - Logseq
  - Git
---

> 原文地址： [Logseq 系列之同步插件 | samwei12's blog](https://blog.samwei12.cn/2022/09/11/Utilities/Logseq/Logseq%20%E7%B3%BB%E5%88%97%E4%B9%8B%E5%90%8C%E6%AD%A5%E6%8F%92%E4%BB%B6/)

前些天看到群友们讨论 Logseq 要是能够在不同设备间同步插件和主题就好了，目前 Logseq 官方同步已经在小规模测试中，预计再有一两个月就会发布，未来大概率会出同步插件的功能。不过考虑到官方同步的免费使用空间肯定不会很大，如果有大量的本地图片或者其他文档类文件，就不适合免费用了，且主要是现在的 Git 同步方案已经非常好用（重点是免费！），所以还是考虑完善一下现有的方案，短时间内不迁移官方同步。

<!--more-->

因为之前在 [Logseq 系列之文档管理 | samwei12's blog](https://blog.samwei12.cn/2022/08/28/Utilities/Logseq%20%E7%B3%BB%E5%88%97%E4%B9%8B%E6%96%87%E6%A1%A3%E7%AE%A1%E7%90%86/#more) 已经分享过个人的文档管理方案，细心的同学可能会发现，我的资源文件中有一个 ebooks 的软链接文件夹，我用它来存放各种电子书，这样的话就可以在不同步电子书到 Git 仓库的同时还能直接使用 Logseq 的本地 PDF 标注功能，这一部分后面有机会可以详细说说。

我这里主要是想提一下系统的文件软链接这个功能，尤其是对于 *nix（包括 Mac）系统来说，软链接几乎可以当作原始文件来使用，由于系统层面的抽象工作做的非常好，对于上层使用的软件来说，可以做到完全无感，这也是上面非 assets 目录下 PDF 标注功能能使用的前提。正因为之前使用过这个功能，所以在考虑插件同步方案的时候，我第一时间就想到了是否可以尝试用软链接来解决呢？答案是肯定的。

对于 Logseq 来说，基本的配置信息都在用户目录下的一个隐藏文件夹 `.logseq` 中，如图：

![20220911225437](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220911225437.png)

其中， plugins 目录存放的就是我们下载的各种插件，以及各种主题； settings 目录则是这些插件的配置信息，而 preferences.json 这个文件则记录了我们的各种配置信息，包括使用的是什么主题、启用了哪些插件等等，graphs 文件夹存放的是各个笔记库的一些数据索引信息，通常我们不需要关心，git 文件夹是开启 git 自动提交使用的，也不需要关心。我们只需要确保 plugins settings 和 preferences.json 这几个文件和目录在不同电脑上内容一致即可。

这里我的思路是，将这几个我们需要同步的目录内容拷贝到自己的 Logseq 文件夹中，使用 git 进行同步，然后在每台电脑上，使用软链接将 `.logseq` 中这几个文件关联到我们 git 仓库中这一份，这样就做到了不同设备之间的插件和配置同步，流程大概长这样：

![20220911230219](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220911230219.png)

确定了方案之后，具体的执行其实就很简单了：

1. 把用户目录下的 `.logseq` 中的内容拷贝到你的 git 仓库中，移除 graphs git 这两个我们不需要的目录。
2. 删除 `.logseq` 中 plugins settings preferences.json 文件
3. 进入到 `.logseq` 目录中，创建对应的软链接

命令如下

```bash
ln -s xxx/.logseq/preferences.json preferences.json
ln -s xxx/.logseq/plugins plugins
ln -s xxx/.logseq/settings settings
```

把其中的 xxx 替换为你电脑上 git 仓库的目录即可，之后在新的设备上重复上述操作。

至此，我们就实现了多设备间插件、主题以及用户配置的同步，而且继续保持我们的优良传统，免费自建，希望本文对你有所帮助～
