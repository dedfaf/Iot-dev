# 物联网应用系统开发与实践 - 开发报告

## 总述

## 模块实现

### 客户端+模拟定位

### 服务端Netty

#### Netty 车辆跟踪

该系统由使用 Java 的 Netty 框架构建的服务器和客户端组成，旨在通过客户端和服务器之间交换 JSON 格式的消息来跟踪车辆位置，并将车辆数据存储和管理在 MySQL 数据库中。

##### Netty 服务器：NettyServerHandler 类

1. NettyServerHandler 类的功能：

    该类继承了 ChannelInboundHandlerAdapter，用来处理从客户端接收到的消息，并将处理后的结果发送回客户端。
    在构造函数中，初始化与 MySQL 数据库的连接，通过 JDBC 驱动程序连接到名为 CarSys2 的数据库。

2. channelActive 方法：

    当客户端连接到服务器时，此方法被调用。它向客户端发送一条 "Connection successful" 的确认消息。

3. channelRead 方法：

   - 读取客户端发送的消息，期望接收到的是 JSON 格式的数据。
   - 解析 JSON 字符串并提取出车牌号 (licensePlate)、令牌 (token)、经度、纬度和速度等信息。
   - 使用 BigDecimal 将经纬度四舍五入到六位小数，以确保精确性。
   - 验证收到的 token 是否有效。如果令牌无效，向客户端发送错误消息并终止进一步处理。
   - 如果令牌有效，更新车辆在数据库中的状态（例如，将车辆的 isOnline 状态设为 true 并更新位置数据）。

4. isValidToken 方法：

    用于验证从客户端接收的令牌是否与数据库中的记录匹配。

5. channelInactive 方法：

    当客户端断开连接时被调用。它将数据库中相应车辆的 isOnline 状态更新为 false。

6. exceptionCaught 方法：

    处理通道中的异常，如果发生异常，它将打印堆栈跟踪并关闭通道。

##### Netty 客户端：NettyClientHandler 类

1. NettyClientHandler 类的功能：

    该类也继承了 ChannelInboundHandlerAdapter，用于处理与服务器的通信。
    使用 ScheduledExecutorService 来定期发送车辆位置的 JSON 消息给服务器。

2. channelActive 方法：

    当客户端与服务器成功连接时调用。它设置一个定时任务，每 500 毫秒发送一次包含车辆位置的 JSON 数据。

3. channelRead 方法：

    读取从服务器接收到的消息并在控制台上输出。

4. Car 内部类：

    - 用于表示车辆，包含车牌号、经纬度、速度和令牌等信息。
    - 提供 updatePosition 方法根据速度更新车辆的位置。
    - 提供 toJson 方法将车辆的状态转换为 JSON 格式字符串。

##### 系统工作流程

客户端与服务器建立连接。

客户端定期发送车辆的位置信息（包括经纬度、速度和令牌）给服务器。

服务器验证接收到的令牌，如果验证通过，服务器将更新数据库中相应车辆的位置信息。

在客户端断开连接时，服务器将车辆状态更新为离线。

#### Netty 车辆令牌管理

该系统使用 Java 和 Netty 框架构建，主要用于管理车辆信息并生成或验证车辆的令牌。系统包括一个服务器端和客户端，双方通过 JSON 格式的消息进行通信，同时使用 MySQL 数据库来存储车辆数据和令牌信息。

##### 客户端：NettyTokenClient 类

1. NettyTokenClient 类的功能：

    负责与服务器建立连接并发送请求数据。
    初始化时需要指定服务器的 IP 地址和端口号。

2. run 方法：

    配置 Netty 客户端启动参数（如使用的通信协议和处理器）。
    创建一个 JSON 对象并设置车牌号，然后将其转换为字符串格式发送给服务器。
    等待服务器的响应，并在通道关闭后终止客户端。

3. NettyTokenClientHandler 类：

    继承自 SimpleChannelInboundHandler，用于处理从服务器接收到的响应消息并将其输出到控制台。
    处理通信中的异常情况并关闭通道。

##### 数据库管理：DatabaseManager 类

1. DatabaseManager 类的功能：

    负责与 MySQL 数据库建立连接和执行 SQL 查询。
    提供方法查找车辆的令牌，如果不存在则生成新的令牌。

2. findVehicleTokenByLicense 方法：

    根据车辆的车牌号在数据库中查找对应的令牌。
    如果找到匹配的车牌号，则返回相应的令牌；否则，插入新记录并生成一个新的令牌。

3. insertVehicleAndGetToken 方法：

    向数据库中插入新的车辆记录，并生成一个新的唯一令牌（UUID）。
    将生成的令牌与车辆记录相关联并保存到数据库。

##### 服务器端：NettyTokenServer 类

1. NettyTokenServer 类的功能：

    负责初始化和启动 Netty 服务器，监听指定端口以等待客户端的连接。
    使用 ServerHandler 类来处理客户端发送的消息。

2. run 方法：

    配置服务器的启动参数（如使用的通信协议和处理器）。
    绑定指定端口，启动服务器，并保持运行直到手动关闭。

3. ServerHandler 类：

    继承自 SimpleChannelInboundHandler，用于处理从客户端接收到的 JSON 消息。
    解析消息中的车牌号，并调用 DatabaseManager 来查找或生成对应的令牌。
    根据查找结果，将令牌或新生成的令牌返回给客户端。

##### 系统工作流程

客户端启动：NettyTokenClient 连接到服务器并发送车辆信息（车牌号）给服务器。

服务器接收请求：NettyTokenServer 通过 ServerHandler 处理接收到的消息，查找或生成车辆的令牌。

数据库操作：DatabaseManager 执行相应的数据库查询或更新操作。

返回响应：服务器将查找到的令牌或生成的新令牌发送回客户端。

客户端输出结果：客户端接收服务器响应并在控制台输出结果。

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
