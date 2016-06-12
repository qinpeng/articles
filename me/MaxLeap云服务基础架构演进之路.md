## 介绍
公司最初是一家移动应用和手游研发公司，积累了很多研发和运营成果，后成立了云服务部门，专注于为企业提供一站式云端服务方案和研发平台。

云服务的发展经历三个阶段：

1.0，定位对内服务，为公司研发的几十款应用提供服务端功能，推送、统一用户管理等API接口，可以说是一个最普通的接口服务。经历了半年左右时间。

2.0，定位对外服务，支撑公司的移动研发以外，对外开放，提供更多的功能接口，很多功能参考行业的通用做法，可以说是一个BaaS的云平台。经历了一年半左右。

3.0，定位基础研发平台，工具链完整，从研发、上线、运维到运营，提供应用管理、支付、IM、推送等SaaS功能，也提供托管、发布、监控、日志等PaaS功能，可以说是一个SaaS + PaaS的研发平台。


## 几个概念

SaaS：云端的业务应用，HR、CRM等– Salesforce、Successfactor、Zendesk

PaaS：云端的研发、测试、部署等，自动化了程序的配置、发布、管理–Heroku、Google App Engine、Force.com

IaaS：云端的基础设计PC、存储、网络等–阿里云、AWS、Azure

BaaS：云端的服务平台，针对移动开发
- 综合类：Parse、Kinvey
- 分析类：友盟、TalkingData、神策数据
- 支付类：Beecloud、Ping++
- IM类：环信、网易
- 消息类：极光、个推

## 1.0时代 单应用

### 背景
- 几十款App需要管理

- 统一的App用户体系

- 统一的云参数体系

- 统一的推送服务

- 统一的数据存储

- 统一的广告服务

- 特殊需求单独实现

### 业务模型

应用的种类包括浏览器、音视频工具、社交工具、清理大师、图片存储类和手游等，门类很多。用户、推送、广告等基础接口和SDK需要统一，提高研发团队的开发效率，降低维护成本。对于特定的需求，增加接口，单独实现。

![MaxLeap1.0业务](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-974c1ea6-4b2f-42b3-9fba-4e964fbb2580.png)

### 设计思路

- 快速研发

- 小成本完成业务需求

- 针对特定服务强调性能

### 架构

结合团队情况和产品需求，决定采用如下实现方式：

![MaxLeap1.0架构](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-63125c6a-c50c-41cb-bd81-da29ca6ec3a5.png)

- 典型的web应用架构方式，使用Nginx做反向代理和负载均衡，后面跟了多个JVM实例；每个JVM实例由Jetty作为应用服务器，提供REST接口；服务层实现具体的逻辑；DAL层对DB和缓存进行封装，提供统一的数据访问接口。

- Redis作为缓存方案，支持多个shard水平扩容，TPS高、性能好。

- Cassandra作为数据存储引擎，无中心、可水平扩展、易维护，没有专门的运维人员，对研发人员非常友好；没有事务操作的需求NoSQL完全满足。

- RabbitMQ作为消息中间件方案，不同进程间通信，支持HA，支持持久化。

- Zookkeeper存储基础配置信息。

### 成果

- 支持了公司几十款应用的运行，Redis没有采用高可用方案，中间出过一次事故

- 日访问量数十亿级别

### 面临的挑战

- Cassandra作为业务数据的存储，查询非常不灵活依赖建索引

- 整个系统没有完全脱单

- 服务部署在一起，出问题时相互影响

- 耦合度高，扩展困难

- 开发效率低，发布新功能时相互依赖

## 2.0时代 - 服务化

### 背景

- 定位公有云

- 服务标准化

- 多租户支持，用户数据需要隔离

- 更灵活的存储和查询服务

- 提供基础数据分析功能

- 提供代码托管服务

- 更多云服务

### 业务模型

对外服务，每个公司有自己的后台管理和账号管理，不同工作人员区分权限职责。每个公司可以允许有多款应用，应用在逻辑上相互独立。每个应用都可以使用所有的服务。

支持的云服务更多，进行标准化。比如推送营销、云存储、云代码、支付、即时通讯等。

![MaxLeap2.0业务](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-4e4f26a7-a4b1-4083-98a3-3bdfd3ee4dae.png)

### 设计思路

- 服务纵向拆分

- MongoDB + MySQL作为数据存储的核心DB

- 服务容器化，引入Docker

- 云分析支持，基于Lambda架构

- 云代码支持，引入Docker

### 云服务架构

![MaxLeap2.0服务](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-58565e2d-d864-48ad-88e4-7bc65e42c91d.png)

#### 网络
最外层增加了ELB，不把IP地址直接暴露在公网，通过DNS配置CNAME指向域名，降低了被DDOS的风险。提高了Nginx的可用性。

#### 服务
每个服务都运行在Docker中，服务之间环境和资源隔离，能够快速迁移和部署，统一了交付流程。

#### 存储
- NoSQL部分主要依赖MongoDB，Wired Tiger + SSD，schema-less，开发者可以根据自己的需求随时调整collection的定义；天生为分布式设计，支持复制集高可用方案，支持shard水平扩展；
- 关系型数据库依赖MySQL，支付、订单、配置信息等需要事务支持的业务，使用AWS的RDS，主要考虑是有HA、降低运维成本。
- 块数据依赖AWS的S3，权限控制、分桶策略比较实用，可用性有保障。
- 缓存 - 采用Redis 3.0的集群方案，shard + replication。


### 云分析架构

![MaxLeap2.0分析](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-63b418a9-d528-4e54-a8e3-9ad7d51ff20b.png)

