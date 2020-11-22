1.Http2
异步连接多
优化Web应用交付层（HTTP 2.0）
优化Web应用交付层（HTTP 2.0） [2]
路复用；
头部压缩；
请求/响应管线化；
保持与HTTP 1.1语义的向后兼容性也是该版本的一个关键目标。SPDY是一种HTTP兼容协议，由Google发起，Chrome、Opera、Firefox以及Amazon Silk等浏览器均已提供支持。HTTP实现的瓶颈之一是其并发要依赖于多重连接。HTTP管线化技术可以缓解这个问题，但也只能做到部分多路复用。此外，已经证实，由于存在中间干扰，浏览器无法采用管线化技术。SPDY在单个连接之上增加了一个帧层，用以多路复用多个并发流。帧层针对HTTP类的请求响应流进行了优化，因此运行在HTTP之上的应用，对应用开发者而言只要很小的修改甚至无需修改就可以运行在SPDY之上。SPDY对当前的HTTP协议有4个改进：
多路复用请求；
对请求划分优先级；
压缩HTTP头；
服务器推送流（即Server Push技术）；
SPDY试图保留HTTP的现有语义，所以cookies、ETags等特性都是可用的。

2.CAP
CAP原则又称CAP定理，指的是在一个分布式系统中，一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）。CAP 原则指的是，这三个要素最多只能同时实现两点，不可能三者兼顾。
CAP原则又称CAP定理，指的是在一个分布式系统中， Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可得兼。
一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）
可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
分区容忍性（P）：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。
CAP原则的精髓就是要么AP，要么CP，要么AC，但是不存在CAP。如果在某个分布式系统中数据无副本， 那么系统必然满足强一致性条件， 因为只有独一数据，不会出现数据不一致的情况，此时C和P两要素具备，但是如果系统发生了网络分区状况或者宕机，必然导致某些数据不可以访问，此时可用性条件就不能被满足，即在此情况下获得了CP系统，但是CAP不可同时满足 [1]  。
因此在进行分布式架构设计时，必须做出取舍。当前一般是通过分布式缓存中各节点的最终一致性来提高系统的性能，通过使用多节点之间的数据异步复制技术来实现集群化的数据一致性。通常使用类似 memcached 之类的 NOSQL 作为实现手段。虽然 memcached 也可以是分布式集群环境的，但是对于一份数据来说，它总是存储在某一台 memcached 服务器上。如果发生网络故障或是服务器死机，则存储在这台服务器上的所有数据都将不可访问。由于数据是存储在内存中的，重启服务器，将导致数据全部丢失。当然也可以自己实现一套机制，用来在分布式 memcached 之间进行数据的同步和持久化，但是实现难度是非常大
的.

3.gRPC
A high-performance, open-source universal RPC framework
所谓RPC(remote procedure call 远程过程调用)框架实际是提供了一套机制，使得应用程序之间可以进行通信，而且也遵从server/client模型。使用的时候客户端调用server端提供的接口就像是调用本地的函数一样。如下图所示就是一个典型的RPC结构图。
gRPC vs. Restful API
gRPC和restful API都提供了一套通信机制，用于server/client模型通信，而且它们都使用http作为底层的传输协议(严格地说, gRPC使用的http2.0，而restful api则不一定)。不过gRPC还是有些特有的优势，如下：

