---
title: 如何使用 Hexo 搭建个人博客
date: 2015-09-01 17:49:16
tags:
- Hexo
- Blog
categories:
- Utilities
---

> [原文链接](http://blog.samwei12.cn/2015/09/01/Utilities/Writing/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-Hexo-%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/)

## 什么是 Hexo ？

[Hexo](https://hexo.io) 是一个简单快速的静态博客框架，可以通过编辑 [Markdown](http://daringfireball.net/projects/markdown/) 文档生成好看的静态博客。

## 搭建 Hexo

### 要求

安装 Hexo 十分简单，只需要 Node.js 和 Git 即可。

* [Node.js](http://nodejs.org/)
* [Git](http://git-scm.com/)

<!--more-->

#### Node.js

最好的安装方式是使用 [nvm](https://github.com/creationix/nvm)，

cURL：

`$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh`

安装好 nvm 之后，重启终端，然后通过

`$ nvm install 0.12`

来安装 Node.js

#### Git

Mac 用户：[Homebrew](http://mxcl.github.com/homebrew/)


### 安装 Hexo

`$ npm install -g hexo-cli`

> 需要注意的是，这里的 npm 相关命令，最好都使用管理员权限操作，否则可能会报错

## 如何使用 Hexo 

一旦安装完毕 Hexo，即可通过如下命令在指定文件夹中初始化 Hexo：

```
$ hexo init <folder>
$ cd <folder>
$ npm install
```

### 配置 Hexo

```
# Hexo Configuration
# Site
title: Hexo #博客标题
subtitle: #博客副标题
description: #博客描述
author: John Doe #作者名字
email: #邮箱地址
language: #语言 中国大陆简体中文的标准语系地区码是zh-CN 台灣是正體中文zh-TW 

# URL
url: http://yoursite.com #博客地址
root: / #博客根目录
permalink: :year/:month/:day/:title/ #博客url地址结构  
tag_dir: tags 
archive_dir: archives
category_dir: categories
code_dir: downloads/code
permalink_defaults:

# Directory
source_dir: source
public_dir: public

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
highlight:
enable: true
line_number: true
tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Archives
archive: 1 #设置为1 是全部展示
category: 2
tag: 2

# Server
port: 4000 #本地服务器端口是4000
server_ip: localhost #本地服务器地址
logger: false
logger_format: dev

# Date / Time format
date_format: MMM D YYYY #日期格式
time_format: H:mm:ss #时间格式

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Disqus
disqus_shortname: #disqus的用户名称

# Extensions
theme: landscape #主题设置
exclude_generator:

# Deployment
deploy:
  type:
```

#### 完善页面

```
hexo new page "about"
```

用于创建 about 页面


### Hexo 常用插件

[Plugins](https://github.com/hexojs/hexo/wiki/Plugins), Hexo官方插件列表地址

####  RSS

安装hexo-generator-feed插件即可

`npm install hexo-generator-feed`

#### 阅读进度

- https://github.com/theme-next/theme-next-reading-progress

#### Sitemap 

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

#### Git

用于发布

```
npm install hexo-deployer-git --save
```

#### baidu sitemap

```
npm install hexo-generator-baidu-sitemap@0.1.1 --save
```

#### sitemap

```
npm install hexo-generator-sitemap --save
```

* 重新clean发布
* 之后访问： https://search.google.com/search-console/settings?resource_id=http%3A%2F%2Fsamwei12.com%2F&hl=zh-CN 填写生成sitemap地址即可

#### Gitalk

* 用于管理评论

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

#### GoogleAnalytics

* 用于分析数据
* https://analytics.google.com/analytics/web/#/report-home/a138655615w199342623p193811219

#### 阅读时间统计

* 参考:https://github.com/theme-next/hexo-symbols-count-time
* 这里有个注意点,必须要在主配置文件即`_config.yml`中提前开启,才能在 next 主题中进行配置,否则失效

#### local_search

* 本地搜索

```yaml
# Local search
# Dependencies: https://github.com/theme-next/hexo-generator-searchdb
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

## 发布到 Github

### 拥有自己的 GithubPage

1. 创建一个与自己用户名相同的项目
2. 随便上传个文件，上传 master 分支
3. 新建 hexo 分支，并切换到该分支，推送到远程

### 发布

* 配置里，新增


```
deploy:
  type: git
  repo: https://github.com/samwei12/samwei12.git
  branch: master
```


* 然后

```
// 生成静态文件
hexo g
// 发布
hexo d
```

即可

## 绑定个人域名

### 购买域名

* 可以从[阿里云](https://www.aliyun.com)上购买域名, 目前我购买了 `samwei12.com` 这个域名，几个注意事项：
  * 需要实名认证
  * 需要备案

### 域名备案

* 可以参考：https://developer.qiniu.com/af/kb/3987/how-to-make-website-and-inquires-the-police-put-on-record-information

### 绑定域名

* 域名解析：
* GitHub中绑定，在项目的设置界面绑定即可
* 参考：https://help.github.com/en/articles/quick-start-setting-up-a-custom-domain

## 迁移到新电脑

- 最近又是好久没有写博客了，换了新电脑之后需要重新配置，这次记录下还原步骤，下次再遇到可以提高效率。

### 还原博客文件

- 我采用的办法是将博客相关内容专门放到 hexo 分支上，然后 git 仓库的默认分支设置为 hexo，切换电脑之后直接拉取最新代码。

### 安装依赖

1. 首先先把主题和插件安装下来
    1. 我使用的主题是 `https://github.com/theme-next/hexo-theme-next.git`， 这里推荐把主题的配置文件放到 `_data`文件夹下，好处是即使换了电脑，配置重新拉取代码仍然同步
    2. 阅读进度插件： `https://github.com/theme-next/theme-next-reading-progress`
    3. 字体插件：`https://github.com/theme-next/theme-next-han`
2. 根目录下执行 `npm install`
3. `hexo g && hexo s` 本地测试是否部署成功

## 注意事项

- 需要注意本地文件名不能有特殊符号，例如空格等，title 里面有没关系，但是文件里面有会导致 Gitalk 插件失效，详见 https://github.com/gitalk/gitalk/issues/162

## 参考文档

* [Hexo静态博客使用指南](http://segmentfault.com/a/1190000002538363)
* [Hexo在github上构建免费的Web应用](http://blog.fens.me/hexo-blog-github/)
* https://xian6ge.cn/posts/5b8c41e7/
* https://zhuanlan.zhihu.com/p/44213627?utm_source=ZHShareTargetIDMore&utm_medium=social&utm_oi=28525297926144
* https://help.github.com/en/articles/troubleshooting-custom-domains#https-errors
