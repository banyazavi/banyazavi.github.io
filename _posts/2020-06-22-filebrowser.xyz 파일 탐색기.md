---
title: "[Docker for a NAS] filebrowser.xyz 파일 탐색기"
excerpt: 이걸 제일 먼저 설치했으면 다른 서비스를 설치하는 과정이 매우 쉬웠을 텐데요...
categories:
  - Docker for a NAS
sidebar:
  - nav: docker_nas
tags:
  - Docker
  - Docker Compose
  - File Browser
  - Portainer
---

저는 나스를 설치한 후에 빠르게 **SMB**나 **FTP**를 설치하고, **SSH**도 이제는 익숙해져서 별 필요가 없다고 생각했는데요. 곰곰이 생각해 보니 이런 기본적인 서비스를 편하게 쓰는 것이 제일 중요하겠더라고요. 이걸 진작 알았어야 했는데요. 지금에서야 설치해 보게 되네요.

---

# filebrowser.xyz

![filebrowser.xyz 로고]({{"/assets/images/2020-06-22/2020-06-22-01-01.png"| relative_url}})  
[**filebrowser.xyz 공식 홈페이지**](https://filebrowser.org/){: target="_blank"}

일단 잠시만요, 저는 이 서비스 이름이 **filebrowser.xyz** 인 줄 알았는데, 정식 명칭은 **File Browser**네요? 심지어 주소도 **filebrowser.org**로 바뀌었습니다. `.xyz`로 접속해도 주소가 리다이렉션되며 접속은 되지만요. 하지만 이런 범용적인 이름으로 소개하면 헷갈릴 것 같아서, 기존에 알려진 이름인 **filebrowser.xyz**로 칭하도록 하겠습니다.

아무튼 **filebrwoser.xyz**는 웹으로 접속할 수 있는 파일 탐색기입니다. 파일을 올리거나 내려받는 것은 당연하고, 간단한 텍스트 파일 작성도 할 수 있습니다. 사용자를 여러 명 추가하여 권한을 주고 관리할 수도 있습니다. 나스를 사실상 설치형 클라우드처럼 사용할 수 있게 되는 셈이죠.

자, 설치해 봅시다.

# 사전 작업

실제 존재하는 폴더를 관리하는 것이기 때문에, 사용자 정보를 저장할 DB 파일만 만들어 주면 됩니다.

## DB 파일 생성

서비스에 접속할 사용자 정보와 설정 등을 저장할 파일이 필요합니다. 처음에 빈 파일을 하나 생성하면 도커에서 알아서 DB를 설치해 주네요.

```
$ mkdir -p /docker/filebrowser
$ touch /docker/filebrowser/filebrowser.db
```

`touch`는 해당 경로에 빈 파일을 생성하는 명령이에요. 이건 이번에 처음 써 보네요.

# 도커 컨테이너 생성

아래의 **docker-compose** 파일 내용을 포테이너의 **Add Stack > Web editor** 아래의 공간에 복사하여 붙여넣습니다. [파일 브라우저의 설치 안내 페이지](https://filebrowser.org/installation#docker){: target="_blank"}에 설치법이 잘 나와 있습니다. 저는 두 가지 상황에 따른 설치법을 안내할 거예요.

## 리눅스 전체를 서비스에 표시

일단은 간단한 것부터 해 봅시다. 리눅스의 `/`(root)를 연동하여 설치합니다.

```
version: "3"
services:
  app:
    image: filebrowser/filebrowser
    ports:
      - 8006:80
    volumes:
      - /docker/filebrowser/filebrowser.db:/database.db
      - /:/srv
    restart: unless-stopped
```

`/:/srv` 이 부분이 나스의 모든 파일을 파일 브라우저의 접근 대상으로 한다는 의미입니다. 그런데 이건 좀 위험할 수도 있다고 생각한다면, 필요한 폴더만 매핑해 주는 것이 좋겠죠.

## 지정 폴더만 서비스에 표시

예를 들어, `/data` 폴더와 `/share` 폴더만 공유하고 싶다고 칩시다. 그럼 이렇게 하면 돼요.

```
version: "3"
services:
  app:
    image: filebrowser/filebrowser
    ports:
      - 8006:80
    volumes:
      - /docker/filebrowser/filebrowser.db:/database.db
      - /data:/srv/data
      - /share:/srv/share
    restart: unless-stopped
```

`{/path/to/directory}:/srv{/directory/name}` 처럼 볼륨 매핑을 해주면 됩니다. 그럼 원하는 폴더만 서비스에 보이게 할 수 있어요. 이편이 보안상으론 더 좋겠죠.

# 서비스 동작 확인 및 초기 설정

오늘은 설치가 쉬운 것 같네요. **http://나스_IP:8006**에 접속해 봅시다.

![filebrowser.xyz 로그인]({{"/assets/images/2020-06-22/2020-06-22-01-02.png"| relative_url}})

초기 ID와 비밀번호는 `admin`/`admin`이에요.

![filebrowser.xyz 접속 화면]({{"/assets/images/2020-06-22/2020-06-22-01-03.png"| relative_url}})

`/`(root) 폴더를 공유하게 되면 이렇게 보이게 될 겁니다. 그럼 설치가 잘 된 거예요.

일단 **Settings**을 눌러 들어가 봅시다.

![filebrowser.xyz 설정 화면]({{"/assets/images/2020-06-22/2020-06-22-01-04.png"| relative_url}})

당연히 언어부터 바꿔야겠죠? 언어를 바꾸고 나면 상단의 **User Management** 탭이 **사용자 관리**로 번역되어 나옵니다. 네, 그곳을 눌러 줍시다.

![filebrowser.xyz 사용자 관리 화면]({{"/assets/images/2020-06-22/2020-06-22-01-05.png"| relative_url}})

**신규** 버튼을 눌러 우리가 쓸 사용자를 추가해 줍니다.

![filebrowser.xyz 사용자 추가 화면]({{"/assets/images/2020-06-22/2020-06-22-01-06.png"| relative_url}})

이렇게 새로운 사용자를 관리자로 추가해 줍니다. 그리고 `admin`을 로그아웃한 다음에 새 관리자로 접속할 거예요.

그리고, 다시 사용자 관리 메뉴에 들어가서 **admin을 지워주시는 것이 좋습니다.** 이런 일반적인 이름의 계정은 해킹당하기 딱 좋거든요.

---

# 그리고 남은 이야기

**Nginx Proxy Manager**를 통해 역방향 프록시 설정을 하면 집 밖에서도 웹 브라우저를 통해 파일을 관리할 수 있습니다. 다만, 해킹에 매우 유의해야 해요. 가능하면 `/`(root) 폴더를 공유하는 방식은 사용하지 않고, `admin` 계정을 삭제하여 사용하기 바랍니다.

그리고 화면의 상단 검색창을 통해 파일을 찾을 수도 있는데요. 안타깝게도 한국어는 잘 안 되네요...
