# 运维mac下终端ssh简单配置

 mac自带的终端还是比较方便，无需单独第三方工具，本文简单的配置基本满足日常工作。

* root 添加以下 # /etc/profile
```shell
if [ -f ~/.bashrc ]; then
source ~/.bashrc
fi
```
* $ vim ~/.bashrc (无则新建)
  添加alias
  如 alias ll=‘ls -l’
* $ cat ~/.ssh/config    (无则新建)
```shell
#添加ssh每次密码免登陆策略
Host *
  ForwardAgent yes
  ForwardX11 yes
ControlMaster auto

ControlPath ~/.ssh/%h-%p-%r
ControlPersist yes
ServerAliveInterval 60
```
* $ cat ~/.bashrc
```shell
export LS_OPTIONS='--color=auto' # 如果没有指定，则自动选择颜色
export CLICOLOR='Yes' #是否输出颜色
export LSCOLORS='CxfxcxdxbxegedabagGxGx' #指定颜色
alias sshhome="ssh -i ~/.ssh/xxxx-key work@ip地址或域名 -p端口号"
```
