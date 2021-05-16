---
title: "使用acme.sh获取Let's Encrypt泛域名证书"
date: "2018-09-16 20:56:52"
---

## 前言

`Let's Encrypt`提供免费的`SSL`证书，官方现已支持泛域名证书的签发。

`Let's Encrypt`官方提供`certbot`工具用于证书的签发，本文介绍的是另一个工具`acme.sh`。

## 安装 acme.sh

```
curl https://get.acme.sh | sh
```

## 设置 DNS API

这里以`Cloudflare`为例，其他`DNS`服务商与之类似

```
export CF_Key="sdfsdfsdfljlbjkljlkjsdfoiwje"
export CF_Email="xxxx@sss.com"
```

## 签发证书

```
acme.sh --issue --dns_cf -d vermouthx.com -d '*.vermouthx.com'
```

## 安装证书

`acme.sh`签发的证书存放在`~/.acme.sh/`下，但是我们不能直接使用证书，在使用之前还必须安装证书。

```
acme.sh --install-cert -d vermouthx.com \
--ca-file        /path/to/keyfile/in/nginx/ca.pem \
--key-file       /path/to/keyfile/in/nginx/key.pem \
--fullchain-file /path/to/fullchain/nginx/cert.pem
```

## 配置 Nginx

下面是一份示例配置

```nginx
server {
    listen 80;
    server_name vermouthx.com;
    # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
    location / {
        return 301 https://$host$request_uri;
    }
}
server {
    listen 443 ssl;
    server_name vermouthx.com;

    access_log /var/log/nginx/blog-access.log;
    error_log /var/log/nginx/blog-error.log;

    root /var/www/hexo;

    location / {
        index index.html;
    }

    # certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
    ssl_certificate /etc/nginx/ssl/vermouthx.com/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/vermouthx.com/key.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
    # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;
    # intermediate configuration. tweak to your needs.
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;
    # HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
    add_header Strict-Transport-Security max-age=15768000;
    # OCSP Stapling ---
    # fetch OCSP records from URL in ssl_certificate and cache them
    ssl_stapling on;
    ssl_stapling_verify on;
    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /etc/nginx/ssl/vermouthx.com/ca.pem;
    resolver 8.8.8.8;
}
```
