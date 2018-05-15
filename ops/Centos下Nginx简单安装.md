# Centos下Yum安装Nginx

[nginx官方](https://nginx.org)分(Stable)稳定版与Mainline(公测版）安装时根据自己的需求。

To set up the yum repository for RHEL/CentOS, create the file named `/etc/yum.repos.d/nginx.repo` with the following contents: 

> ```shell
> [nginx]
> name=nginx repo
> baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
> gpgcheck=0
> enabled=1
> ```

Replace “`OS`” with “`rhel`” or “`centos`”, depending on the distribution used, and “`OSRELEASE`” with “`6`” or “`7`”, for 6.x or 7.x versions, respectively.

根据自己的操作系统更新以上源信息

```shell
# yum install nginx 
```

完成安装后启动检查：

```shell
# /etc/init.d/nginx start
正在启动 nginx：                                           [确定]
# curl 127.0.0.1 -I
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Fri, 19 Jan 2018 06:14:28 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 17 Oct 2017 13:25:44 GMT
Connection: keep-alive
ETag: "59e604d8-264"
Accept-Ranges: bytes
```



