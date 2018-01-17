Centos下Haproxy搭建

操作系统环境: CentOS release 6.9 (Final)

Haproxy ： version 1.5.18

Keepalived: Keepalived v1.2.13

高可用搭建方式： 一主一备共两个节点

haproxy-node01 192.168.1.57

haproxy-node02 192.168.1.47

HAProxy提供高可用性、负载均衡以及基于TCP和HTTP应用的代理，支持虚拟主机，它是免费、快速并且可靠的一种解决方案。

HAProxy特别适用于那些负载特大的web站点，这些站点通常又需要会话保持或七层处理。

HAProxy运行在当前的硬件上，完全可以支持数以万计的并发连接。并且它的运行模式使得它可以很简单安全的整合进您当前的架构中， 同时可以保护你的web服务器不被暴露到网络上。

HAProxy实现了一种事件驱动, 单一进程模型，此模型支持非常大的并发连接数。多进程或多线程模型受内存限制 、系统调度器限制以及无处不在的锁限制，很少能处理数千并发连接。事件驱动模型因为在有更好的资源和时间管理的用户空间(User-Space) 实现所有这些任务，所以没有这些问题。此模型的弊端是，在多核系统上，这些程序通常扩展性较差。这就是为什么他们必须进行优化以 使每个CPU时间片(Cycle)做更多的工作。

安装

    [root@haproxy-node01 work]# yum install haproxy
    [root@haproxy-node01 work]# yum install keepalived

