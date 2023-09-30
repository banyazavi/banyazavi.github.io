---
title: "[OMV6 on HC4] Docker와 Portainer"
excerpt: 도커는 나스의 서비스를 크게 늘려줄 수 있는 플랫폼입니다. 포테이너는 이 도커를 관리하는 간편한 도구이고요.
categories:
  - OMV6 on HC4
sidebar:
  - nav: omv6_nas
tags:
  - Docker
  - NAS
  - OMV6
  - openmediavault
  - Portainer
---

이 글을 따라 하고 나면 OMV6에서도 [**Docker for a NAS**](/2020-02-20/Docker-for-a-NAS)의 가이드를 따라 도커 서비스를 설치할 수 있습니다. 그러니까 이 글은 OMV6를 위한 [**Docker와 Portainer**](/2020-02-23/Docker와-Portainer) 설치 글이에요.

그러니까 이게 뭔지 설명은 생략합니다. 궁금한 분은 Docker NAS 시리즈를 보고 오세요!

---

# Docker 설치

![시스템 > omv-extras > Docker]({{"/assets/images/2022-02-12/2022-02-12-01-01.png"| relative_url}})

**시스템 > omv-extras > Docker** 메뉴에서 도커를 설치할 수 있습니다. 설치 전에는 **상태**가 **Not installed**라고 되어 있는데요. 하단의 **설치** 버튼을 누르면,

![Installing Docker]({{"/assets/images/2022-02-12/2022-02-12-01-02.png"| relative_url}})

위와 같은 화면이 나오며 알아서 설치가 완료됩니다. 직접 명령어를 입력하는 것보다 훨씬 간단해요. ~~물론 이전 방식도 이해 없이 복붙한다면 간편하지만~~

![시스템 > omv-extras > Docker (설치 후)]({{"/assets/images/2022-02-12/2022-02-12-01-03.png"| relative_url}})

설치한 다음에는 **상태**에 **Installed and running**이라고 표시되고, 버전 정보가 적혀 있는 것을 볼 수 있어요. 도커 설치는 이게 끝입니다.

# Portainer

![시스템 > omv-extras > Portainer]({{"/assets/images/2022-02-12/2022-02-12-01-04.png"| relative_url}})

포테이너 설치도 똑같아요. **시스템 > omv-extras > Portainer** 메뉴로 들어간 다음에, **설치** 버튼을 눌러주면 됩니다.

![Installing Portainer]({{"/assets/images/2022-02-12/2022-02-12-01-05.png"| relative_url}})

그럼 포테이너도 똑같이 설치 로그가 화면에 표시되고, **닫기** 버튼이 활성화될 때까지 기다린 후에 창을 닫으면,

![시스템 > omv-extras > Portainer (설치 후)]({{"/assets/images/2022-02-12/2022-02-12-01-06.png"| relative_url}})

**상태**가 **Up**으로 바뀌며 설치가 완료되죠. 하지만 포테이너는 초기 계정 설정을 해 줘야 해요. **http://나스_IP:9000**으로 접속하거나, 아니면 이 화면 아래의 **Open web** 버튼을 눌러도 돼요!

![New Portainer installation]({{"/assets/images/2022-02-12/2022-02-12-01-07.png"| relative_url}})

아쉽게도 포테이너는 한국어 번역을 제공하지 않습니다. 하지만 어렵지 않아요. ~~왜냐하면 기능을 몇 개 안 사용하거든요~~

포테이너에 최초로 접속하게 되면 위와 같이 초기 계정 설정을 해야 합니다. **Username**과 **Password**에 사용할 계정 정보를 입력한 다음 **Create user** 버튼을 누릅니다.

![Welcome to Portainer]({{"/assets/images/2022-02-12/2022-02-12-01-08.png"| relative_url}})

그러면 위와 같이 환영 페이지가 나오는데요, 추가로 환경설정을 할 것이 없으니 바로 **Get Started**를 눌러 접속해볼게요.

![Portainer - Home]({{"/assets/images/2022-02-12/2022-02-12-01-09.png"| relative_url}})

