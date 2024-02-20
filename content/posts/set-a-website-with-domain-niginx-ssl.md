+++
authors = ["Jun Wu"]
title = "Set a websit with domain+nginx+ssl"
date = "2019-11-02"
description = "Set a websit with domain+nginx+ssl"
tags = [
    "linux",
    "https",
    "ssl",
    "nginx",
    "website"
]
categories = [
    "beckend",
    "devops",
]
series = ["tech"]
+++

:::info{title="What will you get after reading this article"}
1. Use Cloudflare to register a domain name and configure Cloudflare domain name resolution
2. Installation and use of acme.sh
3. nginx binds free HTTPS certificate
:::

<!-- more -->

## The domain name of the website

Cloudflare is the first choice abroad, Tencent and Alibaba Cloud are both available in China. The price difference is not big. However, if you want to bind a domestic domain name to a server you bought in China, you need to register the domain name. This is likely to block many first-time users. Scholar's enthusiasm, so I personally recommend purchasing it on Cloudflare, because Cloudflare's service will also be used for subsequent dns domain name resolution.


![image.png](https://blog.shouyicheng.com/static/img/4a294eaf0766fda95309f3a28d76714f.4a294eaf0766fda95309f3a28d76714f.image.png)

### DNS resolution of domain name

Pay attention to the navigation on the left. You need to select the domain name you registered yourself, and then enter the DNS submenu on the left.

![image.png](https://blog.shouyicheng.com/static/img/8c6164c13f71fed924e18e1e0a52b1e0.8c6164c13f71fed924e18e1e0a52b1e0.image.png)

This article will need to reference the domain name you registered later. Suppose

- Your domain name is `xyz.com`
- Your server IP is `1.2.3.4`

Then you need to create a new record with Type = A as shown in the example picture, fill in `@` for Name and `1.2.3.4` for Content.
Also remember to turn off Proxy and do the same as the example picture. This is a function provided by Cloudflare to help you act as a proxy for free, because we need to bind our own certificates ourselves, and we need to complete the direct connection first, and then enable this advanced feature of Cloudflare.

## HTTPS certificate application and binding

Because the HTTPS certificate is a communication mechanism based on trust, security is the first priority. If you want to get a certificate that is legal and recognized by others, then first you have to prove the IP of the server you currently need to bind the domain name to and the domain name you are applying for. It is bound one to one

:::warning{title="Note"}
A free certificate does not mean it is a fake certificate
:::

### Get Cloudflare API Token

> Why does acme.sh need it?

The acme.sh script essentially verifies whether the domain name is registered under your name by calling the API of your domain name resolution vendor, and has been bound to your server for resolution, which means you need to verify the above mentioned** first. trust**

The first step is to click on the pop-up menu in the upper right corner and select: `My Profile`, then click on the submenu `API Tokens`

![image.png](https://blog.shouyicheng.com/static/img/8e3ec12d9bf5cb02d6070ca5172ec91a.8e3ec12d9bf5cb02d6070ca5172ec91a.image.png)

In the second step, click the big blue button: `Create Token`


![image.png](https://blog.shouyicheng.com/static/img/92959284ed6cd65e3dd7ee6753383004.92959284ed6cd65e3dd7ee6753383004.image.png)

Then select the blue button `Use template` corresponding to the first `Edit zond DNS`

![image.png](https://blog.shouyicheng.com/static/img/eedefc71d2d63b052dc50df31f69d581.eedefc71d2d63b052dc50df31f69d581.image.png)

Make your selections as follows, and then click the blue button at the bottom `Continue to summary`

![image.png](https://blog.shouyicheng.com/static/img/6dd6acd29d480a8c469acb0935bb1042.6dd6acd29d480a8c469acb0935bb1042.image.png)

The last page is the confirmation box, click `Create Token` (I won’t take a screenshot of this one, it’s too simple)

After the generation is completed, he will give you a large string of strings, copy it to your computer, and save it temporarily. I assume the string you applied for is `abcdefg1234567`

Next, you need to find two important IDs of the domain name you need to operate.

### Zone ID and Account ID

Click to return to the homepage, find your domain name, click in, and you will enter the domain name management interface.

![image.png](https://blog.shouyicheng.com/static/img/2e8d9577e7ea40e7daa8fd24c99de34b.2e8d9577e7ea40e7daa8fd24c99de34b.image.png)

I assume your Zone ID is `z123` and your Account ID is `a456`

All the materials are ready, we need to enter our server to operate.

### Use acme.sh to automatically obtain the certificate

For more detailed information about acme.sh, please click on the official github [https://github.com/acmesh-official/acme.sh](https://github.com/acmesh-official/acme.sh)


:::warning{title="Please use root privileges"}
Try to use root when logging in via ssh. Some vps do not allow root users to log in by default, but the default user they assign to you has root permissions, so you need to execute sudo -s to become the root user.
:::
```
$ ssh root@1.2.3.4 #It doesn’t matter if you are not a root user, as long as you can obtain sudo permissions sudo -s
```

Then install acme.sh

```
$ curl https://get.acme.sh | sh -s email=yourname@example.com # It is recommended to use the email address you registered with Cloudflare.
```

Then you will see the following output

```
   % Total % Received % Xferd Average Speed Time Time Time Current
                                  Dload Upload Total Spent Left Speed
100 1032 0 1032 0 0 1465 0 --:--:-- --:--:-- --:--:-- 1467
   % Total % Received % Xferd Average Speed Time Time Time Current
                                  Dload Upload Total Spent Left Speed
100 212k 100 212k 0 0 212k 0 0:00:01 0:00:01 --:--:-- 221k
[Sun Sep 25 19:10:03 CST 2022] Installing from online archive.
[Sun Sep 25 19:10:03 CST 2022] Downloading https://github.com/acmesh-official/acme.sh/archive/master.tar.gz
[Sun Sep 25 19:10:09 CST 2022] Extracting master.tar.gz
[Sun Sep 25 19:10:10 CST 2022] It is recommended to install socat first.
[Sun Sep 25 19:10:10 CST 2022] We use socat for standalone server if you use standalone mode.
[Sun Sep 25 19:10:10 CST 2022] If you don't use standalone mode, just ignore this warning.
[Sun Sep 25 19:10:10 CST 2022] Installing to /root/.acme.sh
[Sun Sep 25 19:10:10 CST 2022] Installed to /root/.acme.sh/acme.sh
[Sun Sep 25 19:10:11 CST 2022] Installing alias to '/root/.bashrc'
[Sun Sep 25 19:10:11 CST 2022] OK, Close and reopen your terminal to start using acme.sh
[Sun Sep 25 19:10:13 CST 2022] Installing cron job
no crontab for root
no crontab for root
[Sun Sep 25 19:10:15 CST 2022] Good, bash is found, so change the shebang to use bash as preferred.
[Sun Sep 25 19:10:27 CST 2022] OK
[Sun Sep 25 19:10:27 CST 2022] Install success!
```

View current file directory

```
$ls -a
```
Make sure the current directory contains a folder called `.acem.sh`.

### Write the three important fields obtained from Cloudflare in the previous step into the environment variables of the server

```
$ export CF_Token="abcdefg1234567"
$ export CF_Account_ID="a456"
$ export CF_Zone_ID="z123"
```

:::note{title="Note"}
Note, please replace the above string with the content you obtained.
:::

Execute the following command to obtain the certificate. This is a more advanced pan-domain name certificate, which means that your subdomain names such as `blog.xyz.com` can be trusted. For example, the domain name of your current blog also uses this kind of certificate.

```
$ acme.sh --issue -d xyz.com -d '*.xyz.com' --dns dns_cf
```

:::note{title="acme.sh's default certificate provider is ZeroSSL"}
If you still want to use letsencrypt, you can add `--server letsencrypt` at the end
```
acme.sh --issue -d shouyicheng.com -d '*.shouyicheng.com' --dns dns_cf --server letsencrypt
```
:::

After success, the address where the certificate is stored on your server will be printed:

```
[Sun Sep 25 20:18:58 CST 2022] Your cert is in: /root/.acme.sh/xyz.com/xyz.com.cer
[Sun Sep 25 20:18:58 CST 2022] Your cert key is
in: /root/.acme.sh/xyz.com/xyz.com.key
[Sun Sep 25 20:18:58 CST 2022] The intermediate CA cert is in: /root/.acme.sh/xyz.com/ca.cer
[Sun Sep 25 20:18:58 CST 2022] And the full chain certs is there: /root/.acme.sh/xyz.com/fullchain.cer
```

### Use HTTPS certificate

#### Install nginx

There are too many software that can be used as proxies like nginx. Caddy and Traefik both do very well. If you are familiar with the configuration of the certificate under each proxy software, you don’t need to read it later.

```
$ sudo apt install nginx
```

Then you use your browser to open your domain name http://xyz.com or directly use IP to access http://1.2.3.4, you should be able to see it.


![image.png](https://blog.shouyicheng.com/static/img/2e81cac041dad7197a65cdbc0d970631.2e81cac041dad7197a65cdbc0d970631.image.png)

Now let’s configure nginx

:::warning{title="Note"}
All subsequent operations require root privileges
:::

Enter `/etc/nginx`,

```
$ cd /etc/nginx
```

Then modify the `/etc/nginx/sites-available/default` file

>(Vim or nano can be used, but there is an artifact called [VSCode - Remote development over SSH](https://code.visualstudio.com/docs/remote/ssh-tutorial). I will introduce this in a separate article later.


Change it to the following:

```
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
listen 80 default_server;
listen [::]:80 default_server;

# SSL configuration
#
listen 443 ssl default_server;
listen [::]:443 ssl default_server;
#
ssl_certificate /root/.acme.sh/xyz.com/fullchain.cer;
ssl_certificate_key /root/.acme.sh/xyz.com/xyz.com.key;
# Note: You should disable gzip for SSL traffic.
# See: https://bugs.debian.org/773332
#
# Read up on ssl_ciphers to ensure a secure configuration.
# See: https://bugs.debian.org/765782
#
# Self signed certs generated by the ssl-cert package
# Don't use them in a production server!
#
# include snippets/snakeoil.conf;

root /var/www/html;

# Add index.php to the list if you are using PHP
index index.html index.htm index.nginx-debian.html;

server_name _;

location/{
# First attempt to serve request as file, then
# as directory, then fall back to displaying a 404.
try_files $uri $uri/ =404;
}

# pass PHP scripts to FastCGI server
#
#location ~ \.php$ {
# include snippets/fastcgi-php.conf;
#
# # With php-fpm (or other unix sockets):
# fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
# # With php-cgi (or other tcp sockets):
# fastcgi_pass 127.0.0.1:9000;
#}

# deny access to .htaccess files, if Apache's document root
# concurs with nginx's one
#
#location ~ /\.ht {
# deny all;
#}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
# listen 80;
# listen [::]:80;
#
# server_name example.com;
#
# root /var/www/example.com;
# index index.html;
#
# location / {
# try_files $uri $uri/ =404;
# }
#}
```

Then need to execute

```
$ sudo nginx -s reload
```
Let nginx refresh the configuration.

Then access it again through https://xyz.com. Congratulations, your website has been successfully installed and the HTTPS certificate is enabled.


![image.png](https://blog.shouyicheng.com/static/img/9481d5261ddfa46e6c8db3ac1c83857b.9481d5261ddfa46e6c8db3ac1c83857b.image.png)

## Let the certificate automatically renew and refresh automatically

The official acme.sh also said that the certificate in the folder `/root/.acme.sh/xyz.com/` should not be used directly. Because it is not guaranteed, no version will be placed here, so we have to Set it up a little further.

### Use acme.sh to install the certificate to the specified location

Create the following file `/certs/xyz.com` in the `/root/` directory

```
$ cd /root
$ mkdir -p certs/xyz.com/
```

Then install the certificate into this folder

```
$ acme.sh --install-cert -d example.com \
--key-file /root/certs/xyz.com/key.pem \
--fullchain-file /root/certs/xyz.com/cert.pem \
--reloadcmd "sudo nginx -s reload"
```

Because the certificate is stored in a new location, we need to return to nginx to modify the certificate location and find the following configuration

```
ssl_certificate /root/.acme.sh/xyz.com/fullchain.cer;
ssl_certificate_key /root/.acme.sh/xyz.com/xyz.com.key;
```
change it to

```
ssl_certificate /root/certs/xyz.com/cert.pem;
ssl_certificate_key /root/certs/xyz.com/key.pem;
```

Then reload the nginx configuration

```
$sudo nginx -s reload
```