配置haproxy

    ###########全局配置#########
    global
    　　log 127.0.0.1 local0 #[日志输出配置，所有日志都记录在本机，通过local0输出]
    　　log 127.0.0.1 local1 notice #定义haproxy 日志级别[error warringinfo debug]
    　　daemon #以后台形式运行harpoxy
    　　nbproc 1 #设置进程数量
    　　maxconn 4096 #默认最大连接数,需考虑ulimit-n限制
    　　#user haproxy #运行haproxy的用户
    　　#group haproxy #运行haproxy的用户所在的组
    　　#pidfile /var/run/haproxy.pid #haproxy 进程PID文件
    　　#ulimit-n 819200 #ulimit 的数量限制
    　　#chroot /usr/share/haproxy #chroot运行路径
    　　#debug #haproxy 调试级别，建议只在开启单进程的时候调试
    　　#quiet

    ########默认配置############
    defaults
    　　log global
    　　mode http #默认的模式mode { tcp|http|health }，tcp是4层，http是7层，health只会返回OK
    　　option httplog #日志类别,采用httplog
    　　option dontlognull #不记录健康检查日志信息
    　　retries 2 #两次连接失败就认为是服务器不可用，也可以通过后面设置
    　　#option forwardfor #如果后端服务器需要获得客户端真实ip需要配置的参数，可以从Http Header中获得客户端ip
    　　option httpclose #每次请求完毕后主动关闭http通道,haproxy不支持keep-alive,只能模拟这种模式的实现
    　　#option redispatch #当serverId对应的服务器挂掉后，强制定向到其他健康的服务器，以后将不支持
    　　option abortonclose #当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接
    　　maxconn 4096 #默认的最大连接数
    　　timeout connect 5000ms #连接超时
    　　timeout client 30000ms #客户端超时
    　　timeout server 30000ms #服务器超时
    　　#timeout check 2000 #心跳检测超时
    　　#timeout http-keep-alive10s #默认持久连接超时时间
    　　#timeout http-request 10s #默认http请求超时时间
    　　#timeout queue 1m #默认队列超时时间
    　　balance roundrobin #设置默认负载均衡方式，轮询方式
    　　#balance source #设置默认负载均衡方式，类似于nginx的ip_hash
    　　#balnace leastconn #设置默认负载均衡方式，最小连接数
    　　
    　--defaults-----------
    　defaults
        mode                    tcp
        log                     global
        option                  dontlognull
        option                  redispatch
        option                  abortonclose
        retries                 3
        timeout queue           7h
        timeout connect         3s
        timeout client          7h
        timeout server          7h
        timeout check           3s
        default-server maxconn  4096
        maxconn                 40960
    　---------------------

    ########统计页面配置########
    listen stats
    　　bind 0.0.0.0:1080 #设置Frontend和Backend的组合体，监控组的名称，按需要自定义名称
    　　mode http #http的7层模式
    　　option httplog #采用http日志格式
    　　#log 127.0.0.1 local0 err #错误日志记录
    　　maxconn 10 #默认的最大连接数
    　　stats refresh 30s #统计页面自动刷新时间
    　　stats uri /stats #统计页面url
    　　stats realm XingCloud\ Haproxy #统计页面密码框上提示文本
    　　stats auth admin:admin #设置监控页面的用户和密码:admin,可以设置多个用户名
    　　stats auth Frank:Frank #设置监控页面的用户和密码：Frank
    　　stats hide-version #隐藏统计页面上HAProxy的版本信息
    　　stats admin if TRUE #设置手工启动/禁用，后端服务器(haproxy-1.4.9以后版本)

    ########设置haproxy 错误页面#####
    #errorfile 403 /home/haproxy/haproxy/errorfiles/403.http
    #errorfile 500 /home/haproxy/haproxy/errorfiles/500.http
    #errorfile 502 /home/haproxy/haproxy/errorfiles/502.http
    #errorfile 503 /home/haproxy/haproxy/errorfiles/503.http
    #errorfile 504 /home/haproxy/haproxy/errorfiles/504.http

    ########frontend前端配置##############
    frontend main
    　　bind *:80 #这里建议使用bind *:80的方式，要不然做集群高可用的时候有问题，vip切换到其他机器就不能访问了。
    　　acl web hdr(host) -i www.abc.com  #acl后面是规则名称，-i为忽略大小写，后面跟的是要访问的域名，如果访问www.abc.com这个域名，就触发web规则，。
    　　acl img hdr(host) -i img.abc.com  #如果访问img.abc.com这个域名，就触发img规则。
    　　use_backend webserver if web   #如果上面定义的web规则被触发，即访问www.abc.com，就将请求分发到webserver这个作用域。
    　　use_backend imgserver if img   #如果上面定义的img规则被触发，即访问img.abc.com，就将请求分发到imgserver这个作用域。
    　　default_backend dynamic #不满足则响应backend的默认页面

    ########backend后端配置##############
    backend webserver #webserver作用域
    　　mode http
    　　balance roundrobin #balance roundrobin 负载轮询，balance source 保存session值，支持static-rr，leastconn，first，uri等参数
    　　option httpchk /index.html HTTP/1.0 #健康检查, 检测文件，如果分发到后台index.html访问不到就不再分发给它
    　　server web1 10.16.0.9:8085 cookie 1 weight 5 check inter 2000 rise 2 fall 3
    　　server web2 10.16.0.10:8085 cookie 2 weight 3 check inter 2000 rise 2 fall 3
    　　#cookie 1表示serverid为1，check inter 1500 是检测心跳频率
    　　#rise 2是2次正确认为服务器可用，fall 3是3次失败认为服务器不可用，weight代表权重

    backend imgserver
    　　mode http
    　　option httpchk /index.php
    　　balance roundrobin
    　　server img01 192.168.137.101:80 check inter 2000 fall 3
    　　server img02 192.168.137.102:80 check inter 2000 fall 3

    backend dynamic
    　　balance roundrobin
    　　server test1 192.168.1.23:80 check maxconn 2000
    　　server test2 192.168.1.24:80 check maxconn 2000


    listen tcptest
    　　bind 0.0.0.0:5222
    　　mode tcp
    　　option tcplog #采用tcp日志格式
    　　balance source
    　　#log 127.0.0.1 local0 debug
    　　server s1 192.168.100.204:7222 weight 1
    　　server s2 192.168.100.208:7222 weight 1
    　　
    # Elasticseach
    listen ipr-elasticsearch
        bind                    0.0.0.0:9200
        mode                    http
        balance                 roundrobin
        fullconn                40960
        server es01 elasticsearch.node01:9200 weight 5 check inter 2s rise 2 fall 3
        server es02 elasticsearch.node02:9200 weight 5 check inter 2s rise 2 fall 3
        server es03 elasticsearch.node03:9200 weight 5 check inter 2s rise 2 fall 3
    # zookeeper
    listen ipr-zookeeper
        bind                    0.0.0.0:2181
        mode                    tcp
        balance                 roundrobin
        fullconn                40960
        server zookeeper01 zookeeper.node01:2181 weight 5 check inter 2s rise 2 fall 3
        server zookeeper02 zookeeper.node02:2181 weight 5 check inter 2s rise 2 fall 3
        server zookeeper03 zookeeper.node03:2181 weight 5 check inter 2s rise 2 fall 3
    # swoole
    listen ipr-swoole
        bind                    0.0.0.0:19501
        mode                    tcp
        balance                 roundrobin
        fullconn                40960
        server swoole01 swoole.node01:19501 weight 5 check inter 2s rise 2 fall 3
        server swoole02 swoole.node02:19501 weight 5 check inter 2s rise 2 fall 3
        server swoole03 swoole.node03:19501 weight 5 check inter 2s rise 2 fall 3

