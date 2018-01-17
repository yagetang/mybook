Python3.X 安装 for Centos

1. yum -y install openssl*  gcc-c++

(pip依赖ssl环境)

1. 编译安装python3

下载地址:https://www.python.org/ftp/python/

    1 tar -zxvf  Python-3.6.3.tgz
    2 cd Python-3.6.3
    3 ./configure
    4 make && make install

1. 配置python3与pip3

    # ln -s /usr/local/bin/python3.6 /usr/bin/python3
    # python3 -V
    Python 3.6.3

    # ln -s /usr/local/bin/pip3 /usr/bin/pip3
    # pip3 list --format=columns
    Package    Version
    ---------- -------
    pip        8.1.1
    setuptools 28.8.0
    ## pip指定国内aliyun源
    # vim ~/.pip/pip.conf #如果没这文件则创建
    [global]
    trusted-host=mirrors.aliyun.com
    index-url=http://mirrors.aliyun.com/pypi/simple/


1. 安装完成可以装个模块试试

    # pip3 install pymysql
    Collecting pymysql
      Downloading http://mirrors.aliyun.com/pypi/packages/e5/07/c0f249aa0b7b0517b5843eeab689b9ccc6a6bb0536fc9d95e65901e6f2ac/PyMySQL-0.8.0-py2.py3-none-any.whl (83kB)
        100% |████████████████████████████████| 92kB 2.8MB/s
    Installing collected packages: pymysql
    Successfully installed pymysql-0.8.0
    You are using pip version 8.1.1, however version 9.0.1 is available.
    You should consider upgrading via the 'pip install --upgrade pip' command.

上面提示有新版本了  可以升级.

使用它提示的命令就可以升级pip3了..

但是注意要把pip命令替换成pip3

     1 # pip3 install --upgrade pip
     2 Collecting pip
     3   Downloading pip-9.0.1-py2.py3-none-any.whl (1.3MB)
     4     100% |████████████████████████████████| 1.3MB 3.2kB/s
     5 Installing collected packages: pip
     6   Found existing installation: pip 8.1.1
     7     Uninstalling pip-8.1.1:
     8       Successfully uninstalled pip-8.1.1
     9 Successfully installed pip-9.0.1
    10 # pip3 --version
    11 pip 9.0.1 from /usr/local/lib/python3.6/site-packages (python 3.6)

升级完成



PS:

如果使用pip3安装插件的时候提示:

pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.

是因为系统缺少openssl-devel包

yum install openssl-devel -y  安装一下即可.

再按照上面的方法重新 编译一下即可.

