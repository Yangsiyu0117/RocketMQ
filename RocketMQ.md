# RocketMQ

[TOC]

## <u>文档标识</u>

| 文档名称 | RocketMQ |
| -------- | -------- |
| 版本号   | <V1.0.0> |

## <u>文档修订历史</u>

| 版本   | 日期       | 描述   | 文档所有者 |
| ------ | ---------- | ------ | ---------- |
| V1.0.0 | 2022.11.17 | create | 杨丝雨     |
|        |            |        |            |
|        |            |        |            |



## <u>服务器规划</u>

| IP地址 | 服务器配置 | OS   | remarks |
| ------ | ---------- | ---- | ------- |
|        |            |      |         |

## <u>路径规划</u>

| 路径 | 描述 | 文件名 | remarks |
| ---- | ---- | ------ | ------- |
|      |      |        |         |
|      |      |        |         |
|      |      |        |         |

## <u>端口规划</u>

| 端口  | 协议 | remrks                       |
| ----- | ---- | ---------------------------- |
| 9876  | all  | nameserver 监听端口          |
| 10911 | all  | broker 监听端口              |
| 10909 | all  | broker vip 监听端口          |
| 10912 | all  | broker HA 端口，用于主从同步 |

## <u>软件包</u>

| 安装包             | 版本   | 下载地址                                                     | remarks                            |
| ------------------ | ------ | ------------------------------------------------------------ | ---------------------------------- |
| JDK                | v1.18  | https://repo.huaweicloud.com/java/jdk/8u202-b08/jdk-8u202-linux-x64.tar.gz |                                    |
| Maven              | v3.6.3 | https://mirrors.bfsu.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz | 安装Rocketmq-dashboard需要依赖启动 |
| Rocketmq           | v4.9.4 | https://archive.apache.org/dist/rocketmq/4.9.4/rocketmq-all-4.9.4-bin-release.zip |                                    |
| Rocketmq-dashboard | v1.0.0 | https://github.com/apache/rocketmq-dashboard/archive/refs/tags/rocketmq-dashboard-1.0.0.tar.gz |                                    |
|                    |        |                                                              |                                    |

## <u>相关文档参考</u>

[Rocketmq官方文档]: https://rocketmq.apache.org/zh/docs/
[Rocketmq官方下载地址]: https://rocketmq.apache.org/zh/download
[Rocketmq文档]: https://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/
[Rocketmq文档]: https://www.jianshu.com/p/2ae8e81718d3
[Rocketmq文档]: 



### 一、概述