- 标准Lambda架构，分为Speed Layer、Batch Layer、Serving Layer三层。
- Speed Layer基于Storm，从kafka中获取后数据，实时把一些指标计算完成，写入展示层；Batch Layer，基于Hadoop生态构建，通过Camus，把数据存储在hfs，然后通过hive或者M/R，计算各种指标，工作流通过oozie来调度，最终结果也写入展示层，并且覆盖掉实时计算的结果。
- DT，DataTransform，用于数据标准化、过滤、流控等用途，基于vert.x实现。
- 展示层，数据用Cassandra存储，有很高的写速度，无限水平扩展，antlr4j实现SQL解析，访问Cassandra的数据。数据根据不同指标，按天、周、月分别存储。

### 云代码架构

![MaxLeap2.0云代码](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-1140a322-cd79-4ffc-82af-818b4eb5ccfd.png)

类似hosting的一种服务，按照平台提供的函数模板和SDK去实现复杂一些的或者有安全考量的业务逻辑，运行在服务器端，目前支持java、python和node.js。

#### 流程

1. 用户上传云代码后，hydra会制作一个docker镜像，并且打上版本标签。

2. 用户选择实例的数量和配置，CPU、内存，并且启动

3. 通过特定的规则和认证方式，用REST，能够访问到实现的云函数。

4. hydra，开始监控每个容器的状态，记录日志，并且实现failover、扩展实例、调度功能。


### 面临的挑战

- Nginx的反向代理不能动态设置，不支持动态域名设置

- 需要统一日志系统

- 需要统一监控系统

- 基础功能需要组件化

### 成果

- 公有服务初具规模

- 完成移动应用研发大部分组件（统计分析、支付、IM、社交、在线参数、推送、营销、云数据、云存储、云代码)

- 使应用的研发、上线、运维、运营形成闭环

## 3.0时代 - 平台化

### 背景

- 支撑公司另一款SaaS平台的研发

- 支持容器托管

- 数据源服务支持

- 域名支持

### 业务模型

![MaxLeap3.0业务](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-7f313e1d-1195-490d-a7e4-de3ca74ad14b.png)

新的业务系统对基础平台提出了新的要求。第一是需要支撑公司一个全网营销的SaaS产品线，产品支持配置操作生成App + 微信商城 + 手机网站 + PC网站，以及营销。第二是整个基础组件需要进行升级、数据访问、日志、监控系统等。

### 设计思路

- 统一网关 - 转发规则、限速限流

- 统一基础组件 - 数据访问

- 统一日志系统

- 统一监控

### 网关

- 规则转发

- 主机注册

- 容器状态检查

- load balance

![MaxLeap3.0网关](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-1937b0f8-70a7-46b7-b579-c3bf7c036ce7.png)

由网关组件和容器管理组件两部分组成。应用在启动后，向hydra注册自己的服务、ip和端口信息，hydra进行健康检查，failover处理，并把相关的信息保存在zk和MySQL中。网关在请求到达后，进行规制匹配、路由转发、限制、限速等处理。

目前网关用在了大多数服务中，下步计划是所有服务统一由网关进行管理，最终形态：

![MaxLeap3.0网关02](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-bfb33726-5fa2-4cb8-bdb5-a5a5295e8ba9.png)

### 基础组件

结构化业务数据主要基于MySQL和MongoDB进行存储，抽象了数据代理层，代理主要有几个功能。

1. 数据访问路由

2. 数据访问鉴权

3. 日志采集

4. 流控和风控

![MaxLeap3.0公共组件](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-3a4087a5-8801-49c3-a541-bec4f1369f7f.png)


### 日志系统

采用成熟实用的ELK，每台主机的Logstash Angent采集格式化的日志数据，向Logstash Server发送，并将数据保存在ES中。在Kibana中建立各种服务的面板和视图，进行浏览和日志定位。

![MaxLeap3.0日志](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-f965239a-389d-4dfd-b54e-a7b415d916d0.png)

部分主机收集到的系统Metrics：

![MaxLeap3.0日志02](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-b07edfdf-1109-4496-b072-e4243821ad3f.png)

利用Kibana定位问题：

![MaxLeap3.0日志03](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-59c90d0b-d839-49a3-8783-5aa9ae95d822.png)


### 监控系统

基于日志系统的基础架构，不同服务将Metrics输出到文件系统，通过LogStash和ES收集。在Kibana中定义视图，Kibana使用ES作为自己的存储引擎。比如新增一个视图后，Kibana会在ES的.kibana这个索引中创建一份视图的定义。

![MaxLeap3.0监控](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-62dfcf16-f314-4e79-9332-ec6fa518c56e.png)

有了Metrics数据和视图的展示，下一步是报警。利用Nagios的报警机制，基于JNRPE实现报警的逻辑。报警逻辑通过读取ES中.kibana某一个视图的定义，和对应的阈值设置，通过比较当前值和阈值，返回某一个服务对应特定指标的监控状态。Nagios拿到状态后，决定是否报警。

![MaxLeap3.0监控02](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-ff054737-e013-4ac6-9e53-bdabbd96e115.png)

Kibana中定义视图

![MaxLeap3.0监控03](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-1619f498-dafa-4e13-b75e-df4e23b63ec0.png)

在ES中设置报警阈值

![MaxLeap3.0监控04](https://cscdn.maxleap.cn/2.0/download/NTczMjkwMzkxNjllN2QwMDAxN2ZmZGM5/zcf-ab347413-21dd-42a7-8210-5eec0736efcd.png)

## 总结

第一，架构随业务而变，不能脱离业务去谈技术，满足业务是原动力

第二，成熟简单的技术就是最合适的

第三，快速迭代，方便上线

















