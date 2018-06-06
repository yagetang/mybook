# Tengine下nginx-auth-ldap认证

**官方网站:**

<https://github.com/kvspb/nginx-auth-ldap>

操作系统版本：CentOS Linux release 7.4.1708 (Core)

Tengine版本：[Tengine-2.2.2](http://tengine.taobao.org/download/tengine-2.2.2.tar.gz)

OpenLDAP版本： 2.4.44

请参看[Tengine源码安装配置](https://mybook.yagetang.cn/ops/Tengine%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE.html)

[OpenLDAP 2.4.x安装配置](https://mybook.yagetang.cn/stand-by/CentOS%E4%B8%8BOpenLDAP%E5%AE%89%E8%A3%85.html)



#### 一. 添加nginx-auth-ldap nginx模块

编译nginx-auth-ldap模块需要ldap.h头文件,所以需要先安装ldap库

yum -y install openldap-devel

在编译nginx时,添加上模块编译参数,如

cd /usr/local/src

git clone https://github.com/kvspb/nginx-auth-ldap.git

--add-module=/usr/local/src/nginx-auth-ldap

#### 二. 配置ldap认证

```nginx
http {

        ldap_server openldap {
        url ldap://IP地址:389/dc=innosail,dc=org?uid?sub?(&(objectClass=inetOrgPerson));
        binddn "cn=root,dc=innosail,dc=org";
        binddn_passwd "密码";
        group_attribute 用户组;
        group_attribute_is_dn on;
        require valid_user;
      }

}

server {

       location /status {
            stub_status on;
            access_log off;
            auth_ldap "Restricted Space";
            auth_ldap_servers openldap;
        }
        
}
```



参考资料：https://github.com/kvspb/nginx-auth-ldap