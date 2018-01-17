# Centos 下yum 安装Mariadb 

版本：Centos6  

Linux下安装MariaDB官方文档参见：<https://mariadb.com/kb/zh-cn/installing-mariadb-with-yum/>

1.创建MariaDB.repo文件

```shell
vi /etc/yum.repos.d/MariaDB.repo
```

插入以下内容：

```shell
# MariaDB 10.2 CentOS repository list - created 2018-01-11 11:41 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos6-x86
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

系统及版本选择：<https://downloads.mariadb.org/mariadb/repositories/#mirror=tuna>

2.运行安装命令安装MariaDB

```shell
 yum -y install MariaDB-server MariaDB-client
```

首先下载安装包，然后进行自动安装，安装成功之后启动MariaDB服务。

```shell
#centos 6
service mysql start #启动服务
chkconfig mysql on #设置开机启动
service mysql stop #停止服务
#centos 7
systemctl start mariadb #启动服务
systemctl enable mariadb #设置开机启动
systemctl restart mariadb #重新启动
systemctl stop mariadb.service #停止MariaDB
```

3.登录到数据库

　　用mysql -uroot命令登录到MariaDB，此时root账户的密码为空。

4.进行MariaDB的相关简单配置

　　使用mysql_secure_installation命令进行配置。　　![img](https://images2015.cnblogs.com/blog/311068/201608/311068-20160811164627418-997803593.png)

　　回车设置root账户的密码

　　![img](https://images2015.cnblogs.com/blog/311068/201608/311068-20160811164806840-1919620792.png)

　　输入两次密码

　　![img](https://images2015.cnblogs.com/blog/311068/201608/311068-20160811164828887-701337546.png)

　　其他配置：是否删除匿名用户、是否允许远程登录、 是否删除test数据库、是否重新加载权限表如果都选是，直接回车。

![img](https://images2015.cnblogs.com/blog/311068/201608/311068-20160811165009106-1129323863.png)

5.配置MariaDB的字符集

　　查看/etc/my.cnf文件内容，其中包含一句!includedir /etc/my.cnf.d 说明在该配置文件中引入/etc/my.cnf.d 目录下的配置文件。

　　1）使用vi server.cnf命令编辑server.cnf文件，在[mysqld]标签下添加

```shell
init_connect='SET collation_connection = utf8_unicode_ci' 
init_connect='SET NAMES utf8' 
character-set-server=utf8 
collation-server=utf8_unicode_ci 
skip-character-set-client-handshake
```

 

　　如果/etc/my.cnf.d 目录下无server.cnf文件，则直接在/etc/my.cnf文件的[mysqld]标签下添加以上内容。

　　2）用vi  client.cnf命令编辑/etc/my.cnf.d/client.cnf文件，在[client]标签下添加 

```shell
default-character-set=utf8
```

　　3）用vi  mysql-clients.cnf命令编辑/etc/my.cnf.d/mysql-clients.cnf文件，在[mysql]标签下添加 

```shell
default-character-set=utf8
```

配置完成后 systemctl restart mariadb 重启服务。

进入到数据库查看字符设置。

```
show variables like "%character%";
show variables like "%collation%";
```

![img](https://images2015.cnblogs.com/blog/311068/201608/311068-20160811170735918-1033876775.png)



6.添加用户，设置权限

　　创建用户命令：

```shell
create user username@localhost identified by 'password';
```

　　授予外网登陆权限：

```shell
grant all privileges on *.* to username@'%' identified by 'password';
```

 

![img](https://images2015.cnblogs.com/blog/311068/201608/311068-20160811170945699-729735477.png)

![img](https://images2015.cnblogs.com/blog/311068/201608/311068-20160811170954418-1990817010.png)

使用新创建的用户连接下数据库OK！