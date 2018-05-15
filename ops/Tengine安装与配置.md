# Tengine 安装与配置

操作系统版本：CentOS Linux release 7.4.1708 (Core)

Tengine版本：[Tengine-2.2.2](http://tengine.taobao.org/download/tengine-2.2.2.tar.gz) 

Tengine版本选择：http://tengine.taobao.org/download_cn.html

Tengine官方文档参见：http://tengine.taobao.org/documentation_cn.html



#### 简介

> [Tengine](http://tengine.taobao.org/)是由淘宝网发起的Web服务器项目。它在[Nginx](http://nginx.org/)的基础上，针对大访问量网站的需求，添加了很多高级功能和特性。Tengine的性能和稳定性已经在大型的网站如[淘宝网](http://www.taobao.com/)，[天猫商城](http://www.tmall.com/)等得到了很好的检验。它的最终目标是打造一个高效、稳定、安全、易用的Web平台。
>
> 从2011年12月开始，Tengine成为一个开源项目，Tengine团队在积极地开发和维护着它。Tengine团队的核心成员来自于[淘宝](http://www.taobao.com/)、[搜狗](http://www.sogou.com/)等互联网企业。Tengine是社区合作的成果，我们欢迎大家[参与其中](http://tengine.taobao.org/source_cn.html)，贡献自己的力量。

#### 特性

> - 继承Nginx-1.8.1的所有特性，兼容Nginx的配置；
> - [动态模块加载（DSO）](http://tengine.taobao.org/document_cn/dso_cn.html)支持。加入一个模块不再需要重新编译整个Tengine；
> - 支持[HTTP/2协议](http://nginx.org/en/docs/http/ngx_http_v2_module.html)，HTTP/2模块替代SPDY模块；
> - [流式上传](http://tengine.taobao.org/document_cn/http_core_cn.html)到HTTP后端服务器或FastCGI服务器，大量减少机器的I/O压力；
> - [支持异步OpenSSL](http://tengine.taobao.org/document_cn/ngx_http_ssl_asynchronous_mode_cn.html)，可使用硬件如:[QAT](http://tengine.taobao.org/document_cn/tengine_qat_ssl_cn.html)进行HTTPS的加速与卸载；
> - 更加强大的负载均衡能力，包括[一致性hash模块](http://tengine.taobao.org/document_cn/http_upstream_consistent_hash_cn.html)、[会话保持模块](http://tengine.taobao.org/document_cn/http_upstream_session_sticky_cn.html)，[还可以对后端的服务器进行主动健康检查](http://tengine.taobao.org/document_cn/http_upstream_check_cn.html)，根据服务器状态自动上线下线，以及[动态解析upstream中出现的域名](http://tengine.taobao.org/document_cn/http_upstream_dynamic_cn.html)；
> - [输入过滤器机制](http://blog.zhuzhaoyuan.com/2012/01/a-mechanism-to-help-write-web-application-firewalls-for-nginx/)支持。通过使用这种机制Web应用防火墙的编写更为方便；
> - 支持设置proxy、memcached、fastcgi、scgi、uwsgi[在后端失败时的重试次数](http://tengine.taobao.org/document_cn/ngx_limit_upstream_tries_cn.html)
> - [动态脚本语言Lua](https://github.com/alibaba/tengine/blob/master/modules/ngx_http_lua_module/README.markdown)支持。扩展功能非常高效简单；
> - 支持按指定关键字(域名，url等)[收集Tengine运行状态](http://tengine.taobao.org/document_cn/http_reqstat_cn.html)；
> - [组合多个CSS、JavaScript文件的访问请求变成一个请求](http://tengine.taobao.org/document_cn/http_concat_cn.html)；
> - [自动去除空白字符和注释](http://tengine.taobao.org/document_cn/http_trim_filter_cn.html)从而减小页面的体积
> - 自动根据CPU数目设置进程个数和绑定CPU亲缘性；
> - [监控系统的负载和资源占用从而对系统进行保护](http://tengine.taobao.org/document_cn/http_sysguard_cn.html)；
> - [显示对运维人员更友好的出错信息，便于定位出错机器；](http://tengine.taobao.org/document_cn/http_footer_filter_cn.html)
> - [更强大的防攻击（访问速度限制）模块](http://tengine.taobao.org/document_cn/http_limit_req_cn.html)；
> - [更方便的命令行参数，如列出编译的模块列表、支持的指令等](http://tengine.taobao.org/document_cn/commandline_cn.html)；
> - 可以根据访问文件类型设置过期时间；
> - ……

#### 安装

1. 安装必要的编译环境好

由于Tengine安装需要使用源代码自行编译，所以在安装前需要安装必要的编译工具：

> ``` shell
> # yum update
> # yum install gcc gcc-c++ autoconf automake12
> ```

2. 安装需要的组件

**A、PCRE** 
PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx rewrite依赖于PCRE库，所以在安装Tengine前一定要先安装PCRE，最新版本的PCRE可在官网（<http://www.pcre.org/>）获取。具体安装流程为：

> ```` shell
> # yum install pcre pcre-devel
> ````

附加信息：

源码的安装一般由3个步骤组成：配置(configure)、编译(make)、安装(make install)。 
Configure是一个可执行脚本，它有很多选项，在待安装的源码路径下使用命令./configure –help输出详细的选项列表。其中–prefix选项是配置安装的路径，如果不配置该选项，安装后可执行文件默认放在/usr /local/bin，库文件默认放在/usr/local/lib，配置文件默认放在/usr/local/etc，其它的资源文件放在/usr /local/share，比较凌乱。 
如果配置–prefix，如：./configure --prefix=/usr/local/test，可以把所有资源文件放在/usr/local/test的路径中，不会杂乱。 
用了--prefix选项的另一个好处是卸载软件或移植软件。当某个安装的软件不再需要时，只须简单的删除该安装目录，就可以把软件卸载得干干净净；移植软件只需拷贝整个目录到另外一个机器即可（相同的操作系统）。当然要卸载程序，也可以在原来的make目录下用一次make uninstall，但前提是make文件指定过uninstall。 
**B、OpenSSL** 
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。，安装OpenSSL（<http://www.openssl.org/source/>）主要是为了让tengine支持Https的访问请求。具体是否安装看需求。 
复制代码 代码如下:

> ``` shell
> # yum install openssl
> ```

**C、Zlib** 
Zlib是提供资料压缩之用的函式库，当Tengine想启用GZIP压缩的时候就需要使用到Zlib（<http://www.zlib.net/>）。

> ``` shell
> # yum install zlib zlib-devel
> ```

**D、jemalloc** 
jemalloc（<http://www.canonware.com/jemalloc/>）是一个更好的内存管理工具，使用jemalloc可以更好的优化Tengine的内存管理。

> ``` shell
> # yum install jemalloc jemalloc-devel
> ```

**E、GeoIP**

GeoIP数据包

> ```shell
> # yum -y install GeoIP GeoIP-devel perl-Geo-IP
> ```

**F、lua支持依赖安装(<http://luajit.org/download.html>) **

> ``` shell
> # wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz
> # tar -zxvf LuaJIT-2.0.5.tar.gz 
> # make install PREFIX=/usr/local/luajit
> 
> # #注意环境变量
> # vim /etc/profile
> 
> export LUAJIT_LIB=/usr/local/luajit/lib
> export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0
> ```



#### 安装Tengine

在主要核心的组件安装完毕以后就可以安装Tegine了，最新版本的Tegine可从官网(<http://tengine.taobao.org/>)获取。 
在编译安装前还需要做的一件事是添加一个专门的用户来执行Tengine。当然你也可以用root（不建议）。 
复制代码 代码如下:

> ``` shell
> # groupadd nginx
> # useradd -M -s /sbin/nologin -g nginx nginx
> ```

接下来才是进行安装：

> ``` shell
> ./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --dso-tool-path=/usr/sbin/ --includedir=/usr/include/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-http_v2_module --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_auth_request_module --with-mail --with-mail_ssl_module --with-http_geoip_module --with-file-aio --with-http_sysguard_module --with-force-exit --with-http_concat_module --with-jemalloc --with-http_dyups_module --with-http_dyups_lua_api --with-pcre-jit --with-openssl=../openssl-1.1.0h --with-ld-opt=-Wl,-rpath,/usr/lib --add-module=../ngx_devel_kit-0.3.1rc1 --add-module=../lua-nginx-module-0.10.13 --add-module=../lua-upstream-nginx-module-0.07 --add-module=../echo-nginx-module-0.61
> 
> # 模块
> 
> --with-openssl=../openssl-1.1.0h
> https://www.openssl.org/source/
> https://www.openssl.org/source/openssl-1.1.0h.tar.gz
> 
> --with-zlib=../zlib-1.2.11
> http://zlib.net/
> https://jaist.dl.sourceforge.net/project/libpng/zlib/1.2.11/zlib-1.2.11.tar.gz 
> 
> --with-pcre=../pcre-8.42
> http://www.pcre.org/
> https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz
> 
> lua支持
> --add-module=/path/to/ngx_devel_kit-0.3.1rc1 \   #ngx_devel_kit 的源码路径 
> https://github.com/simplresty/ngx_devel_kit/tags
> https://github.com/simplresty/ngx_devel_kit/archive/v0.3.1rc1.tar.gz
> 
> --add-module=/path/to/lua-nginx-module-0.10.13  #nginx_lua_module 的源码路径
> https://github.com/openresty/lua-nginx-module/tags
> https://github.com/openresty/lua-nginx-module/archive/v0.10.13.tar.gz
> 
> --add-module=../ngx_devel_kit-0.3.1rc1 --add-module=../lua-nginx-module-0.10.13
> 
> echo 模块
> https://github.com/openresty/echo-nginx-module
> --add-module=../echo-nginx-module-0.61
> 
> lua-upstream-nginx-module
> https://github.com/openresty/lua-upstream-nginx-module/releases
> --add-module=../lua-upstream-nginx-module-0.07
> ```



注意配置的时候 –with-pcre 、–with-openssl、–with-zlib的路径为源文件的路径。



## 4、配置Tengine自动启动

编辑启动文件添加下面内容

```shell
# vim /etc/rc.d/init.d/nginx 
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.1 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
#       It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /etc/nginx/nginx.conf

#Load Nginx Env
if [[ -f /etc/profile.d/sys_env.sh ]] ; then
    source /etc/profile.d/sys_env.sh
else
    echo "error:sys_env.sh file not exist." >> /var/log/nginx/error.log
    exit
fi

nginxd=/usr/sbin/nginx
nginx_config=/etc/nginx/nginx.conf
nginx_pid=/var/run/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ "${NETWORKING}" = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
  echo "nginx already running...."
  exit 1
fi
  echo -n $"Starting $prog: "
  daemon $nginxd -c ${nginx_config}
  RETVAL=$?
  echo
  [ $RETVAL = 0 ] && touch /var/run/nginx.lock
  return $RETVAL
}
# Stop nginx daemons functions.
stop() {
    echo -n $"Stopping $prog: "
    killproc $nginxd
    RETVAL=$?
    echo
    [ $RETVAL = 0 ] && rm -f /var/run/nginx.lock /var/run/nginx.pid
}
# reload nginx service functions.
reload() {
  echo -n $"Reloading $prog: "
  #kill -HUP `cat ${nginx_pid}`
  killproc $nginxd -HUP
  RETVAL=$?
  echo
}
# See how we were called.
case "$1" in
start)
    start
    ;;
stop)
    stop
    ;;
reload)
    reload
    ;;
restart)
    stop
    start
    ;;
status)
    status $prog
    RETVAL=$?
    ;;
*)
    echo $"Usage: $prog {start|stop|restart|reload|status|help}"
    exit 1
esac
exit $RETVAL

# chmod u+x /etc/rc.d/init.d/nginx
```

保存退出 
复制代码 代码如下:

```shell
# vim /usr/lib/systemd/system/nginx.service

[Unit]
Description=nginx - high performance web server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/etc/rc.d/init.d/nginx start 
ExecReload=/etc/rc.d/init.d/nginx reload
ExecStop=/etc/rc.d/init.d/nginx  stop

[Install]
WantedBy=multi-user.target
```



#### 启动nginx服务

```
systemctl start nginx.service
```

#### 设置开机自启动

```
systemctl enable nginx.service
```

#### 停止开机自启动

```
systemctl disable nginx.service
```

#### 查看服务当前状态

```
systemctl status nginx.service
```

#### 重新启动服务

```
systemctl restart nginx.service
```

#### 查看所有已启动的服务

```
systemctl list-units --type=service
```

异常处理

```Shell
# nginx 
nginx: error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory

# ln -s /usr/local/luajit/lib/libluajit-5.1.so.2.0.5 /lib/libluajit-5.1.so
# ln -s /usr/local/luajit/lib/libluajit-5.1.so.2.0.5 /lib/libluajit-5.1.so.2 
# ldconfig

# nginx
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (2: No such file or directory)

# /var/cache/nginx

#安全
# cd /etc/ssl/certs
# openssl dhparam -out dhparam.pem 4096
```

