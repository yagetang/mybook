## 用 Python 快速实现 HTTP 和 FTP 服务器

**用 Python 快速实现 HTTP 服务器**

有时你需临时搭建一个简单的 `Web Server`，但你又不想去安装 `Apache`、`Nginx` 等这类功能较复杂的 `HTTP` 服务程序时。这时可以使用 `Python` 内建的 `SimpleHTTPServer` 模块快速搭建一个简单的 `HTTP` 服务器。

`SimpleHTTPServer` 模块可以把你指定目录中的文件和文件夹以一个简单的 `Web` 页面的方式展示出来。

假设我们需要以 `Web` 方式共享目录 `/Users/Mike/Docker`，只需要以下这个命令行就可以轻松实现：

> ```shell
> $ cd /Users/Mike/Docker
> $ python -m SimpleHTTPServer
> Serving HTTP on 0.0.0.0 port 8000 ...
> ```

`SimpleHTTPServer` 模块默认会在 8000 端口上监听一个 `HTTP` 服务，这时就可以打开浏览器输入 `http://IP:Port`访问这个 `Web` 页面。例如类似下面的 URL：

> ```
> http://192.168.1.91:8000
> ```

如果你需要 `Web` 服务有一个默认页，可以在目录下创建一个名为 index.html 的文件。如果没有默认页，那么会以列表的形式将目录中的内容显示出来。

如果默认的 8000 端口已经被占用，你想换成使用其它端口号，可以使用如下的命令：

> ```shell
> $ python -m SimpleHTTPServer 8080
> ```

### 用 Python 快速实现 FTP 服务器

有时当你想快速搭建一个 `FTP` 服务器来临时实现文件上传下载时，这是特别有用的。我们这里利用 `Python` 的 `Pyftpdlib` 模块可以快速的实现一个 `FTP` 服务器的功能。

首先安装 `Pyftpdlib` 模块

> ```shell
> $ sudo pip install pyftpdlib
> ```

通过 `Python` 的 `-m` 选项将 `Pyftpdlib` 模块作为一个简单的独立服务器来运行，假设我们需要共享目录 `/Users/Mike/Docker`，只需要以下这个命令行就可以轻松实现：

> ```Shell
> $ cd /Users/Mike/Docker
> $ python -m pyftpdlib
> [I 2018-01-02 16:24:02] >>> starting FTP server on :::2121, pid=7517 <<<
> [I 2018-01-02 16:24:02] concurrency model: async
> [I 2018-01-02 16:24:02] masquerade (NAT) address: None
> [I 2018-01-02 16:24:02] passive ports: None
> ```

至此一个简单的 `FTP` 服务器已经搭建完成，访问 `ftp://IP:PORT` 即可。例如类似下面的 URL：

> ```
> ftp://192.168.1.91:2121
>
> ```

- 默认 IP 为本机所有可用 IP，端口为 2121。
- 默认登陆方式为匿名。
- 默认权限是只读。

如果你要建一个有认证且可写的 `FTP` 服务器，可使用类似以下指令：

> ```shell
> $ python -m pyftpdlib -i 0.0.0.0 -w -d /tmp/ -p 2121 -u admin -P "xxxxxx"
> ```

小插曲：测试时一直使用密码 `000000` 这样的弱密码做认证密码，在客户端登陆时一直提示认证失败。看来 `Pyftpdlib` 模块还做了基本的安全策略哟，不错的！

常用可选参数说明:

> ```
> -i 指定IP地址（默认为本机所有可用 IP 地址）
> -p 指定端口（默认为 2121）
> -w 写权限（默认为只读）
> -d 指定目录 （默认为当前目录）
> -u 指定登录用户名
> -P 指定登录密码
> ```

更多参数可以使用以下指令查询：

> ```shell
> $ python -m pyftpdlib --help
>
> Usage: python -m pyftpdlib [options]
>
> Start a stand alone anonymous FTP server.
>
> Options:
> -h, --help
> show this help message and exit
>
> -i ADDRESS, --interface=ADDRESS
> specify the interface to run on (default all interfaces)
>
> -p PORT, --port=PORT
> specify port number to run on (default 2121)
>
> -w, --write
> grants write access for logged in user (default read-only)
>
> -d FOLDER, --directory=FOLDER
> specify the directory to share (default current directory)
>
> -n ADDRESS, --nat-address=ADDRESS
> the NAT address to use for passive connections
>
> -r FROM-TO, --range=FROM-TO
> the range of TCP ports to use for passive connections (e.g. -r 8000-9000)
>
> -D, --debug
> enable DEBUG logging evel
>
> -v, --version
> print pyftpdlib version and exit
>
> -V, --verbose
> activate a more verbose logging
>
> -u USERNAME, --username=USERNAME
> specify username to login with (anonymous login will be disabled and password required if supplied)
>
> -P PASSWORD, --password=PASSWORD
> specify a password to login with (username required to be useful)
> ```

如果你需卸载 `Pyftpdlib` 模块，可以通过以下命令：

> ```
> $ pip uninstall pyftpdlib
> ```