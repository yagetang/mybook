## Maven安装

操作系统：`CentOS Linux release 7.6.1810 (Core) `

Nexus版本：`maven-3.6.1`

下载地址：[http://maven.apache.org](http://maven.apache.org/download.cgi)

#### 环境准备
[JDK安装](https://www.jianshu.com/p/239edba08493)

#### 安装
下载后二进制包进行解包后包括两个文件夹如下：
```
# cd /usr/local/src/
# webt http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz
# tar -zxvf apache-maven-3.6.1-bin.tar.gz
# mv apache-maven-3.6.1 /usr/local/maven3
```
#### 配置环境变量
```
# vim /etc/profile
export M2_HOME=/usr/local/maven3
export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
# source /etc/profile
```

#### 版本验证
```
# mvn -v
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /usr/local/maven3
Java version: 1.8.0_202, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_202-amd64/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.1.3.el7.x86_64", arch: "amd64", family: "unix"
```

#### 配置镜象
在maven安装目录的conf目录下的settings.xml文件：
```
# vim /usr/local/maven3/conf/settings.xml
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
 </mirrors>