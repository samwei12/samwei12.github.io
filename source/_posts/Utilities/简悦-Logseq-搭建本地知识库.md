---
title: 简悦+Logseq 搭建本地化个人知识库
tags:
  - 简悦
  - Logseq
  - PKM
  - 稍后读
categories:
  - Utilities
date: 2022-03-21 22:14:04
---

最近在少数派上看到了 [简悦 +Logseq 个人知识库搭建 | 从零开始完全指南 - 少数派](https://sspai.com/post/72022)， 一时间感觉打开了新世界，其实我很早就买了简悦 2.0，但由于一直没有很好的使用场景，外加配置实在过于复杂， 始终吃灰。直到看到这篇文章，发现原来简悦也可以直接跟 Logseq 打通，那么稍后读的最大弊端终于有了完美的解决方案。

不过实际按照教程配置的过程中，发现原作者是基于坚果云搭建的，不符合我本地化的思路（主要是公司不允许用云盘），因此尝试摸索一个本地任意文件夹均可实现的方案，过程中踩了非常多的坑，记录下来，希望对大家有帮助。

<!--more-->

## 核心配置步骤

因为原文已经写的非常详细了，所以这里我不再一一列举配置，而是仅把本地化配置需要注意的事项标注出来。

### 安装同步与导出工具

这里坚果云不再是必选项，而是任意本地目录即可，注意必须要保证目录结构为 `xx/SimpRead` 才行，这个是硬性要求，否则会无法使用简悦的本地知识库功能。其实只需要在同步文件夹里选择一个名为 SimpRead 的文件夹即可，导出部分使用默认

![20220321222459](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321222459.png)

![20220321222627](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321222627.png)

详细服务设定参考原文，全部打开。

> 关于简悦知识库相关内容，可以查阅官方文档 [建立知识库](https://kb.simpread.pro/#/page/%E5%BB%BA%E7%AB%8B%E7%9F%A5%E8%AF%86%E5%BA%93)

### 通用&高级

- 自动同步相关参考原文，不需要做改动（也不需要覆盖本地配置文件这一步）
- 标注部分保持一致，确保导出时候一定要选 `全文（含标注）+标注`

### 服务

这里是配置大头，也是容易出错的地方，重点说一下。

【授权】
- 因为没有用坚果云，这里可以不用关心授权

【定制导出】
- 自定义标题一定要开，这个也是知识库使用的前提，同时标题必须要保证是 `{{id}}{{un_title}}{{mode}}`  (这个主要是为了后面稍后读能够加载本地文件)
- 模板部分，我个人做了一定的改动，这个可以自定义

```markdown
tags:: #[[SimpRead]] {{tags}}
type:: [[Literature Notes]]
time:: [[{{date_format|now|yyyy-MM-dd}} ]]
source:: [{{title}}]({{url}}) 
desc:: {{desc}}

{{#each}}
- > [📌](<{{an_int_uri}}>) {{an_html}}
  
{{  - |an_note}}{{an_tags}}
{{/each}}
```

主要改动包括：
- 添加部分元数据，用于 Logseq 的 query 功能，例如： type、tags、time 等
- 标注部分把原文放在前面，同时加引文标识，自己的话放在后面，且添加缩进

效果图：

![20220321223653](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321223653.png)

这里一定要注意，`[[{{date_format|now|yyyy-MM-dd}} ]]` 后面是有一个空格的，它不是作者笔误，而是必须要这样才能正常添加时间戳，参见： [定制导出](http://ksria.com/simpread/docs/#/%E5%AE%9A%E5%88%B6%E5%8C%96%E5%AF%BC%E5%87%BA?id=markdown) ，![image-20220321224231231](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321224231.png)

### 增强导出（保持原文配置）

### 自动化

这里有一个坑花了我三天时间，必须要重点讲一下。

> 但这里更建议是保存为 [Textbundle ](http://textbundle.org/)格式，因为离线 HTML 需要将图片转译为 Base64 的代码，碰到多图杀猫的文章很容易卡顿或失败。

因此在自动化中，我导出了 HTML、Markdown、Textbundle，结果就是，尝试了各种方案，包括数次卸载重装、排查本地端口占用、卸载同步助手重装等，都无法成功在本地使用 `http://localhost:7026/reading/` 跳转文件，一直提示 404，![](https://user-images.githubusercontent.com/12758352/158979936-06fa9e67-8a68-4fd8-ab13-e854377e87e9.png)，中间尝试了各种方案，一度怀疑是否只能使用坚果云才能用知识库功能。 结果发现全部卸载重装之后，坚果云也无法使用，中间联系简悦作者 **[Kenshin](https://github.com/Kenshin)** 排查了很久，最终发现是因为同步助手无法识别 `.textbundle` 文件，导致不能正常建立索引。

这个问题有两种解决方案： 1. 放弃 Textbundle，改为使用离线 HTML  2. 通过文件规则，将 `.textbundle` 格式文件移动到其他目录下。我选择了第二种，原因就是离线 HTML 下载容易造成卡顿。不过遇到了另一个问题，那就是 **Hazel** 实在是太快了，在简悦刚开始创建 Textbundle 文件的时候，就已经把文件夹挪到了其他文件，导致此时简悦正在下载的图片找不到存放目录，引发大量报错。这个问题也卡了我不少时间，具体解决方案稍后单独介绍。

关于这部分，Kenshin 戏称说这个 issue 的长度可以在 2000 个 issue 里面排到前五，过程确实比较曲折，我中间走了非常多的弯路（不然也不会花掉一个周末的时间折腾），一度都想卸载简悦放弃了。有兴趣的同学可以看一下 [自定义导出无法使用稍后读加载本地缓存文件 · Issue #3554 · Kenshin/simpread](https://github.com/Kenshin/simpread/issues/3554)

第二个坑，这里原文作者也有提到，那就是稍后读的自动化流程，必须要在阅读模式下才可以，否则不生效， 参考： [部分操作方式加入稍后读时，无法触发自动化 · Discussion #2362 · Kenshin/simpread](https://github.com/Kenshin/simpread/discussions/2362)

### 开放平台（本地方案不需要，跳过）

### 定制导出

- 这里一定要注意： 必须要跟原作者保持一致， `http://localhost:7026/reading/` 路径是简悦知识库必须的配置，否则会导致链接无法跳转以及稍后读无法使用本地缓存文件。
- 另一个问题就是简悦存在 bug，刷新完一定要检查一下配置是否符合预期

### 稍后读后台设置

- 基本保持一致，唯一一个小改动点就是 ![image-20220321225751261](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321225751.png)，因为我只用简悦来进行深度阅读，并不需要在这里查看标注，所以通常都是进入阅读模式。

### MD 文件自动移动（保持一致，使用 Hazel 即可，实际上后面我放弃了使用 Hazel）

至此，整个核心配置基本就已经完成了，我们实现的功能包括：

1. 添加标注自动保存 HTML、Markdown 到 output 文件夹
2. 标注文件自动移动到 Logseq 目录中
3. 稍后读自动加载本地缓存 HTML 文件 ![image-20220321230602224](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321230602.png)

## 优化点

### Textbundle永久保存方案（Mac、Linux 适用）

接着上面提到的，详细展开一下如何处理 Textbundle 文件。

首先，我一下子想到的就是利用 Hazel，新建一个规则就可以，当我实际上这么操作的时候，发现遇到了简悦写入图片时文件夹已经被移动导致报错问题。

此时，我的思路是，如果有办法能够延长移动文件夹的速度就好了。但简悦跟 Hazel 是相互不知道的，很可惜的是 Hazel 中也没有配置延迟操作或者定时任务这一功能，那么我们只能想办法自己动手了（当然这里还有一条路，希望简悦后续的文件写入能够先把图片下载完，之后再生成文件夹）。

我想到的解决方案是使用 *nix 系统的 cron 功能，不了解的小伙伴可以看一下 [linux - Why is my crontab not working, and how can I troubleshoot it? - Server Fault](https://serverfault.com/questions/449651/why-is-my-crontab-not-working-and-how-can-i-troubleshoot-it) ，非常详细的介绍了 cron crontab 分别是什么， 如何使用，以及如何排查错误，受益匪浅。

这里需要做的事情非常简单，将所有后缀为 `.textbundle` 的文件移动到另一个目录中，实际操作起来并不简单，因为 `mv` 命令无法直接移动文件夹，而 textbundle 其实是一个文件夹而不是文件，包括了 json、图片以及 markdown，所以第一步就卡住了。我改为使用 `rsync` 来实现这个功能，可以很方便的把某个目录下所有的 Textbundle 移动到另一个目录。之后我们把源文件夹中的 Textbundle 文件删除。这里由于无法使用 Hazel 实现全部功能，那我干脆把移动 `@annotate.md` 文件的部分也放在自己的脚本里面实现了。完整的脚本文件如下：

```
#!/bin/bash

cd 你的目录/SimpRead
mv output/*@annote.md  Logseq目录/SimpleRead/
rsync -axvP output/*.textbundle archieves（存放 Textbundle 的目录）
rm -rf output/*.textbundle
```

可以直接新建一个 `simpread.sh` 之后把上面的代码粘贴过去，记得把上面中文说明的部分都替换掉，同时为了手动执行方便，可以 `chmod +x simpread.sh` 一下。

然后就是使用 cron 让电脑自动执行这个脚本，这里可以使用 [Crontab.guru - The cron schedule expression editor](https://crontab.guru/#*/5_*_*_*_*) 这个网站调试一下定时任务，比如我目前的配置是每隔 5 分钟执行一次脚本。注意不能执行太频繁，否则一样会遇到简悦下载出错（5 分钟也不能保证完全不出错，理论上间隔越久，出错概率越小）。

具体的添加步骤（我是 Mac，以下请在命令行中执行）

```
 export EDITOR='vi' (不是必须)
 crontab -e
 
 然后输入
 */5 * * * * /bin/sh 你的简悦目录/SimpRead/simpread.sh
 之后输入 :wq 保存即可
```

提示： ![image-20220321232140468](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321232144.png)

就代表定时任务添加成功，可以把 Hazel 卸载掉了。

### 快速进入稍后读

看到简悦同步助手里面有如何快速打开稍后读的方式，参见： [稍后读](http://ksria.com/simpread/docs/#/%E7%A8%8D%E5%90%8E%E8%AF%BB?id=%E5%88%9B%E5%BB%BA%E5%BF%AB%E6%8D%B7%E6%96%B9%E5%BC%8F) ，我就琢磨了一下，感觉效率还可以再提高一下。

### 配置快捷方式

文档中把快捷方式跟独立窗口分开成两步，实际上 Chrome 浏览器支持直接创建一个独立窗口的 PWA 应用（关于 PWA，参考： [Progressive web apps (PWAs) | MDN](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)）。

打开稍后读： ![image-20220321232814407](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321232814.png)

选择独立窗口：

![image-20220321232832834](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321232832.png)

然后你可以在用户目录下找到它 

![image-20220321232955685](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321232955.png)

也可以通过 Alfred 或者 Spotlight 找到它 

![image-20220321233044890](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321233044.png)

### Manico 配置快速跳转 

这里不得不强推一个效率软件了，官方网站： [Manico - macOS 下的快速 App 切换器](https://manico.im/)

基本上可以说这个软件是我目前每天使用频率最高的软件，应该没有之一，大概是 18 年左右 25￥购入。它的主要作用就是快速切换软件，比如你可以配置 option+1 打开 Chrome，option+2 打开微信等等，而且再次点击快捷键可以隐藏该软件，能够极大的节约软件切换的时间。这虽然是一款付费软件，但绝对是我在 Mac 上买过的最物超所值的软件。

核心配置：快速隐藏应用 ![image-20220321233606404](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321233606.png)

如何配置稍后读快速打开隐藏： ![image-20220321233522695](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20220321233522.png)

## 总结

我一般的处理流程是手机浏览公众号或者 RSS 初筛到 Cubox，之后电脑上打开 Cubox，跳转到简悦进行深度阅读，添加标注，同步到 Logseq，之后进行定期整理。

这样，整个从文章到标注，到 Logseq 整理的过程就打通了。简悦作为一个稍后读软件，最大的亮点就是作为一个粘合剂，可以很好地跟各种笔记类软件打通，称为知识管理工作流中不可或缺的一部分，把浏览阅读到的信息和知识真正的经过深度阅读，让它有机会融入到已有的知识体系中，内化为自己的东西，就这一点已经远远超越了 Evernote、Cubox、Pinbox 等同类剪藏产品。

不过简悦问题也很明显，太极客了，配置使用成本巨高无比，我身为一个程序员，都好几次尝试无果弃坑了，个人感觉它的功能已经足够强大了，希望作者能够多花点心思在交互跟实际使用体验上，真的不需要再加那么多新功能了，适当的做减法非常有必要。希望我的踩坑经历能够对你有一些帮助。