[消息中间件](https://so.csdn.net/so/search?q=消息中间件&spm=1001.2101.3001.7020)是分布式系统中重要的组件，主要解决***\*应用解耦，异步消息，流量削锋\****等问题，实现高性能，高可用，可伸缩和最终一致性[架构](https://so.csdn.net/so/search?q=架构&spm=1001.2101.3001.7020)。目前使用较多的消息队列有ActiveMQ，RabbitMQ，Kafka，RocketMQ等。从这篇文章开始，将会通过几篇文章系统学习下RocketMQ。

[RocketMQ](https://so.csdn.net/so/search?q=RocketMQ&spm=1001.2101.3001.7020)是一个队列模型的消息中间件，具有***\*高性能、高可靠、高实时、分布式\****特点。由阿里巴巴研发，借鉴参考了JMS规范的MQ实现，更参考了优秀的开源消息中间件[Kafka](https://so.csdn.net/so/search?q=Kafka&spm=1001.2101.3001.7020)，并且结合阿里实际业务需求，在天猫双十一的场景，实现业务削峰，分布式事务的优秀框架。

![img](https://img-blog.csdnimg.cn/img_convert/22851ce78e1c1e47803526a87b7e5544.png)

Producer 向一些队列轮流发送消息，队列集合称为 Topic，Consumer 如果做广播消费，则一个 consumer 实例消费这个 Topic 对应的所有队列，如果做集群消费，则多个 Consumer 实例平均消费这个 topic 对应的队列集合。

下面介绍一下RocketMQ的一些特性：

- 具有高性能、高可靠、高实时、分布式（ Producer、Consumer、队列都可以分布式 ）特点；
- 底层采用Netty NIO框架实现数据通信； 
- 内部使用更轻量级的NameServer进行网络路由，提高服务性能，并支持消息失败重试机制；
- 支持集群模式、消费者负载均衡、水平扩展能力，支持广播模式；
- 采用零拷贝原理，顺序写盘、支持亿级消息堆积能力；
- 提供丰富的消息机制，比如顺序消息、事务消息；

### 二、核心概念

> 在正式进入RocketMQ的学习之前，有必要熟悉一下RocketMQ核心概念，为后面学习RocketMQ打下基础。

(一)、消息模型（Message Model）

- RocketMQ主要由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker。Message Queue 用于存储消息的物理地址，每个Topic中的消息地址存储于多个 Message Queue 中。ConsumerGroup 由多个Consumer 实例构成。

(二)、消息生产者（Producer）

- Producer负责生产消息，一般由业务系统负责生产消息。一个消息生产者会把业务应用系统里产生的消息发送到broker服务器。RocketMQ提供多种发送方式，同步发送、异步发送、顺序发送、单向发送。同步和异步方式均需要Broker返回确认信息，单向发送不需要。

(三)、消息消费者（Consumer）

- Consumer负责消费消息，一般是后台系统负责异步消费。一个消息消费者会从Broker服务器拉取消息、并将其提供给应用程序。从用户应用的角度而言提供了两种消费形式：拉取式消费（Pull方式）、推动式消费(Push方式)。
- 拉取式消费：应用通常主动调用Consumer的拉消息方法从Broker服务器拉消息、主动权由应用控制。一旦获取了批量消息，应用就会启动消费过程。
- 推动式消费：Broker收到数据后会主动推送给消费端，该消费模式一般实时性较高。

(四)、消息（Message）

- Message其实就是要发送的信息内容，生产和消费数据的最小单位，每条消息必须属于一个主题。RocketMQ中每个消息拥有唯一的Message ID，且可以携带具有业务标识的Key。系统提供了通过Message ID和Key查询消息的功能。

(五)、主题（Topic）

- 表示一类消息的集合，每个主题包含若干条消息，每条消息只能属于一个主题，是RocketMQ进行消息订阅的基本单位。
- Topic可以理解为对消息的分类，或者打标签。Topic与Message之间的关系是一对多的关系，即一个Topic可以有多个Message，但是一个Message只能属于一个Topic。

(六)、标签（Tag）

- 为消息设置的标志，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。标签能够有效地保持代码的清晰度和连贯性，并优化RocketMQ提供的查询系统。消费者可以根据Tag实现对不同子主题的不同消费逻辑，实现更好的扩展性。
- 可以简单理解为：Topic主题是消息的大分类，而标签Tag则是大分类下的小分类。

(七)、代理服务器（Broker Server）

- 消息存储服务器，同时也是消息中转角色，负责存储消息、转发消息。代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。
- Broker分为两种角色：Master与Slave。主服务Master承担读写操作，从服务器Slave作为一个备份。所有Broker，包含Slave服务器每隔30s会向Nameserver发送心跳包，心跳包中会包含存在于Broker上所有的topic的路由信息。

(八)、名字服务（Name Server）

- NameServer是Topic的路由注册中心，为客户端根据Topic提供路由服务，从而引导客户端向Broker发送消息。生产者或消费者能够通过NameServer查找各主题相应的Broker的IP列表。
- 多个NameServer实例组成集群，但相互独立，它们之间没有信息交换。

(九)、生产者组（Producer Group）

- 同一类Producer的集合，这类Producer发送同一类消息且发送逻辑一致。如果发送的是事务消息且原始生产者在发送之后崩溃，则Broker服务器会联系同一生产者组的其他生产者实例以提交或回溯消费。

(十)、消费者组（Consumer Group）

- 同一类Consumer的集合，这类Consumer通常消费同一类消息且消费逻辑一致。消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。要注意的是，消费者组的消费者实例必须订阅完全相同的Topic。RocketMQ 支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。

  - 集群消费（Clustering）：集群消费模式下，相同Consumer Group的每个Consumer实例平均分摊消息。
    例如某个Topic有10条消息，然后存在两个Consumer Group，其中一个Consumer Group有3个实例(可能是3个进程或3台机器)，另外一个Consumer Group有2个实例(可能是2个进程或2台机器)，那么MQ会将负载均衡分给两个Consumer Group，一个5条。那么Consumer Group A会有5条消息，B也有5条。A有三个Consumer，再进行负载均衡，那么可能会是2，2，1这样进行分配消费。B有两个Consumer，那么可能是3，2这样进行分配消费。

  > RocketMQ天然支持消费者负载均衡，并且负载均衡不仅仅局限于消费者，还有消费者组的负载均衡。集群模式是非常普遍的模式，符合分布式架构的基本理念，即横向扩容，当前消费者如果无法快速及时处理消息时，可以通过增加消费者的个数横向扩容，快速提高消费能力，及时处理挤压的消息。

  - 广播消费（Broadcasting）：广播消费模式下，相同Consumer Group的每个Consumer实例都接收全量的消息。例如某个Topic有一条消息，其中一个Consumer Group有3个实例(可能是3个进程或3台机器)，另外一个Consumer Group有2个实例(可能是2个进程或2台机器)，广播消费的话，消息会被Consumer Group中的每个Consumer都消费一次，即每个实例都会消费这条消息。

![img](https://img-blog.csdnimg.cn/img_convert/c6e08bede36e0a3897c9280f6d056989.png)

> 生产者、消费者与Topic主题之间的关系是，一个Topic可以由多个生产者发送消息，反过来一个生产者也可以发送多个Topic消息。一个Topic可以由多个消费者消费，反过来一个消费者可以消费多个Topic消息。

为什么选择RocketMQ作为你们项目中的消息中间件？可以从下面几点进行回答：

1. RocketMQ集群无单点，可扩展，任意一点高可用，水平可扩展；
2. 支持海量消息堆积能力，消息堆积后，写入低延迟；
3. 支持上万个队列（与ActiveMQ进行对比）；
4. 支持消息失败重试机制；
5. 消息可查询；
6. 开源社区活跃；
7. 成熟度（经过双十一考验）；



### JDK安装与配置

1、下载JDK安装包

```shell
wget https://repo.huaweicloud.com/java/jdk/8u202-b08/jdk-8u202-linux-x64.tar.gz
```

2、建立JDK目录

```shell
mkdir /usr/local/java
cp jdk-8u202-linux-x64.tar.gz /usr/local/java/
cd /usr/local/java/
```

3、解压

```shell
tar xvf jdk-8u202-linux-x64.tar.gz
```

4、配置环境变量

```shell
vim /etc/profile
# 添加到末尾
export   JAVA_HOME=/usr/local/java/jdk1.8.0_202
export   CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export  PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
export   JRE_HOME=$JAVA_HOME/jre
```

5、验证

```shell
# 刷新环境变量
source /etc/profile
```

6、验证环境变量

```shell
# 输入
echo $JAVA_HOME
# 输出
/usr/local/java/jdk1.8.0_202
```

7、验证JDK

```shell
# 输入
java -version
# 输出
java version "1.8.0_202"
Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
```

### Maven安装与配置

1、下载Maven安装包

```shell
wget https://mirrors.bfsu.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```

2、解压

```shell
mkdir /usr/local/maven3.6
mv apache-maven-3.6.3-bin.tar.gz /usr/local/maven3.6/
cd /usr/local/maven3.6/
tar xvf apache-maven-3.6.3-bin.tar.gz
```

3、配置环境变量

```shell
vim /etc/profile
# 添加到末尾
export MAVEN_HOME=/usr/local/maven3.6/apache-maven-3.6.3
export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
source /etc/profile
```

4、验证

```shell
# 输入
mvn -v
# 输出
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /usr/local/maven3/apache-maven-3.6.3
Java version: 1.8.0_202, vendor: Oracle Corporation, runtime: /usr/local/java/jdk1.8.0_202/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-1062.el7.x86_64", arch: "amd64", family: "unix"
```

### Rocketmq部署安装

1、下载安装包

```shell
yum install -y unzip zip
wget https://archive.apache.org/dist/rocketmq/4.9.4/rocketmq-all-4.9.4-bin-release.zip
```

2、编译RocketMQ

```shell
unzip rocketmq-all-4.9.4-bin-release.zip
mv rocketmq-all-4.9.4-bin-release /usr/local/
mv /usr/local/rocketmq-all-4.9.4-bin-release  /usr/local/rocketmq
cd /usr/local/rocketmq
```

3、配置nameserver启动文件

```shell
# nameserver
cat << EOF > /usr/lib/systemd/system/rocketmqname.service
[Unit]
Description=rocketmq-nameserver
Documentation=http://mirror.bit.edu.cn/apache/rocketmq/
After=network.target
 
[Service]
Type=sample
User=root
Environment="JAVA_HOME=/usr/local/java/jdk1.8.0_202/"
ExecStart=/usr/local/rocketmq/bin/mqnamesrv
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
Restart=0
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

### RocketMQ 集群部署架构

![1](https://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/RocketMQ%20%E5%AE%9E%E6%88%98%E4%B8%8E%E8%BF%9B%E9%98%B6%EF%BC%88%E5%AE%8C%EF%BC%89/assets/20200726212547918.png)

### Rocketmq集群部署

#### 集群规划

| 名称     | IP           | 服务                                   | 端口                        | remarks                                                      |
| -------- | ------------ | -------------------------------------- | --------------------------- | ------------------------------------------------------------ |
| master-1 | 192.168.0.10 | Nameserver、broker-a、rocket-dashboard | 9876\8080\10911\10912\10909 | broker配置文件路:/usr/local/rocketmq/conf/2m-2s-sync/broker-a.properties |
| slave-1  | 192.168.1.11 | broker-a-s                             | 10911\10912\10909           | broker配置文件路:/usr/local/rocketmq/conf/2m-2s-sync/broker-a-s.properties |
| master-2 | 192.168.1.20 | Nameserver、broker-b                   | 9876\10911\10912\10909      | broker配置文件路:/usr/local/rocketmq/conf/2m-2s-sync/broker-b.properties |
| slave-2  | 192.168.1.21 | broker-b-s                             | 10911\10912\10909           | broker配置文件路:/usr/local/rocketmq/conf/2m-2s-sync/broker-b-s.properties |

#### 配置文件

master-1

```shell
#broker集群名称，同集群需要一致
brokerClusterName=RocketCluster
#Broker的名称，Master和Slave通过使用相同的Broker名称来表明相互关系，以说明某个Slave是哪个Master的Slave。
brokerName=broker-a
#一个Master Borker可以有多个Slave，0表示Master，大于0表示不同Slave的ID。
brokerId=0
#与fileReservedTime参数呼应，表明在几点做消息删除动作，默认值04表示凌晨4点。
deleteWhen=04
#文件过期时间,从文件最后一次的更新时间到现在为止，如果超过该时间，则是过期文件可被删除，单位h
fileReservedTime=48
#brokerRole有3种：SYNC_MASTER（同步）、ASYNC_MASTER（异步）、SLAVE。
brokerRole=SYNC_MASTER
#刷盘策略，分为SYNC_FLUSH（同步）和ASYNC_FLUSH（异步）两种。同步刷盘情况下，消息真正写入磁盘后再返回成功状态；异步刷盘情况下，消息写入page_cache后就返回成功状态。
flushDiskType=SYNC_FLUSH
#namesrc的ip地址
namesrvAddr=192.168.0.10:9876;192.168.0.20:9876
#Broker监听客户端请求的端口
listenPort=10911
#存储消息以及一些配置信息的根目录。
storePathRootDir=/usr/local/rocketmq/store

####调优
#打开线程锁，配合下面的属性使用
useReentrantLockWhenPutMessage=true
#发送线程池大小
sendMessageThreadPoolNums=4
#不自动创建topic
autoCreateTopicEnable=false
```

master-2

```shell
brokerClusterName=RocketCluster
brokerName=broker-b
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=SYNC_FLUSH
namesrvAddr=192.168.0.10:9876;192.168.0.20:9876
listenPort=10911
storePathRootDir=/usr/local/rocketmq/store
```

slave-1

```shell
brokerClusterName=RocketCluster
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=SYNC_FLUSH
namesrvAddr=192.168.0.10:9876;192.168.0.20:9876
listenPort=10911
storePathRootDir=/usr/local/rocketmq/store
#开启可从slave读消息
slaveReadEnable=true
```

slave-2

```shell
brokerClusterName=RocketCluster
brokerName=broker-b
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=SYNC_FLUSH
namesrvAddr=192.168.0.10:9876;192.168.0.20:9876
listenPort=10911
storePathRootDir=/usr/local/rocketmq/store
#开启可从slave读消息
slaveReadEnable=true
```

#### 启动文件

```shell
cat << EOF > /usr/lib/systemd/system/rocketmqbroker.service
[Unit]
Description=rocketmq-broker
Documentation=http://mirror.bit.edu.cn/apache/rocketmq/
After=network.target
 
[Service]
Type=sample
User=root
Environment="JAVA_HOME=/usr/local/java/jdk1.8.0_202/"
# 根据配置文件的地址修改
ExecStart=/usr/local/rocketmq/bin/mqbroker -c /usr/local/rocketmq/conf/2m-2s-sync/broker-a.properties
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
Restart=0
LimitNOFILE=65536
 
[Install]
WantedBy=multi-user.target
EOF
```

#### 安装rocketmq dashborad

1、下载安装包

```shell
wget https://github.com/apache/rocketmq-dashboard/archive/refs/tags/rocketmq-dashboard-1.0.0.tar.gz
```

2、解压并放到指定目录

```shell
tar zxvf rocketmq-dashboard-rocketmq-dashboard-1.0.0.tar.gz -C /usr/local/
mv /usr/local/rocketmq-dashboard-rocketmq-dashboard-1.0.0 /usr/local/rocketmq-dashboard
```

3、配置启动文件

```shell
cat> /usr/lib/systemd/system/rocketmqdashboard.service << EOF
[Unit]
After=network.target

[Service]
Type=simple
User=root
Environment="MAVEN_HOME=/usr/local/maven3.6/apache-maven-3.6.3/"
Environment="JAVA_HOME=/usr/local/java/jdk1.8.0_202/"
WorkingDirectory=/usr/local/rocketmq-dashboard
ExecStart=/usr/local/maven3.6/apache-maven-3.6.3/bin/mvn  spring-boot:run
ExecStop=/bin/kill -3 \$MAINPID

[Install]
WantedBy=multi-user.target
EOF
```

#### 创建存储根目录

```shell
mkdir -p /usr/local/rocketmq/store
```

#### 启动程序

```shell
# 在mster-1和master-2上启动nameserver\broker
systemctl enable  --now rocketmqname.service
systemctl enable  --now rocketmqbroker.service
# 在slave-1和slave-2上启动broker
systemctl enable  --now rocketmqbroker.service
# 在mster-1上启动rocketmq-dashboard
systemctl enable  --now rocketmqdashboard.service
```

