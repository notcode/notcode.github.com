---
layout: post
title: "refactor database design"
date: 2013-10-13 19:40
comments: true
categories: 
    design
    chaos
---

#### 原来的设计

最近被临时调入一个项目，任务是重写后台计算程序, 所以要保持现有的接口和数据库设计。
原有的数据库被设计成每个task一张表，表名为`<prefix>_<taskid>`，简单说明一下:
> * prefix 用于区别报表类型，每个task有10多种报表类型
> * taskid 整数，用于标识task
每个task有一个开始日期和结束日期，在这个日期区间内，每天计算一次该task的报表。
每个table都有date列, 表明该行记录是哪一天的计算结果

<!-- more -->

#### 新的设计

后台重构后，需要将修改数据库结构，表名为`<prefix>_<date>`，表内有个taskid指明该条记录属于哪个task.
而为了不影响前端访问数据库展示报表，希望不要修改表结构。

#### 解决方案

* 预先创建一组名为`<prefix>`的表,每天的`<prefix>_<date>`继承自`<prefix>`.
* 为了不影响前端，每个task创建一个view:
> create view &lt;prefix&gt;&lt;taskid&gt; as select * from &lt;prefix&gt; where taskid=&lt;taskid&gt;

前端都没有察觉到后台数据库结构变了，后台程序有可以按照希望的方式建表。
