# GitLab启用HTTPS

## 启用HTTPS 

### 警告 

Nginx配置会告诉浏览器和客户端，只需在未来24个月通过安全连接与您的GitLab实例进行通信。通过启用HTTPS，您需要至少在24个月内为您的实例提供安全连接。

默认情况下，omnibus-gitlab不使用HTTPS。如果要为gitlab.example.com启用HTTPS，请将以下语句添加到`/etc/gitlab/gitlab.rb`：

```
# note the ‘https‘ below
external_url "https://gitlab.example.com"

```

因为我们的示例中的主机名是“gitlab.example.com”，所以omnibus-gitlab将分别查找名为“ `/etc/gitlab/ssl/gitlab.example.com.key`和”的 密钥和证书文件 `/etc/gitlab/ssl/gitlab.example.com.crt`。创建`/etc/gitlab/ssl`目录并在那里复制您的密钥和证书。

```
sudo mkdir -p /etc/gitlab/ssl
sudo chmod 700 /etc/gitlab/ssl
sudo cp gitlab.example.com.key gitlab.example.com.crt /etc/gitlab/ssl/

```

现在运行`sudo gitlab-ctl reconfigure`。当重新配置完成后，您的GitLab实例应该可以访问`https://gitlab.example.com`。

如果您使用防火墙，您可能必须打开端口443以允许入站HTTPS流量。

```
# UFW example (Debian, Ubuntu)
sudo ufw allow https

# lokkit example (RedHat, CentOS 6)
sudo lokkit -s https

# firewall-cmd (RedHat, Centos 7)
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld

```

## 重定向`HTTP`请求`HTTPS` 

默认情况下，当您以“https”开头指定一个external_url时，Nginx将不再侦听端口80上未加密的HTTP流量。如果要将所有HTTP流量重定向到HTTPS，您可以使用该`redirect_http_to_https` 设置。

```
external_url "https://gitlab.example.com"
nginx['redirect_http_to_https'] = true

```

## 更改默认端口和SSL证书位置 

如果您需要使用除默认端口（443）之外的HTTPS端口，请将其指定为external_url的一部分。

```
external_url "https://gitlab.example.com:2443"

```

要设置SSL证书的位置创建`/etc/gitlab/ssl`目录，将`.crt`与`.key`目录中的文件，并指定以下配置：

```
# For GitLab
nginx[‘ssl_certificate‘] = "/etc/gitlab/ssl/gitlab.example.com.crt"
nginx[‘ssl_certificate_key‘] = "/etc/gitlab/ssl/gitlab.example.com.key"

```

运行`sudo gitlab-ctl reconfigure`使更改生效。

## 更改默认代理标头 

默认情况下，当指定`external_url`omn??ibus-gitlab将会设置几个在大多数环境中被认为是完美的NGINX代理头。

例如，omnibus-gitlab将设置：

```
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl" => "on"

```

如果您已经指定了`https`模式`external_url`。

不过，如果你遇到这样的情况你GitLab是一个更复杂的设置像后面的反向代理，您将需要调整代理头，以避免类似的错误`The change you wanted was rejected`或 `Can‘t verify CSRF token authenticity Completed 422 Unprocessable`。

这可以通过覆盖默认标头来实现，例如。指定在`/etc/gitlab/gitlab.rb`：

```
 nginx[‘proxy_set_headers‘] = {
  "X-Forwarded-Proto" => "http",
  "CUSTOM_HEADER" => "VALUE"
 }

```

