## 使用curl 命令模拟POST/GET请求

curl命令是一个利用URL规则在命令行下工作的文件传输工具。它支持文件的上传和下载。curl支持包括HTTP、HTTPS、ftp等众多协议，还支持POST、cookies、认证、从指定偏移处下载部分文件、用户代理字符串、限速、文件大小、进度条等特征。

在进行web后台程序开发测试过程中，常常会需要发送url进行测试，使用curl可以方便地模拟出符合需求的url命令

假设目标url 为：`127.0.0.1:8080/login`

使用curl发送GET请求：`curl protocol://address:port/url?args`

```
curl http://127.0.0.1:8080/login?admin&passwd=12345678
```
使用curl发送POST请求：`curl -d "args" protocol://address:port/url`
```
curl -d "user=admin&passwd=12345678" http://127.0.0.1:8080/login
```
这种方法是参数直接在header里面的，如需将输出指定到文件可以通过重定向进行操作
```
curl -H "Content-Type:application/json" -X POST -d 'json data' URL
curl -H "Content-Type:application/json" -X POST -d '{"user": "admin", "passwd":"12345678"}' http://127.0.0.1:8000/login
```
curl POST 上传文件
上面的两种请求，都是只传输字符串，我们在测试上传接口的时候，会要求传输文件，其实这个对于 curl 来说，也是小菜一碟。

我们用 -F "file=@__FILE_PATH__" 的请示，传输文件即可。命令如下：
```
curl http://127.0.0.1:8000/api/v1/upimg -F "file=@/Users/fungleo/Downloads/401.png" -H "token: 222" -v
```
其它功能介绍
#### 一、get请求

curl "http://www.baidu.com"  如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地
- curl -i "http://www.baidu.com"  显示全部信息
- curl -l "http://www.baidu.com" 只显示头部信息
- curl -v "http://www.baidu.com" 显示get请求全过程解析
- wget "http://www.baidu.com"也可以

#### 二、post请求

- curl -d "param1=value1&param2=value2" "http://www.baidu.com"


用途说明
curl命令是一个功能强大的网络工具，它能够通过http、ftp等方式下载文件，也能够上传文件。其实curl远不止前面所说的那些功能，大家可以通过man curl阅读手册页获取更多的信息。类似的工具还有wget。

curl命令使用了libcurl库来实现，libcurl库常用在C程序中用来处理HTTP请求，curlpp是libcurl的一个C++封装，这几个东西可以用在抓取网页、网络监控等方面的开发，而curl命令可以帮助来解决开发过程中遇到的问题。

常用参数
curl命令参数很多，这里只列出我曾经用过、特别是在shell脚本中用到过的那些。

-A:随意指定自己这次访问所宣称的自己的浏览器信息

-b/--cookie <name=string/file> cookie字符串或文件读取位置，使用option来把上次的cookie信息追加到http request里面去。

-c/--cookie-jar <file> 操作结束后把cookie写入到这个文件中

-C/--continue-at <offset>  断点续转

-d/--data <data>   HTTP POST方式传送数据

-D/--dump-header <file> 把header信息写入到该文件中

-F/--form <name=content> 模拟http表单提交数据

-v/--verbose 小写的v参数，用于打印更多信息，包括发送的请求信息，这在调试脚本是特别有用。

-m/--max-time <seconds> 指定处理的最大时长

-H/--header <header> 指定请求头参数

-s/--slient 减少输出的信息，比如进度

--connect-timeout <seconds> 指定尝试连接的最大时长

-x/--proxy <proxyhost[:port]> 指定代理服务器地址和端口，端口默认为1080

-T/--upload-file <file> 指定上传文件路径

-o/--output <file> 指定输出文件名称

--retry <num> 指定重试次数

-e/--referer <URL> 指定引用地址

-I/--head 仅返回头部信息，使用HEAD请求

-u/--user <user[:password]>设置服务器的用户和密码

-O:按照服务器上的文件名，自动存在本地

-r/--range <range>检索来自HTTP/1.1或FTP服务器字节范围

-T/--upload-file <file> 上传文件

使用示例
1，抓取页面内容到一个文件中

　　[root@xi mytest]# curl -o home.html http://www.baidu.com   --将百度首页内容抓下到home.html中

　　   [root@xi mytest]#curl -o #2_#1.jpghttp://cgi2.tky.3web.ne.jp/~{A,B}/[001-201].JPG

           由于A/B下的文件名都是001，002...，201，下载下来的文件重名，这样，自定义出来下载下来的文件名，就变成了这样：原来： A/001.JPG —-> 下载后： 001-A.JPG 原来： B/001.JPG ---> 下载后： 001-B.JPG


- 用-O（大写的），后面的url要具体到某个文件，不然抓不下来。还可以用正则来抓取东西

　　[root@xi mytest]# curl -O http://img.voidcn.com/vcimg/000/000/767/511_420_fe4.gif

         运行结果如下：

        % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                                                   Dload  Upload   Total   Spent    Left  Speed
       100  1575  100  1575    0     0  14940      0 --:--:-- --:--:-- --:--:-- 1538k

          会在当前执行目录中生成一张bdlogo.gif的图片。

