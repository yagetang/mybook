# Centos下elasticsearch集群搭建

**操作系统环境:** CentOS release 6.9 (Final)

**elasticsearch ：** elasticsearch-2.4.0

**集群搭建方式：** 一个集群3个节点

elasticsearch-node01 192.168.1.54

elasticsearch-node01 192.168.1.55

elasticsearch-node01 192.168.1.56

**必备环境:  **java运行环境

1. JDK安装

```shell
#所有节点安装
[root@elasticsearch-node01 ~]# yum install java-1.8.0
```

2. elasticsearch 安装RPM

```shell
#所有节点都安装
#https://www.elastic.co/downloads/past-releases/elasticsearch-2-4-0
[root@elasticsearch-node01 ~]# wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/rpm/elasticsearch/2.4.0/elasticsearch-2.4.0.rpm
[root@elasticsearch-node01 ~]# rpm -ivh elasticsearch-2.4.0.rpm
[root@elasticsearch-node01 ~]# chkconfig --add elasticsearch
#配置文件修改
[root@elasticsearch-node01 ~]# cat /etc/elasticsearch/elasticsearch.yml |grep -v "^#"
cluster.name: ipr-elasticsearch #群集名
node.name: node-01 #节点名
network.host: 0.0.0.0 #可以授权访问的地址，这里设置都可以访问
discovery.zen.ping.unicast.hosts: ["192.168.1.54:9300","192.168.1.55:9300"] #设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点
[root@elasticsearch-node02 work]# cat /etc/elasticsearch/elasticsearch.yml |grep -v "^#"
cluster.name: ipr-elasticsearch
node.name: node-02
network.host: 0.0.0.0
discovery.zen.ping.unicast.hosts: ["192.168.1.54:9300","192.168.1.55:9300"]
[root@elasticsearch-node03 work]# cat /etc/elasticsearch/elasticsearch.yml |grep -v "^#"
cluster.name: ipr-elasticsearch
node.name: node-03
network.host: 0.0.0.0
discovery.zen.ping.unicast.hosts: ["192.168.1.54:9300","192.168.1.55:9300"]
[root@elasticsearch-node03 work]# service elasticsearch start #所有节点启动
# 创建牵引
[root@elasticsearch-node01 work]# curl -XPUT 'http://localhost:9200/twitter/tweet/1' -d '{
    "user" : "kimchy",
    "post_date" : "2017-09-15T14:12:12",
    "message" : "trying out Elasticsearch"
}'
# 查看
[root@elasticsearch-node01 work]# curl -XGET 'http://192.168.1.5[4-6]:9200/twitter/tweet/1'

[1/3]: http://192.168.1.54:9200/twitter/tweet/1 --> <stdout>
--_curl_--http://192.168.1.54:9200/twitter/tweet/1
{"_index":"twitter","_type":"tweet","_id":"1","_version":1,"found":true,"_source":{
    "user" : "kimchy",
    "post_date" : "2017-09-15T14:12:12",
    "message" : "trying out Elasticsearch"

[2/3]: http://192.168.1.55:9200/twitter/tweet/1 --> <stdout>
}}--_curl_--http://192.168.1.55:9200/twitter/tweet/1
{"_index":"twitter","_type":"tweet","_id":"1","_version":1,"found":true,"_source":{
    "user" : "kimchy",
    "post_date" : "2017-09-15T14:12:12",
    "message" : "trying out Elasticsearch"

[3/3]: http://192.168.1.56:9200/twitter/tweet/1 --> <stdout>
}}--_curl_--http://192.168.1.56:9200/twitter/tweet/1
{"_index":"twitter","_type":"tweet","_id":"1","_version":1,"found":true,"_source":{
    "user" : "kimchy",
    "post_date" : "2017-09-15T14:12:12",
    "message" : "trying out Elasticsearch"
# 集群验证成功
-----------------------------------------------------------------------------------------其它方法：yum安装 

1.下载并安装GPG key
[root@linux-node1 ~]# rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch

2.添加yum仓库
[root@linux-node1 ~]# cat /etc/yum.repos.d/elasticsearch.repo
[elasticsearch-2.x]
name=Elasticsearch repository for 2.x packages
baseurl=http://packages.elastic.co/elasticsearch/2.x/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1

3.安装elasticsearch
[root@hadoop-node1 ~]# yum install -y elasticsearch
-----------------------------------------------------------------------------------------
```

3. 安装head插件

```shell
#使用的节点安装
[root@elasticsearch-node01 work]# /usr/share/elasticsearch/bin/plugin install mobz/elasticsearch-head
-> Installing mobz/elasticsearch-head...
Trying https://github.com/mobz/elasticsearch-head/archive/master.zip ...
Downloading ...................................................................................................DONE
Verifying https://github.com/mobz/elasticsearch-head/archive/master.zip checksums if available ...
NOTE: Unable to verify checksum for downloaded plugin (unable to find .sha1 or .md5 file to verify)
Installed head into /usr/share/elasticsearch/plugins/head
```

4. 访问：http://192.168.1.54:9200/_plugin/head/ 

   成功