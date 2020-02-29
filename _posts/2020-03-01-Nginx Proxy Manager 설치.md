---
title: Nginx Proxy Manager 설치
key: post-2020-03-01-01
categories:
  - Docker Cookbook
sidebar:
  - nav: "docker_cookbook"
tags:
  - Docker
  - Nginx Proxy Manager
  - 역방향 프록시
---

> 대부분의 서비스를 웹 페이지를 통해 접속할 거에요. 그러면 HTTPS 설정 서비스부터 설치하는 것이 좋겠죠?

<!--more-->

> **[NOTE]**
>
> 이 서비스는 **x86-64**와 **arm** 아키텍처의 설치 방법에 차이가 있습니다.

---

# Nginx Proxy Manager

![Nginx Proxy Manager 로고]({{"/assets/images/2020-03-01/2020-03-01-01-01.png"| relative_url}})  
[Nginx Proxy Manager 공식 홈페이지](https://nginxproxymanager.jc21.com){: target="_blank"}

역방향 프록시 개념은 OMV4 나스 구축 가이드의 [**Nginx 역방향 프록시 설정**](/2019-12-01/Nginx-역방향-프록시-설정)에서 한번 다룬 내용입니다. 역방향 프록시를 사용하게 되면 여러분의 홈서버에 **HTTPS** 적용을 쉽게 할 수 있어요.

또한 이 서비스는 [**Let's Encrypt SSL/TLS 인증서 적용**](/2019-12-14/Let's-Encrypt-SSL-TLS-인증서-적용)도 쉽게 할 수 있습니다.

그리고 이 서비스가 저와 같은 OMV 사용자에게 중요해진 이유가 있습니다. OMV5부터는 Nginx와 Let's Encrypt 플러그인이 삭제 되었거든요. 따라서 편하게 설치하려면 이러한 서비스가 필요합니다.

자 그럼, 엔진엑스 프록시 매니저를 도커로 설치해봅시다. 이 글은 해당 서비스의 [**설치 가이드**](https://github.com/jc21/nginx-proxy-manager/blob/master/doc/INSTALL.md?utm_source=npm-site){: target="_blank"}를 한국어로 쉽게 풀어 설명한 것에 가까우니, 해당 링크를 참조하셔도 좋습니다.

# 사전 작업

다른 서비스들은 사전 작업으로 볼륨 매핑할 폴더를 만들어두거나, 기껏해야 설정 파일을 미리 구성하는 정도 밖에 없는데요. 이 서비스는 홈서버의 웹 접속을 총괄하는 역할을 가지고 있기 때문에 포트포워딩도 해 줘야 합니다. 도메인 연결도 이 참에 해 둬야겠네요.

## 도메인 연결 (DNS or DDNS)

[**Duck DNS**](https://www.duckdns.org){: target="_blank"}와 같은 DDNS 서비스나, 도메인을 구입하여 홈서버에 연결을 시켜주세요.

Duck DNS를 사용하실 경우에는 [**IP 갱신 스크립트 설치**](https://www.duckdns.org/install.jsp){: target="_blank"}를 참고하여 설정하시거나, [**DuckDNS DDNS 설치**](/2019-11-10/DuckDNS-DDNS-설치)하는 OMV 가이드를 보셔도 좋습니다.

DNS를 연결하실 때는 네임서버에 **\*.yourdomain.com**과 같은 방식으로 서브도메인 전체를 홈서버에 연결해 주셔야 합니다. 아니면 프록시 매니저에서 설정한 각각의 서브도메인에 대한 네임서버 연결을 모두 홈서버로 지정해 주셔도 됩니다. 다만 그건 좀 귀찮겠죠.

## 공유기 포트포워딩 설정

공유기에 **80 (HTTP)**, **443 (HTTPS)** 포트에 대한 포트포워딩을 해 줍시다. 인터넷 서비스 제공자가 80/443 같은 기본 포트를 막아두었거나, 다른 포트를 사용하시려면 포트포워딩 규칙을 바꾼 후에 컨테이너 배포 시 서비스 설정도 바꿔주시면 됩니다.

아, 그리고 80 포트는 기존 홈서버에 설치되어 있는 서비스가 이미 사용하고 있을 확률이 높습니다. 저는 기존 서비스의 포트를 바꾸는 것을 추천하는 편이에요. 그만큼 이 서비스는 HTTP/HTTPS 접속의 기본이 되는 서비스에요.

![HTTP 포트포워딩 설정]({{"/assets/images/2020-03-01/2020-03-01-01-02.png"| relative_url}})

HTTP는 이렇게, 내부 IP주소는 당연히 홈서버의 IP이구요.

![HTTPS 포트포워딩 설정]({{"/assets/images/2020-03-01/2020-03-01-01-03.png"| relative_url}})

HTTPS 또한 포트만 바꿔서 동일한 방식으로 설정해 줍니다.

## 맵핑 폴더 생성

서비스에 필요한 파일을 저장할 폴더를 만들 거에요. 이 시리즈에서 저는 특이사항이 없다면 일관적으로 루트 폴더 아래 `/docker/서비스명/`과 같은 서비스에 대한 폴더를 만들고, 그 아래에 추가적으로 관련 폴더를 넣을 생각입니다.

만약에 여러분의 홈서버가 루트 폴더 아래에 폴더를 만들면 안 되는 경우 다른 폴더를 지정할 수 있습니다.

시놀로지와 OMV는 아래와 같은 사유로 다른 위치에 맵핑 폴더를 만드셔야 할 거에요.

- **시놀로지**: 루트 바로 아래 쓰기가 금지되어 있어 `/volmue1/` 과 같은 디스크 마운트 위치 아래에 만들어야 합니다.
- **OMV**: 루트 볼륨의 용량이 부족한 경우가 대부분이라 `/srv/dev-disk-by-label-SOMETHING/` 과 같은 디스크 마운트 절대경로 아래에 만드시는 게 좋아요.

폴더 경로를 바꾸어 생성하실 때는 아래 내용을 그대로 복붙해서 진행하시면 안되고, 관련 경로를 수정해서 적용하셔야 합니다.

### 서비스 기본 폴더 생성

아까 `/docker/서비스명/`과 같은 규칙으로 서비스 기본 폴더를 만든다고 했었죠? 따라서 아래와 같은 명령을 입력하면 됩니다. 다른 경로로 기본 폴더를 만드실 경우에는 이 경로가 보이는 족족 바꿔주신 경로로 치환해주시면 됩니다.

```
$ mkdir -p /docker/nginx-proxy-manager
```

### Data 폴더 생성

서비스 데이터를 보관하고 있는 위치에요. 사실 대부분의 폴더 생성 부분은 도커 이미지 제작자가 만들라는 대로 만드는 거라서 정확히 어떻게 쓰이는 지 알고 하는 편은 아닙니다.

```
$ mkdir -p /docker/nginx-proxy-manager/data
```

### DB 폴더 생성

`data` 폴더 아래에 **MySQL (MariaDB) DB**를 저장할 폴더도 만들 거에요.

```
$ mkdir -p /docker/nginx-proxy-manager/data/mysql
```

### Let's Encrypt 폴더 생성

**Let's Encrypt**의 인증 정보를 담은 폴더도 생성합니다. 여기에 있는 인증서를 다른 서비스에서도 활용할 수 있어 보이는데요. 시도해봤는데 이 폴더 안에선 인증서 이름이 숫자로 관리되고 있어 쉽지 않더라구요.

```
$ mkdir -p /docker/nginx-proxy-manager/letsencrypt
```

## DB 설정파일 생성

이 서비스는 DB 설정파일도 만들어 넣어주어야 합니다. 

기본 경로 아래 `config.json` 파일을 만들어서 아래와 같은 내용을 적어줍니다. 아래와 같이 `nano` 에디터로 파일을 열어서, (또는 `vi` 에디터를 쓰시거나)

```
$ nano /docker/nginx-proxy-manager/config.json
```

아래 내용을 복붙하시고,

```
{
  "database": {
    "engine": "mysql",
    "host": "db",
    "name": "npm",
    "user": "npm",
    "password": "npm",
    "port": 3306
  }
}
```

**Ctrl + O** 누르고 **엔터** 쳐서 저장하시면 돼요.

그런데 이것도 어렵고 귀찮죠? 그래서 이 블로그에서 직접 받을 수도 있게 했습니다. 이 사이트가 없어지지 않는 이상 아래 명령어로 설정 파일을 받을 수 있을 거에요.

```
$  wget -O /docker/nginx-proxy-manager/config.json https://blog.banyazavi.com/assets/files/2020-03-01/config.json
```

# 도커 컨테이너 생성

여기까지 잘 따라하셨다면, 아래의 **docker-compose** 파일 내용을 포테이너의 **Add Stack > Web editor** 아래의 공간에 복사하여 붙여 넣습니다.

```
---
version: "2"
services:
  app:
    image: jc21/nginx-proxy-manager:2
    restart: always
    ports:
      # Public HTTP Port:
      - 80:80
      # Public HTTPS Port:
      - 443:443
      # Admin Web Port:
      - 81:81
    volumes:
      # Make sure this config.json file exists as per instructions above:
      - /docker/nginx-proxy-manager/config.json:/app/config/production.json
      - /docker/nginx-proxy-manager/data:/data
      - /docker/nginx-proxy-manager/letsencrypt:/etc/letsencrypt
    depends_on:
      - db
  db:
    image: mariadb:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: "npm"
      MYSQL_DATABASE: "npm"
      MYSQL_USER: "npm"
      MYSQL_PASSWORD: "npm"
    volumes:
      - /docker/nginx-proxy-manager/data/mysql:/var/lib/mysql
```

그런데 **arm** 아키텍처에서는 이 구성으로 실행할 수가 없습니다. 제작자가 NPM 자체는 arm을 지원하도록 만들긴 했는데요, **MariaDB**의 공식 이미지가 arm 지원을 하지 않거든요. 그래서 arm 아키텍처는 DB 이미지를 **webhippie/mariadb**로 바꿔서 생성합니다. 이에 따라 환경 변수도 차이가 나요.

```
---
version: "2"
services:
  app:
    image: jc21/nginx-proxy-manager:2
    restart: always
    ports:
      # Public HTTP Port:
      - 80:80
      # Public HTTPS Port:
      - 443:443
      # Admin Web Port:
      - 81:81
    volumes:
      # Make sure this config.json file exists as per instructions above:
      - /docker/nginx-proxy-manager/config.json:/app/config/production.json
      - /docker/nginx-proxy-manager/data:/data
      - /docker/nginx-proxy-manager/letsencrypt:/etc/letsencrypt
    depends_on:
      - db
  db:
    image: webhippie/mariadb:latest
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: "npm"
      MARIADB_DATABASE: "npm"
      MARIADB_USERNAME: "npm"
      MARIADB_PASSWORD: "npm"
    volumes:
      - /docker/nginx-proxy-manager/data/mysql:/var/lib/mysql
```

그리고 **Deploy the stack** 버튼을 눌러주세요.

# 서비스 동작 확인

도커 컴포즈 코드에서 알아채신 분도 있으실 거에요. NPM 서비스는 81번 포트로 접속할 수 있습니다. **http://홈서버_IP:81**로 접속하면 아래와 같은 화면이 나올 거에요.

![Nginx Proxy Manager 접속 화면]({{"/assets/images/2020-03-01/2020-03-01-01-04.png"| relative_url}})

초기 로그인 정보는 아래와 같습니다. 아래 정보로 최초 접속하게 되면 바로 새로운 이메일과 비밀번호를 설정하셔야 할 거에요.

- Email:    admin@example.com
- Password: changeme

---

# 그리고 남은 이야기

이 서비스는 다른 글에서도 종종 등장할 거에요. 외부에 노출시켜서 여러 사람들과 사용해야 하는 **Wordpress**와 같은 웹 사이트 서비스나 **Nextcloud**와 같은 설치형 클라우드가 대표적이겠네요.
