---
title: hot, warm and cold data 热数据, 温数据, 冷数据
categories:
- DataScience
tags:
- 大数据
- 数据科学
- 数据仓库
- data warehouse
- Data Science and Big Data Technology
date: 2022-05-04T09:16:12+08:00
year: 2022
week: 18
updated: 2022-05-04T09:16:12+08:00
---

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/C6A8224158E488BE8A58B933BDD1F104985501DB.webp)

<!-- more -->

* 分类依据 classification criteria
  * data access frequency 数据访问频率

## 热数据

* 指频繁访问的在线类数据
* 对存储性能要求高

> Hot data is frequently accessed on faster storage

> 热数据是需要被计算节点频繁访问的在线类数据，比如可以是半年以内的数据，用户经常会查询它们，  
> 适合放在数据库中存储，比如MySQL、MongoDB和HBase。

部分热数据是需要通过 redis memcache 等 cache (缓冲) 非持久化内存数据库进行访问， 数据的实际存储还是放在持久化数据库中

## 温数据

* 访问频率和存储性能要求介于冷热数据之间

> warm data is accessed less frequently and stored on slightly slower storage

温数据是非即时的状态和行为数据，也可以简单理解为把热数据和冷数据混在一起就成了温数据。

如果整体数据量不大, 不区分温数据和热数据。

## 冷数据

* 不经常访问的离线类数据
  * 备份数据
  * 归档数据
* 存储性能要求相对低
* 存储介质要求容量大

> cold data is rarely accessed and stored on even slower storage.

冷数据是指离线类不经常访问的数据，用于灾难恢复的备份或者因为要遵守法律规定必须保留一段时间，比如企业备份数据、业务与操作日志数据、话单与统计数据。通常会存储在性能较低、价格较便宜的文件系统里，适用于离线分析，比如机器学习中的模型训练或者大数据分析。