---
title: "[Docker for a NAS] WordPress 설치형 블로그"
excerpt: Jekyll과 같은 정적 웹 사이트 빌더로 블로그 운영이 어려운 분은 Wordpress가 좋을 수 있어요. 저는 워드프레스도 쉽지 않다고 생각하지만요...
categories:
  - Docker for a NAS
sidebar:
  - nav: docker_nas
tags:
  - CMS
  - Docker
  - Docker Compose
  - Portainer
  - WordPress
---

# Github Pages

![Wordpress.org 로고]({{"/assets/images/2020-04-11/2020-04-11-01-01.png"| relative_url}})  
[**WordPress.org 공식 홈페이지**](https://ko.wordpress.org/){: target="_blank"}

**WordPress**는 가장 많이 쓰이는 오픈 소스 설치형 블로그 툴이자 **CMS**<sup>Contents Management System</sup>입니다. 국내의 설치형 블로그는 **태터툴즈**나, 이에서 발전된 **텍스트큐브**가 많이 쓰였죠. 반면 웹 사이트 빌더의 경우, 보통은 **제로보드**나 **그누보드**와 같은 게시판 모듈을 사용하거나 **XpressEngine** 정도가 쓰였죠. 아무튼, 지금은 다 옛날 이야기네요. 말했던 솔루션들도 아직 오래된 사이트에서는 여전히 찾아볼 수 있긴 하지만요.

과거의 솔루션도 편리한 면이 있긴 하지만 워드프레스를 소개하는 이유는 간단합니다. **도커허브에 없어요.** ~~대원칙: 편해야 한다~~ 물론 개인이 만들어 둔 이미지가 있긴 하지만, 공식적으로 관리되지 않는 이미지는 아무래도 불안하니까요. 그렇다고 지속해서 관리되고 있는 것 같지도 않고요.

아무튼, 워드프레스를 설치해 보도록 합시다.

# 사전 작업

사실 [**워드프레스 도커허브**](https://hub.docker.com/_/wordpress/){: target="_blank"}에는 볼륨 컨테이너를 사용하여 구성하게 되어있습니다. 따라서 아무런 사전 작업이 필요가 없었는데요. 저는 볼륨 바인딩 방식을 사용하여 도커 서비스를 구성할 예정이므로 여전히 폴더를 만들어야 합니다. 굳이 볼륨 바인딩을 사용하는 이유는요. 첫째, 나스에서는 도커가 설치된 시스템 파티션의 용량이 작은 경우가 많고, 둘째, 혹시나 나스를 이전하거나 재설치할 때 설정이나 내용을 이전하기 편하거든요.

## DB 폴더 생성

워드프레스의 자료를 저장할 DB 폴더를 만듭니다.

```
$ mkdir -p /docker/wordpress/mysql
```

## 서비스 파일 폴더 생성

워드프레스도 넥스트클라우드와 같이 PHP로 만들어져 있습니다. 이 PHP 파일을 저장할 폴더를 만들어 줍니다.

```
$ mkdir -p /docker/wordpress/html
```

# 도커 컨테이너 생성

아래의 **docker-compose** 파일 내용을 포테이너의 **Add Stack > Web editor** 아래의 공간에 복사하여 붙여넣습니다.

사실 도커허브의 설정을 최대한 그대로 가져왔는데요, 지금 보니 DB 정보는 좀 바꿨어도 되겠네요...

```
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8023:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - /docker/wordpress/html:/var/www/html

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - /docker/wordpress/mysql:/var/lib/mysql
```

# 서비스 동작 확인 및 초기 설정

이제 **http://나스_IP:8023**에 접속해 봅시다. 워드프레스 셋업 화면이 반겨줄 거예요.

![WordPress 설치 언어 선택]({{"/assets/images/2020-04-11/2020-04-11-01-02.png"| relative_url}})

영어로 반겨주는 건 별로 반갑지 않네요. 아래에 **한국어**가 있으니 선택해 줍시다.

![WordPress 설치 정보 설정]({{"/assets/images/2020-04-11/2020-04-11-01-03.png"| relative_url}})

사이트 제목과 기본 사용자 정보를 설정해주세요. 암호 필드에 기본 암호가 굉장히 어렵게 쓰여있는데, 저처럼 까먹어서 재설치하는 일이 없도록 합시다.

![WordPress 로그인]({{"/assets/images/2020-04-11/2020-04-11-01-04.png"| relative_url}})

자, 설치는 끝났습니다. 이제 로그인하여 ~~더 어려운~~ 워드프레스 세팅을 해 봅시다.

---

# 그리고 남은 이야기

홈 네트워크 내에서만 워드프레스를 사용하려는 분은 없죠? **Nginx Proxy Manager**에서 프록시 호스트 설정을 해주면 외부에서도 접속할 수 있어요. **HTTPS** 설정도 쉽게 할 수 있으니 반드시 연동하는 것이 좋습니다.
