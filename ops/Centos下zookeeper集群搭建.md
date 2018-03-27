

# Centos下zookeeper集群搭建

最近搭建服务的过程了解到zookeeper分布式应用程序协调服务被其它应用使用，如，solr,kafka,codis缓存等。为保证zookeeper的可用性是很重要的。

#### What is ZooKeeper?

ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them ,which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.

Learn more about ZooKeeper on the [ZooKeeper Wiki](https://cwiki.apache.org/confluence/display/ZOOKEEPER/Index).

#### 安装与配置

1. 环境

操作系统版本：CentOS release 6.9 (Final) 

服务器信息：

> ``` shell
> 192.168.1.71 zookeeper-node001 
> 192.168.1.72 zookeeper-node002
> 192.168.1.74 zookeeper-node003
> ```

2. zookeeper安装

下载与安装

> ``` shell
> # wget http://www.eu.apache.org/dist/zookeeper/zookeeper-3.3.6/zookeeper-3.3.6.tar.gz
> # tar -zxvf zookeeper-3.3.6.tar.gz -C /usr/local/
> # mv /usr/local/zookeeper-3.3.6 /usr/local/zookeeper
>
> # cp /usr/local/zookeeper/conf/zoo_sample.cfg  /usr/local/zookeeper/conf/zoo.cfg
> # vim /usr/local/zookeeper/conf/zoo.cfg #添加修改以下内容
> tickTime=2000
> initLimit=10
> syncLimit=5
> dataDir=/var/lib/zookeeper
> dataLogDir=/var/log/zookeeper
> clientPort=2181
> maxClientCnxns=300
> autopurge.snapRetainCount=20
> autopurge.purgeInterval=72
> dataLogDir=/var/log/zookeeper
> server.1001=zookeeper-node001:2888:3888
> server.1002=zookeeper-node002:2888:3888
> server.1003=zookeeper-node003:2888:3888
>
> # 创建数据目录和日志目录
> # mkdir -p /var/lib/zookeeper  
> # mkdir -p /var/log/zookeeper
> ```

如不创建启动时会报如下异常：

> ``` shell
> # /usr/local/zookeeper/bin/zkServer.sh status
> JMX enabled by default
> Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
> Error contacting service. It is probably not running.
> ```

创建myid文件， id 与 zoo.cfg 中的序号对应

`echo 1001 > /var/data/zookeeper/myid`

注意：

> - 注意，如果是`zookeeper-node002`和`zookeeper-node003`中进行相应的修改
> - `zookeeper-node002`上应改为：`echo 1002 > /var/lib/zookeeper/myid`
> - `zookeeper-node003`上应改为：`echo 1003 > /var/lib/zookeeper/myid`

配置hosts文件：编辑`/etc/hosts`，加入如下内容：

> ``` shell
> 192.168.1.71 zookeeper-node001 
> 192.168.1.72 zookeeper-node002
> 192.168.1.74 zookeeper-node003
> ```

启动

> ``` shell
> # /usr/local/zookeeper/bin/zkServer.sh start
> JMX enabled by default
> Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
> Starting zookeeper ... STARTED
> ```

查看

> ```shell
> # /usr/local/zookeeper/bin/zkServer.sh status
> JMX enabled by default
> Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
> Mode: follower
> ```