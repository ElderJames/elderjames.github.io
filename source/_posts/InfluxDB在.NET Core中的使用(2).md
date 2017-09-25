---
layout: post
title: 时序数据库InfluxDB在.NET Core中的使用——（2）IoC和仓储模式
date: 2017-09-26 00:44:01
permalink: influxdb-in-dotNEt-Core-2-ioc-and-repository-pattern
tags:
- .NET Core
- InfluxDB
categories:
- .NET Core
description:
thumbnail: https://www.msolution.io/wp-content/uploads/2015/12/influxdb.png
---

我上一篇文章[《InfluxDB在.NET Core中的使用——（1）InfluxDB简介》](/influxdb-in-dotNEt-Core-1-Introduction.html)中简单介绍了InfluxDB的相关概念，本文将继续介绍下半部分，就是在.NET Core中的使用方法。

## 安装与使用

安装方法很简单，除了在官方网站[https://influxdata.com/downloads/#influxdb](https://influxdata.com/downloads/#influxdb)下载安装程序外，官方还提供了Docker镜像，我就是在Docker Hub拉取镜像并且部署的，毕竟才14MB大小，而且还与本机环境隔离。

安装好后，数据库服务的地址是http://localhost:8086，初始用户名是admin，初始密码也是admin。数据库可以通过发送Http请求来操作。

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

## .NET 客户端

正因为InfluxDB是通过http请求来操作数据库，所以在.NET Core中使用很简单，只需要用HttpClient封装一个客户端、并且按照InfluxDB的数据结构去解析数据就行了。而且，已经有大神开发出了一个开源的可用于.NET Core的InfluxDB客户端项目 [InfluxData.Net](https://github.com/pootzko/InfluxData.Net) ，本文将利用这个客户端项目结合.NET Core自带的依赖注入框架和仓储模式，去封装一个事件存储的InfluxDB实现的类库。

---
未完待续。