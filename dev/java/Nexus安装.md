## Nexus安装

操作系统：`CentOS Linux release 7.6.1810 (Core) `

Nexus版本：`nexus-3.16.1`

下载地址：[www.sonatype.com](https://www.sonatype.com/download-oss-sonatype)

#### 环境准备
[JDK安装](https://www.jianshu.com/p/239edba08493)

#### 安装
下载后二进制包进行解包后包括两个文件夹如下：
```
# tar -xzvf nexus-3.16.1-02-unix.tar.gz
# mv nexus-3.16.1-02 /usr/local/src/
# mv sonatype-work /usr/local/src/
# ln -sv /usr/local/src/nexus-3.16.1-02 /usr/local/nexus
```

#### 配置启动用户
```
# vim /usr/local/nexus/bin/nexus.rc
run_as_user="root"
```
#### 配置环境变量

```
# vim /etc/profile
export NEXUS_HOME=/usr/local/nexus
export PATH=$NEXUS_HOME/bin:$PATH
# source /etc/profile
```
#### 添加启动服务
```
# vim /usr/lib/systemd/system/nexus.service
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/usr/local/nexus/bin/nexus start
ExecStop=/usr/local/nexus/bin/nexus stop
User=root
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

#### 启动服务
```
# systemctl enable nexus.service 
Created symlink from /etc/systemd/system/multi-user.target.wants/nexus.service to /usr/lib/systemd/system/nexus.service.

# systemctl start nexus.service
```
#### 检查服务

```
# systemctl status nexus.service 
● nexus.service - nexus service
   Loaded: loaded (/usr/lib/systemd/system/nexus.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2019-05-05 16:28:59 CST; 1min 7s ago
  Process: 27917 ExecStop=/usr/local/nexus/bin/nexus stop (code=exited, status=0/SUCCESS)
  Process: 28093 ExecStart=/usr/local/nexus/bin/nexus start (code=exited, status=0/SUCCESS)
 Main PID: 28229 (java)
    Tasks: 74
   Memory: 1.2G
   CGroup: /system.slice/nexus.service
           └─28229 /usr/java/jdk1.8.0_202-amd64/jre/bin/java -server -Dinstall4j.jvmDir=/usr/java/jdk1.8.0_202-amd64/jre -Dexe4j.moduleName=/usr/local/nexus/bin/nexus -XX:+UnlockDiagnosticVMOptions -Dinstall4j.launcherId=245 -Dinstall4j.swt=false -Di4jv=0 -Di4jv=0 -Di4...

5月 05 16:28:59 localhost.localdomain systemd[1]: Starting nexus service...
5月 05 16:28:59 localhost.localdomain nexus[28093]: WARNING: ************************************************************
5月 05 16:28:59 localhost.localdomain nexus[28093]: WARNING: Detected execution as "root" user.  This is NOT recommended!
5月 05 16:28:59 localhost.localdomain nexus[28093]: WARNING: ************************************************************
5月 05 16:28:59 localhost.localdomain systemd[1]: Started nexus service.
5月 05 16:28:59 localhost.localdomain nexus[28093]: Starting nexus

```

注：

- Web打开 `http://ip:8081`
- 默认账号：`admin` 密码：`admin123`

官方文档：[https://help.sonatype.com/repomanager3](https://help.sonatype.com/repomanager3)