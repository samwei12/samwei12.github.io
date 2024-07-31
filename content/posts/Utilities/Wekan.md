---
title: 使用 Wekan 搭建个人看板
date: 2020-05-03  15:10:57
categories:
  - Utilities
tags: 
  - Wekan
  - 看板
---

> [原文链接](http://blog.samwei12.cn/2020/05/03/Utilities/Wekan/)

## 看板？

术语：

- 看板：TODO
- 列表：同类型的事务放在一起，按照任务进度，通常为：未完成、进行中、已完成
- 卡片：最小单位，即任务，
- 泳道：不同类型的任务横向对比，例如，团队中不同分组等

<!--more-->


## 为什么要用看板？

TODO

## 关于 wekan

[wekan](https://github.com/wekan/wekan) 是一款开源的看板系统，从功能上来讲模仿看板类软件鼻祖 [Trello](https://trello.com/)。整体界面大致如下：

![](http://www.scienjus.com/uploads/2016/03/wekan.jpeg)

采用 wekan 的原因也比较简单， 为了安全性考虑，公司内部文档禁止使用部署在外部服务器上的工具。

## 如何搭建 wekan？

>  官方教程：https://github.com/wekan/wekan/wiki/Install-Wekan-Docker-in-production

1. 安装 Docker
2. 下载 https://github.com/wekan/wekan-mongodb
3. 进入 mongodb 文件夹
4. 修改 yaml 中的 url 信息
5. 执行 `docker-compose up -d` 或者 `docker-compose up`

## wekan 简易说明

### 初步接触

目前是在 Mac 服务器上搭建了 wekan，暂时没有域名，大家先使用 IP 地址登录， [http://30.3.131.128](http://30.3.131.128)，之后就是注册账号： --统一约定使用中文花名，邮箱使用公司邮箱--，

![](http://gw.alicdn.com/mt/TB1Qu_nd8mWBuNkSndVXXcsApXa-1566-950.png)

之后进入看板首页，这里可以管理个人名下所有项目，新增看板也在这里，下面以工程组为例，简单介绍下如何使用看板。

首先，创建一个看板，![](http://gw.alicdn.com/mt/TB1UY6MmgmTBuNjy1XbXXaMrVXa-303-210.png)

接下来可以看到看板的整体页面，

![](http://gw.alicdn.com/mt/TB1ThO.d3KTBuNkSne1XXaJoXXa-1917-956.png)

- 左上角为工程属性，例如是否私有，是否提示通知
- 左侧整体面板为看板主体内容
- 右侧为属性栏，包括系统日志、项目成员、标签设置等
- 右上角为工具栏，包括搜索、过滤、列表/泳道切换、多选、看板菜单等

### 需求看板

接下来需要做的事情就是建立列表和泳道，这里我们约定，需求跟任务划分为两个看板，例如，所有工程618的需求放到一个看板中，包括待定需求、进行中的需求、已完成需求，不需要写明需求详细工期，仅列出本期都有哪些需求即可；所有的需求细分出来的任务单独放一个看板中，需要细化到每人每天的工作。需求看板示例：

![](http://gw.alicdn.com/mt/TB18GyQmDJYBeNjy1zeXXahzVXa-1919-943.png)

这里我们约定划分这么几个列表：

- 待梳理需求：需要进一步评估是否需要做或者如何做
- 长期规划：一般是3个月以上，可以视项目情况而定，移除该列表也可以
- 中期规划：一般是1-3个月
- 本月：当前月需要完成的需求
- 已交付：该版本已完成需求
- 后续可以看情况自定义部分列表，例如已延期
- 不同泳道即不同模块，整体大图的话就可以按照算法、2D、3D 这样划分，小组内部可以根据功能划分，例如 CPU、GPU 等

右侧面板中需要添加其他成员，这样其他人才能访问

### 任务看板

之后我们就需要根据刚才整理出来的需求进一步分配工时，列表可以参考下图：

![](http://gw.alicdn.com/mt/TB1_ZzUd4uTBuNkHFNRXXc9qpXa-1633-949.png)

- 需求列表：即本月需要完成的需求描述
- Task-todo：待完成任务
- Task-doing：进行中的任务
- Task-done：已完成任务
- 需求-testing：测试中的需求
- 需求-bug：测试中发现的 bug 列表
- 需求-done：需求完成

### 任务详情

![](http://gw.alicdn.com/mt/TB1x8O1mDJYBeNjy1zeXXahzVXa-807-885.png)

主要分为描述、成员、标签、耗时、评论等。

### 标签使用

![](http://gw.alicdn.com/mt/TB1BCoRmbGYBuNjy0FoXXciBFXa-299-283.png)

按照需求优先级划分标签

## 项目约定

1. 账号统一使用中文花名
2. 邮箱使用公司邮箱
3. 所有项目均设置为私有
4. 每个组分为两个看板，分别为需求看板及任务看板
5. 标签按照需求优先级划分

