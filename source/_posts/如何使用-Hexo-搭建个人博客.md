title: 如何使用 Hexo 搭建个人博客
date: 2015-09-01 17:49:16
tags:
categories: Daily
---


## 什么是 Hexo ？

[Hexo](https://hexo.io) 是一个简单快速的静态博客框架，可以通过编辑 [Markdown](http://daringfireball.net/projects/markdown/) 文档生成好看的静态博客。

<!--more-->

##  搭建 Hexo

### 要求

安装 Hexo 十分简单，只需要 Node.js 和 Git 即可。

* [Node.js](http://nodejs.org/)
* [Git](http://git-scm.com/)

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

#### 主题

 Alex

### Hexo 常用插件

[Plugins](https://github.com/hexojs/hexo/wiki/Plugins), Hexo官方插件列表地址

####  RSS

安装hexo-generator-feed插件即可

`npm install hexo-generator-feed`

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

#### Disqus评论系统

* 登录`http://disqus.com/`网站，申请一个新网站的shortname，配置到`_config.yml`中，格式为：`disqus_shortname: samwei12`
* 之后再 Disqus 中申请接入评论系统

![](http://blog.fens.me/wp-content/uploads/2014/05/disqus.png)

```
<div id="disqus_thread"></div>
<script type="text/javascript">
    /* * * CONFIGURATION VARIABLES * * */
    var disqus_shortname = 'samwei12';
    
    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
```

* 然后选择 universal code,将生成的 js保存下来，新建`themes/主题名称/layout/_partial/comment.ejs`，将代码拷贝到里面

## 发布到 GitCafe

### 拥有自己的 GitPage

1. 创建一个与自己用户名相同的项目
2. 随便上传个文件，上传 master 分支
3. 新建 gitcafe-pages 分支，并切换到该分支，推送到远程

### 发布

* 配置里，新增


```
deploy:
  type: git
  repo: https://gitcafe.com/samwei12/samwei12.git
  branch: gitcafe-pages
```


* 然后

```
// 生成静态文件
hexo g
// 发布
hexo d
```

即可


## 参考文档：

* [Hexo静态博客使用指南](http://segmentfault.com/a/1190000002538363)
* [Hexo在github上构建免费的Web应用](http://blog.fens.me/hexo-blog-github/)
* [如何搭建一个独立博客——简明Github Pages与Hexo教程](http://www.jianshu.com/p/05289a4bc8b2)
* [进阶设置](https://vxhly.github.io/2017/10/hexo-next-advanced-settings/)
