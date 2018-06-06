# Docker安装Self Service Password

## 介绍

[自助服务密码](https://ltb-project.org/documentation/self-service-password) (Self Service Password)是一个PHP应用程序，允许用户在LDAP目录中更改其密码。

该应用程序可用于标准LDAPv3目录（OpenLDAP, OpenDS, ApacheDS, ~~Sun~~ Oracle DSEE, Novell, etc）以及Active Directory上。



Docker版本：18.05.0-ce

Self Service Password版本：[Self Service Password v1.2](https://ltb-project.org/documentation/self-service-password/1.2/start)



#### 1. 下载self Service Password镜像

```shell
# docker search "self Service Password" #搜索
# docker pull grams/ltb-self-service-password #下载
# docker images  #查看下载
```

#### 2. 加载镜像

``` shell
# docker run --detach \
--hostname service-yagetang-ldap-web.0100101001.tang \
--publish 8765:80 \
--name service-yagetang-ldap-web.0100101001.tang \
--restart always \
grams/ltb-self-service-password
```

#### 3. 配置文件

```shell
# docker exec  -ti CONTAINER ID或NAMES /bin/bash #进入容器

# vim /usr/share/self-service-password/conf/config.inc.php #修改对应值，见官方文档

# /etc/init.d/apache2 reload  #重新加载
```

config.inc.php部分配置

```php
# LDAP
$ldap_url = "ldap://ip地址";
$ldap_starttls = false;
$ldap_binddn = "cn=root,dc=innosail,dc=org";
$ldap_bindpw = '密码';
$ldap_base = "dc=innosail,dc=org";
$ldap_login_attribute = "uid";
$ldap_fullname_attribute = "cn";
$ldap_filter = "(&(objectClass=inetOrgPerson)($ldap_login_attribute={login}))";

## Mail for QQ
# LDAP mail attribute
$mail_attribute = "mail";
# Who the email should come from
$mail_from = "发送账号";
$mail_from_name = "Ldap Password";
# Notify users anytime their password is changed
$notify_on_change = true;
# PHPMailer configuration (see https://github.com/PHPMailer/PHPMailer)
$mail_sendmailpath = '/usr/sbin/sendmail';
$mail_protocol = 'smtp';
$mail_smtp_debug = 0;
$mail_debug_format = 'html';
$mail_smtp_host = 'smtp.exmail.qq.com';
$mail_smtp_auth = true;
$mail_smtp_user = '登陆账号';
$mail_smtp_pass = '密码';
$mail_smtp_port = 465;
$mail_smtp_timeout = 30;
$mail_smtp_keepalive = false;
#$mail_smtp_secure = 'tls';
$mail_smtp_secure = 'ssl';
$mail_contenttype = 'text/plain';
$mail_charset = 'utf-8';
$mail_priority = 3;
$mail_newline = PHP_EOL;

```



#### 4. 访问

http://服务器地址:8765



参考文档：https://ltb-project.org/documentation/self-service-password/latest/start