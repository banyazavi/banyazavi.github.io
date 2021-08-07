---
title: "[Docker for a NAS] Transmission BitTorrent 클라이언트"
excerpt: OMV5에는 트랜스미션 플러그인이 없죠. 그래서 이 방법으로 설치해야 합니다.
key: post-2020-03-07-01
categories:
  - Docker for a NAS
sidebar:
  - nav: docker_nas
tags:
  - BitTorrent
  - Docker
  - Docker Compose
  - Portainer
  - Transmission
---

# Transmission

[**Transmission Project 공식 홈페이지**](https://transmissionbt.com/){: target="_blank"}

**Transmission**은 리눅스에서 널리 쓰이는 오픈 소스 **BitTorrent** 클라이언트입니다. 윈도우에서 많이 쓰이는 [**µTorrent**](https://www.utorrent.com/){: target="_blank"}나 [**qBittorrent**](https://www.qbittorrent.org/){: target="_blank"}와 같은 종류의 프로그램이에요.

위의 요약문에서 얘기했지만 OMV5에서는 이 플러그인이 삭제되었습니다. 아마 이렇게 도커로 설치하는 편이 더 쉽고 관리하기 편해서일 듯해요.

# 사전 작업

이 서비스는 파일을 주고받기 위해서 사용자와 밀접하게 연계해야 합니다. 단순하게 표현하자면 내려받은 것을 이용하기 편해야 한다는 거예요. 그러기 위해서 다운로드 폴더는 최대한 접근하기 쉬운 곳을 지정해야 하고, 권한 오류가 생기지 않도록 설정도 해야 합니다.

## 사용자 및 그룹 확인

예를 들어 토렌트를 받기 위해 `/data/torrent`라는 폴더를 미리 만들어 두었다고 합시다. 나스의 공유 폴더 권한 등의 설정으로 이 폴더에 의도한 사용자가 정상적으로 파일을 이용할 수 있는지 확인이 되었다면, 터미널에서 해당 폴더의 상위 폴더에 `ls -l` 명령을 해서 설정을 확인할 수 있습니다.

```
$ ls -l /data

drwxr-xr-x   1 banyazavi users   4096 Mar 07 00:00 torrent
```

여기서 확인할 내용은 `torrent` 폴더가 `banyazavi`라는 사용자의 소유, `users`라는 그룹에 속해있다는 점이에요. 앞으로 생성할 트랜스미션이 이 폴더에 정상적으로 읽고 쓰기를 하려면 해당 사용자와 그룹을 가지고 있어야 한다는 의미입니다.

여기에 더해서 이 사용자와 그룹의 ID도 알아야 합니다. `id 사용자명`명령을 통해 알 수 있어요.

```
$ id banyazavi

uid=1000(banyazavi) gid=100(users) groups=100(users),27(sudo),111(ssh),1001(Members)
```

여러 정보가 나오는데요. 저희는 `banyazavi` 사용자의 ID (uid) 는 `1000`이고, `users` 그룹의 ID (gid) 는 `100`이라는 정보만 얻어 가면 됩니다.

## 바인딩 폴더 작성

앞서 말한 `/data/torrent`와 같이 사용자와 상호 작용해야 하는 폴더들은 터미널이 아니라 다른 방식으로 생성하고, `pwd` 명령을 통하여 전체 경로만 확인해 두는 것이 편리합니다. 그렇지 않으면 권한 문제가 생길 수 있어요.

### 설정 폴더 생성

트랜스미션의 설정이 저장되는 공간입니다. 이 폴더를 바인딩해 두면 나중에 컨테이너를 재생성하더라도 속도 제한 등의 설정이 복구됩니다. 직접 접근할 일은 거의 없으므로 터미널 명령어로 생성해도 괜찮아요.

```
$ mkdir -p /docker/transmission/config
```

### 다운로드 폴더 생성

내려받는 파일이 저장되는 공간입니다. 그런데 앞에서 이 폴더를 `/data/torrent`로 이미 생성해 두었죠? 그렇다면 이 부분은 생략해도 됩니다.

```
$ mkdir -p /data/torrent
```

그런데 내려받는 파일은 한 종류가 더 있어요. 바로 **내려받는 중인 파일**입니다. 또한, 이 공간은 읽고 쓰기가 자주 발생하여 토렌트 클라이언트가 하드 파괴범이라는 악명을 가지게 하는 데 큰 역할을 하기도 해요. 그게 신경 쓰인다면 이 폴더를 SSD 드라이브 내의 공간에 바인딩시켜주세요. 저는 일단 기본 경로로 하겠습니다.

```
$ mkdir -p /docker/transmission/incomplete
```

### 토렌트 시드 폴더 생성

이 폴더는 트랜스미션에서는 **Watch** 폴더라고 합니다. `.torrent` 확장자의 토렌트 시드를 이 폴더에 넣어두면, 트랜스미션이 토렌트를 자동으로 추가하고 해당 시드는 `.torrent.added` 확장자로 바꿔둬요. 트랜스미션 웹페이지에 접속하는 것보다 FTP 접속과 같은 폴더 직접 접근이 더 편하거나, 해당 폴더를 응용해서 자동 다운로드 스크립트를 만들 때 매우 유용합니다. 저는 `/data/seeds` 폴더로 만들었습니다.

```
$ mkdir -p /data/seeds
```

## 생성 폴더의 소유자 및 그룹 수정

위에서 만든 모든 폴더에 트랜스미션 도커가 접근 가능해야 합니다. 우리는 트랜스미션을 `banyazavi` 사용자 (1000), `users` 그룹 (100) 으로 실행할 예정이니까 이에 맞게 폴더를 수정해 줍시다. 혹시 생성한 폴더가 다르다면 아래 명령에서 해당 경로 역시 바꾸어서 해야 합니다.

```
$ sudo chown banyazavi:users /data/torrent /docker/transmission/config /docker/transmission/incomplete /data/seeds
```

# 도커 컨테이너 생성

여기까지 잘 따라 하셨다면, 아래의 **docker-compose** 파일 내용을 포테이너의 **Add Stack > Web editor** 아래의 공간에 복사하여 붙여넣습니다. 이 부분은 [**linuxserver/transmission** 도커허브 설명서](https://hub.docker.com/r/linuxserver/transmission){: target="_blank"}에서 상세한 설정까지 확인할 수 있어요.

```
version: "2"
services:
  transmission:
    image: linuxserver/transmission
    container_name: transmission
    environment:
      - PUID=1000
      - PGID=100
      - TZ=Asia/Seoul
      - USER=yourid
      - PASS=yourpasswd
    volumes:
      - /docker/transmission/config:/config
      - /data/torrent:/downloads/complete
      - /docker/transmission/incomplete:/downloads/incomplete
      - /data/seeds:/watch
    ports:
      - 9091:9091
      - 51413:51413
      - 51413:51413/udp
    restart: unless-stopped
```

몇 가지 설정은 반드시 확인하고 값을 바꿔서 넘어가야 해요.

아래 두 값은 위에서 `id` 명령을 통해 숫자를 확인했었죠?

- **PUID**: 트랜스미션을 실행할 사용자입니다. 이 글에서는 **1000** (`banyazavi`) 이었죠.
- **GUID**: 트랜스미션을 실행할 그룹입니다. **100** (`users`) 으로 설정했어요.

트랜스미션 관리 웹 페이지에 사용할 아이디와 비밀번호도 설정해야 합니다. 해당 줄 자체를 지워버리면 인증 없이도 접속할 수 있기는 해요.

- **USER**: 트랜스미션 관리 페이지 사용자명
- **PASS**: 트랜스미션 관리 페이지 비밀번호

이 도커 이미지는 관리 웹 페이지 디자인도 몇 가지 선택이 가능합니다. 관심 있으면 아래 세 개 중 하나를 `enviroment` 하위 목록에 추가해 보세요.

- \- TRANSMISSION_WEB_HOME=/combustion-release/
- \- TRANSMISSION_WEB_HOME=/transmission-web-control/
- \- TRANSMISSION_WEB_HOME=/kettu/

그리고 **Deploy the stack** 버튼을 눌러주세요.

# 서비스 동작 확인

**9091**가 트랜스미션 관리 웹 페이지의 포트입니다. **http://나스_IP:81**에 접속해 봅시다.

![Transmission 웹 패널 인증]({{"/assets/images/2020-03-07/2020-03-07-01-01.png"| relative_url}})

여기에 아까 컨테이너 설정에 적었던 **USER**와 **PASS** 값을 넣어주면,

![Transmission 웹 패널]({{"/assets/images/2020-03-07/2020-03-07-01-02.png"| relative_url}})

이렇게 트랜스미션 페이지가 나오게 됩니다. ~~기본 디자인 굉장히 안 예쁘다~~ ~~TRANSMISSION_WEB_HOME 넣어서 바꾸세요~~
