---
title: Logseq 系列之文档管理
date: 2022-08-28 18:07:05
categories:
  - Utilities
tags: 
  - Logseq
---

本文主要是介绍一下我自己在使用 Logseq 进行文档管理时遇到的一些问题以及解决方案。

## 现状

目前 Logseq 针对各种文档、图片等外部资源的管理形式比较简单，统一放到 `assets` 这里目录下进行管理，在实际使用中通过相对路径进行引入，大概是下图这样：

![20220828181355](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220828181355.png)

<!--more-->

可以看到，这里面包含了 PDF 标注文件、音频文件、视频文件、各种图片资源等，这样做的好处非常明显，可以统一管控全部资源，做到各平台体验一致，包括在移动端上也能够正常查看图片、音频等，例如下图的思维导图，引入之后，点击链接电脑上可以打开进行文件编辑等，比较方便。

![20220828182234](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220828182234.png)

![20220828182126](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220828182126.png)

同样的，这样做的缺点也比较明显，你在 Logseq 中使用的全部资源都会拷贝一份到它自己的资源目录中，如果是截图这种偶尔使用一下的资源还好，如果是工作学习中使用到的文档，就会很不方便，无法进行管理，毕竟你并不是只在 Logseq 中使用这些文档。

### 问题一：文件检索管理

首先，Logseq 本身并不是一个用于管理文档的软件，只有最基本的检索和打开功能，一旦文件变多了之后，没有目录层级、没有标签，文档的检索会成为一个很严重的问题，当然你可以通过单独建立一个 page，把相关文档都放在其中，以实现打标等功能，但操作路径会比较别扭且略有成本。

同时当你拖拽文档进入 Logseq 之后，你的电脑中存放了两份相同内容的文档，且以后你只能以 Logseq 中这一份为准，之前的拷贝必须删掉，不然你就会面临更新编辑无法同步到 Logseq 的问题。之后，当你想要进行文件编辑的时候，只能通过 Logseq 点击打开进行，因为当你的 assets 文件夹中包含了成千上万截图、文档资料的时候，你几乎不可能在文件夹中进行查看管理。
有时候你可能并不是想编辑，而仅仅是想要分享给同事，你会发现，如何在本地文件中找到并发给他并不容易，我目前想到的路径是找到该资源对应的 block，点击查看 Logseq 进行改动后的文件名，然后把相对链接拼接你本地 Logseq 的路径，转换成绝对路径，这样就可以在系统资源管理器中打开了，毫无疑问，这个操作成本非常高（这里也可以考虑使用一些类似 Spotlight 或Alfred 这样的软件进行文件索引，相对成本低很多）。

### 问题二：同步问题

