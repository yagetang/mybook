# CentOS下OpenLDAP安装

操作系统版本：CentOS Linux release 7.4.1708 (Core)

OpenLDAP版本： 2.4.44 

LDAP 全称轻量级目录访问协议（英文：Lightweight Directory Access Protocol），是一个运行在 TCP/IP 上的目录访问协议。目录是一个特殊的数据库，它的数据经常被查询，但是不经常更新。其专门针对读取、浏览和搜索操作进行了特定的优化。目录一般用来包含描述性的，基于属性的信息并支持精细复杂的过滤能力。比如 DNS 协议便是一种最被广泛使用的目录服务。

LDAP 中的信息按照目录信息树结构组织，树中的一个节点称之为条目（Entry），条目包含了该节点的属性及属性值。条目都可以通过识别名 dn 来全局的唯一确定，可以类比于关系型数据库中的主键。比如 dn 为 `uid=ada,ou=People,dc=xinhua,dc=org` 的条目表示在组织中一个名字叫做 Ada Catherine 的员工，其中 `uid=ada` 也被称作相对区别名 rdn。

一个条目的属性通过 LDAP 元数据模型（Scheme）中的对象类（objectClass）所定义，下面的表格列举了对象类 inetOrgPerson（Internet Organizational Person）中的一些必填属性和可选属性。

| 属性名        | 是否必填 | 描述                                    |
| ------------- | -------- | --------------------------------------- |
| `cn`          | 是       | 该条目被人所熟知的通用名（Common Name） |
| `sn`          | 是       | 该条目的姓氏                            |
| `o`           | 否       | 该条目所属的组织名（Organization Name） |
| `mobile`      | 否       | 该条目的手机号码                        |
| `description` | 否       | 该条目的描述信息                        |



## 1.首先安装OpenLDAP相关组件：

```shell
[root@ldap-service ~]# yum install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel migrationtools -y
```

启动slapd并查看状态

``` shell
[root@ldap-service ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
[root@ldap-service ~]# chown ldap:ldap -R /var/lib/ldap
[root@ldap-service ~]# chmod 700 -R /var/lib/ldap
[root@ldap-service ~]# ll /var/lib/ldap/
总用量 4
-rwx------ 1 ldap ldap 845 5月  30 14:01 DB_CONFIG

[root@ldap-service ~]# systemctl enable slapd.service 
Created symlink from /etc/systemd/system/multi-user.target.wants/slapd.service to /usr/lib/systemd/system/slapd.service.
[root@ldap-service ~]# systemctl start slapd.service 
[root@ldap-service ~]# systemctl status slapd.service #查看状态
[root@ldap-service ~]# netstat -nltp |grep slapd
tcp        0      0 0.0.0.0:389             0.0.0.0:*          LISTEN      16076/slapd         
tcp6       0      0 :::389                  :::*               LISTEN      16076/slapd 
```



## 2.设置管理用户和密码

Openldap顶级域（以 `dc=innosail,dc=org` 为例）

最终管理用户为：cn=root,dc=innosail,dc=org

密码为slapasswd自己输入的密码

**olcSuffix** – 数据库后缀，通常它是你服务的域名。

**olcRootDN** – Root Distinguished Name (DN)。root用户的DN。

**olcRootPW** – root 密码。

``` shell
[root@ldap-service ~]# slappasswd
New password: 
Re-enter new password: 
{SSHA}生成的加密串

##添加/root/chrootpw.ldif文件
[root@ldap-service ~]# vim /root/chrootpw.ldif

#specify the password generated above for “olcRootPW” sectio
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}生成的加密串

##导入openldap属性
[root@ldap-service ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
[root@ldap-service ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
[root@ldap-service ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

[root@ldap-service ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /root/chrootpw.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={0}config,cn=config"

##添加/root/chdomain.ldif文件
[root@ldap-service ~]# vim /root/chdomain.ldif

# replace to your own domain name for "dc=***,dc=***" section
# specify the password generated above for "olcRootPW" section
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=root,dc=innosail,dc=org" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=innosail,dc=org

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=root,dc=innosail,dc=org

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}生成的加密串

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=root,dc=innosail,dc=org" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=root,dc=innosail,dc=org" write by * read

[root@ldap-service ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={1}monitor,cn=config"
modifying entry "olcDatabase={2}hdb,cn=config"
modifying entry "olcDatabase={2}hdb,cn=config"
modifying entry "olcDatabase={2}hdb,cn=config"
modifying entry "olcDatabase={2}hdb,cn=config"

##创建一个 root 的组织角色（该角色内的用户具有管理整个 LDAP 的权限）和 People 和 Group 两个组织单元
[root@ldap-service ~]# vim /root/basedomain.ldif

# replace to your own domain name for "dc=***,dc=***" section
dn: dc=innosail,dc=org
objectClass: top
objectClass: dcObject
objectclass: organization
o: Server Org
dc: Innosail

dn: cn=root,dc=innosail,dc=org
objectClass: organizationalRole
cn: root
description: LDAP Manager

dn: ou=People,dc=innosail,dc=org
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=innosail,dc=org
objectClass: organizationalUnit
ou: Group

[root@ldap-service ~]# ldapadd -x -D cn=root,dc=innosail,dc=org -W -f /root/basedomain.ldif
Enter LDAP Password: 
adding new entry "dc=innosail,dc=org"
adding new entry "cn=root,dc=innosail,dc=org"
adding new entry "ou=People,dc=innosail,dc=org"
adding new entry "ou=Group,dc=innosail,dc=org"

```



