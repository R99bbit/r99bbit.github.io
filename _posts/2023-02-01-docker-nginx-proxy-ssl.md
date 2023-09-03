---
title: "[Docker] Nginx를 이용해서 SSL 설정하기(HTTPS)"
date: 2023-02-01 00:56:01 +0900
categories: [DOCKER]
tags: [docker, nginx, proxy, https, ssl]
---

## Why?
> 최근에 브라우저와 내 서버 간의 HTTP 통신을 다뤄야 하는 일이 있었다. 이 때 브라우저로 접속한 서버는 HTTPS, 내가 구현한 서버는 HTTP 였기 때문에 당연하게도 Mixed Contents 문제가 발생했다. 문제를 해결하기 위해 내 서버에 SSL 인증을 넣어주기로 했다. 서버에 SSL 인증을 받는 방법에는 여러가지가 있지만, 매번 프레임워크 별 HTTPS 적용 방법을 검색하는 것이 지쳤기 때문에 항상 눈독들여 왔던 "nginx reverse proxy로 ssl 인증 적용하기(무려 docker로!)" 를 시도해보려고 한다.


> +) 금방 구현할 수 있을 줄 알았는데 생각보다 많은 시간이 걸렸다. 관련 글들은 많으나 어딘가 난잡했고, 여기저기서 config 내용을 복붙하다보니 설정이 꼬여서 삽질을 조금 했다.. 이 글을 통해 "docker + nginx + reverse proxy + ssl" 이 필요한 사람들에게 문제를 해결하는 힌트가 되었으면 하는 바램이다.

## Proxy Server?

`proxy server`는 클라이언트와 접속하고자 하는 서버 사이에 중계(proxy) 역할을 수행하는 것을 말한다. 간단하게 말하자면 클라이언트와 서버 통신 사이에 징검다리가 하나 생긴 것이다. 일반적으로 Proxy에는 `Forward Proxy`와 `Reverse Proxy`가 존재한다.

### Forward Proxy

클라이언트가 `proxy server` 에 외부 자산에 접근을 요청하면 `proxy server` 가 클라이언트를 대신하여 인터넷에 접속한 후 결과를 전달해준다. 웹 사이트 취약점 점검시 이용하는 `Burp Suite` 나 `Paros` 등의 도구가 이에 해당한다.

### Reverse Proxy

`Forward Proxy` 에서 `proxy server`가 클라이언트의 역할을 대신 하였다면, `Reverse Proxy` 에서는 서버의 역할을 대신하는 것이라고 할 수 있다. `클라이언트<->프록시서버<->내부서버` 와 같은 구조로 되어있을 때 클라이언트가 `proxy server`에 접근을 요청하면 `proxy server` 가 다시 내부 서버에 데이터를 요청하여 클라이언트에게 결과를 전달해준다.
이렇게 되면 사용자는 내부 서버에 직접 접근할 수 없기 때문에 서버를 더 안전하게 운영할 수 있다. 또한, 사용자가 굳이 내부 서버의 아이피를 몰라도 되며, 여러 제품을 각기 다른 서버에서 운영하고 있을 때 외부로 노출시키는 포트를 하나만 사용하더라도 `reverse proxy`가 사용자의 요청에 따라 마치 라우팅 하듯이 사용할 수 있어 효과적인 토폴로지를 구성할 수 있다.

## 목표
- 내 경우 현재 개발중인 Flask 서버 앞단에 `Reverse Proxy`를 붙여서 해당 도메인을 접속했을 때 HTTPS가 적용되도록 할 것이다.
- 개발중인 애플리케이션이 꼭 Flask가 아니여도 가능하니 참고

