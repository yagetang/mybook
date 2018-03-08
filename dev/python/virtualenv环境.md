# virtualenv环境

在开发Python应用程序的时候，系统安装的Python3只有一个版本：3.6。所有第三方的包都会被`pip`安装到Python3的`site-packages`目录下。

如果我们要同时开发多个应用程序，那这些应用程序都会共用一个Python，就是安装在系统的Python 3。如果应用A需要jinja 2.7，而应用B需要jinja 2.6怎么办？

这种情况下，每个应用可能需要各自拥有一套“独立”的Python运行环境。virtualenv就是用来为一个应用创建一套“隔离”的Python运行环境。

首先，我们用`pip`安装virtualenv：

```
$ pip3 install virtualenv
```

然后，假定我们要开发一个新的项目，需要一套独立的Python运行环境，可以这么做：

第一步，创建目录：

```
[work@test-server yagetang]$ mkdir test-project
[work@test-server yagetang]$ cd test-project/
[work@test-server test-project]$ ll
总用量 0
```

第二步，创建一个独立的Python运行环境，命名为`venv`：

```
[work@test-server test-project]$  virtualenv --no-site-packages venv
Using base prefix '/usr/local'
New python executable in /data/yagetang/test-project/venv/bin/python3.6
Also creating executable in /data/yagetang/test-project/venv/bin/python
Installing setuptools, pip, wheel...
done.
[work@test-server test-project]$ 
[work@test-server test-project]$ ll
总用量 4
drwxrwxr-x 5 work work 4096 3月   8 14:40 venv
```

命令`virtualenv`就可以创建一个独立的Python运行环境，我们还加上了参数`--no-site-packages`，这样，已经安装到系统Python环境中的所有第三方包都不会复制过来，这样，我们就得到了一个不带任何第三方包的“干净”的Python运行环境。

新建的Python环境被放到当前目录下的`venv`目录。有了`venv`这个Python环境，可以用`source`进入该环境：

```
[work@test-server test-project]$ source venv/bin/activate
(venv) [work@test-server test-project]$ 
```

注意到命令提示符变了，有个`(venv)`前缀，表示当前环境是一个名为`venv`的Python环境。

下面正常安装各种第三方包，并运行`python`命令：

```
(venv) [work@test-server test-project]$ pip3 list
DEPRECATION: The default format will switch to columns in the future. You can use --format=(legacy|columns) (or define a format=(legacy|columns) in your pip.conf under the [list] section) to disable this warning.
pip (9.0.1)
setuptools (38.5.2)
wheel (0.30.0)
```

在`venv`环境下，用`pip`安装的包都被安装到`venv`这个环境下，系统Python环境不受任何影响。也就是说，`venv`环境是专门针对`myproject`这个应用创建的。

退出当前的`venv`环境，使用`deactivate`命令：

```
(venv) [work@test-server test-project]$ deactivate
[work@test-server test-project]$ 
```

此时就回到了正常的环境，现在`pip`或`python`均是在系统Python环境下执行。

完全可以针对每个应用创建独立的Python运行环境，这样就可以对每个应用的Python环境进行隔离。

virtualenv是如何创建“独立”的Python运行环境的呢？原理很简单，就是把系统Python复制一份到virtualenv的环境，用命令`source venv/bin/activate`进入一个virtualenv环境时，virtualenv会修改相关环境变量，让命令`python`和`pip`均指向当前的virtualenv环境。

### 小结

virtualenv为应用提供了隔离的Python运行环境，解决了不同应用间多版本的冲突问题。