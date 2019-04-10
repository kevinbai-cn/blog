---
title: 简单几步让你的网站支持 HTTPS
date: 2018-11-01 17:03:18
categories:
- Web
tags:
- Nginx
- HTTPS
---

最近在玩小程序，调用自己的 API 的时候需要支持 HTTPS。在网上试了一些方法，这里分享一个简单实用的供大家使用。

证书提供商有很多，这里我使用的是 Let’s Encrypt 家的免费 SSL 证书。官方推荐使用 Certbot 工具生成，本文教程也基于此。

<!-- more -->

# 1 Nginx 安装与配置

为了能描述的更清楚一些，教程从 Nginx 的使用开始。

**注意：当前系统为 Ubuntu 16.04**

安装 Nginx

```
# apt-get update
# apt-get install nginx
```

备份配置文件

```
# cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
```

编辑配置文件 `/etc/nginx/sites-available/default` 内容为

```
server {
    listen 80;
    server_name test.kevinbai.com;

    location / {
        default_type text/html;
        return 200 'Hello World!';
    }
}
```

保存后启动 Nginx

```
# /etc/init.d/nginx start
```

浏览器访问 `http://test.kevinbai.com`，看到如下内容

```
Hello World!
```

Nginx 配置成功。

**注意：Nginx 中的域名配置前记得去域名服务商添加一条 A 记录，将域名解析到目标 IP**

# 2 证书生成与配置

更新源后安装 `software-properties-common`，这个主要是提供一些便携工具，比如添加软件源工具，一行命令就能添加源，不用自己手动去编辑相应的配置文件。

```
# apt-get update
# apt-get install software-properties-common
```

添加 Certbot 的源并安装

```
# add-apt-repository ppa:certbot/certbot
# apt-get update
# apt-get install python-certbot-nginx
```

生成证书

```
# certbot --nginx -d test.kevinbai.com
```

将上面的 `test.kevinbai.com` 换成你的域名即可。

生成过程中会让你填写必要的信息，输入你的邮箱

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): kevinbai.cn@gmail.com
```

同意协议

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel:A
```

是否愿意 Let's Encrypt 给你的邮箱推送一些消息，这个你自己随意选择

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
```

选择是否自动修改 Nginx 的一些配置，以支持 HTTPS，1 为否，2 为是。不熟悉相关配置的可以选择 2

```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):2
```

最后如果看到类似下面的输出，则说明证书已经生成并配置成功

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/test.kevinbai.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/test.kevinbai.com/privkey.pem
   Your cert will expire on 2019-01-29. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

此时，Nginx 的配置文件内容已经被修改成这样了

```
server {
    server_name test.kevinbai.com;

    location / {
        default_type text/html;
        return 200 'Hello World!';
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/test.kevinbai.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/test.kevinbai.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}

server {
    if ($host = test.kevinbai.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name test.kevinbai.com;
    return 404; # managed by Certbot
}
```

我们访问 `http://test.kevinbai.com`，会发现域名被重定向到了 HTTPS，并且能看到如下内容

```
Hello World!
```

说明网站已经成功支持 HTTPS。如果未生效，试着重启一下 Nginx，若还是不行，参考第 3 节的内容。

如果你不想 Certbot 自动给你配置，你可以使用 certonly 选项，类似这样

```
# certbot --nginx certonly -d test.kevinbai.com
```

# 3 可能会碰到的问题

## 3.1 Nginx 安装并配置后无法访问

重启一下 Nginx

```
# /etc/init.d/nginx restart
```

如果还不行，可能是 80 端口未开放，打开就好。

## 3.2 Another instance of Certbot is already running

安装过程中，不小心中断，再次执行下面的命令

```
# certbot --nginx -d test.kevinbai.com
```

可能会出现这样的问题

```
# Another instance of Certbot is already running
```

执行如下命令找到 `.certbot.lock` 文件

```
# find / -type f -name ".certbot.lock"
```

将其删除

```
# find / -type f -name ".certbot.lock" -exec rm {} \;
```

然后重试即可。


## 3.3 证书生成并且配置成功，访问域名超时

这个可能是 HTTPS 的 443 端口被禁引起的，打开就可以了。

# 4 自动延长免费期限

Let's Encrypt 的证书有效期只有 90 天，到期后我们需要重新生成。

不过我们不需要手动去生成，上面的生成命令在生成证书后，已经为我们配置了定时任务。查看 `/etc/cron.d/certbot`，主要内容如下

```
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(43200))' && certbot -q renew
```

表面上每隔 12 个小时会重建一次证书，事实上只有当证书的有效期只有 30 天的时候，重建才真正生效。

这里我们只要确认下重建命令没有问题就可以了，执行一下命令

```
# certbot renew --dry-run
```

如果执行过程中没有报错的话，也就没有什么问题了。

# 5 SSL 状态检测

我们可以使用 MySSL 检测证书详细信息、证书链详细信息、当前支持协议、加密套件详细信息等，地址如下

```
https://myssl.com
```

访问后，输入你的域名即可测试。

这里我们可以看到支持的 SSL 版本，类似这样

```
TLS 1.3	不支持	
TLS 1.2	支持	
TLS 1.1	支持	
TLS 1.0	支持	
SSL 3	不支持	
SSL 2	不支持
```

有些应用场景下对 SSL 版本有要求，比如小程序的 API 等，我们就可以从这确定我们的 HTTPS 是否满足需求。当然，还有其它的一些信息，有需要的时候查看就好。

除了 MySSL 外，下面的两个在线工具也比较常用

```
https://www.ssllabs.com/ssltest
https://www.htbridge.com/ssl
```

# 6 参考

```
https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx
https://www.f2ecoder.net/477.html
```
