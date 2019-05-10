# SSH通过TUNNEL把本地开发环境公布到互联网

​       我们有时候需要把内部的地址或应用映射到公网，对于没有固定IP出口的人员来说是非常难办的事。这里使用 SSH 在本地电脑与公网服务器之间打开一个通道，简单的处理实现。

必备条件：

一台可以公网登陆的服务器（需要该台机器做代理，反向端口映射）。

通道

> 我们要在本地与公网服务器之间，使用 SSH 打开一个通道。命令如下：

```
ssh -vnNT -R 服务器端口:localhost:本地端口 服务器用户名@服务器 IP 地址
```

示例

```shell
# 本地的80端口
$ curl http://127.0.0.1:80/favicon.ico -I
HTTP/1.1 200 OK
Server: Tengine
Date: Mon, 21 May 2018 02:45:43 GMT
Content-Type: image/x-icon
Content-Length: 4286
Last-Modified: Fri, 18 May 2018 04:05:35 GMT
Connection: keep-alive
ETag: "5afe510f-10be"
Accept-Ranges: bytes

# 开启本地端口80到公网服务器47.98.109.88的7689端口通道
$ ssh -vnNT -R 7689:localhost:80 root@47.98.109.88

# 公网服务器
# curl http://127.0.0.1:7689/favicon.ico -I
HTTP/1.1 200 OK
Server: Tengine
Date: Mon, 21 May 2018 02:46:17 GMT
Content-Type: image/x-icon
Content-Length: 4286
Last-Modified: Fri, 18 May 2018 04:05:35 GMT
Connection: keep-alive
ETag: "5afe510f-10be"
Accept-Ranges: bytes
```

> 在上面这个例子里，7689 指的是公网服务器的端口，localhost 后面的 80 是本地电脑用的端口。root 是登录到公网服务器的用户，47.98.109.88 是公网服务器的 IP 地址。

