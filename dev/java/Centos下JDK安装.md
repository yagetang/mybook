## Centos下JDK安装

操作系统：`CentOS Linux release 7.4.1708 (Core) `

JDk版本：JDK 1.8 

JDK下载地址:[https://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html](https://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html)

下载对应版本，这里采用RPM包与二进制包两个方法分别安装



## 方法一

```
# yum install jdk-8u202-linux-x64.rpm -y
```
#### 查看版本
```
# java -version
java version "1.8.0_202"
Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
```
#### 检查是哪个RPM包提供了java文件

```
# rpm -q --whatprovides java
jdk1.8-1.8.0_202-fcs.x86_64
```

## 方法二

#### 二进制包安装
```
# tar -xzvf jdk-8u40-linux-x64.tar.gz
# mv jdk1.8.0_40 /usr/local/jdk
```

#### 配置环境变量
```
# vim /etc/profile
export JAVA_HOME=/usr/local/jdk
export PATH=$JAVA_HOME/bin:$PATH
  
# source /etc/profile
```
#### 查看版本
```
# java –version
java version "1.8.0_40"
Java(TM) SE Runtime Environment (build 1.8.0_40-b26)
Java HotSpot(TM) 64-Bit Server VM (build 25.40-b25, mixed mode)
```
参考资料：[https://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html](https://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html)
