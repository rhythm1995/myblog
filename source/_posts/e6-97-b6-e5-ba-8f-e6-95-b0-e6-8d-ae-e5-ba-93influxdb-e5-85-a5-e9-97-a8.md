---
title: 时序数据库Influxdb入门
url: 313.html
id: 313
comments: false
categories:
  - node.js
  - 后端
  - 数据库
date: 2019-09-12 20:06:37
tags:
---

简介
==

InfluxDB是一个时间序列数据库，旨在处理高写入和查询负载。它是TICK堆栈的组成部分。InfluxDB旨在用作涉及大量带时间戳数据的任何用例的后备存储，包括DevOps监控，应用程序指标，物联网传感器数据和实时数据分析。

特点
==

InfluxDB具备以下特点：

*   专为时间序列数据编写的自定义高性能数据存储。TSM引擎允许高摄取速度和数据压缩
*   完全写在Go。它编译成单个二进制文件，没有外部依赖项。
*   简单，高性能的写入和查询HTTP API。
*   插件支持其他数据提取协议，如Graphite，collectd和OpenTSDB。
*   为类似SQL的查询语言量身定制，可轻松查询聚合数据。
*   标签允许对系列进行索引以实现快速有效的查询。
*   保留策略有效地自动使过时数据过期。
*   连续查询自动计算聚合数据，以提高频繁查询的效率。

安装
==

在mac上的安装使用brew

```sh
    brew update
    brew install influxdb
    ln -sfv /usr/local/opt/influxdb/*.plist ~/Library/LaunchAgents

    # 配置文件在/etc/influxdb/influxdb.conf ，如果没有就将/usr/local/etc/influxdb.conf 拷一个过去
    配置缓存：cache-max-memory-size

    #启动服务
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.influxdb.plist

    #停止服务
    launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.influxdb.plist

    #前台启动
    influxd -config /usr/local/etc/influxdb.conf

    #查看influxdb运行配置
    influxd config

    #启动交互式客户端
    influx -precision rfc3339
```


基本概念
====


**time**是时间(time)，具体格式必须是一个时间戳或者RFC3339时间。 **butterflies**、**honeybees**是字段（field），包括字段键（表头）和字段值（表格内容）。 、**location**、**scientist**是标签（tag），同样包括了标签建（表头）和标签值（表格内容）。注意标签和字段第一眼看上去非常相似，其区别是标签是一个类似枚举的结构，只有几种可选的标签值。这与字段不同，标记是索引的。这意味着标签上的查询更快，并且该标签非常适合存储常用查询元数据。 在influxDB中，有个叫做series的概念，这个series是数据可视化中的数据，是通过tags排列组合出来的。一般在echarts等库中也可以看到相似的概念。

基本操作
====

数据库操作
-----
```sql
    /*创建数据库*/
    CREATE DATABASE mydb

    /*使用数据库*/
    use DATABASE mydb

    /*删除数据库*/
    drop DATABASE mydb
```

数据measurement的操作
----------------

```sql
    /*插入数据：插入了一条数据*/
    insert testTable<表名字>, butterflies=3 ,honeybees=28 , location=1 ,scientist=perpetua<内容>
    /*查询数据：查询最近的3条数据*/
    SELECT * FROM weather ORDER BY time DESC LIMIT 3
```

HTTP接口
------

InfluxDB直接提供了一套HTTP接口，部分操作如下：

```sql
    # 插入数据：插入了一条数据
    curl -i -XPOST 'http://localhost:8086/write?db=myDB' --data-binary 'testTable, butterflies=3 ,honeybees=28 , location=1 ,scientist=perpetua'

    # 查询数据：查询最近的3条数据
    curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=myDB" --data-urlencode "q=SELECT * FROM testTable ORDER BY time DESC LIMIT 3"
```

node.js中的实用
-----------

目前已经有第三方的npm包，所以在node.js中也非常方便

```js
    /**@type InfluxDB*/
    var influx = require('influx')
    var async = require("async")
    var ut = require("./../../util/util.js")
    var dbName = "mydb"
    var tableName = "testTable"
    var client = influx({
        host : '0.0.0.0',
        port : 8086, // optional, default 8086
        protocol : 'http', // optional, default 'http'
        username : '',
        password : '',
        database : mydb
    })
    var altitudes = [1000, 5000]
    var areas = ["北", "上", "广", "深"]
    async.waterfall([
            function(cb){ // 创建数据库
                client.createDatabase(dbName, function(err,result){
                    ut.log("createDatabase", result)
                    cb(err, null)
                } )
            },
            function(result, cb){ // 获取数据库名字
                client.getDatabaseNames( function(err, result){
                    ut.log("getDatabaseNames", result)
                    cb(err, null)
                } )
            },
            function(result, cb){ // 写入数据
                var points = [
                    [
                        {
                            temperature: ut.RandByRange(0, 100), humidity : ut.RandByRange(-15, 30)
                        },
                        {
                            altitude: altitudes[ut.RandByRange(0, altitudes.length)], area : areas[0]
                        },
                    ],
                    [
                        {
                            temperature: ut.RandByRange(0, 100), humidity : ut.RandByRange(-15, 30)
                        },
                        {
                            altitude: altitudes[ut.RandByRange(0, altitudes.length)], area : areas[1]
                        },
                    ],
                ]
                client.writePoints(tableName, points, function(err, result){
                    ut.log("writePoint", result)
                    cb(err, null)
                } )
            },
            function(result, cb){ // 查询数据
                client.query( 'SELECT * FROM weather ORDER BY time DESC LIMIT 3', function(err,result){
                    ut.log("query", result)
                    cb(err, null)
                } )
            },
            function(result, cb){
                client.getMeasurements( function(err,result){
                    ut.log("getMeasurements", JSON.stringify(result))
                    cb(err, null)
                })
            }
        ]
        , function(err, result){
            ut.log("finish...", err, result)
        }
    )
```