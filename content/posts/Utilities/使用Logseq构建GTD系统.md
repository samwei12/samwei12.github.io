---
title: 使用Logseq构建GTD系统
tags:
  - Logseq
  - GTD
categories:
  - Utilities
date: 2021-05-08 15:07:34
---

> 2021-05-08： 1.0版本初步完成，待完善已完成任务回顾
> 2021-05-10： 1.1版本完成，修改不重要不紧急为将来清单，且新增每周回顾

## 前言

最近在阅读《小强升职记》，感觉里面讲到的时间管理工具和方法十分有用，自己之前虽然也了解一些工具，例如四象限原则、两分钟方法等，但看完这本书之后感觉之前的理解还是太肤浅了。是时候升级一下自己的时间管理系统了。

<!-- more -->

在此之前我的工具大致包括： 稍后读软件、GTD工具（滴答清单）、写作工具（一般是VSCode+Hexo）。最近半年沉迷于使用双链笔记软件，目前在用的是 Logseq，后面专门展开介绍一下。目前在尝试将GTD工具、笔记都整合到 Logseq 里面，主要是受了这篇文章的启发 [OKR + GTD + Note => Logseq](https://www.bmpi.dev/self/okr-gtd-note-logseq/)，下面就详细介绍一下我的实践过程。

## 前置知识

### Datalog

首先，需要简单学习一下Datalog语法。Logseq是基于Clojure开发的，它的底层可以简单地理解为是类似于数据库的结构，因此可以天然的支持各种查询，这也是我们能够利用它来实现 GTD 管理的前提。因此这里如果需要掌握如何使用 Logseq 来进行查询的话，最好先学习一下如何使用 Datalog。可以参考 [官方的查询教程](https://logseq.github.io/#/page/advanced%20queries) 以及这篇 [Learn Datalog Today](http://www.learndatalogtoday.org/)，粗略扫一遍，能够看懂大概意思就可以了。

### CSS

其次，Logseq本身是列表大纲形式，无法直接支持四象限这种样式，因此需要一些 CSS功底，这里我是直接参考了 [OKR + GTD + Note => Logseq](https://www.bmpi.dev/self/okr-gtd-note-logseq/) 里面的样式，原始版本在 [[css+template] eisenhower matrix](https://discuss.logseq.com/t/css-template-eisenhower-matrix/526)，可以简单看下。

## 使用步骤

### 块属性

《小强升职记》里面提到日常待办事项是分为项目、任务、行动三种的，只有行动才是可以立马去执行的，项目和任务都需要进行分解，拆成行动。

结合Logseq，我目前考虑使用属性来解决，也可以考虑使用标签，不过我认为会对内容有一定的污染，各有利弊吧。

针对一个新的行动或者任务，添加属性type字段，然后根据具体的类型添加行动、任务、项目字段。

样例：![20210508175442](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20210508175442.png)

### 四象限查询条件

注意：**所有的待办都必须要有明确的优先级和截止日期**，这么做有几个原因：
    1. 要求自己对每个待办都设置明确的时间点，避免添加到收集箱中一直不处理
    2. 添加了优先级之后才好进行四象限统计
    3. 目前还没有找到Logseq的query条件中如何同时查找DDL在限定时间内以及无DDL限制的任务，只能查找其中一种case，这个后续解决

实际使用过程中，可以把一下的查询条件汇总到一个模板中，

```
## #.v-eisenhower-matrix （用于CSS选择器，最好不要改）
:PROPERTIES:
:template: 四象限
:END:
#### [[重要且紧急]]
##### 
#+BEGIN_QUERY
...
#+END_QUERY
#### [[重要不紧急]]
#### [[紧急不重要]]
#### [[不重要不紧急]]
```

具体查询语句如下：可以根据自己的实际使用需求将属性限制移除或者修改时间限制，代码中均已经添加注释。

> 2021-05-10 补充： 将设置为Later的事项移动到第四象限，这样就可以定期回顾；同时新增一个每周回顾的query语句，由于目前Logseq还不支持查询当前时间戳，因此用了个笨办法，每次查询的时候手动输入当前时间戳ms进行查询。

#### [[重要且紧急]] ==(避免太多进入该象限)==

```
#+BEGIN_QUERY
{:query [:find (pull ?b [*])
            :in $ ?end
            :where
            [?b :block/marker ?marker]
            [?b :block/priority "A"] ;; 优先级在A的才认为是重要
            [?b :block/properties ?p]
            [(get ?p "type") ?t]
            [(contains? #{"任务" "行动" "项目"} ?t)] ;; 属性中type字段包含这三种
            [(contains? #{"TODO" "DOING" "NOW"} ?marker)] ;; 不包含已完成任务，不包含LATER
            (or
            [?b :block/scheduled ?d]
            [?b :block/deadline ?d])
            [(<= ?d ?end)]
            ]
    :inputs [:5d-after] ;; 时间跨度，5天内的算作紧急
    }
#+END_QUERY
```
#### [[重要不紧急]] ==(优先完成此象限)==
```
#+BEGIN_QUERY
{    :query [:find (pull ?b [*])
            :in $ ?end
            :where
            [?b :block/marker ?marker]
            [?b :block/priority "A"] ;; 优先级在A的才认为是重要
            [?b :block/properties ?p]
            [(get ?p "type") ?t]
            [(contains? #{"任务" "行动" "项目"} ?t)] ;; 属性中type字段包含这三种
            [(contains? #{"TODO" "DOING" "NOW"} ?marker)] ;; 不包含已完成任务，不包含LATER
            (or
            [?b :block/scheduled ?d]
            [?b :block/deadline ?d])
            [(> ?d ?end)]  ;; DDL在5天以上
    ]
    :inputs [:5d-after] ;; 时间跨度，5天内的算作紧急
    }
#+END_QUERY
```
#### [[紧急不重要]] ==(尽量委派给其他人)==
```
#+BEGIN_QUERY
{    :query [:find (pull ?b [*])
            :in $ ?end
            :where
            [?b :block/marker ?marker]
            [?b :block/properties ?p]
            [?b :block/priority ?priority]
            [(get ?p "type") ?t]
            [(contains? #{"任务" "行动" "项目"} ?t)] ;; 属性中type字段包含这三种
            [(contains? #{"TODO" "DOING" "NOW"} ?marker)] ;; 不包含已完成任务，不包含LATER
            [(!= "A" ?priority)] ;; 优先级在A的才认为是重要
            (or
            [?b :block/scheduled ?d]
            [?b :block/deadline ?d])
            [(<= ?d ?end)]
            ]
    :inputs [:5d-after] ;; 时间跨度，5天内的算作紧急
    }
#+END_QUERY
```
#### [[待办清单]] ==(不紧急不重要，定期回顾确认)==
```
#+BEGIN_QUERY
{    :query [:find (pull ?b [*])
            :where
            [?b :block/marker ?marker]
            [(contains? #{"LATER"} ?marker)]
            ]
    }
#+END_QUERY
```

#### [[本周已完成]] ==(近七天统计)==
```
#+BEGIN_QUERY
{   :query [:find (pull ?b [*])
            :where
            [?b :block/marker ?marker]
            [?b :block/properties ?p]
            [(get ?p "done") ?finishedTime]
            [(- 1620657776000 604800000) ?weekbefore] ;; 由于目前Logseq不支持直接获取当前时间戳，只能使用trick的方式，每次查询前手动输入当前时间戳ms值,算出一周前的时间戳
            [(>= ?finishedTime ?weekbefore)] ;; 一周内完成的工作
            [(= "DONE" ?marker)]
            ]
    }
#+END_QUERY
```

### CSS优化

这里直接照抄了 [OKR + GTD + Note => Logseq](https://www.bmpi.dev/self/okr-gtd-note-logseq/)， 不过它的CSS中内容较多，我单独提取了四象限相关的部分，放在 [四象限CSS](https://github.com/samwei12/LogseqMisc/blob/master/%E5%9B%9B%E8%B1%A1%E9%99%90.css) 中，查询模板也都放在这里，便于大家使用。

### 最终效果

目前我的每日笔记大概长这样：

![20210508181043](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20210508181043.png)

> 1.1 版本已经解决

~~可以算是1.0 版本，目前存在的问题包括：~~

1. ~~没有地方定期回顾那些没有设置DDL和优先级的任务~~
2. ~~缺少查看已完成任务和行动的地方（做周报时候很有用）~~

Logseq是个很棒的工具，期待尽快稳定下来，后面长期使用，作为笔记+GTD一站式平台~






