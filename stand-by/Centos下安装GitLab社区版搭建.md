# Centos下安装GitLab社区版搭建

GitLab 是一个用于仓库管理系统的开源项目，使用[Git](https://about.gitlab.com/)作为代码管理工具，并在此基础上搭建起来的web服务。

推荐最低配置内存4G

#### 1.必须的依赖安装

On CentOS 6 (and RedHat/Oracle/Scientific Linux 6), the commands below will also open HTTP and SSH access in the system firewall.

> ```shell
> sudo yum install -y curl policycoreutils-python openssh-server cronie
> sudo lokkit -s http -s ssh
> ```

#### 2.  邮件服务

Next, install Postfix to send notification emails. If you want to use another solution to send emails please skip this step and [configure an external SMTP server](https://docs.gitlab.com/omnibus/settings/smtp.html) after GitLab has been installed.

> ```shell
> sudo yum install postfix
> sudo service postfix start
> sudo chkconfig postfix on
> ```

#### 3. 安装gpg签名名与yum工具包

> ```shell
> sudo yum install pygpgme yum-utils
> ```

#### 4. 创建源： /etc/yum.repos.d/gitlab_gitlab-ce.repo 

Make sure to replace el and 6 in the config below with your [Linux distribution and version](https://packagecloud.io/docs#os_distro_version):

> ```shell
> # vim /etc/yum.repos.d/gitlab_gitlab-ce.repo #该源为官网源，可能速度慢
> [gitlab_gitlab-ce]
> name=gitlab_gitlab-ce
> baseurl=https://packages.gitlab.com/gitlab/gitlab-ce/el/6/$basearch
> repo_gpgcheck=1
> gpgcheck=1
> enabled=1
> gpgkey=https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
>        https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg
> sslverify=1
> sslcacert=/etc/pki/tls/certs/ca-bundle.crt
> metadata_expire=300
>
> [gitlab_gitlab-ce-source]
> name=gitlab_gitlab-ce-source
> baseurl=https://packages.gitlab.com/gitlab/gitlab-ce/el/6/SRPMS
> repo_gpgcheck=1
> gpgcheck=1
> enabled=1
> gpgkey=https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
>        https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg
> sslverify=1
> sslcacert=/etc/pki/tls/certs/ca-bundle.crt
> metadata_expire=300
> ```

因速度原因这里我选择配置清华大学源

> ```shell
> # vim /etc/yum.repos.d/gitlab_gitlab-ce.repo
> [gitlab_gitlab-ce]
> name=gitlab_gitlab-ce
> baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6
> Repo_gpgcheck=0
> Enabled=1
> gpgkey=https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
>        https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg
> ```
>
> 

#### 5. 安装GitLab社区版

> ```shell
> sudo EXTERNAL_URL="http://gitxx.xxx.com" yum  install gitlab-ce      #自动安装最新版并分配指定域名
> sudo EXTERNAL_URL="http://gitxx.xxx.com" yum install gitlab-ce-10.3.0    #安装指定版本并分配指定域名
> ```

#### 6. 完成

浏览器输入服务器地址或绑定hosts访问域名，就可以看到界面了。首页登陆需要更改root用户密码。

#### 7. 邮件服务器配置

我这里使用的是腾讯的企业邮箱，官方推荐[更多](https://docs.gitlab.com/omnibus/settings/smtp.html)

> ```shell
> gitlab_rails['smtp_enable'] = true
> gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
> gitlab_rails['smtp_port'] = 465
> gitlab_rails['smtp_user_name'] = "xxxx@xx.com"
> gitlab_rails['smtp_password'] = "your password"
> gitlab_rails['smtp_authentication'] = "login"
> gitlab_rails['smtp_enable_starttls_auto'] = true
> gitlab_rails['smtp_tls'] = true
> gitlab_rails['gitlab_email_from'] = 'xxxx@xx.com'
> ```
>
> 

#### last. GitLab常用命令

> ```shell
> sudo gitlab-ctl start    # 启动所有 gitlab 组件；
> sudo gitlab-ctl stop        # 停止所有 gitlab 组件；
> sudo gitlab-ctl restart        # 重启所有 gitlab 组件；
> sudo gitlab-ctl status        # 查看服务状态；
> sudo gitlab-ctl reconfigure        # 启动服务；
> sudo vim /etc/gitlab/gitlab.rb        # 修改默认的配置文件；
> gitlab-rake gitlab:check SANITIZE=true --trace    # 检查gitlab；
> sudo gitlab-ctl tail        # 查看日志；
> ```