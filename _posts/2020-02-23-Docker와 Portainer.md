---
title: "[Docker for a NAS] Docker와 Portainer"
excerpt: 이 시리즈를 위한 가장 기본적인 도구 두 개를 먼저 설치해야 합니다.
categories:
  - Docker for a NAS
sidebar:
  - nav: docker_nas
tags:
  - Docker
  - Docker Compose
  - Portainer
---

# Docker와 Portainer

굉장히 여러 번 말씀드리는 것 같지만, **Docker**는 컨테이너 기반의 리눅스 가상화 도구입니다. 그러나 우리는 나스에 서비스를 편리하게 설치하기 위한 일종의 플랫폼으로 생각하자고 했죠.

그리고 **Portainer**는 도커를 웹으로 편하게 관리하기 위한 도구입니다. SSH에 접속하여 코드를 칠 필요 없이, 포테이너 웹페이지에서 마우스 클릭으로 운영할 수 있다는 것이죠.

그래서 이 두 개를 이 시리즈의 핵심 기반으로 두고, 이 도구들을 설치하는 것을 첫 번째 목표로 하였습니다.

# Docker 설치

도커는 대부분의 리눅스 시스템에 설치할 수 있습니다. **Ubuntu**나 **Debian**, **CentOS** 등은 각자의 패키지 관리 툴을 이용한 설치 방법도 제공하고 있고요. 바이너리를 직접 내려받아 설치할 수도 있습니다. **시놀로지**는 터미널에 접속할 필요 없이 웹 패널의 **패키지 센터**에서 설치할 수 있습니다. 각자 편한 방법으로 설치하면 됩니다.

따라서 아래의 설치 명령어는 **우분투를 기준으로 작성**했어요. 다른 배포판의 설치 스크립트는 여기에서 확인할 수 있습니다. [Docker 공식 Install 문서](https://docs.docker.com/install/){: target="_blank"} (**Docker Engine > Linux > 배포판 선택**하여 볼 수 있어요)

## 오래된 Docker 제거

이미 도커가 설치된 경우에, 오래된 도커 버전을 제거합니다. 도커가 설치되어 있지 않으면 실행이 무시되므로 일단 실행시키면 됩니다.

```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

## 패키지 업데이트

일단 패키지 업데이트를 한번 해줍니다. 가능하면 최신 버전을 깔아야 하니까요.

```
$ sudo apt-get update
```

## 관련 패키지 설치

도커 저장소 설정을 위해 **HTTPS** 사용을 할 수 있게 하는 패키지들을 설치합니다.

```
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

## Docker GPG Key 추가

도커 패키지의 인증을 위한 GPG Key를 추가해 줍니다.

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

## Docker 저장소 추가

아키텍처별 저장소를 추가해 줍니다.

```
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## 그리고 다시 패키지 업데이트

원래는 저장소 추가하면서 패키지 업데이트를 한번 해줄 텐데, 혹시 모르니 한 번 더 해줍시다.

```
$ sudo apt-get update
```

## Docker 패키지 설치

이제 아래 명령어만 입력하면 도커의 설치가 완료됩니다!

```
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 설치 확인

제대로 설치되었는지 확인해 봅시다. 일단 버전부터 확인해 볼까요?

```
$ sudo docker --version

Docker version 19.03.6, build 369ce74
```

작성일 기준으로 **19.03.6** 버전이 최신인 것 같네요.

```
$ sudo docker ps -a

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

생성된 모든 컨테이너를 조회하는 명령입니다. 아무것도 만들지 않았으니 당연히 없겠죠. 설치 오류 등으로 서비스가 제대로 실행되어 있지 않다면 `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?` 이런 메시지가 나올 거예요.

이렇게 도커 설치는 완료되었습니다. 이제 포테이너를 설치해 봅시다.

# Portainer 설치

포테이너는 도커 서비스입니다. 도커 관련된 서비스라는 것도 맞는데, 이 서비스 자체도 도커 컨테이너로 실행시키거든요! 그래서 명령어가 굉장히 간단한 편에 속하죠.

[공식 설치 문서](https://docs.portainer.io/start/install/server/docker/linux){: target="_blank"}에서도 단 두 줄로 안내하고 있습니다. 따라 해 봅시다.

## Portainer 데이터 볼륨 설정

포테이너 데이터를 담아둘 볼륨을 설정합니다. 

```
$ sudo docker volume create portainer_data
```

## Portainer 컨테이너 생성

그리고 아래 명령어를 입력하여 포테이너 이미지를 내려받고 실행합니다.

```
$ sudo docker run -d \
    -p 8000:8000 \
    -p 9000:9000 \
    --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce
```

# Portainer 초기 설정

설치가 완료되었다면 **http://나스_IP:9000**과 같이, 나스의 9000번 포트로 접속할 수 있습니다.

![Portainer 초기 설정]({{"/assets/images/2020-02-23/2020-02-23-01-01.png"| relative_url}})

최초 접속 시 관리자 계정을 설정하는 화면이 나옵니다. 원하는 이름과 비밀번호를 설정해주세요.

![Portainer 관리 환경 설정]({{"/assets/images/2020-02-23/2020-02-23-01-02.png"| relative_url}})

도커 관리 환경 설정 화면입니다. 우리는 포테이너가 설치된 서버에서 도커를 같이 실행시킬 것이므로, **Local**이에요.

![Portainer 홈]({{"/assets/images/2020-02-23/2020-02-23-01-03.png"| relative_url}})

그럼 이제 홈 화면이 보입니다. 여기에 보이는 **local**이 우리 도커의 현재 상황이에요. 자세한 설정을 위해 카드를 눌러봅시다.

![Portainer Docker 상태 요약]({{"/assets/images/2020-02-23/2020-02-23-01-04.png"| relative_url}})

좀 더 자세한 상태를 볼 수 있어요. 컨테이너가 하나 실행되고 있죠? 그건 포테이너입니다.

앞으로 가이드에 주로 쓰일 메뉴는 **Stacks**입니다. 눌러서 들어가 봅시다.

![Portainer Stack 목록]({{"/assets/images/2020-02-23/2020-02-23-01-05.png"| relative_url}})

스택 관리 화면입니다. 스택은 포테이너에서 정의하는 컨테이너 묶음인데요. 저번 글에서 잠깐 언급했지만, **Docker Compose** 구성들을 의미한다고 보면 됩니다. 현재는 아무것도 구성하지 않았으니 목록이 비어 있어요.

앞으로의 글은 이 화면에서 **Add Stack**을 누른 다음,

![Portainer Stack 생성]({{"/assets/images/2020-02-23/2020-02-23-01-06.png"| relative_url}})

여기의 **Web editor** 아래의 공간에 들어갈 내용을 안내할 겁니다.

---

# 그리고 남은 이야기

도커를 원활하게 사용하려면 몇 가지 리눅스 명령어는 알고 있어야 해요. 아래 대표적인 명령들입니다.

- `ls`: **List Segments**. 현재 폴더의 내용을 보는 명령입니다.
- `cd [path]`: **Change Directory**. 폴더를 이동하는 명령입니다.
- `pwd`: **Print Working Directory**. 현재 폴더의 절대 경로를 출력하는 명령입니다. 나중에 볼륨 바인딩할 때 유용하게 사용하게 됩니다.
- `mkdir -p [path]`: **MaKe DIRectory**. 폴더를 생성하는 명령입니다. `-p` 옵션을 넣으면 해당 폴더를 만드는 데 필요한 상위 폴더까지 자동으로 만들게 돼요.

이 정도만 알고 있으면, 나머지는 포테이너에서 거의 다 할 수 있을 거예요.