## Nginx Docker를 이용해서 SSL 설정하기
- 환경 : Ubuntu 22.04 LTS
- `docker`, `docker-compose`가 설치 되어있다고 가정
- [참고한 링크](https://pentacent.medium.com/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71){:target="_blank"}
- 계획
  - 본인의 상황에 맞게 `docker-compose.yml` 파일을 작성한다.
  - `nginx.conf` 파일을 작성한다.
  - certbot으로 nginx를 자동으로 ssl 인증해주는 스크립트를 돌린다.

### 0. 디렉토리 만들기

우선 작업할 디렉토리를 하나 파도록 하자. 우선은 이런식으로 구조가 된다는 것만 인지하도록 하자.

```
.
├── app # 이건 내 Flask 서버이다. 상황에 맞게 본인의 애플리케이션을 위치시키자
│   ├── Dockerfile
│   └── src
│       └── app.py
├── data # data 디렉토리를 만들고 
│   └── nginx.conf # 안에 nginx.conf 파일을 만들어주자
├── docker-compose.yml # 바로 뒤에 작성할 docker-compose.yml 파일이다.
└── init-letsencrypt.sh # 이건 지금 신경쓰지 말고 뒤에 스크립트 나오는거 복붙하면 된다.
```

### 1. docker-compose.yml 작성하기

```yaml
version: '3'
services:
  your-application:
    # 이 부분에 본인의 애플리케이션 config를 넣도록 하자.
  nginx:
    image: nginx:1.18-alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./data/nginx.conf:/etc/nginx/nginx.conf
      - ./data/nginx:/etc/nginx/conf.d
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot
    command: "/bin/sh -c 'while :; do sleep 6h & wait $${!}; nginx -s reload; done & nginx -g \"daemon off;\"'"
    depends_on:
      - your-application
  certbot:
    image: certbot/certbot
    volumes:
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"
```

작성해야 하는 docker-compose 파일은 위와 같다. 다른건 건들지 말고 `your-application` 이라고 해놓은 부분만 본인의 상황에 맞게 수정하면 된다. 간단하게 설명하자면

- nginx
  - image : nginx 기본 이미지를 사용할 것이다.
  - ports : HTTP(80)와 HTTPS(443)를 열어놓을 것이다. 추후 80으로 들어오는건 HTTPS로 Redirect 하는 설정을 할 것이다.
  - volumes : nginx에 ssl 인증을 할것인데, 기본적으로 host에서 ssl 인증을 하고 그 파일을 docker container 내부로 volume share 해 줄 것이다.
  - command : 인증서 갱신용
- certbot
  - image : letsencrypt에서 친절하게도 인증 컨테이너를 push 해주셨다.
  - entrypoint : 인증서 갱신용



### 2. data/nginx.conf 작성하기

`docker-compose.yml` 작성을 완료했으면 같은 디렉토리에 `data/` 디렉토리를 생성하고 그 안에 `nginx.conf` 를 작성한다. 기본 틀은 아래와 같은데, 주석으로 표시한 부분을 본인에 맞게 변경하도록 하자(도메인 및 proxy alias)

여기서 29번 라인에 URL에는 `docker-compose.yml` 에 작성한 본인의 애플리케이션 container name을 적어주면 된다. (예시는 your-application)

~~인증이 안될 경우 처음에는 443 부분을 지우고 했던 것 같다~~

```nginx
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
	worker_connections 1024;
}

http {
	include mime.types;
	server {
		listen 80;

		server_name your.domain.com; # modify required!!!

		location /.well-known/acme-challenge/ {
			root /var/www/certbot;
		}

		location / {
			return 301 https://$host$request_uri;
		}    
	}
	server {
		listen 443 ssl;
		server_name your.domain.com; # modify required!!!
		
		location / {
			proxy_pass http://your-application:8080/; # modify required!!! & check your application port
		}
		ssl_certificate /etc/letsencrypt/live/your.domain.com/fullchain.pem; # modify required!!!
		ssl_certificate_key /etc/letsencrypt/your.domain.com/privkey.pem; # modify required!!!

		include /etc/letsencrypt/options-ssl-nginx.conf;
		ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
	}
	log_format main '$remote_addr - $remote_user [$time_local] "$request" '
	'$status $body_bytes_sent "$http_referer" '
	'"$http_user_agent" "$http_x_forwarded_for"';
	access_log /var/log/nginx/access.log main;

	sendfile on;
	keepalive_timeout 65;
}
```

### 3. init-letsencrypt.sh

서버 구성에 필요한 파일들은 모두 작성했다. 마지막으로, 서버에 ssl 인증을 넣어줘야 하는데 원래대로라면 굉장히 노가다를 해야한다.

그런데 감사하게도 누군가 아래와 같은 nginx ssl 인증 스크립트를 작성해놓았다. 감사한 마음으로 pull 해서 쓰도록 하자.

```bash
curl -L https://raw.githubusercontent.com/wmnnd/nginx-certbot/master/init-letsencrypt.sh > init-letsencrypt.sh
```


이제 `init-letsencrypt.sh` 를 실행시키기만 하면 되는데 그 전에 몇가지 수정해줘야 하는 것이 있다.

```bash
#!/bin/bash

if ! [ -x "$(command -v docker-compose)" ]; then
  echo 'Error: docker-compose is not installed.' >&2
  exit 1
fi

domains=(domain.com, www.domain.com) # 여기 도메인 변경!!
rsa_key_size=4096
data_path="./data/certbot"
email="" # 여기 이메일 넣기!!
staging=0 # 테스트할 때는 여기 1로 바꾸기!! release 시에는 다시 0으로 변경
# ... 후략 ...
```

- L8 : 본인의 도메인으로 변경해주자.
- L11 : 본인의 이메일을 넣어주자.
- L12 : 테스트 할 때는 staging 플래그를 1로 바꿔서 돌려야 한다. 이거 설정 안하면 letsencrypt에 ssl 인증 요청하는 횟수가 제한되어 있어서 block될 수도 있다.

### 4. ssl 인증 받기

모든 준비가 다 되었으면 `sudo` 로 스크립트를 실행해주자.

```bash
sudo ./init-letsencrypt.sh
```

중간에 Y/N 질의가 있는데 Y 선택하고 엔터 쳐주면

![](/assets/img/posts/230201-01.png)

**성공!! 한것을 볼 수 있다.**