第二个问题，就是随着你的长期使用，assets 的大小会逐步膨胀，这在多设备之间同步的时候就会慢慢变成一个你无法忽视的问题。如果你用的是同步盘，这个问题还稍微好点（不过同步盘自身就有很大问题）。如果你像我一样，使用的是 Git 同步 （参考
[Logseq 系列之 Git 同步 | samwei12's blog](https://blog.samwei12.cn/2022/08/12/Utilities/Logseq%E7%B3%BB%E5%88%97%E4%B9%8BGit%E5%90%8C%E6%AD%A5/) ），你会发现随着你的使用，你的 Logseq 文件夹膨胀速度比你想象中的快很多。这是因为 git 本身的原理是基于记录你每次提交的文件变化来进行跟踪，当你使用文本文件的时候（也就是非二进制文件），通常每次提交 git 会准确的把其中几行改动记录下来以便进行版本管理；但当你使用的是二进制文件，由于本身比对改动非常不方便（即使把改动的二进制变化贴给你你也看不懂，自然也就失去了意义），所以很多时候都会直接把改动前后的版本都记录下来，这也就意味着，假如你有一个 100MB 大小的 PS 文件，即使你只改动了其中一个像素，那么提交的时候也会把改动后的这 100MB 大小的新文件提交上去，不难想象，随着你改动的文档以及频次变多，你的 assets 文件夹会轻松变为一个几十 GB 的庞然大物，让你的同步变得非常困难。

> 注： Git 本身提供了对于大文件管理的专用方案，[Git Large File Storage | Git Large File Storage (LFS) replaces large files such as audio samples, videos, datasets, and graphics with text pointers inside Git, while storing the file contents on a remote server like GitHub.com or GitHub Enterprise.](https://git-lfs.github.com/)，但有一定的使用成本。

## 个人解决方案

那么如何解决这些问题呢？首先，我们需要一个专门的文件管理软件，它能够进行正常的文件目录、标签、检索、预览编辑等操作，同时，我们还需要能够在 Logseq 中插入对应的文件链接，以便在笔记中进行引用。

针对上面的诉求，我第一时间想到的就是 [DEVONtechnologies | DEVONthink, professional document and information management for the Mac and iOS](https://www.devontechnologies.com/apps/devonthink) 这款软件，它同时支持苹果全家桶，是专业的个人知识文档管理软件，不光能够进行检索、标签，甚至还可以进行网页抓取，防止网址失效，可以说是目前 Mac 平台里面最专业的最好用的管理工具了，唯一的问题大概就是价格过于高昂，当然可能这不是它的问题。

有了文档管理软件之后，我们再解决文件链接引用的问题，这里我们引入一个术语，URI scheme， 可以通过 [List of URI schemes - Wikipedia](https://en.wikipedia.org/wiki/List_of_URI_schemes) 了解一下， 简单点来说就是一种约定好的资源标识链接格式，通过这个链接以及一些约定好的参数，就可以通过跳转链接来唯一定位软件中资源，然后就可以实现一些功能。 最常用的例子就是文件管理器中的文件路径，例如，文件管理器规范为 `file://[host]/path`， 这是大家统一约定好的，路径大概长这样 `file:///Users/samwei12/Downloads/docker/Dockerfile`，通过这个路径就能唯一定位到文件夹中的这个文件。

![20220828204040](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220828204040.png)

通过 URI Scheme，我们就可以在 Logseq 中引入文件链接来做到点击跳转，跟普通使用一个网址没有什么区别，就像这样： 

![20220828204645](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220828204645.png)

同样的， Devonthink 也支持 URI Scheme，而且支持的更好。它的 Scheme 格式长这样 `x-devonthink-item://C81BB953-8659-4AE7-9E89-143C4F6036BC` ，看起来就是普通的 UUID，但它比 Finder 强大太多了，除了能够直接点击定位以外，它最大的优势就是**可以在文件位置变更之后依然能够定位到文件**！有没有感觉很棒？ 试想一下，当你的文件夹位置稍微有所调整，所有已经引用的文件路径全部作废，有没有很难受？这时候你就会发现 DevonThink 是多么的强大，甚至改了名字之后，这个链接依然有效！包括搜索的时候，不光可以名字搜索、标签搜索、文件夹搜索，还会根据你的文档使用和打开频率给他们打分，把分数高的排在前面，非常的方便。基本上使用了它之后，你就不会再想用系统文件夹进行文件管理了。

![20220828223456](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220828223456.png)

那么我们的第一个问题，文件资源管理器以及引用就解决了，那么现在来解决第二个同步问题，这里目前我的解决方案还不是很完美，但基本满足我个人的诉求了。

![20220828224142](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220828224142.png)

DevonThink 支持两种方式建立索引，方式一是 Import，也就是类似 Logseq 这种，把文档全部拷贝到自己的管理系统中统一处理，这样做的好处就是可以使用它自己的一整套管理体系，包括数据同步、客户端等，但也会受限于它的服务器连接速度，可以说各有利弊，毕竟自由控制跟统一管理总是没法同时满足需求的，想要做到统一的体验，就需要牺牲一些东西。如果你不愿意把自己限定在 DevonThink 的体系里面，它很人性的提供了方式二，也就是 Index 这种，它的使用方式是你自己提供一个已有的本地文件夹，它仅仅通过检索文件信息来进行管理，这样做最大的好处就是既能够享受它强大的标签、索引系统，还可以自己本地保留管理的权利，不必 Lock-in 在这个软件里面，如果你选择了方式一，想要自己手动进行管理会麻烦很多。

我选择使用方式二，我借鉴了 PARA 的概念，本地专门有一个名为 Resources 的文件夹，它的作用就是用于管理我的全部个人文档，可以使用树状目录，也可以使用 DevonThink 的标签系统，或者 Finder 的标签系统，然后使用方式二把它添加索引，拷贝链接到 Logseq 中进行使用即可。如果你需要的话，只要把这个文档文件夹使用同步盘进行同步，就可以做到跨电脑使用，而 Logseq 目录中仅包含文本文件，体积非常可控。不过日常使用中为了方便，图片资源我还是都使用 assets 文件夹，而 pdf 文件，我有单独的管理方式，也可以认为不在 assets 中，后面有机会再分享。这样做的好处就是我整个包含了全部笔记的 Logseq 文件夹，也就不到 500MB，对 Git 的压力就非常小了。
当然，有利就有弊，方式二最大的问题就是多平台体验会不一致，由于文档文件本身还是在电脑设备上，通过云盘还可以做到跨电脑使用，但移动端就不可用了，原因是 iOS 端苹果的系统限制，软件通常只能访问自己应用目录下的文件，无法跨应用进行访问，DevonThink 移动端也只能访问方式一那种自己管理的数据，不过由于我个人在移动端仅仅是查看笔记居多，几乎也没有这方面的诉求，所以这个影响可以忽略不计。

### 提效小 tips

这里不得不赞一下国外软件的开放特性，通常大家都是在思考如何把自己的功能做的标准化，可以让更多的软件融入进来，而国内软件大部分都是思考如何让自己的软件封闭化，全部功能只靠自己就可以做到，好像做开放一点自己的用户就会被其他人抢走一样，思考一下你平时用的国产软件有多少支持 URI Scheme？又有多少国外软件支持 URI Scheme？
扯远了，回到正题，DevonThink 有一款 Alfred 插件。Alfred 同样是一款非常棒的软件，类似于 Windows 下的 everything，但功能要强大太多了。不光可以替代掉各种剪切板软件、计算器、文本替代软件，还提供了非常棒的插件功能。 

![20220828230343](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220828230343.png)

安装了这个插件之后，可以直接在 Alfred 中输入 dnt 或者 dnw 缩写，即可实现与检索本地的 DevonThink 软件相同的功能，非常方便，详见 [mpco/AlfredWorkflow-DEVONthink-Search: Powerful Tool for Searching in DEVONthink.](https://github.com/mpco/AlfredWorkflow-DEVONthink-Search) 。

## 总结

至此，本文就介绍完了我自己如何在 Logseq 中进行文档的管理和使用，虽然是依托了 DevonThink 这个非常棒的软件，但整体思路是不受限于此的，但凡是可以提供稳定 URI Scheme 的软件均可以替换掉它，比如 Obsidian 也很棒。只是说在文档管理本身上功能略有偏差，但对于减负 Logseq 来说足够用了，再配合一个同步盘，可以做到多设备同步。
如果本文对你有所帮助或者你有其他方式，欢迎留言交流~