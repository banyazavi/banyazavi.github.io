---
title: "[Docker for a NAS] Nextcloud 설치형 클라우드"
excerpt: 드롭박스나 구글 드라이브 같은 클라우드 저장소를 만들 수 있습니다.
categories:
  - Docker for a NAS
sidebar:
  - nav: docker_nas
tags:
  - Cloud Storage
  - Docker
  - Docker Compose
  - Nextcloud
  - Portainer
---

설치형 클라우드는 나스에 설치하여 나스의 공간을 클라우드 스토리지처럼 사용할 수 있는 서비스입니다. 설치하여 사용해 보니, 나스를 복잡하게 구축할 생각 없이 간단하게 여러 사람과 파일 공유하여 사용할 생각이라면 이것만 깔아도 괜찮은 것 같습니다.

설치형 클라우드를 설치하려고 보니, 다음과 같은 유명한 몇 종류가 있었어요.

- [**Nextcloud**](https://nextcloud.com/){: target="_blank"}
- [**ownCloud**](https://owncloud.org/){: target="_blank"}
- [**Pydio**](https://pydio.com/){: target="_blank"}
- [**Seafile**](https://www.seafile.com/){: target="_blank"}

네 종류 모두 도커허브에 공식 또는 공식에 준하는 이미지가 배포되고 있고요, 플레이스토어에 모바일 앱도 있습니다. 그래서 어떤 것을 설치해 볼까 고민이 있었어요. 결국, 하나를 선택하여 알려드리지만 다른 설치형 클라우드도 꽤 쓸만할 것으로 생각합니다.

---

# Nextcloud

![Nextcloud 로고]({{"/assets/images/2020-03-14/2020-03-14-01-01.png"| relative_url}})  
[**Nextcloud 공식 홈페이지**](https://nextcloud.com/){: target="_blank"}

**Nextcloud**를 선택한 데에는 매우 중요한 이유가 있습니다. **플레이스토어 다운로드 수**가 제일 많았어요. ~~퍽이나 중요한 이유네...~~ 그리고 우분투 서버 설치 시 넥스트클라우드 설치 옵션이 제공되기도 했고요. 이런 점으로 보아 Nextcloud가 제일 대중적으로 사용되는 서비스이니, 팁이나 문제 해결에 대한 도움을 쉽게 받을 수 있으리라 생각했습니다.

# 사전 작업

이 서비스는 보안을 위해 접속 주소를 확인합니다. 설정에 등록되지 않은 주소로 접속하게 되면 경고 메시지가 발생할 거예요. 설정에서 이를 수정해 줄 수는 있지만, 귀찮죠. 그러니까 컨테이너를 만들기 전에 도메인 연결부터 시켜줍니다.

## Nginx Proxy Manager 도메인 연결

[**Nginx Proxy Manager 설치**](/2020-03-01/Nginx-Proxy-Manager-설치)는 모두 하셨죠? **Hosts > Proxy Hosts**에서 **Add Proxy Host**를 눌러 넥스트클라우드를 위한 도메인 연결 규칙을 생성해 줍시다.

![Nextcloud Proxy Host 추가]({{"/assets/images/2020-03-14/2020-03-14-01-02.png"| relative_url}})

**Forward IP**에 **나스의 IP**를 넣고, **Forward Post**에는 이 글에서 사용할 **8014** 포트를 넣습니다.

![Nextcloud SSL 발급]({{"/assets/images/2020-03-14/2020-03-14-01-03.png"| relative_url}})

HTTPS 사용을 위해 SSL 인증도 발급해줍시다.

## 바인딩 폴더 작성

DB와 서비스 파일이 저장될 폴더를 만들어야 합니다. 그리고 자료가 저장될 공간은 용량이 큰 공간에 연결해 주어야 좋겠죠.

### DB 폴더 생성

넥스트클라우드의 자료 정보를 저장할 DB 폴더를 만듭니다.

```
$ mkdir -p /docker/nextcloud/mysql
```

### 서비스 파일 폴더 생성

넥스트클라우드는 PHP로 만들어진 서비스거든요. 이 PHP 파일을 저장할 폴더를 만들어 줍니다.

```
$ mkdir -p /docker/nextcloud/html
```

### 클라우드 자료 폴더 생성

넥스트클라우드의 도커 공식 가이드에는 이를 구분해 놓지는 않았지만, 저는 대량의 파일이 저장될 가능성이 있는 공간은 항상 분리해 놓습니다. SSD를 사용할 경우, 위의 두 폴더는 SSD에 생성하고 이 폴더는 대용량 HDD에 생성하면 빠른 속도와 큰 용량을 모두 노릴 수 있습니다.

```
$ mkdir -p /data/nextcloud
```

# 도커 컨테이너 생성

여기까지 잘 따라 하셨다면, 아래의 **docker-compose** 파일 내용을 포테이너의 **Add Stack > Web editor** 아래의 공간에 복사하여 붙여넣습니다. 늘 그렇듯이[**Nextcloud** 도커허브](https://hub.docker.com/_/nextcloud){: target="_blank"}의 설치 코드를 조금 바꾼 수준입니다.

```
version: '2'

services:
  db:
    image: mariadb
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    restart: always
    volumes:
      - /docker/nextcloud/mysql:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=nextcloud
      - MYSQL_PASSWORD=nextcloud
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  app:
    image: nextcloud
    ports:
      - 8014:80
    links:
      - db
    volumes:
      - /docker/nextcloud/html:/var/www/html
      - /data/nextcloud:/var/www/html/data
    restart: always
```

# 서비스 동작 확인 및 초기 설정

최초 접속을 해서 서비스 초기 설정을 해야 하는데요. 여기서 내부 IP로 접속하면 안 됩니다! NPM에서 연결한 도메인으로 접속해 주세요. 저는 **https://nextcloud.banyazavi.duckdns.org/**로 연결했으니 여기로 접속했어요.

![Nextcloud 관리자 설정]({{"/assets/images/2020-03-14/2020-03-14-01-04.png"| relative_url}})

사용할 관리자 계정을 만들고 아래 **데이터베이스 설정**을 그림과 같이 만들어 줍니다. 여기서는 **DB명**부터 **사용자명**, **비밀번호** 모두 `nextcloud`로 만들었으니 위와 같이 쓰고, **DB의 호스트 주소** 역시 도커 컴포즈의 설정 때문에 **DB**라는 이름으로 사용할 수 있습니다. 결과적으로 데이터베이스 설정은 위 그림과 똑같이 설정하면 됩니다.

그리고 설치 완료를 DB를 생성하는 약간의 시간이 지난 후에 서비스에 접속되게 됩니다.

---

# 그리고 남은 이야기

이 서비스는 여러 사용자와 함께 쓸수록, 데스크톱 그리고 모바일 앱과 같이 쓰면 더욱 편리합니다. 달력이나 문서 편집 등의 기능을 활용하면 그룹웨어로도 쓸 수 있겠네요.
