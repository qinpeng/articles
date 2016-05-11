#### MaxLeap 移动云平台的基础架构之旅-云应用篇

```info 
作者:马传林,从事开发工作已经有5年。当前在MaxLeap数据服务组担任开发工程师，主要负责MaxWon服务器开发。
```



#### 背景

当下10后都能在手机键盘上敲字如飞，60后的都可以坦然的摇微信，移动互联网可谓炙手可热。随着智能手机的快速发展，移动APP作为登入移动互联网最便捷的方式，扼守着移动互联网的入口。现在这类软件被越来越多的人所青睐，在没有大量资金的情况下，手机APP是中小企业发展方向的一个很好选择。对于个人和企业移动APP 已经是创业和发展的必备工具。移动互联网APP开发，对于企业来说面临着项目周期，资源投入，推广等诸多问题，而对于个人更是望而却步。

传统移动开发技术方案：

  ![maxwon_chuangtong](http://o6wkmqikd.bkt.clouddn.com/maxwon_chuangtong.png)



##### 难题

1. 终端移动平台太多：Android,IOS,Windows Phone,微信 ….  而且不同平台还有版本差异，对于开发调试简直是一场噩梦，要想实现统一覆盖没有雄厚的资本支持是非常困难的。
2. 开发成本：招人难，找到合适的更难，成本高，移动开发门槛障碍
3. 捡了芝麻丢了西瓜：企业把精力投入到自己不擅长的领域大多时候并不是一件好事情，对于个人来说更是如此。
4. 全站解决方案：一个优秀的APP除了核心业务同时也需要其他优秀的组件，如:推送，用户行为分析，市场活动，数据统计等等。
5. 运维困难：要保证APP的稳定可靠运行，运维是必不可少的一部分，这些工作需要专业的运维团队来做。这样也加重了企业的负担。




现在是一个屌丝逆袭的时代，为了帮助企业和个人无门槛拥有属于自己的APP，[MaxWon](http://www.maxwon.cn/) 应运而生。

[MaxWon](http://www.maxwon.cn/) 是基于[MaxLeap SaaS](https://maxleap.cn/s/web/zh_cn/index.html) 云服务，集成不同行业模块，集 APP 生成，运营，分析，自动化运维与一体的服务，用户只需要关心自己的业务，完全摆脱上面的各种难题。

  ![maxwon_jiahua](http://o6wkmqikd.bkt.clouddn.com/maxwon_yegou.png)

用户组合自己想要的模块，点击生成APP，就可以生成自己想要的不同平台的APP，包括Android，IOS,微官网，PC官网。

##### 需要解决的的问题

1. 差异化服务。由于是面向`多租户`的服务，不同的APP产生的流量可能差异很大，系统要能做到服务隔离和水平扩展。
2. 数据隔离与扩展。为了保证数据安全，每一个APP 都会有一个独立的DB，数据只能被自己的APP访问，防止数据hack，保证数据安全。对于大数据量的APP，DB 能够支持无限扩展。
3. 快速部署与自动化运维。
4. 服务的监控。由于服务遍布在集群的不同机器上，需要能够监控所有租户服务的健康状态，保证服务的高可用行，并且能够水平扩展。
5. 支持服务和数据的迁移


#### 能独立运行的1.0

由于[MaxWon](http://www.maxwon.cn/) 需要支持不同行业，业务就会比复杂，比较多。项目业务层是按模块来划分，通过不同模块的组合来不同满足行业的需求。

   ![maxwon_yewu2](http://o6wkmqikd.bkt.clouddn.com/maxwon_yewu_005.png)

第一版架构遵循两个原则:第一， 以业务实现为目标，尽快做出产品原型。由于MaxLeap已经有很多基础的中间件可以直接拿来使用，如:推送，FAQ&Issue，支付，IM&社交等。现在只需要把精力放在[MaxWon](http://www.maxwon.cn/) 自己的业务中去。第二，快速响应产品的需求，产品指导研发，很多场景、很多的玩法必须帮助产品实现，而且速度要非常快，要快速迭代。



##### 主要技术栈

|  描述   |                    名称                    |
| :---: | :--------------------------------------: |
|  平台   |                 JDK 1.8                  |
| Web框架 | [vertx-jersey](https://github.com/englishtown/vertx-jersey)(异步) |
|  ORM  |             [JOOQ](jooq.org)             |
|  RDS  |                  Mysql                   |
| NoSql |                  Mongo                   |

对于大部分人来说 [Vert.x](http://vertx.io/)可能会有点陌生,它是基于`netty` 实现的异步架构，和node.js 极其相似。由于之前做MaxLeap服务，一直在用[Vert.x](http://vertx.io/)做为基础架构，整个团队对Vert.x 也很熟悉，该踩得坑也都踩过了，通过[Verx-Rpc](https://github.com/MaxLeap/vertx-rpc) 可以直接访问的MaxLeap 的微服务。在使用Vert.x 时最大的感受就是不能写同步代码，否则就会阻塞，导致导致服务不可用，所以我们的服务全是基于异步的方式来写的。由于它是一个轻量级高性能JVM应用平台，支持多语言开发，它的简单actor-like 机制能帮助脱离直接基于多线程编程，天生支持分布式，以后对于服务扩展也是水到渠成的事情。

对于ORM 并没有使用主流的 Hibernate或者IBATIS,而是使用小众的JOOQ。JOOQ 相对于其他ORM算是很轻量，提供了强大的DSL 来访问数据库，灵活，上手很容易，代码非常接近sql。

JOOQ [runtime schema mapping](http://www.jooq.org/doc/3.4/manual/sql-building/dsl-context/runtime-schema-mapping/) 对于多租户应用程序有很好的支持，可以很容易的实现为每个租户分配独立的DB。

还有一个重要的原因就是 JOOQ 已经和Java8 的Stream API 完全融合,cool!!。函数式编程表达性强，并且非常通用。它是数据及数据流处理的核心。Java开发人员现在也都知道函数式编程，而且大家又都用过SQL。想象一下，你用SQL来声明表来源，把数据转化成新的元组流，然后要么将它们作为派生表提供给其它更高级的SQL语句来使用，要么将它们交给你的应用程序来处理。

下面就是一段典型的Java代码

```java
DSLContext create = DSL.using(connection, dialect);
create.select(AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME, count())
      .from(AUTHOR)
      .join(BOOK).on(BOOK.AUTHOR_ID.equal(AUTHOR.ID))
      .where(BOOK.LANGUAGE.equal("DE"))
      .and(BOOK.PUBLISHED.greaterThan("20017-01-01"))      
      .limit(2)
      .offset(1)     
      .fetch(record -> transfer(record))
      .stream()
      .filter(ele -> null != ele)
      .collect(Collectors.toList());
```

有了JOOQ,Java 8以及Streams API,你可以写出强大的数据转化的API，而且简单易懂。

 ![maxwon_001](http://o6wkmqikd.bkt.clouddn.com/maxwon_10_001.png)



##### 架构特点

  将架构特点划分为优点和缺点进行描述。那么优点是：

- 简单，易于实现，不需要额外的基础支撑

- 利于业务的功能快速实现

- 服务都是以Docker Container 启动，可以实现快速发布与部署

  ​

缺点：

- 不同租户的应用无法隔离，所有的APP 都使用相同的Container，这样会带来APP之间相互影响，导致服务不稳定的风险。
- 缺少服务健康检查。
- 运维成本过大。


1.0的架构就是一个简单的Web系统。负载均衡使用Nignx，并没有细化到`租户`级别。业务系统通过代码模块的形式组织各种业务就是一个简单的Web系统，后面直接挂了数据库，比如商品、订单、会员、客服，等等。可以看到，我们这个基础的架构，对外就是HTTP。当时两个人的小团队开发各种业务，我们考虑只能用最简单、最粗暴的方式实现，能快速地实现业务。当时的流量不是第一重要的问题，也不是最主要的矛盾。

对于这个阶段，总结了三点。第一，技术来源于业务同时提升业务发展，业务发展又反过来推动技术的前进，他们是一个相互影响相互促进的关系。和业务共同发展的技术才是有生命力的。第二，成熟简单的技术就是最合适的，这个理念一直贯穿始终。不要把事情复杂的形态呈现给大家，脑子要保持简单，不要想那么复杂的事儿。第三，要把能遇到的场景尽量到考虑到，以后架构升级不至于很被动。大家看到初始的架构等于没有架构，但是这种形式在这时是最符合业务需求的一个，能快速迭代，能非常方便上线。




#### 面向多租户的2.0

在MaxWon1.0时代的时候，我们的关注点更偏向业务的实现，随着用户增长，性能和稳定性问题逐渐浮上水面，作为一个多租户的应用系统，系统不稳定，是非常致命的，2.0解决这些问题也迫在眉睫。

##### 要解决的问题

首先要解决的就是服务分离。其中有两种方案 ：

1. 每一个租户APP都有属于自己的 服务 Container，这样就解决了租户之间的相互影响。但是 大部分 APP 访问点可能很小，甚至是僵尸应用。虽然Docker 容器使用的资源很小，但是大量的不活跃应用还是会浪费掉太多的系统资源，资源利用率低。
2. 按租户的真实的访问量划分为不同的组，普通规模应用或者是僵尸应用都公用同一组Container，中等规模应用 某几个使用一组Container，对于大量数据流量的应用 独占 同一组Container，这样的话资源利用率就会很高。缺点就是 普通规模和中等规模应用 服务之间还是会有影响，由于这两种规模的数据访问的会少很多，出现慢查询而导致拖慢整个系统的可能性会很小。

对比上面的两个方案优缺点，基于现实的考虑最终选择了第二种方案。这就需要能够随时监控APP的数据访问量，当某个APP访问量快速上升时能够随时独立出服务来，这样就可以最大限度的防止租户之前相互影响而产生的服务抖动。

对于服务监控，则采用心跳检测的方式，每个服务Container对外暴露一组健康检查的接口，监控系统会定时的巡视所有服务的健康状态，如果由于某种原因被Kill掉，则重启对应Container的并产生告警。

 ![](http://o6wkmqikd.bkt.clouddn.com/maxwon_elb.png)

对于**数据存储分离** 也采用了同样的思路。对于Mongo ,`Pandora` 本身就支持按不同App 数据分治。对于Mysql代理则采用 MaxLeap 研发的 Circe组件，可以实现不同App数据的隔离。使用AWS ELB 解决了Circe的负载均衡与高可用。



2.0的采用了服务和数据分离的思想，现在回顾也并不复杂，对于码农来说这种思想已经是非常熟悉的了。如果你的产品功能不多，迭代不是很快，可以放慢一下脚步，停下来一段时间来集中一次重构。但对于MaxWon来说这一版本的迭代就像是鸟枪换炮，满足了大部分的应用场景。对于业务快速迭代，上线时间紧迫的系统来说，这次重构也是一个不小的挑战。

##### 优势

- 继承了原有1.0的特点，保留了其优势
- 解决了数据和服务隔离与扩容的问题
- 实现不同租户的差异化服务
- 添加了服务监控与检查


##### Docker 构建和发布

使用docker 构建可以完美的解决环境冲突的问题，也可以方便快速部署和扩容。

```java
FROM 10.10.10.160:8010/maxleap/vertx:3.2.1
MAINTAINER ben.ma <cma@maxleap.com>
#----------------------------Copy 项目目录到容器里------------------------------------------
RUN \
mkdir -p /opt/maxwon
#覆盖vert.x 相关配置
ADD lib/ $VERTX_HOME/lib/
ADD log4j2.xml $VERTX_HOME/conf/
ADD zookeeper.properties $VERTX_HOME/conf/
ADD config.json /opt/maxwon/
WORKDIR /opt/maxwon
ENTRYPOINT ["vertx", "run", "java-hk2:as.leap.ama.module.jersey.JerseyVerticle", "--conf", "config.json"]
```

 通过[spotify](https://github.com/spotify)  `docker-maven-plugin` 插件，根据事先定义在项目中的DockerFile可以轻松的把项目打包成可执行的docker Image并push到生产环境中。

```shell
$ mvn clean deploy -DpushImage -Pcn 
```




##### 好用的中间件

 **Hydra**: 海德拉 古希腊神话人物,是一种传说中有九个头的大蛇，为冥王看守门户。在这里Hydra 作为  [MaxWon](http://www.maxwon.cn/)的API网关，管理来自不同端的请求，根据请求的来源转发到相应的的[MaxWon](http://www.maxwon.cn/)的服务容器组中。同时它也会管理和监控容器状态以及对服务的动态扩容。

 ![hydra](http://o6wkmqikd.bkt.clouddn.com/hydra.png)



**Circe**:希腊神话里一个能制造幻觉的女巫,这里用来隐喻能够制造Mysql服务的代理的项目.通过它可以实现不同租户的数据隔离，过滤非法,有毒的sql语句，保证数据隐私和安全。

**Pandora**:访问MongoDB的基础组件，也是MaxLeap 核心组件之一,提供了同步和异步的两种接口。Pandora最为核心的功能是实现了资源限制和数据库访问的路由策略，这对数据库进行平滑的动态扩展及迁移提供了可靠的支持。感兴趣的可以参考同事写的[MONGO 集群设计](https://blog.maxleap.cn/archives/365)



#### 总结

脱离业务谈架构都是扯淡，利用技术手段提升工作效率是好事，别陷进去，产品最终拿出来说话的还是有没有解决用户的问题，而不是解决你自己的问题。对于MaxWon 这种快速迭代的系统，系统也会考虑更多的业务场景，体积也越来越庞大，遇到棘手的问题也会越来越多，做好优化的准备。

系统要尽量保持简单，技术架构的选型建议是寻找当前最短路径，然后进行不断优化迭代，想一口吃个大胖子不太可能。

代码不要写死。