　　[root@xi mytest]# curl -O http://XXXXX/screen[1-10].JPG  --下载screen1.jpg~screen10.jpg

#### 3、模拟表单信息，模拟登录，保存cookie信息

　　[root@xi mytest]# curl -c ./cookie_c.txt -F log=aaaa -F pwd=******http://www.XXXX.com/wp-login.php

#### 4、模拟表单信息，模拟登录，保存头信息

　　[root@xi mytest]# curl -D ./cookie_D.txt -F log=aaaa -F pwd=******http://www.XXXX.com/wp-login.php

　　-c(小写)产生的cookie和-D里面的cookie是不一样的。

#### 5、使用cookie文件

　　[root@xi mytest]# curl -b ./cookie_c.txt http://www.XXXX.com/wp-admin

#### 6、断点续传，-C(大写)

　　[root@xi mytest]# curl -C -O http://img.voidcn.com/vcimg/000/000/767/511_420_fe4.gif

#### 7、传送数据,最好用登录页面测试，因为你传值过去后，curl回抓数据，你可以看到你传值有没有成功

　　[root@xi mytest]# curl -d log=aaaa http://www.XXXX.com/wp-login.php

#### 8、显示抓取错误，下面这个例子，很清楚的表明了。

　　[root@xi mytest]# curl -fhttp://www.XXXX.com/asdf

　　curl: (22) The requested URL returned error: 404

　　[root@xi mytest]# curlhttp://www.XXXX.com/asdf

　　<HTML><HEAD><TITLE>404,not found</TITLE>

#### 9、伪造来源地址，有的网站会判断，请求来源地址，防止盗链。

　　[root@xi mytest]# curl -ehttp://localhosthttp://www.XXXX.com/wp-login.php

#### 10、当我们经常用curl去搞人家东西的时候，人家会把你的IP给屏蔽掉的,这个时候,我们可以用代理

　　[root@xi mytest]# curl -x 24.10.28.84:32779 -o home.htmlhttp://www.XXXX.com

#### 11、比较大的东西，我们可以分段下载

　　[root@xi mytest]# curl -r 0-100 -o img.part1http://www.XXXX.com/wp-content/uploads/2010/09/compare_varnish.jpg

　　% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

　　Dload  Upload   Total   Spent    Left  Speed

　　100   101  100   101    0     0    105      0 --:--:-- --:--:-- --:--:--     0

　　[root@xi mytest]# curl -r 100-200 -o img.part2http://www.XXXX.com/wp-ontent/uploads/2010/09/compare_varnish.jpg

　　% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

　　Dload  Upload   Total   Spent    Left  Speed

　　100   101  100   101    0     0     57      0  0:00:01  0:00:01 --:--:--     0

　　[root@xi mytest]# curl -r 200- -o img.part3http://www.XXXX.com/wp-content/uploads/2010/09/compare_varnish.jpg

　　% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

　　Dload  Upload   Total   Spent    Left  Speed

　　100  104k  100  104k    0     0  52793      0  0:00:02  0:00:02 --:--:-- 88961

　　[root@xi mytest]# ls |grep part | xargs du -sh

　　4.0K    one.part1

　　112K    three.part3

　　4.0K    two.part2

　　用的时候，把他们cat一下就OK,cat img.part* >img.jpg

#### 12、不会显示下载进度信息

　　[root@xi mytest]# curl -s -o aaa.jpg http://img.voidcn.com/vcimg/000/000/767/511_420_fe4.gif

#### 13、显示下载进度条

　　[root@xi mytest]# curl  -0 http://img.voidcn.com/vcimg/000/000/767/511_420_fe4.gif     (以http1.0协议请求)

####################################################################### 100.0%

#### 14、通过ftp下载文件

　　[xifj@Xi ~]$ curl -u用户名:密码 -Ohttp://www.XXXX.com/demo/curtain/bbstudy_files/style.css

　　% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

　　Dload  Upload   Total   Spent    Left  Speed

　　101  1934  101  1934    0     0   3184      0 --:--:-- --:--:-- --:--:--  7136

　　[xifj@Xi ~]$ curl -u 用户名:密码 -O http://www.XXXX.com/demo/curtain/bbstudy_files/style.css

　　% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

　　Dload  Upload   Total   Spent    Left  Speed

　　101  1934  101  1934    0     0   3184      0 --:--:-- --:--:-- --:--:--  7136

　　或者用下面的方式

　　[xifj@Xi ~]$ curl -O ftp://用户名:密码@ip:port/demo/curtain/bbstudy_files/style.css

　　[xifj@Xi ~]$ curl -O ftp://用户名:密码@ip:port/demo/curtain/bbstudy_files/style.css

　　15，通过ftp上传

　　[xifj@Xi ~]$ curl -T test.sql ftp://用户名:密码@ip:port/demo/curtain/bbstudy_files/

　　[xifj@Xi ~]$ curl -T test.sql ftp://用户名:密码@ip:port/demo/curtain/bbstudy_files/

#### 15、模拟浏览器头

　　[xifj@Xi ~]$ curl -A "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)" -x 123.45.67.89:1080 -o page.html -D cookie0001.txthttp://www.www.baidu.com