## 3. 新增用户

``` shell
[root@ldap-service ~]# vim /root/addldap_user.ldif

dn: uid=user01,ou=People,dc=innosail,dc=org
objectClass: inetOrgPerson
cn: user01
sn: user01
givenName: user01
displayName: user01
userPassword: {SSHA}生成的加密串
employeeNumber: XXJS-001
mobile: 19912345678
mail: user01@innosail.cn
postalAddress: 400000

[root@ldap-service ~]# ldapadd -x -D cn=root,dc=innosail,dc=org -W -f /root/addldap_user.ldif
Enter LDAP Password: 
adding new entry "dc=innosail,dc=org"
adding new entry "cn=root,dc=innosail,dc=org"
adding new entry "ou=People,dc=innosail,dc=org"
adding new entry "ou=Group,dc=innosail,dc=org"

```

注意：dn和userPassword，这个user01用户可以登陆已接入LDAP的应用



## 4.OpenLDAP 使用 SSL/TLS

``` shell
[root@ldap-service ~]# openssl req -new -x509 -nodes -out /etc/openldap/certs/ldap.cert -keyout /etc/openldap/certs/ldap.key -days 3650

[root@ldap-service ~]# cat /roo/certs.ldif
dn: cn=config
changetype: modify
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/ldap.cert
 
dn: cn=config
changetype: modify
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/ldap.key
 
[root@ldap-service ~]# ldapmodify -Y EXTERNAL  -H ldapi:/// -f /root/certs.ldif
 
##配置文件新增一个ldaps:///
 
[root@ldap-service ~]# vim /etc/sysconfig/slapd

# OpenLDAP server configuration
# see 'man slapd' for additional information

# Where the server will run (-h option)
# - ldapi:/// is required for on-the-fly configuration using client tools
#   (use SASL with EXTERNAL mechanism for authentication)
# - default: ldapi:/// ldap:///
# - example: ldapi:/// ldap://127.0.0.1/ ldap://10.0.0.1:1389/ ldaps:///
SLAPD_URLS="ldapi:/// ldap:/// ldaps:///"

# Any custom options
#SLAPD_OPTIONS=""

# Keytab location for GSSAPI Kerberos authentication
#KRB5_KTNAME="FILE:/etc/openldap/ldap.keytab"

##查看端口 636
[root@ldap-service ~]# netstat -nltp |grep slapd
tcp        0      0 0.0.0.0:636             0.0.0.0:*               LISTEN      1215/slapd          
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      1215/slapd          
tcp6       0      0 :::636                  :::*                    LISTEN      1215/slapd          
tcp6       0      0 :::389                  :::*                    LISTEN      1215/slapd  
```



## 5.开启日志

默认情况下OpenLDAP是没有启用日志记录功能的，但是在实际使用过程中，我们为了定位问题需要使用到OpenLDAP日志。

```shell
[root@ldap-service ~]# vim loglevel.ldif
dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: stats

##导入到OpenLDAP中，并重启OpenLDAP服务
[root@ldap-service ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/loglevel.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"

##修改rsyslog配置文件，并重启rsyslog服务
[root@ldap-service ~]# echo "local4.* /var/log/slapd.log" >> /etc/rsyslog.conf
[root@ldap-service ~]# systemctl restart rsyslog.service
[root@ldap-service ~]# systemctl restart slapd.service

##查看日志
[root@ldap-service ~]# tailf /var/log/slapd.log
 
```



## 6.服务重启重启，防火墙开启

```shell
[root@ldap-service ~]# firewall-cmd --add-service=ldap --permanent 
success
[root@ldap-service ~]# firewall-cmd --reload 
success
```



## 7.检查命令

```shell
##查看
[root@ldap-service ~]# ldapsearch -x -b "dc=innosail,dc=org" -H ldap://127.0.0.1
[root@ldap-service ~]# ldapsearch -LLL -x -D 'cn=root,dc=innosail,dc=org' -w '密码' -b 'dc=innosail,dc=org'

##删除用户
[root@ldap-service ~]# ldapdelete -x -W -D 'cn=root,dc=innosail,dc=org' "uid=user01,ou=People,dc=innosail,dc=org"

```

参考：

https://www.server-world.info/en/note?os=CentOS_7&p=openldap&f=1

https://myanbin.github.io/post/openldap-in-centos-7.html

 