保存文件并[重新配置GitLab](https://docs.gitlab.com/ce/administration/restart_gitlab.html#omnibus-gitlab-reconfigure) ，使更改生效。

这样，您可以指定您需要的NGINX支持的任何头。

## 配置GitLab `trusted_proxies`和NGINX `real_ip`模块

默认情况下，NGINX和GitLab将记录连接的客户端的IP地址。

如果您的GitLab位于反向代理之后，您可能不希望代理的IP地址显示为客户端地址。

您可以通过将反向代理添加到`real_ip_trusted_addresses`列表中，使NGINX可以查找不同的地址：

```
# Each address is added to the the NGINX config as ‘set_real_ip_from <address>;‘
nginx[‘real_ip_trusted_addresses‘] = [ ‘192.168.1.0/24‘, ‘192.168.2.1‘, ‘2001:0db8::/32‘ ]
# other real_ip config options
nginx[‘real_ip_header‘] = ‘X-Real-IP‘
nginx[‘real_ip_recursive‘] = ‘on‘

```

选项说明：

- <http://nginx.org/en/docs/http/ngx_http_realip_module.html>

默认情况下，omnibus-gitlab将使用IP地址`real_ip_trusted_addresses` 作为GitLab的信任代理，这将阻止用户从这些IP中登录。

保存文件并[重新配置GitLab](https://docs.gitlab.com/ce/administration/restart_gitlab.html#omnibus-gitlab-reconfigure) ，使更改生效。

## 配置HTTP2协议 

默认情况下，当您指定您的Gitlab实例应通过HTTPS指定时`external_url "https://gitlab.example.com"`， [http2协议](https://tools.ietf.org/html/rfc7540)也被启用。

omn??ibus-gitlab软件包设置与http2协议兼容的所需ssl_ciphers。

如果您在配置中指定自定义ssl_ciphers，并且密码位于[http2密码黑名单中](https://tools.ietf.org/html/rfc7540#appendix-A)，一旦您尝试访问GitLab实例`INADEQUATE_SECURITY`，您的浏览器将显示错误。

考虑从密码列表中删除违规密码。只有使用非常具体的自定义设置，才需要更改密码。

有关为什么要启用http2协议的更多信息，请查看[http2白皮书](https://assets.wp.nginx.com/wp-content/uploads/2015/09/NGINX_HTTP2_White_Paper_v4.pdf?_ga=1.127086286.212780517.1454411744)。

如果更改密码不是一个选项，可以通过指定在`/etc/gitlab/gitlab.rb`http：

```
nginx[‘http2_enabled‘] = false

```

保存文件并[重新配置GitLab](https://docs.gitlab.com/ce/administration/restart_gitlab.html#omnibus-gitlab-reconfigure) ，使更改生效。

## 使用非捆绑的Web服务器 

默认情况下，omnibus-gitlab使用捆绑的Nginx安装GitLab。Omnibus-gitlab允许网络服务器通过`gitlab-www`位于同一组名的用户进行访问。要允许外部Web服务器访问GitLab，外部Web服务器用户需要添加`gitlab-www`组。

要使用其他Web服务器（如Apache）或现有的Nginx安装，您必须执行以下步骤：

1. 禁用捆绑的Nginx

   在`/etc/gitlab/gitlab.rb`集合：

   ```
   nginx[‘enable‘] = false

   ```

2. 设置非捆绑的Web服务器用户的用户名

   默认情况下，omnibus-gitlab没有外部Web服务器用户的默认设置，您必须在配置中指定它。对于Debian / Ubuntu，默认用户是`www-data`Apache / Nginx，而对于RHEL / CentOS，Nginx用户是`nginx`。

   *注意：确保先安装了Apache / Nginx，以便创建网络服务器用户，否则在重新配置时omnibus将失败。*

   比方说，网络服务器用户是`www-data`。在`/etc/gitlab/gitlab.rb`集合：

   ```
   web_server[‘external_users‘] = [‘www-data‘]

   ```

   *注意：此设置是一个数组，因此您可以指定多个用户添加到gitlab-www组。*

   运行`sudo gitlab-ctl reconfigure`使更改生效。

   *注意：如果您使用SELinux，并且您的Web服务器运行在受限制的SELinux配置文件下，您可能必须松开Web服务器上的限制。*

   *注意：确保网络服务器用户对外部Web服务器使用的所有目录具有正确的权限，否则您将收到`failed (XX: Permission denied) while reading upstream`错误。

3. 将非捆绑的Web服务器添加到可信代理列表中

   通常，omnibus-gitlab将可信代理列表默认为捆绑的NGINX的real_ip模块中配置的代理。

   对于非捆绑的Web服务器，列表需要直接配置，如果Web服务器与GitLab不在同一台机器上，则应包含该IP地址。否则用户将被显示为从您的Web服务器的IP地址登录。

   ```
   gitlab_rails[‘trusted_proxies‘] = [ ‘192.168.1.0/24‘, ‘192.168.2.1‘, ‘2001:0db8::/32‘ ]

   ```

4. （可选）如果使用Apache，请设置正确的gitlab-workhorse设置

   *注意：以下值在GitLab 8.2中添加，请确保已安装最新版本。*

   Apache无法连接到UNIX套接字，而需要连接到TCP端口。允许gitlab-workhorse在TCP上侦听（默认端口8181）编辑`/etc/gitlab/gitlab.rb`：

   ```
   gitlab_workhorse[‘listen_network‘] = "tcp"
   gitlab_workhorse[‘listen_addr‘] = "127.0.0.1:8181"

   ```

   运行`sudo gitlab-ctl reconfigure`使更改生效。

5. 下载正确的Web服务器配置

   转到[GitLab配方存储库，](https://gitlab.com/gitlab-org/gitlab-recipes/tree/master/web-server)并在您选择的webserver目录中查找omnibus配置。确保您选择正确的配置文件，具体取决于您是否选择使用SSL提供GitLab。您唯一需要更改的是`YOUR_SERVER_FQDN`使用您自己的FQDN，如果您使用SSL，SSL密钥当前所在的位置。您还可能需要更改日志文件的位置。

## 设置NGINX监听地址或地址 

默认情况下，NGINX将接受所有本地IPv4地址的传入连接。您可以更改地址列表`/etc/gitlab/gitlab.rb`。

```
nginx[‘listen_addresses‘] = ["0.0.0.0", "[::]"] # listen on all IPv4 and IPv6 addresses

```

## 设置NGINX监听端口 

默认情况下，NGINX将侦听指定的端口`external_url`或隐式使用正确的端口（HTTP为80，HTTPS为443）。如果您在反向代理之后运行GitLab，则可能需要将监听端口重写为其他。例如，要使用端口8081：

```
nginx[‘listen_port‘] = 8081

```

## 支持代理SSL 

默认情况下，如果`external_url` 包含，NGINX将自动检测是否使用SSL `https://`。如果您在反向代理之后运行GitLab，则可能希望在另一个代理服务器或负载均衡器上终止SSL。为此，请确保`external_url`包含`https://`并应用以下配置`gitlab.rb`：

```
nginx[‘listen_port‘] = 80
nginx[‘listen_https‘] = false
nginx[‘proxy_set_headers‘] = {
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl" => "on"
}

```

请注意，您可能需要配置反向代理服务器或负载平衡器来转发特定的头文件（例如`Host`，`X-Forwarded-Ssl`，`X-Forwarded-For`，`X-Forwarded-Port`（如果你使用一个和Mattermost）），以GitLab。如果您忘记了此步骤，您可能会看到不正确的重定向或错误（例如“422不可处理实体”，“无法验证CSRF令牌真实性”）。有关详细信息，请参阅：

- <http://stackoverflow.com/questions/16042647/whats-the-de-facto-standard-for-a-reverse-proxy-to-tell-the-backend-ssl-is-used>
- <https://wiki.apache.org/couchdb/Nginx_As_a_Reverse_Proxy>

## 设置HTTP严格的传输安全性 

默认情况下，GitLab启用严格的传输安全性，通知浏览器他们应该只使用HTTPS联系网站。当浏览器访问GitLab实例时，即使用户显式地输入`http://`url，它也会记住不再尝试不安全的连接。这样的URL将被浏览器自动重定向到`https://`变体。

```
nginx[‘hsts_max_age‘] = 31536000
nginx[‘hsts_include_subdomains‘] = false

```

默认`max_age`设置为一年，这是浏览器记住仅通过HTTPS连接的时间。设置`max_age`为0将禁用此功能。有关更多信息，请参阅

- <https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/>

## 使用自定义SSL密码 

默认情况下，GitLab使用的SSL密码是gitlab.com上的测试和GitLab社区贡献的各种最佳做法。

但是，您可以通过添加到更改ssl密码`gitlab.rb`：

```
  nginx[‘ssl_ciphers‘] = "CIPHER:CIPHER1"

```

并运行重新配置。

您还可以启用`ssl_dhparam`指令。

首先，生成`dhparams.pem`用`openssl dhparam -out dhparams.pem 2048`。然后，`gitlab.rb`添加生成文件的路径，例如：

```
  nginx[‘ssl_dhparam‘] = "/etc/gitlab/ssl/dhparams.pem"

```

变更运行后`sudo gitlab-ctl reconfigure`。

## 启用双向SSL客户端身份验证 

要求Web客户端使用受信任的证书进行身份验证，您可以通过添加到`gitlab.rb`以下方式启用双向SSL ：

```
  nginx[‘ssl_verify_client‘] = "on"

```

并运行重新配置。

还可以配置NGINX支持配置SSL客户端身份验证的其他选项：

```
  nginx[‘ssl_client_certificate‘] = "/etc/pki/tls/certs/root-certs.pem"
  nginx[‘ssl_verify_depth‘] = "2"

```

进行更改运行后`sudo gitlab-ctl reconfigure`。

## 将自定义NGINX设置插入到GitLab服务器块中 

请记住，如果您的`gitlab.rb`文件中定义了相同的设置，这些自定义设置可能会产生冲突。

如果您需要将自定义设置添加到`server`GitLab 的NGINX 块中，某些原因可以使用以下设置。

```
# Example: block raw file downloads from a specific repository
nginx[‘custom_gitlab_server_config‘] = "location ^~ /foo-namespace/bar-project/raw/ {\n deny all;\n}\n"

```

运行`gitlab-ctl reconfigure`以重写NGINX配置并重新启动NGINX。

这将定义的字符串插入`server`块 的末尾`/var/opt/gitlab/nginx/conf/gitlab-http.conf`。

### 笔记： 

- 如果您要添加新的位置，则可能需要包含

  ```
  proxy_cache off;
  proxy_pass http://gitlab-workhorse;

  ```

  在字符串或包含的nginx配置中。没有这些，任何子位置都将返回404。参见 [gitlab-ce＃30619](https://gitlab.com/gitlab-org/gitlab-ce/issues/30619)。

- 您不能将根`/`位置或`/assets`位置添加为已存在的位置`gitlab-http.conf`。

## 将自定义设置插入NGINX配置 

如果您需要将自定义设置添加到NGINX配置中，例如要包括现有服务器块，则可以使用以下设置。

```
# Example: include a directory to scan for additional config files
nginx[‘custom_nginx_config‘] = "include /etc/nginx/conf.d/*.conf;"

```

运行`gitlab-ctl reconfigure`以重写NGINX配置并重新启动NGINX。

这将定义的字符串插入`http`块 的末尾`/var/opt/gitlab/nginx/conf/nginx.conf`。

## 自定义错误页面 

您可以使用`custom_error_pages`默认GitLab错误页面修改文本。这可以用于任何有效的HTTP错误代码; 例如404,502。

作为示例，以下内容将修改默认的404错误页面。

```
nginx[‘custom_error_pages‘] = {
  ‘404‘ => {
    ‘title‘ => ‘Example title‘,
    ‘header‘ => ‘Example header‘,
    ‘message‘ => ‘Example message‘
  }
}

```

这将导致下面的404错误页面。

[![imgs](../imgs/error_page_example.png)](https://docs.gitlab.com/omnibus/settings/img/error_page_example.png)

运行`gitlab-ctl reconfigure`以重写NGINX配置并重新启动NGINX。

## 使用现有的Passenger / Nginx安装 

在某些情况下，您可能希望使用现有的Passenger / Nginx安装来托管GitLab，但仍然可以使用omnibus软件包更新和安装。

### 组态 

首先，你需要设置你的`/etc/gitlab/gitlab.rb`Nginx和Unicorn内置的：

```
# Define the external url
external_url ‘http://git.example.com‘

# Disable the built-in nginx
nginx[‘enable‘] = false

# Disable the built-in unicorn
unicorn[‘enable‘] = false

# Set the internal API URL
gitlab_rails[‘internal_api_url‘] = ‘http://git.example.com‘

# Define the web server process user (ubuntu/nginx)
web_server[‘external_users‘] = [‘www-data‘]

```

确保运行`sudo gitlab-ctl reconfigure`更改才能生效。

注意：如果您运行的版本早于8.16.0，则必须手动删除独角兽服务文件（`/opt/gitlab/service/unicorn`）（如果存在）才能重新配置成功。

### Vhost（服务器块） 

然后，在您的自定义Passenger / Nginx安装中，创建以下站点配置文件：

```
upstream gitlab-workhorse {
  server unix://var/opt/gitlab/gitlab-workhorse/socket fail_timeout=0;
}

server {
  listen *:80;
  server_name git.example.com;
  server_tokens off;
  root /opt/gitlab/embedded/service/gitlab-rails/public;

  client_max_body_size 250m;

  access_log  /var/log/gitlab/nginx/gitlab_access.log;
  error_log   /var/log/gitlab/nginx/gitlab_error.log;

  # Ensure Passenger uses the bundled Ruby version
  passenger_ruby /opt/gitlab/embedded/bin/ruby;

  # Correct the $PATH variable to included packaged executables
  passenger_env_var PATH "/opt/gitlab/bin:/opt/gitlab/embedded/bin:/usr/local/bin:/usr/bin:/bin";

  # Make sure Passenger runs as the correct user and group to
  # prevent permission issues
  passenger_user git;
  passenger_group git;

  # Enable Passenger & keep at least one instance running at all times
  passenger_enabled on;
  passenger_min_instances 1;

  location ~ ^/[\w\.-]+/[\w\.-]+/(info/refs|git-upload-pack|git-receive-pack)$ {
    # ‘Error‘ 418 is a hack to re-use the @gitlab-workhorse block
    error_page 418 = @gitlab-workhorse;
    return 418;
  }

  location ~ ^/[\w\.-]+/[\w\.-]+/repository/archive {
    # ‘Error‘ 418 is a hack to re-use the @gitlab-workhorse block
    error_page 418 = @gitlab-workhorse;
    return 418;
  }

  location ~ ^/api/v3/projects/.*/repository/archive {
    # ‘Error‘ 418 is a hack to re-use the @gitlab-workhorse block
    error_page 418 = @gitlab-workhorse;
    return 418;
  }

  # Build artifacts should be submitted to this location
  location ~ ^/[\w\.-]+/[\w\.-]+/builds/download {
      client_max_body_size 0;
      # ‘Error‘ 418 is a hack to re-use the @gitlab-workhorse block
      error_page 418 = @gitlab-workhorse;
      return 418;
  }

  # Build artifacts should be submitted to this location
  location ~ /ci/api/v1/builds/[0-9]+/artifacts {
      client_max_body_size 0;
      # ‘Error‘ 418 is a hack to re-use the @gitlab-workhorse block
      error_page 418 = @gitlab-workhorse;
      return 418;
  }

  # Build artifacts should be submitted to this location
  location ~ /api/v4/jobs/[0-9]+/artifacts {
      client_max_body_size 0;
      # ‘Error‘ 418 is a hack to re-use the @gitlab-workhorse block
      error_page 418 = @gitlab-workhorse;
      return 418;
  }


  # For protocol upgrades from HTTP/1.0 to HTTP/1.1 we need to provide Host header if its missing
  if ($http_host = "") {
  # use one of values defined in server_name
    set $http_host_with_default "git.example.com";
  }

  if ($http_host != "") {
    set $http_host_with_default $http_host;
  }

  location @gitlab-workhorse {

    ## https://github.com/gitlabhq/gitlabhq/issues/694
    ## Some requests take more than 30 seconds.
    proxy_read_timeout      3600;
    proxy_connect_timeout   300;
    proxy_redirect          off;

    # Do not buffer Git HTTP responses
    proxy_buffering off;

    proxy_set_header    Host                $http_host_with_default;
    proxy_set_header    X-Real-IP           $remote_addr;
    proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
    proxy_set_header    X-Forwarded-Proto   $scheme;

    proxy_pass http://gitlab-workhorse;

    ## The following settings only work with NGINX 1.7.11 or newer
    #
    ## Pass chunked request bodies to gitlab-workhorse as-is
    # proxy_request_buffering off;
    # proxy_http_version 1.1;
  }

  ## Enable gzip compression as per rails guide:
  ## http://guides.rubyonrails.org/asset_pipeline.html#gzip-compression
  ## WARNING: If you are using relative urls remove the block below
  ## See config/application.rb under "Relative url support" for the list of
  ## other files that need to be changed for relative url support
  location ~ ^/(assets)/ {
    root /opt/gitlab/embedded/service/gitlab-rails/public;
    gzip_static on; # to serve pre-gzipped version
    expires max;
    add_header Cache-Control public;
  }

  error_page 502 /502.html;
}

```

不要忘记在上面的例子中更新“git.example.com”作为您的服务器URL。

注意：如果您使用403禁止，可能您尚未启用/etc/nginx/nginx.conf中的乘客，只能取消注释：

```
# include /etc/nginx/passenger.conf;

```

那么，‘sudo service nginx reload‘

### 启用/禁用nginx_status 

默认情况下，您将有一个配置为127.0.0.1:8060/nginx_status的nginx健康检查端点来监视您的Nginx服务器状态。

#### 将显示以下信息： 

```
Active connections: 1
server accepts handled requests
 18 18 36
Reading: 0 Writing: 1 Waiting: 0

```

- 活动连接 - 总共打开连接。
- 显示3个数字。
  - 所有接受的连接。
  - 所有处理的连接。
  - 处理的请求总数
- 读取：Nginx读取请求头
- 写入：Nginx读取请求体，处理请求或向客户端写入响应
- 等待：保持连接。该值取决于keepalive-timeout。

## 组态 

编辑`/etc/gitlab/gitlab.rb`：

```
nginx[‘status‘] = {
  "listen_addresses" => ["127.0.0.1"],
  "fqdn" => "dev.example.com",
  "port" => 9999,
  "options" => {
    "stub_status" => "on", # Turn on stats
    "access_log" => "on", # Disable logs for stats
    "allow" => "127.0.0.1", # Only allow access from localhost
    "deny" => "all" # Deny access to anyone else
  }
}

```

如果您没有发现此服务对您当前的基础设施有用，可以通过以下方式禁用它：

```
nginx[‘status‘] = {
  ‘enable‘ => false
}

```

确保运行sudo gitlab-ctl reconfigure以使更改生效。

#### 警告 

为了确保用户上传可以访问，您的Nginx用户（通常`www-data`）应该添加到`gitlab-www`组中。这可以使用以下命令完成：

```
sudo usermod -aG gitlab-www www-data

```

#### 模板 

除了乘客配置代替独角兽和缺少HTTPS（尽管可以启用），这些文件大体上与以下相同：

- [捆绑的Gitlab Nginx配置](https://gitlab.com/gitlab-org/omnibus-gitlab/tree/master/files/gitlab-cookbooks/gitlab/templates/default/nginx-gitlab-http.conf.erb)

不要忘记重新启动Nginx以加载新配置（在基于Debian的系统上`sudo service nginx restart`）。

#### 故障排除 

##### 400错误请求：太多主机头 

确保您没有设置中的`proxy_set_header`配置 `nginx[‘custom_gitlab_server_config‘]`，而是使用文件中的 [“proxy_set_headers”](https://docs.gitlab.com/omnibus/settings/nginx.html#supporting-proxied-ssl)配置`gitlab.rb`。

##### javax.net.ssl.SSLHandshakeException：接收致命警报：handshake_failure 

从GitLab 10开始，默认情况下，omnibus-gitlab软件包不再支持TLSv1协议。当与您的GitLab实例进行交互时，这可能会导致与一些较旧的基于Java的IDE客户端的连接问题。我们强烈要求您升级服务器上的密码，与[用户评论中](https://gitlab.com/gitlab-org/gitlab-ce/issues/624#note_299061)提到的相似。

如果不可能使此服务器更改，您可以通过更改以下内容的值来默认返回到旧的行为`/etc/gitlab/gitlab.rb`：

```
nginx[‘ssl_protocols‘] = "TLSv1 TLSv1.1 TLSv1.2"
```