---
layout: post
title: 时序数据库InfluxDB在.NET Core中的使用——（1）InfluxDB简介及使用
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

根据网络上的众多测评文章，同样的时序型数据，时序数据库的读写性能都比关系型数据库、NoSQL数据库要快，占用资源更少。所以，时序数据库适合数据结构固定、与时间顺序关联性强、数据量大并且读写性能要求高的使用场景，比如日志、监控数据、大数据存储等，以我这个DDD践行者来说，更是事件溯源架构中事件存储方式的最优选择。

### 特色

1. 基于时间序列，支持与时间有关的相关函数（如最大，最小，求和等）
2. 可度量性：你可以实时对大量数据进行计算
3. 基于事件：它支持任意的事件数据

### 特点

1. 无结构（无模式）：可以是任意数量的列
2. 可拓展的，通过配套的代理插件可实现分布式
3. 支持min, max, sum, count, mean, median 等一系列函数，方便统计
4. 原生的HTTP支持，内置HTTP API
5. 强大的类SQL语法
6. 自带管理界面，方便使用

### 缺点

只有一个，只适合固定的数据结构，如果结构有变动，变动前的数据讲很难查询到，需要从数据源重新导入，所以说是用灵活性的牺牲来换取超高的性能，但是它的应用场景依然宽泛。

## 概念

##### Database

数据库名，在 InfluxDB 中可以创建多个数据库，不同数据库中的数据文件是隔离存放的，存放在磁盘上的不同目录。

##### Measurements

相当于传统数据库的表，例如 cpu_usage 表示 cpu 的使用率。

##### Tag sets

标签，tags 在 InfluxDB 中会被建立索引，且放在内存中。如果某种数据经常用来被作为查询条件，可以考虑设为Tag。

##### Fields

字段，是各种数据类型的存储和查询时的主要对象。

##### Points

点，代表一条记录，由时间戳（time）、数据（field）、标签（tags）组成。

- time : 每个数据记录时间，是数据库中的主索引(会自动生成)
- tags : 各种有索引的属性，只支持字符串
- fields : 各种记录值（没有索引的属性）也就是记录的值

##### Series

序列，以retention policy、 measurement、 tagset共同组成的定位测点作为序列的唯一标识的所有记录，每个时刻的记录就是Point。

##### Timestamp

时间戳，每一条数据都需要指定一个时间戳，在 TSM 存储引擎中会特殊对待，以为了优化后续的查询操作。

##### Retention Policies

存储策略，用于设置数据保留的时间，每个数据库刚开始会自动创建一个默认的存储策略 autogen，数据保留时间为永久，之后用户可以自己设置，例如保留最近2小时的数据。插入和查询数据时如果不指定存储策略，则使用默认存储策略，且默认存储策略可以修改。InfluxDB 会定期清除过期的数据。

##### Continuous Queries

连续查询，使用连续查询是最优的降低采样率的方式，连续查询和存储策略搭配使用将会大大降低InfluxDB的系统占用量。而且使用连续查询后，数据会存放到指定的数据表中，这样就为以后统计不同精度的数据提供了方便。

### 注意

由于Tag与Field的不同特性，在编写SQL进行查询时，Tag与Field支持不同的操作，总结如下:

##### Tag

- 只能使用Tag进行Group
- 只能使用Tag进行正则表达式操作

##### Field 

- 只能使用Field进行函数操作，例如sum()
- 只能使用Field进行比较操作
- 如果需要使用int,float,boolean类型进行存储,只能使用Field

## 安装与使用

安装方法很简单，除了在官方网站[https://influxdata.com/downloads/#influxdb](https://influxdata.com/downloads/#influxdb)下载安装程序外，官方还提供了Docker镜像，我就是在Docker Hub拉取镜像并且部署的，毕竟才14MB大小，而且还与本机环境隔离。

安装好后，数据库服务的地址是`http://localhost:8086`，初始用户名是admin，初始密码也是admin。数据库可以通过发送Http请求来操作，操作语句与sql语言基本保持一致。

- 创建一个数据库：

    `curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE mydb"`

- 写入一个Point(记录)：

    `curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu_load_short,host=server01,region=us-west value=0.64 1434055562000000000'`

- 查询操作：

    `curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=mydb" --data-urlencode "q=SELECT \"value\" FROM \"cpu_load_short\" WHERE \"region\"='us-west'"`

    返回的会是一段Json数据：

```json
{
    "results": [
        {
            "statement_id": 0,
            "series": [
                {
                    "name": "cpu_load_short",
                    "columns": [
                        "time",
                        "value"
                    ],
                    "values": [
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            2
                        ],
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            0.55
                        ],
                        [
                            "2015-06-11T20:46:02Z",
                            0.64
                        ]
                    ]
                }
            ]
        }
    ]
}
```

## 小结

本篇文章主要介绍了InfluxDB的基本概念以及安装和基本操作方法，更详细的介绍请参阅[官方文档](https://docs.influxdata.com/influxdb/v1.3/guides)以及热心开发者发表的[系列教程](https://www.linuxdaxue.com/series/influxdb-series/)

性能测试参考：[http://www.cnblogs.com/MikeZhang/p/InfluxDBTest20170212.html](http://www.cnblogs.com/MikeZhang/p/InfluxDBTest20170212.html)

下一篇，我将动手利用一款InfluxDB客户端结合.NET Core自带IoC框架和仓储模式实现事件存储。