gRPC可以通过protobuf来定义接口，从而可以有更加严格的接口约束条件。关于protobuf可以参见笔者之前的小文Google Protobuf简明教程
另外，通过protobuf可以将数据序列化为二进制编码，这会大幅减少需要传输的数据量，从而大幅提高性能。
gRPC可以方便地支持流式通信(理论上通过http2.0就可以使用streaming模式, 但是通常web服务的restful api似乎很少这么用，通常的流式数据应用如视频流，一般都会使用专门的协议如HLS，RTMP等，这些就不是我们通常web服务了，而是有专门的服务器应用。）

使用场景
需要对接口进行严格约束的情况，比如我们提供了一个公共的服务，很多人，甚至公司外部的人也可以访问这个服务，这时对于接口我们希望有更加严格的约束，我们不希望客户端给我们传递任意的数据，尤其是考虑到安全性的因素，我们通常需要对接口进行更加严格的约束。这时gRPC就可以通过protobuf来提供严格的接口约束。
对于性能有更高的要求时。有时我们的服务需要传递大量的数据，而又希望不影响我们的性能，这个时候也可以考虑gRPC服务，因为通过protobuf我们可以将数据压缩编码转化为二进制格式，通常传递的数据量要小得多，而且通过http2我们可以实现异步的请求，从而大大提高了通信效率。
但是，通常我们不会去单独使用gRPC，而是将gRPC作为一个部件进行使用，这是因为在生产环境，我们面对大并发的情况下，需要使用分布式系统来去处理，而gRPC并没有提供分布式系统相关的一些必要组件。而且，真正的线上服务还需要提供包括负载均衡，限流熔断，监控报警，服务注册和发现等等必要的组件。不过，这就不属于本篇文章讨论的主题了，我们还是先继续看下如何使用gRPC。

4.go-kit
https://gokit.io/

5.Kratos
Kratos
Kratos是bilibili开源的一套Go微服务框架，包含大量微服务相关框架及工具。
https://gitee.com/mirrors/Kratos

6.Kafka
ActiveMQ 演进
Kafka 是一种高吞吐量的分布式发布订阅消息系统，有如下特性：
通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。
高吞吐量：即使是非常普通的硬件Kafka也可以支持每秒数百万的消息。
支持通过Kafka服务器和消费机集群来分区消息。
支持Hadoop并行数据加载。
Kafka通过官网发布了最新版本2.5.0
http://kafka.apache.org/

7.Redis
Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由Pivotal赞助
https://redis.io/

8.K8s
k8s全称kubernetes，这个名字大家应该都不陌生，k8s是为容器服务而生的一个可移植容器的编排管理工具，越来越多的公司正在拥抱k8s，并且当前k8s已经主导了云业务流程，推动了微服务架构等热门技术的普及和落地.
具体来说，主要包括以下几点：

服务发现与调度
负载均衡
服务自愈
服务弹性扩容
横向扩容
存储卷挂载
https://kubernetes.io/

9.BFF:Backends for Frontends
https://segmentfault.com/a/1190000020623009
https://www.jianshu.com/p/9cca72f9e93c

10.负载均衡-Golang
https://segmentfault.com/a/1190000014924862

11.Service Mesh
https://zhuanlan.zhihu.com/p/61901608'

12.Zookeeper
https://zhuanlan.zhihu.com/p/69114539
https://github.com/apache/zookeeper

13.Nacos
Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

14.Eureka发现服务
https://github.com/Netflix/eureka/

15.Nginx
http://nginx.org/
Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。
其将源代码以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名。2011年6月1日，nginx 1.0.4发布。
Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。

16.YAPI
https://hellosean1025.github.io/yapi/
https://github.com/YMFE/yapi

17.Canal
canal [kə'næl]，译意为水道/管道/沟渠，主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费

早期阿里巴巴因为杭州和美国双机房部署，存在跨机房同步的业务需求，实现方式主要是基于业务 trigger 获取增量变更。从 2010 年开始，业务逐步尝试数据库日志解析获取增量变更进行同步，由此衍生出了大量的数据库增量订阅和消费业务。

基于日志增量订阅和消费的业务包括

数据库镜像
数据库实时备份
索引构建和实时维护(拆分异构索引、倒排索引等)
业务 cache 刷新
带业务逻辑的增量数据处理
当前的 canal 支持源端 MySQL 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x
https://github.com/alibaba/canal

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。
https://nacos.io/zh-cn/docs/quick-start.html

18.Binlog
https://www.jianshu.com/p/ea666baf0d82

19.protocol buffer
protocolbuffer(以下简称PB)是google 的一种数据交换的格式，它独立于语言，独立于平台。google 提供了多种语言的实现：java、c#、c++、go 和 python，每一种实现都包含了相应语言的编译器以及库文件。由于它是一种二进制的格式，比使用 xml 进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域。
https://developers.google.cn/protocol-buffers/