포테이너는 원래 원격 PC의 도커나 쿠버네티스라고 하는 가상화 환경도 관리할 수 있는데요. 그러면 이 화면에 여러 관리 대상이 표시되고, 그중 하나에 접속할 수 있어요.

그런데 우리는 이 나스에 설치되어 있는 도커만 관리할 것이므로, **local**만 보이고, 항상 여기에 접속하면 됩니다.

![Portainer - Dashboard]({{"/assets/images/2022-02-12/2022-02-12-01-10.png"| relative_url}})

그럼 이제 로컬 도커의 대시보드가 보이게 되는데요. 현재는 아무것도 설치하지 않았는데... 하나가 있죠? 이건 포테이너 자신의 도커입니다. 이전 시리즈를 읽은 분은 알겠지만, 포테이너 또한 도커를 통해 실행되는 서비스거든요.

[**Docker NAS**](/category/docker_nas/) 시리즈에서는 **Stack**을 통해 서비스를 설치하고 관리하도록 안내하고 있는데요. **Stack** 메뉴를 눌러 어떤 화면인지 구경해보도록 합시다.

![Portainer - Stacks list]({{"/assets/images/2022-02-12/2022-02-12-01-11.png"| relative_url}})

도커 나스 시리즈를 통해 서비스를 설치한다면 이제 여기에 등록한 서비스 종류가 기록되게 될 거예요. (Stack 또는 Docker Compose를 통해 설치하지 않은 서비스는 여기에 뜨지 않습니다) 지금은 당연히 아무것도 없죠. **Add stack** 버튼 한번 눌러볼까요?

![Portainer - Create stack]({{"/assets/images/2022-02-12/2022-02-12-01-12.png"| relative_url}})

이 화면에서 스택을 관리하고 설치할 수 있어요. 다른 곳에서 도커를 조금 공부한 분들을 위해 설명하자면, 이곳의 **Web editor**에는 **Docker Compose**라고 하는 도커 서비스 구성 파일의 내용을 넣으면 됩니다.

무슨 말인지 잘 모르겠다고요? [**Docker NAS**](/category/docker_nas/) 시리즈는 그걸 몰라도 설치할 수 있게 되어 있어요!

---

# 그리고 남은 이야기

도커를 사용하기 위해서는 나스의 CPU 아키텍처를 알아야 하는데요. **ODROID-HC4**는 **64비트 ARM**이예요! `aarch64`, `arm64` 등 부르는 이름은 다양한데요. `x86-64`나 `amd64`가 아니라는 것만 알고 계시면 되겠습니다.

사실 OMV6에서는 [**Yacht**](https://yacht.sh/){: target="_blank"}라고 하는 다른 도커 관리 도구도 쉽게 설치할 수 있도록 지원하고 있습니다. 그런데 ~~야추~~ 요트는 대시보드에서 모든 컨테이너 (=서비스) 의 자원 점유 현황을 파악하는 것은 간편하긴 한데, 서비스 관리는 포테이너보다 불편하더라고요. 단순히 익숙함의 차이일 수도 있으니, 도커를 잘 이해한 분은 요트도 사용해 보면 좋겠네요.

# 그리고 또 남은 이야기

**OMV6 on HC4** 시리즈는 여기까지입니다. **ODROID-HC4**를 대상으로 가이드를 작성했는데요, 아무래도 이 제품이 많이 쓰이지 않는 모델이다 보니 많은 분께 도움이 되었을지는 잘 모르겠네요. ~~그런데 이제 라즈베리 파이 4는 너무 비싸~~ 처음에 데비안 설치 과정을 제외하고는 다른 컴퓨터에서도 똑같이 가이드를 따라 할 수 있으니, 다른 제품에서 OMV6를 설치하려는 분에게도 이 시리즈가 도움이 되었으면 좋겠네요.

그런데 아직도 OMV6는 정식 릴리즈를 하지 않았네요. 올해 봄에는 안정 버전으로 배포를 하겠죠?
