# Docker运行Gitlab

## Gitlab简介

[GitLab](https://about.gitlab.com/)是一个Git的代码托管工具，有免费的社区版允许我们在本地搭建代码托管网站，也有付费的企业版网站，能够在线托管代码。传统方式是手动下载Gitlab的软件包，然后搭建相关运行环境。不过这种方式非常麻烦，而且如果要更换机器所有配置工作又得重来一边，如果有同学学过Java的话应该记得初学Java时配置环境变量的恐惧吧？因此更好的办法就是使用现在非常流行的Docker。

那么[Docker](https://www.docker.com/what-docker)又是个什么东西呢？这是一个虚拟化的运行工具，主要目的是将软件和整个运行环境打包起来，让我们不需要配置即可快速运行软件。由于Docker依赖于Linux内核的某些特性，所以Docker只能在Linux上运行。Windows上的Docker实际上是开了一个虚拟机。Docker目前好像没有比较好的中文社区，我谷歌了一下只找到了这个[Docker中文社区](http://www.docker.org.cn/index.html)，看起来还行。



操作系统版本：CentOS Linux release 7.4.1708 (Core)

Docker版本：18.05.0-ce

Gitla版本：GitLab Community Edition 10.8.1

Docker 版本选择：<https://download.docker.com/linux/centos/7/x86_64/stable/Packages/>

Centos下安装Docker官方文档参见：<https://docs.docker.com/v1.13/engine/installation/linux/centos/>



## 安装docker

安装命令如下

```
#删除老版本
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
#组件支持                   
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

#更新yum源  
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
#更新yum源
$ sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#启用稳定版
$ yum-config-manager --enable docker-ce-edge

#安装docker-ce
$ sudo yum install docker-ce

注：Starting with Docker 17.06, stable releases are also pushed to the edge and test repositories.
```

安装好之后，来看看Docker的版本，显示类似下面这样的信息。Docker客户端的版本最好在1.10以上。

```
docker version
Client:
 Version:      18.05.0-ce
 API version:  1.37
 Go version:   go1.9.5
 Git commit:   f150324
 Built:        Wed May  9 22:14:54 2018
 OS/Arch:      linux/amd64
 Experimental: false
 Orchestrator: swarm

Server:
 Engine:
  Version:      18.05.0-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.5
  Git commit:   f150324
  Built:        Wed May  9 22:18:36 2018
  OS/Arch:      linux/amd64
  Experimental: false
```

这样Docker就安装成功了。

## 使用阿里云加速Docker

Docker官方镜像网站部署在外网，因此我们国内下载比较慢。看了一下国内最好的Docker加速服务就是阿里云了。阿里云的其他镜像比如Maven镜像之类的也都不错。

首先需要注册一个[阿里云](https://www.aliyun.com/)的帐号，可能还需要其他一点信息。然后进入[容器Hub服务控制台](https://cr.console.aliyun.com/?spm=5176.2020520152.aliyun_topbar.12.765316ddUTMAOj#/imageList)，中间有一个加速器。我们点击它之后，阿里云会为我们创建一个专属加速器地址。

然后需要检查Docker客户端的版本，如果小于1.10，只能按照自己系统版本寻找相应的办法了。如果大于等于1.10，就可以直接使用下面的配置方法。配置方法很简单，在`/etc/docker/daemon.json`中添加一段配置。如果没有该文件则创建。

```
{
    "registry-mirrors": ["<your accelerate address>"]
}
```

然后重启Docker服务。

```
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker
```

## 下载Gitlab

配置好加速器之后，下载就很快了。直接执行下面的命令，稍等片刻之后，Docker就会将Gitlab下载好了。

```
sudo docker pull gitlab/gitlab-ce:latest
```

## 启动Gitlab

用下面的命令启动一个默认配置的Gitlab。

```shell
$ sudo mkdir -p /srv/gitlab
    
docker run --detach \
 --hostname service-yagetang-git-web.0100011001.tang \
 --publish 10443:443 \
 --publish 10080:80 \
 --publish 10022:22 \
 --name service-yagetang-git-web.0100011001.tang \
 --restart always\
 --volume /srv/gitlab/config:/etc/gitlab \
 --volume /srv/gitlab/logs:/var/log/gitlab \
 --volume /srv/gitlab/data:/var/opt/gitlab \
 gitlab/gitlab-ce:latest

eg:docker run -i -t --hostname java-yagetang-demo-web.0100011001.tang --name java-yagetang-demo-web.0100011001.tang --restart always centos /bin/bash

```

首次启动可能比较慢，需要等待一分钟左右的时间。我们可以使用`sudo docker ps`命令查看当前所有Docker容器的状态。当它的状态由starting变为运行时间时，说明成功启动了。我们直接使用上面配置的IP地址加端口号在浏览器中访问即可。

初次使用需要我们创建默认管理员密码，随便指定一个就行了。然后我们需要注册一个普通用户。以后的使用方法和Github这样的工具很相似了。

## 配置Gitlabs

刚刚启动Gitlab的时候需要我们输入一个密码，这个密码是管理员用户的密码。我们在登录那里使用root作为用户名，然后用刚刚设置的密码，就可以以管理员身份登录Gitlab。

登录进去之后，点击右上角的齿轮图标，即可进入到管理员页面。在这里我们可以设置很多东西。比如说，默认情况下每个用户只能创建10个仓库，我们可以改变这个设置。在管理员页面点击右面的齿轮，再点击设置，就会进入到系统设置中。然后找到Default projects limit一项，我们给它设个小目标，设它一个亿，这样就相当于无限仓库了。当然如果你实际硬盘满了也就不能在创建更多项目了。

如果这些配置还是不能满足你的需求的时候，还可以直接配置Gitlab。首先进入到Docker环境中。我们使用下面的命令进入Docker环境的bash中。gitlab是刚刚指定的Gitlab名称。

```
sudo docker exec -it gitlab /bin/bash
```

然后就进入了Docker的环境中，我们可以把它当作一个独立的系统来使用。然后编辑`/etc/gitlab/gitlab.rb`文件，这是Gitlab的全局配置文件。所有选项都可以在这里配置。



# 开放gitlab的https支持

仅仅由nginx反向代理https是不行的，因为还需要打开gitlab的https支持。

- 修改配置文件，在/srv/gitlab/config/ 目录下的gitlab.rb中添加：

```
#复制证书文件,没有证书需要自行申请
$ sudo mkdir  /srv/gitlab/config/ssl
$ sudo cp yagetang.cn.key /srv/gitlab/config/ssl/yagetang.cn.key
$ sudo cp yagetang.cn.cer /srv/gitlab/config/ssl/yagetang.cn.cer

$ sudo vim /srv/gitlab/config/gitlab.rb 
external_url 'https://git.yagetang.cn'
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/yagetang.cn.cer"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/yagetang.cn.key"

```

- 进入docker 

```
# docker exec -ti service-yagetang-git-web.0100011001.tang /bin/bash
root@service-yagetang-git-web:/# gitlab-ctl reconfigure #让配置文件生效
```

注意：如果ssl证书是自己生成的，并不具有全网通用性，需要选择相信证书。在本地设置环境变量：

```
$ export GIT_SSL_NO_VERIFY=1
```



# 配置nginx，支持https

参考：[gitlab set nginx](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md#enable-https)
 nginx 配置文件

```
server {
    listen       80;
    server_name  git.yagetang.cn;
    charset utf-8;
    location / {
      rewrite ^/(.*)$ https://$host/$1 permanent;
    }
    access_log off;
}

server {
    listen       443 ssl;
    server_name  git.yagetang.cn;
    charset utf-8;
    ssl on;
    ssl_certificate         /etc/nginx/ssl/yagetang.cn.cer;
    ssl_certificate_key     /etc/nginx/ssl/yagetang.cn.key;
    ssl_session_timeout     10m;
    ssl_session_cache       shared:SSL:10m;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    proxy_pass    https://127.0.0.1:10443;
     }
  #  set_by_lua $envType 'return os.getenv("envType")';
  #  set_by_lua $envArea 'return os.getenv("envArea")';
  #  set $group "git.yagetang.cn_${envType}_${envArea}_root_static";

  #  include rule.conf;

  #  location  / {
  #      include proxy.conf;
  #      proxy_set_header X-Forwarded-Proto https;
  #      proxy_pass https://$group;
 # }

  access_log   /var/log/nginx/access.log main;

}
```



# 开启邮件服务

默认的邮件服务不太好使，所以这里配置自己的邮件服务。参考官方[gitlab stmp文档](https://docs.gitlab.com/omnibus/settings/smtp.html )。

使用腾讯邮箱, 按照官方文档配置，注意端口及服务地址。

```shell
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "用户@yagetang.cn"
gitlab_rails['smtp_password'] = "密码"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = '用户@yagetang.cn'
gitlab_rails['smtp_domain'] = "exmail.qq.com"
```



# ssh方式访问

因为是使用docker部署的，通过ssh方式(比如git clone ssh://git@git.yagetang.cn/test-group/shell-test.git 访问会有两层认证:

- 一层是freelancer服务器的认证
- 另一层是gitlab的认证。

后者需要使用ssh-key
 前者可能需要ssh本身的反向代理(现在使用的nginx不支持除http，https以外的反向代理)，

现在发现使用端口转发的形式比较困难，但是可以改变默认的gitlab的ssh端口为非标准端口：
 直接修改gitlab配置文件中的变量：

```
gitlab_shell_ssh_port = 10022
```

然后重新启动docker容器，就可以在web界面中看到相应的ssh地址发生了改变:
 ssh://git@git.yagetang.cn:10022/test-group/shell-test.git  然后就直接可以继续使用git clone来继续操作了



## LDAP配置

参考地址：[https://docs.gitlab.com/omnibus/settings/ldap.html](https://docs.gitlab.com/omnibus/settings/ldap.html )

``` shell
# gitlab_rails['ldap_enabled'] = false
gitlab_rails['ldap_enabled'] = true

###! **remember to close this block with 'EOS' below**
gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
main: # 'main' is the GitLab 'provider ID' of this LDAP server
     label: 'LDAP'
     host: 'LDAP服务器地址'
     port: 端口号
     uid: 'uid'
     bind_dn: 'cn=root,dc=yagetang,dc=org'
     password: '密码'
     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
     verify_certificates: true
     active_directory: true
     allow_username_or_email_login: false
     lowercase_usernames: false
     block_auto_created_users: false
     base: 'dc=yagetang,dc=org'
     user_filter: ''
     ## EE only
     group_base: 'ou=People'
     admin_group: ''
     sync_ssh_keys: false
#
#   secondary: # 'secondary' is the GitLab 'provider ID' of second LDAP server
#     label: 'LDAP'
#     host: '_your_ldap_server'
#     port: 389
#     uid: 'sAMAccountName'
#     bind_dn: '_the_full_dn_of_the_user_you_will_bind_with'
#     password: '_the_password_of_the_bind_user'
#     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
#     verify_certificates: true
#     active_directory: true
#     allow_username_or_email_login: false
#     lowercase_usernames: false
#     block_auto_created_users: false
#     base: ''
#     user_filter: ''
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
EOS


# gitlab-ctl reconfigure #进入docker让配置文件生效 
```





## 更新Gitlab

以后如果需要更新Gitlab版本，首先需要停止并删除当前的Gitlab实例。

```
sudo docker stop gitlab
sudo docker rm gitlab
```

然后在拉取最新版的Gitlab。

```
sudo docker pull gitlab/gitlab-ce:latest
```

然后在使用上次的配置运行Gitlab即可。不用担心数据会丢失。只要你的volume参数指定还和上次一样，Gitlab就会自动读取这些配置。

```shell
docker run --detach \
 --hostname service-yagetang-git-web.0100011001.tang \
 --publish 10443:443 \
 --publish 10080:80 \
 --publish 10022:22 \
 --name service-yagetang-git-web.0100011001.tang \
 --restart always\
 --volume /srv/gitlab/config:/etc/gitlab \
 --volume /srv/gitlab/logs:/var/log/gitlab \
 --volume /srv/gitlab/data:/var/opt/gitlab \
 gitlab/gitlab-ce:latest
```

最后来看看使用Docker的优势。还是在Gitlab的Bash中。我们依次输入下面的命令，看看有什么反应。

```shell
ruby --version
git --version
redis-cli --version
psql --version
```

不出意外的话应该会显示对应软件的版本。我们看到Gitlab使用了4个开源软件或运行环境：ruby、git、redis和postgresql。如果我们手动安装Gitlab的话，这几个软件也必须分别安装和配置好。这个任务的难度可是非常大的。而且如果需要在多台机器上配置，那么任务量就更大了。但是如果使用Docker的话，我们甚至完全没必要知道这几个软件的存在，简单两条命令即可创建和运行Gitlab。**这正是Docker的魅力，难怪现在越来越多的公司在使用Docker。**



## 参考资料

- [https://yq.aliyun.com/articles/29941](https://yq.aliyun.com/articles/29941)
- [https://docs.gitlab.com/omnibus/docker/README.html#gitlab-docker-images](https://docs.gitlab.com/omnibus/docker/README.html#gitlab-docker-images)
- [https://www.jianshu.com/p/05e3bb375f64](https://www.jianshu.com/p/05e3bb375f64)

 
