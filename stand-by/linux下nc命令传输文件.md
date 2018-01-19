# linux nc 命令传输文件

### 1. 接收端pc命令(目的主机监听)：

nc -l 监听端口<未使用端口>  > 要接收的文件名

ip:192.168.1.100

```shell
nc -l  5555 > /tmp/nginx.tar.gz 
```

### 2. 发送端pc命令(源主机发起请求)：

nc  目的主机ip    目的端口 < 要发送的文件名

ip:192.168.1.101 

```shell
nc -w 1000 192.168.1.100 5555 < /tmp/nginx.tar.gz
```

### 3. 命令语法

想要连接到某处: nc [-options] hostname port[s][ports] … 
绑定端口等待连接: nc -l -p port [-options][hostname] [port] 
参数: 
-g gateway source-routing hop point[s], up to 8 
-G num source-routing pointer: 4, 8, 12, … 
-h 帮助信息 
-i secs 延时的间隔 
-l 监听模式，用于入站连接 
-n 指定数字的IP地址，不能用hostname 
-o file 记录16进制的传输 
-p port 本地端口号 
-r 任意指定本地及远程端口 
-s addr 本地源地址 
-u UDP模式 
-v 详细输出——用两个-v可得到更详细的内容 
-w secs timeout的时间 
-z 将输入输出关掉——用于扫描时，其中端口号可以指定一个或者用lo-hi式的指定范围。

注：nc 命令非常强大，不仅限此功能。