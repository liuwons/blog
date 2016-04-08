---
layout: post
title: nginx实现请求转发
date: 2016-04-08 18:49:02
categories: tech
tags: nginx
---

# nginx实现请求转发

反向代理适用于很多场合，负载均衡是最普遍的用法。

[**nginx**](http://nginx.org/) 作为目前最流行的web服务器之一，可以很方便地实现反向代理。

**nginx** 反向代理官方文档: [NGINX REVERSE PROXY](https://www.nginx.com/resources/admin-guide/reverse-proxy/)

当在一台主机上部署了多个不同的web服务器，并且需要能在80端口同时访问这些web服务器时，可以使用 **nginx** 的反向代理功能: 用 **nginx** 在80端口监听所有请求，并依据转发规则(比较常见的是以 URI 来转发)转发到对应的web服务器上。

例如有 ***webmail*** , ***webcom*** 以及 ***webdefault*** 三个服务器分别运行在 *portmail* , *portcom* , *portdefault* 端口，要实现从80端口同时访问这三个web服务器，则可以在80端口运行 **nginx**， 然后将 `/mail` 下的请求转发到 ***webmail*** 服务器， 将 `/com`下的请求转发到 ***webcom*** 服务器， 将其他所有请求转发到 ***webdefault*** 服务器。

假设服务器域名为example.com，则对应的 **nginx** http配置如下：

```conf
http {
    server {
            server_name example.com;

            location /mail/ {
                    proxy_pass http://example.com:protmail/;
            }

            location /com/ {
                    proxy_pass http://example.com:portcom/main/;
            }

            location / {
                    proxy_pass http://example.com:portdefault;
            }
    }
}
```

以上的配置会按以下规则转发请求( `GET` 和 `POST` 请求都会转发):

  - 将 `http://example.com/mail/` 下的请求转发到 `http://example.com:portmail/`
  - 将 `http://example.com/com/` 下的请求转发到 `http://example.com:portcom/main/`
  - 将其它所有请求转发到 `http://example.com:portdefault/`

需要注意的是，在以上的配置中，***webdefault*** 的代理服务器设置是没有指定URI的，而 ***webmail*** 和 ***webcom*** 的代理服务器设置是指定了URI的(分别为 `/` 和 `/main/`)。
如果代理服务器地址中是带有URI的，此URI会替换掉 `location` 所匹配的URI部分。
而如果代理服务器地址中是不带有URI的，则会用完整的请求URL来转发到代理服务器。

官方文档描述：

>If the URI is specified along with the address, it replaces the part of the request URI that matches the location parameter.
>If the address is specified without a URI, or it is not possible to determine the part of URI to be replaced, the full request URI is passed (possibly, modified).

以上配置的转发示例：
  - `http://example.com/mail/index.html` -> `http://example.com:portmail/index.html`
  - `http://example.com/com/index.html` -> `http://example.com:portcom/main/index.html`
  - `http://example.com/mail/static/a.jpg` -> `http://example.com:portmail/static/a.jpg`
  - `http://example.com/com/static/b.css` -> `http://example.com:portcom/main/static/b.css`
  - `http://example.com/other/index.htm` -> `http://example.com:portdefault/other/index.htm`