启动

    [root@haproxy-node01 work]# service haproxy start
    [root@haproxy-node01 work]# netstat -nltp
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name
    tcp        0      0 0.0.0.0:32425               0.0.0.0:*                   LISTEN      1412/haproxy
    tcp        0      0 0.0.0.0:9200                0.0.0.0:*                   LISTEN      1412/haproxy
    tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1318/sshd
    tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1397/master
    tcp        0      0 :::22                       :::*                        LISTEN      1318/sshd
    tcp        0      0 ::1:25                      :::*                        LISTEN      1397/master

查看状态

    http://192.168.1.57:1080/stats

    #说明：
    #1080即haproxy配置文件中监听端口
    s#tats 即haproxy配置文件中的监听名称

配置keepalived

    #第一台服务器
    [work@haproxy-node01 ~]# cat /etc/keepalived/keepalived.conf
    ! Configuration File for keepalived
    global_defs {
       router_id LVS_DEVEL
    }
    vrrp_script chk_haproxy {
             script "/usr/local/script/check_haproxy.sh"
            interval 2
            weight 2
    }
    vrrp_instance VI_1 {
        state MASTER
        interface eth0
        virtual_router_id 51
        priority 100
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 5555
        }
    track_script {
            chk_haproxy
        }
        virtual_ipaddress {
            192.168.1.100/24
        }
    }

    #第二台服务器
    [root@haproxy-node02 ~]# cat /etc/keepalived/keepalived.conf
    ! Configuration File for keepalived
    global_defs {
       router_id LVS_DEVEL
    }
    vrrp_script chk_haproxy {
             script "/usr/local/script/check_haproxy.sh"
            interval 2
            weight 2
    }
    vrrp_instance VI_1 {
        state BACKUP #不同点
        interface eth0
        virtual_router_id 51
        priority 99 #不同点
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 5555
        }
    track_script {
            chk_haproxy
        }
        virtual_ipaddress {
            192.168.1.100/24
        }
    }

    #检测脚本，为了防止haproxy服务关闭导致keepalived不自动切换
    [work@haproxy-node01 ~]# cat /usr/local/script/check_haproxy.sh
    #!/bin/bash
    if [ $(ps -C haproxy --no-header | wc -l) -eq 0 ]; then
         /etc/init.d/haproxy  start
    fi
    sleep 2
    if [ $(ps -C haproxy --no-header | wc -l) -eq 0 ]; then
           /etc/init.d/keepalived stop
    fi


启动

    [work@haproxy-node01 ~]# service keepalived  start
    [work@haproxy-node01 ~]# ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether fa:16:3e:8b:96:89 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.57/24 brd 192.168.1.255 scope global eth0
        inet 192.168.1.100/24 scope global secondary eth0
        inet6 fe80::f816:3eff:fe8b:9689/64 scope link
           valid_lft forever preferred_lft forever

官网内核优化建议

    echo 1024 60999 > /proc/sys/net/ipv4/ip_local_port_range
    echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout
    echo 4096 > /proc/sys/net/ipv4/tcp_max_syn_backlog
    echo 262144 > /proc/sys/net/ipv4/tcp_max_tw_buckets
    echo 262144 > /proc/sys/net/ipv4/tcp_max_orphans
    echo 300 > /proc/sys/net/ipv4/tcp_keepalive_time
    echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle
    echo 0 > /proc/sys/net/ipv4/tcp_timestamps
    echo 0 > /proc/sys/net/ipv4/tcp_ecn
    echo 1 > /proc/sys/net/ipv4/tcp_sack
    echo 0 > /proc/sys/net/ipv4/tcp_dsack
