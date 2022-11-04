---
title: Google Fonts 反向代理
categories:
- DevOps
tags:
- Google
- 反向代理
date: 2022-02-28T13:00:39+08:00
year: 2022
week: 9
updated: 2022-02-28T13:00:39+08:00
---

解决需求痛点：

博客需要谷歌开源字体， 访问有时不太稳定， 加载速度慢甚至加载不出来。

国内镜像服务有些不提供 https，有些服务也不稳定

<!-- more -->

* 方法
  * 通过 Nginx 反向代理 fonts.google.com
* 环境
  * CentOS 7

``` shell
curl 'https://fonts.haowei.ch/css?family=Galdeano:300,300italic,400,400italic,700,700italic%7CLong+Cang:300,300italic,400,400italic,700,700italic&display=swap&subset=latin,latin-ext' \
  -H 'sec-ch-ua: " Not A;Brand";v="99", "Chromium";v="98", "Google Chrome";v="98"' \
  -H 'DNT: 1' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.109 Safari/537.36' \
  -H 'sec-ch-ua-platform: "macOS"' \
  --compressed
```

Nginx 关键配置

```
        proxy_pass https://fonts.googleapis.com;
        proxy_buffering off;
        proxy_cookie_domain fonts.googleapis.com fonts.haowei.ch;
        proxy_redirect https://fonts.googleapis.com/ /;
        proxy_set_header X-Real_IP $remote_addr;
        proxy_set_header User-Agent $http_user_agent;
        proxy_set_header Accept-Encoding '';
        proxy_set_header referer "https://fonts.googleapis.com$request_uri";
```