---
title: Hexo-常用插件&配置
date: 2024-08-17 12:49:56
tags:
    - Hexo
    - Blog
    - Plugins
categories:
    - Utilities
---


参考文档地址：

> [Plugins](https://github.com/hexojs/hexo/wiki/Plugins), Hexo官方插件列表地址
> [theme-next/awesome-next: :sunglasses: Theme NexT, AWESOME NexT!](https://github.com/theme-next/awesome-next?tab=readme-ov-file)

这里汇总一下自己比较常用的插件以及相关的配置，希望对你有所帮助。
注意： 我使用的是 `next` 主题，很多配置可能是主题专用！

<!-- more -->

###  RSS

安装 `hexo-generator-feed` 插件即可

`npm install hexo-generator-feed`

在配置中开启 RSS

```yaml
# RSS
feed:
    type: atom
    path: atom.xml
    # 用于限制 RSS 中初始多少 feed
    limit: 20
```

这样其他人就可以在网页上抓取 RSS 了，

![202408171255722](https://learner.oss-cn-hangzhou.aliyuncs.com/img/202408171255722.png)

同时还可以在文章结尾添加一个订阅提示

```yaml
follow_me:
  #Twitter: https://twitter.com/username || fab fa-twitter
  #Telegram: https://t.me/channel_name || fab fa-telegram
  WeChat: /images/wechat_channel.jpg || fab fa-weixin
  RSS: /atom.xml || fa fa-rss
```

这里同时还可以把自己的微信公众号等放在这里。

### 中英文自动加空格

[next-theme/hexo-pangu: 🀄️ Server side pangu.js filter plugin for Hexo.](https://github.com/next-theme/hexo-pangu)

再也不用手动在中英文之间敲空格了，虽然现在很多输入法已经带了这个功能，但经常是英文前不加后面加等，各种问题，有这个插件之后就不用考虑这个问题了。

原文：

```
1. 测试一下 test   中英文。
2. 测试一下test   中英文。
3. 测试一下test中英文。
```

插件效果：

1. 测试一下 test   中英文。
2. 测试一下test   中英文。
3. 测试一下test中英文。

感觉还是挺不错的，基本涵盖了各种 case。

### 阅读进度

可以在文章顶部显示阅读进度条，简介美观。

```yaml
# Reading progress bar
# Dependencies: https://github.com/theme-next/theme-next-reading-progress
reading_progress:
  enable: true
  color: "#37c6c0"
  height: 5px
```

### Sitemap 

给搜索引擎使用

```
npm install hexo-generator-sitemap
```

装完之后 记得在全局配置里面开启插件

```
plugins:
- hexo-generator-feed
- hexo-generator-sitemap
```

* 重新clean发布
* 之后访问： https://search.google.com/search-console/settings?resource_id=http%3A%2F%2Fsamwei12.com%2F&hl=zh-CN 填写生成sitemap地址即可

配合  GoogleAnalytics 使用

* https://analytics.google.com/analytics/web/#/report-home/a138655615w199342623p193811219

### 百度 sitemap

同上，不过是国内统计使用

```
npm install hexo-generator-baidu-sitemap@0.1.1 --save
```

### Git

用于发布

```
npm install hexo-deployer-git --save
```

安装后需要在配置中开启

```yaml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  # repo: https://gitcafe.com/samwei12/samwei12.git
  # branch: gitcafe-pages
  # repo: https://github.com/samwei12/samwei12.github.io.git
  repo: git@github.com:samwei12/samwei12.github.io.git
  branch: main
```

不过建议直接使用 Github Actions，见 [Hexo-Github Actions 自动部署方案 | samwei12's blog](https://blog.samwei12.cn/2024/08/17/Utilities/Writing/Hexo-Github-%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%E6%96%B9%E6%A1%88/)

### Gitalk-待优化

用于管理评论

```yaml
# Gitalk
# Demo: https://gitalk.github.io
gitalk:
  enable: true
  github_id: samwei12  # Github repo owner
  repo: Gitalk # Repository name to store issues 注意这里必须要填名称，而不是链接
  client_id:  # Github Application Client ID
  client_secret:  # Github Application Client Secret
  admin_user: samwei12 # GitHub repo owner and collaborators, only these guys can initialize github issues 这里填名称即可，可以是数组
  distraction_free_mode: true # Facebook-like distraction free mode
  # Gitalk's display language depends on user's browser or system environment
  # If you want everyone visiting your site to see a uniform language, you can set a force language value
  # Available values: en, es-ES, fr, ru, zh-CN, zh-TW
  language:
```

### 阅读时间统计

建议使用： [next-theme/hexo-word-counter: ⏱️ Word count and time to read of articles for Hexo, written in Rust](https://github.com/next-theme/hexo-word-counter) ， 相较于 其他几个字数统计更加精准，速度也会更快。


* 这里有个注意点,必须要在主配置文件即`_config.yml`中提前开启,才能在 next 主题中进行配置,否则失效

### local_search

开启本地搜索功能

```yaml
# Local search
# Dependencies: https://github.com/next-theme/hexo-generator-searchdb
local_search:
  enable: true
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
```

效果图

![202408171306783](https://learner.oss-cn-hangzhou.aliyuncs.com/img/202408171306783.png)


### 相关文章推荐

这里 next 主题中本身自带了一个配置，如下：

```yaml
# Related popular posts
# Dependencies: https://github.com/tea3/hexo-related-popular-posts
related_posts:
  enable: true
  # title: "相关文章" # custom header, leave empty to use the default one
  display_in_home: true
```

但这个已经是被废弃的插件依赖，最新的配置方案参考： [sergeyzwezdin/hexo-related-posts: Hexo plugin that generates related posts list with TF/IDF algorithm](https://github.com/sergeyzwezdin/hexo-related-posts)

配置开启记得直接在 `_config.yml` 中进行。

效果图：

![202408171422767](https://learner.oss-cn-hangzhou.aliyuncs.com/img/202408171422767.png)

默认是每次构建都会生成相关文章索引，本地查看效果会比较慢，如果希望只在 Github 上部署的时候生效，可以在配置中开启

```yaml
related_posts:
  enabled: true
  enable_env_name: prod
```

这样只有在环境变量中包含 `prod` 的时候才会生效，也就是我们执行 `hexo generate --prod` 时，对应的 [Hexo-Github Actions 自动部署方案 | samwei12's blog](https://blog.samwei12.cn/2024/08/17/Utilities/Writing/Hexo-Github-%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%E6%96%B9%E6%A1%88/) 中使用的 Action 代码是

```yaml
- name: Build
        run: npm run build
```

也就是直接调用 package.json 中的这一段里面的 build 对应的命令，所以我们在这里把 环境变量 `prod` 添加上即可。

```
"scripts": {
    "build": "hexo generate --prod",
    "clean": "hexo clean",
    "deploy": "hexo deploy",
    "server": "hexo server"
  }
```

这样就可以做到本地调试的时候没有推荐文章功能，远程部署的时候带上该功能。