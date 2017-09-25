---
layout: post
title: 时序数据库InfluxDB在.NET Core中的使用——（1）InfluxDB简介
date: 2017-09-25 22:56:01
permalink: influxdb-in-dotNEt-Core-1-Introduction
tags:
- .NET Core
- InfluxDB
categories:
- .NET Core
description:
thumbnail: https://www.msolution.io/wp-content/uploads/2015/12/influxdb.png
---

## 简介

> InfluxDB是一个由InfluxData开发的开源时序型数据库。它由Go写成，着力于高性能地查询与存储时序型数据。InfluxDB被广泛应用于存储系统的监控数据，IoT行业的实时数据等场景。
>
> ——维基百科

根据网络上的众多测评文章，同样的时序型数据，时序数据库的读写性能都比关系型数据库、NoSQL数据库要快，占用资源更少。所以，时许数据库适合数据结构固定、与时间顺序关联性强、数据量大并且读写性能要求高的使用场景，比如日志、监控数据等，更是事件溯源架构中事件存储方式的最优选择。

## 概念

##### Database

 数据库名，在 InfluxDB 中可以创建多个数据库，不同数据库中的数据文件是隔离存放的，存放在磁盘上的不同目录。

##### Measurements

相当于传统数据库的表，例如 cpu_usage 表示 cpu 的使用率。

##### Tag sets

tags 在 InfluxDB 中会被建立索引，且放在内存中。如果某种数据经常用来被作为查询条件，可以考虑设为Tag。

##### Fields

记录值，是查询的主要对象。

##### Points

代表一条记录，由时间戳（time）、数据（field）、标签（tags）组成。

- time : 每个数据记录时间，是数据库中的主索引(会自动生成)
- tags : 各种有索引的属性，只支持字符串
- fields : 各种记录值（没有索引的属性）也就是记录的值

##### Series

以retention policy、 measurement、 tagset共同组成的定位测点作为序列的唯一标识的所有记录，每个时刻的记录就是Point。

##### Timestamp

每一条数据都需要指定一个时间戳，在 TSM 存储引擎中会特殊对待，以为了优化后续的查询操作。

##### Retention Policies

存储策略，用于设置数据保留的时间，每个数据库刚开始会自动创建一个默认的存储策略 autogen，数据保留时间为永久，之后用户可以自己设置，例如保留最近2小时的数据。插入和查询数据时如果不指定存储策略，则使用默认存储策略，且默认存储策略可以修改。InfluxDB 会定期清除过期的数据。

##### Continuous Queries

连续查询，

### 注意

由于Tag与Field的不同特性，在编写SQL进行查询时，Tag与Field支持不同的操作，总结如下:

##### Tag

- 只能使用Tag进行Group
- 只能使用Tag进行正则表达式操作

##### Field 

- 只能使用Field进行函数操作，例如sum()
- 只能使用Field进行比较操作
- 如果需要使用int,float,boolean类型进行存储,只能使用Field

## 在.NET Core中的应用

本文用了比较大的篇幅去介绍InfluxDB的基础概念， 下一篇文章会介绍如何在.NET Core中使用。