# 物联网应用系统开发与实践 - 开发报告

## 总述

## 模块实现

### 客户端+模拟定位

### 服务端Netty

### 数据库

数据库服务器使用MySql 8.4.2 - GPL，其中维护了两个表：

1. Vehicle

    记录车辆的基本信息

2. VehicleLoc

    记录车辆的位置

``` mermaid
erDiagram
    Vehicle ||--|| VehicleLoc : "ShowLoc"
    Vehicle {
        int id PK
        string License
        bit isOnline
        string Token
    }
    VehicleLoc {
        int id PK, FK
        double Lat
        double Log
        double Speed
        timestamp Time_update "default current_timestamp()"
    }
```

基础表和位置表一一对应，拆开为两个目的：

1. 若需要实现服务器数据库存储历史位置，只需要将映射关系改为1 : n对应
2. 方便了地图端的查询

本数据库维护了两个触发器，仅用于周五的展示，不作为最终版本的功能，目前用于实现：

1. [goOffline](../sql/Trigger-LocGoOffline.sql)：实现车辆下线后，自动将位置表中对应的信息删除
2. [UpdateTime](../sql/Trigger-LocUpdateTime.sql)：实现位置表更新后自动更新时间至当前时间

#### api封装

将数据库封装为webapi，用于数据库的数据读取

使用[database2api](https://github.com/mrhuo/database2api)，一个使用ktor框架的轻量级数据库封装程序

[部署的Api Index：http://182.92.68.227:3301](http://182.92.68.227:3301/)

### 在线地图

## 总结
