#### 一、安装saltstack

官网安装

```http
http://repo.saltstack.com/#rhel   # 注意选择系统版本及python版本
```

saltstack的模块：  https://www.unixhot.com/docs/saltstack/ref/modules/all/

1. Run the following commands to install the SaltStack repository and key:

   ```bash
   sudo yum install https://repo.saltstack.com/yum/redhat/salt-repo-latest.el7.noarch.rpm 
   ```

2. Run `sudo yum clean expire-cache`

3. Install the salt-minion, salt-master, or other Salt components:

   - `sudo yum install salt-master`  # 安装master
   - `sudo yum install salt-minion`  #agent
   - `sudo yum install salt-ssh`
   - `sudo yum install salt-syndic`
   - `sudo yum install salt-cloud`
   - `sudo yum install salt-api`

4. (*Upgrade only*) Restart all upgraded services, for example:

```bash
sudo systemctl restart salt-master # 启动master
sudo systemctl restart salt-minion # 启动agent
```

[saltstack部署配置](https://www.cnblogs.com/renyongbin/p/7994044.html)

共计使用三台虚拟机进行部署实验，系统环境：centos7.3 

在master上进行部署配置：

配置主机名

```
[root@localhost ~]# hostname salt-master

[root@localhost ~]# cat /etc/sysconfig/network

# Created by anaconda

HOSTNAME=salt-master
```

配置hosts

```
cat /etc/hosts

127.0.0.1  localhost localhost.localdomain localhost4 localhost4.localdomain4

::1     localhost localhost.localdomain localhost6 localhost6.localdomain6

salt-master   192.168.143.19

salt-minion-01 192.168.143.28

salt-minion-02 192.168.143.35
```

关闭防火墙

```
[root@localhost ~]# systemctl disable firewalld.service

Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.

Removed symlink /etc/systemd/system/basic.target.wants/firewalld.service.

[root@localhost ~]# systemctl stop firewalld.service
```

 

修改selinux为Permissive模式

```
[root@localhost ~]# setenforce 0

[root@localhost ~]# getenforce

Permissiv
```

安装配置阿里云yum源

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

安装epel-release和salt-master工具包

```
[root@localhost ~]# yum install epel-release –y

[root@localhost ~]# yum install salt-master –y
```

 

配置saltstack开机自启动服务

`[root@localhost ~]# systemctl enable salt-master.service`  

至此，Master部署配置完成！

在第一个minion端部署配置：

配置主机名

```shell
[root@localhost ~]# hostname salt-minion-01

[root@localhost ~]# cat /etc/sysconfig/network

# Created by anaconda

HOSTNAME=salt-minion-01
```

配置hosts

```
cat /etc/hosts

127.0.0.1  localhost localhost.localdomain localhost4 localhost4.localdomain4

::1     localhost localhost.localdomain localhost6 localhost6.localdomain6

salt-master   192.168.143.19

salt-minion-01 192.168.143.28

salt-minion-02 192.168.143.35
```

关闭防火墙

```
[root@localhost ~]# systemctl disable firewalld.service

Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.

Removed symlink /etc/systemd/system/basic.target.wants/firewalld.service.

[root@localhost ~]# systemctl stop firewalld.service
```

 

修改selinux为Permissive模式

```
[root@localhost ~]# setenforce 0

[root@localhost ~]# getenforce

Permissive
```

 

安装配置阿里云yum源

```
[root@localhost ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

安装epel-release工具包和salt-minion客户端

```
[root@localhost ~]# yum install epel-release -y
[root@localhost ~]# yum install salt-minion -y

```



在minion端配置master的ip地址和ID



```shell
##### Primary configuration settings #####

##########################################

# This configuration file is used to manage the behavior of the Salt Minion.

# With the exception of the location of the Salt Master Server, values that are

# commented out but have an empty line after the comment are defaults that need

# not be set in the config. If there is no blank line after the comment, the

# value is presented as an example and is not the default.

 

# Per default the minion will automatically include all config files

# from minion.d/*.conf (minion.d is a directory in the same directory

# as the main minion config file).

#default_include: minion.d/*.conf

 

# Set the location of the salt master server. If the master server cannot be

# resolved, then the minion will fail to start.

#master: salt

maser: 192.168.143.19

 

# If multiple masters are specified in the 'master' setting, the default behavior

# is to always try to connect to them in the order they are listed. If random_master is

# set to True, the order will be randomized instead. This can be helpful in distributing

# the load of many minions executing salt-call requests, for example, from a cron job.

# If only one master is listed, this setting is ignored and a warning will be logged.

# NOTE: If master_type is set to failover, use master_shuffle instead.

#random_master: False

 

# Use if master_type is set to failover.

#master_shuffle: False

 

# Minions can connect to multiple masters simultaneously (all masters

# are "hot"), or can be configured to failover if a master becomes

# unavailable.  Multiple hot masters are configured by setting this

# value to "str".  Failover masters can be requested by setting

# to "failover".  MAKE SURE TO SET master_alive_interval if you are

# using failover.

# master_type: str

 

# Poll interval in seconds for checking if the master is still there.  Only

# respected if master_type above is "failover". To disable the interval entirely,

# set the value to -1. (This may be necessary on machines which have high numbers

# of TCP connections, such as load balancers.)

# master_alive_interval: 30

 

# Set whether the minion should connect to the master via IPv6:

#ipv6: False

 

# Set the number of seconds to wait before attempting to resolve

# the master hostname if name resolution fails. Defaults to 30 seconds.

# Set to zero if the minion should shutdown and not retry.

# retry_dns: 30

 

# Set the port used by the master reply and authentication server.

#master_port: 4506

 

# The user to run salt.

#user: root

 

# Setting sudo_user will cause salt to run all execution modules under an sudo

# to the user given in sudo_user.  The user under which the salt minion process

# itself runs will still be that provided in the user config above, but all

# execution modules run by the minion will be rerouted through sudo.

#sudo_user: saltdev

 

# Specify the location of the daemon process ID file.

#pidfile: /var/run/salt-minion.pid

 

# The root directory prepended to these options: pki_dir, cachedir, log_file,

# sock_dir, pidfile.

#root_dir: /

 

# The directory to store the pki information in

#pki_dir: /etc/salt/pki/minion

 

# Explicitly declare the id for this minion to use, if left commented the id

# will be the hostname as returned by the python call: socket.getfqdn()

# Since salt uses detached ids it is possible to run multiple minions on the

# same machine but with different ids, this can be useful for salt compute

# clusters.

id: salt-minion-01
```

 配置开机minion开启自启动服务

```
[root@localhost ~]# systemctl enable salt-minion.service

Created symlink from /etc/systemd/system/multi-user.target.wants/salt-minion.service to usr/lib/systemd/system/salt-minion.service.
```

至此，第一个minion配置部署完毕，第二个minion部署配置同理第一个minion部署配置！



启动salt-master服务: `systemctl start salt-master.service`

启动salt-minion服务：`systemctl start salt-minion.service`

测试master和minion之间的通信是否正常

```
salt "*" test.ping
```

查看 minion 列表:

```shell
# salt-key –L
Accepted Keys:
Denied Keys:
Unaccepted Keys:
ops-172.16.0.14
Rejected Keys:
```

为客户端添加Key

```shell
# salt-key -a ops-172.16.0.14
The following keys are going to be accepted:
Unaccepted Keys:
ops-172.16.0.14
Proceed? [n/Y] Y
Key for minion ops-172.16.0.14 accepted.
```



认证所有 key，当然你也可以通过 
salt-key -a saltstack-minion 指定某台 minion 进行认证 key

说明：-a ：accept ，-A：accept-all，-d：delete，-D：delete-all。可以使用 salt-key 命令查看到已经签名的客户端。此时我们在客户端的 /etc/salt/pki/minion 目录下面会多出一个minion_master.pub 文件。