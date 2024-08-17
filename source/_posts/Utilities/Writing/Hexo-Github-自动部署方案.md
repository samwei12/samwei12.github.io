---
title: Hexo-Github Actions 自动部署方案
date: 2024-08-17 10:44:43
tags:
    - Hexo
    - Blog
categories:
    - Utilities
---


前阵子因为很久没有捡起来写博客，导致电脑的 node 环境各种版本问题，本地压根运行不起来，所以折腾了一下 Hugo 方案，感觉 Hugo 相较于 Hexo 还是有很多优势的，让我印象比较深的是：
1. 整个环境较为独立，不再像 Hexo 需要依赖电脑 Node 版本，各种插件需要独立版本，随着 Hexo 或者 Node 版本升级还需要考虑插件版本问题，减少了很多折腾的工作。
    1. Hugo 搭建和配置非常简单， ![202408171050885](https://learner.oss-cn-hangzhou.aliyuncs.com/img/202408171050885.png)
    2. Hexo 的环境依赖：
![202408171049445](https://learner.oss-cn-hangzhou.aliyuncs.com/img/202408171049445.png)
2. 其次就是 Hugo 速度真的是非常快，比 Hexo 快非常多
3. 然后就是直接配置 Github Action 就可以完成部署，不再需要像 Hexo 那样安装 git 部署插件，参考： [如何使用 Hexo 搭建个人博客 | samwei12's blog](https://blog.samwei12.cn/2015/09/01/Utilities/Writing/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-Hexo-%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/#Git-1) 这点感觉能节省很多时间，不再需要每次写完文章手动敲命令行部署。

<!-- more -->

不过后面折腾半天之后，觉得 Hugo 上确实找不到一款比较合心意的主题（可能是因为使用 Hugo 的大多是后端开发），另外就是需要把之前折腾的一堆插件，例如评论、时长统计、站点统计、RSS 等全部折腾一遍，想想这个工作就觉得有点过于浪费时间了，毕竟还是要以写作为主，不应该花费太多时间在折腾工具上。
刚好后面发现 GitHub 的部署居然会提示我插件版本升级，于是按照对应的插件版本提示，修复了版本冲突问题，节省了我大量的时间，很赞。

![202408171101402](https://learner.oss-cn-hangzhou.aliyuncs.com/img/202408171101402.png)


接下来就是思考 Hexo 是否也有类似的自动部署方案，因为我是 15 年开始使用 Hexo 的，当时 Github Action 还没有推出。
简单检索了一下，发现确实还真有，但找到几个现成的 Action 都无法使用，对应的 Node 版本已经过时，例如  [Hexo GitHub Action · Actions · GitHub Marketplace](https://github.com/marketplace/actions/hexo-github-action) 。 

后来觉得这种没道理官方不出对应的方案，没必要找别人已经加工过的。于是搜到了 [GitHub Pages | Hexo](https://hexo.io/docs/github-pages)。发现官方文档写的更加完整，建议大家直接去网站上查看，这里只写一下自己在实际使用过程中可能会遇到的问题：

### 分支问题

这里官方给到的示例是直接使用 `main` 分支，包括 yaml 配置中使用的也是这个，但对于我目前的工程来说，并没有把 Markdown 文件跟生成的网页拆分成两个工程（理论上来说拆开比较合适，更加安全，可以把草稿文件等隐藏起来不对外公开），而是通过两个不同的 git 分支来进行分离，hexo 分支存放原始的 `sources` 文件，而 `master` 分支用来存放暴露的静态站点。所以这里花了一点时间来研究 Action 配置中的 git 分支应该填哪个。

```yaml
name: Pages # 这里可以改成你自定义的Action名称

on:
  push:
    branches:
      - hexo # 这里如果你像我一样拆分了两个不同分支，那么应该填你的 source 分支，对于我来说就是 hexo
```


### 版本问题

这个比较简单，确认你本地的Hexo和对应的Node版本，设置成一样的即可，确保不会出现插件跟Node版本不一致问题

```yaml
- name: Use Node.js 18
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "18"
```

### Github Page 设置

![202408171146444](https://learner.oss-cn-hangzhou.aliyuncs.com/img/202408171146444.png)

在 Github 中开启 Actions 即可

### 总结

![202408171148869](https://learner.oss-cn-hangzhou.aliyuncs.com/img/202408171148869.png)


至此，基本上就完成了自动部署，之后只需要提交完 Markdown 之后完成推送，Actions 就会自动帮我们构建网站，不用手动敲命令行了。

这里还告诉我们一个道理，也是陈皓老师多次提到的，我们要尽可能的学习一手资料，而不是别人咀嚼过的，可以通过检索别人的文章博客来简单了解一下某个知识，但想要实际学习其中的细节或者遇到问题能够钻研解决，还是需要直接找官方文档或者论文，不然可能只能照抄别人的流程，中间遇到跟他不一样的环境、配置，就傻眼了。