---
title: Python模板引擎Jinja2使用简介
date: 2018-07-07 15:52:36
categories:
  - Python
tags: 
  - Jinja2
---

> [原文链接](http://blog.samwei12.cn/2018/07/07/Python/Python%E6%A8%A1%E6%9D%BF%E5%BC%95%E6%93%8EJinja2%E4%BD%BF%E7%94%A8%E7%AE%80%E4%BB%8B/)

## 背景

最近在项目开发中，需要针对 Jenkins 项目进行配置，Jenkins 的 job 配置采用的是 xml，在维护配置模板的过程中就遇到了问题，因为逐步发现配置灵活性超出了字符串的范畴，本文旨在简单介绍 Python 下模板引擎模块 Jinja2 的使用。

## 什么是 Jinja2？

> Jinja2 是一个 Python 的功能齐全的模板引擎。它有完整的 unicode 支持，一个可选的集成沙箱执行环境，被广泛使用，以 BSD 许可证授权。

以上是官方说明，简单来说，它提供了替换功能（变量替换）和一些强大的特性（控制流、继承等），可以快速生成数据文件，使得业务与数据分离开来，满足一些灵活多变的配置需求。

<!-- more -->

## 为什么要使用 Jinja2？

拿 Jenkins 配置来说，如果要实现不同平台、不同项目的配置，有以下几种方案：

1. 直接进行字符串替换，使用 Python 字符串语法即可
2. 使用 XML 解析和生成模块，比较简便的是 ElementTree
3. 使用模板引擎，例如 Jinja2

起初，我选用的是方案一，特点是简单粗暴，不需要额外的配置，例如，两个项目的 git 仓库配置可以采用如下代码实现：

```python
# 略去不相关代码
template = b"""<configVersion>2</configVersion>
        <userRemoteConfigs>
            <hudson.plugins.git.UserRemoteConfig>
                <url>%(git_url)s</url>
            </hudson.plugins.git.UserRemoteConfig>
        </userRemoteConfigs>
        <branches>
            <hudson.plugins.git.BranchSpec>
                <name>%(git_branch)s</name>
            </hudson.plugins.git.BranchSpec>
        </branches>"""
xml = template % {'git_url':self.__git_url, 'git_branch':self.__git_branch}
```

这样，只需要外部传入地址及分支，即可获得不同的项目配置 xml，但缺点也很明显如果两个配置节点数目不相同或者里面的字段差别很大，那么就需要使用 `template2`、`template3`、`template4`等等，因此此方案仅适合配置需求较为简单，字段不是很复杂的场景。

紧接着，开始调研快速生成不同 xml 数据的方案，找到了 https://stackoverflow.com/questions/1055108/fast-and-easy-way-to-template-xml-files-in-python， 这里提到了两种方案，也就是上述的2、3。

方案2的特点也很明显，可以完整控制 xml 中每个字段和属性，灵活性确实够了，能够满足我们的需求，但是有点大炮打蚊子的感觉，而且频繁的读写 xml 也比较消耗性能，业务逻辑也会随着配置的灵活性和多样性急遽膨胀，比如 Jenkins 项目配置，仓库地址、分支信息、ssh 信息、tag 字段、打包脚本等等，如果使用 xml 解析的话，针对这些节点都必须创建对应的业务逻辑，不是很推荐。

那么我们的选择也比较简单了，Jinja 可以快速的替换生成我们业务需要的各种各样的数据，而且接入成本低，业务逻辑不会变化太大，下面简单介绍下本次项目中的使用。

## Jinja2的简单使用教程

### 安装

这个比较简单，看下文档即可，执行命令

`pip install jinja2`

### 基本概念

Jinja2中有一个名为`Environment`的对象，用于存储配置、全局变量，并且用于从文件系统中加载模板。即使你通过`Template`用字符串创建一个模板，Jinja2 也会自动为你创建一个环境。推荐使用这种方式进行模板配置而不是使用字符串，大多数程序使用一个环境即可。

### 用法

1. 创建一个环境：

```python
from jinja2 import Environment, PackageLoader, select_autoescape

# 初始化 jinja 环境
self.__env = Environment(
	loader=PackageLoader('模块名称', '模板文件夹'),
	autoescape=select_autoescape(['html', 'xml'])
)
```

2. 加载一个模板：

   ```python
   self.__job_template = self.__env.get_template('job_template.xml')
   ```

3. 接下来就是传递一些业务参数了，

   ```python
   self.__config = self.__job_template.render(
               git_url=self.__git_remote_url,
               git_branch=self.__git_branch,
               git_tag=self.__git_tag
           )
   ```

以上都还是比较简单的 API 调用，核心使用部分在与如何在模板中通过 Jinja 支持的语法灵活的进行数据配置。

#### 基本变量替换

最基础的功能就是进行变量替换了，语法为`{{变量名}}`， 例如将 url 传递到 xml 中，模板中写法为(省略了不相关内容):

```xml
<userRemoteConfigs>
    <hudson.plugins.git.UserRemoteConfig>
        <url>{{git_url}}</url>
    </hudson.plugins.git.UserRemoteConfig>
</userRemoteConfigs>
```

然后在代码中向上面一样调用 `template.render(git_url='xxx')`即可

#### 控制流

Jinja 支持基本的`if`、`for`、宏等语法，这也是它最强大的地方，目前项目中我只用到了判断就够了。用法如下：

```xml
{% if push_tag %}
<tagsToPush>
    <hudson.plugins.git.GitPublisher_-TagToPush>
    <targetRepoName>origin</targetRepoName>
    <tagName>{{git_tag}}</tagName>
    <tagMessage></tagMessage>
    <createTag>true</createTag>
    <updateTag>true</updateTag>
    </hudson.plugins.git.GitPublisher_-TagToPush>
</tagsToPush>
{% endif %}
```

上面这段 xml 的逻辑就是如果传入了 `push_tag`变量且为`True`则在 Jenkins 配置中增加推送 git tag 节点，否则不做任何处理。

除此之外，Jinja2 还支持循环，

```python
<dl>
{% for key, value in my_dict.iteritems() %}
    <dt>{{ key|e }}</dt>
    <dd>{{ value|e }}</dd>
{% endfor %}
</dl>
```

详细说明可以参考 http://docs.jinkan.org/docs/jinja2/templates.html#for

#### 继承

这部分由于目前项目配置还比较简单，暂时没用到，可以参考 http://docs.jinkan.org/docs/jinja2/templates.html#id23

## 结语

对于简单的项目来说，上面提到的一些基本替换加控制流基本都能满足使用了， 稍微复杂一点的程序可以考虑使用继承和扩展。

总的来说，Jinja2 既强大又简便，完美符合项目中一些HTML、XML 的业务配置需求，目前项目中本身有的三四个 xml 文件用了 Jinja2 之后缩减成一个，同时改动工作量大约只花了几个小时，真的很方便。

> 参考资料：
>
> * http://jinja.pocoo.org/docs/2.10/
> * http://docs.jinkan.org/docs/jinja2/index.html
> * https://stackoverflow.com/questions/1055108/fast-and-easy-way-to-template-xml-files-in-python
