---
title: "[OMV6 on HC4] openmediavault 6 설치"
excerpt: OMV6는 아직 정식 출시되지 않아서 공식 설치 안내가 없어요. 하지만 설치하기는 여전히 어렵지 않죠.
categories:
  - OMV6 on HC4
sidebar:
  - nav: omv6_nas
tags:
  - NAS
  - OMV6
  - openmediavault
---

원래 OMV를 설치하는 방법은 OMV 공식 문서의 [**Installation on Debian**](https://openmediavault.readthedocs.io/en/stable/installation/on_debian.html){: target="_blank"} 페이지에서 확인할 수 있습니다. 그러나 OMV6는 아직 정식 출시된 것이 아니라서, 공식 문서에는 아직 OMV5의 설치법만 나와 있어요.

그래서 우리는 다른 방식으로 설치합니다. 사실, 이 방법이 더 쉬워서 정식 출시 이후로도 이렇게 설치하는 것이 더 편할 수도 있어요.

---

# openmediavault 6 설치

그 방법은 바로 **OpenMediaVault Plugin Developers**에서 만든 [**installScript**](https://github.com/OpenMediaVault-Plugin-Developers/installScript){: target="_blank"}를 사용하는 것입니다. 데비안 11을 설치한 오드로이드에 SSH로 연결하거나, 또는 키보드와 모니터를 연결하고 아래 명령어를 입력하여 이 스크립트를 실행할 수 있어요.

```
$ sudo wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```

그리고 한 10분 정도 지나면 OMV6 설치가 완료됩니다. **http://나스_IP**로 접속하여 OMV 워크벤치에 접속할 수 있어요.

자, 모든 설치가 끝났습니다. 이번 글은 여기까지에요.

---

# 그리고 남은 이야기

이렇게 끝내면 좀 아쉽죠. 그래서 기본 설정까지는 이번 글에서 다루기로 하겠습니다.

## openmediavault 초기 관리자 계정

![OMV6 Workbench 로그인]({{"/assets/images/2021-12-11/2021-12-11-01-01.png"| relative_url}})

OMV의 초기 관리자 계정은 `admin / openmediavault` 입니다. 이 계정으로 접속해야 설정 관리가 가능해요.

## 관리자 계정 비밀번호 변경

![User Management > Change Password]({{"/assets/images/2021-12-11/2021-12-11-01-02.png"| relative_url}})

비밀번호는 바로 바꾸는 것이 좋겠죠? 비밀번호 변경은 상단 톱니바퀴의 **Change Password** 메뉴에서 할 수 있습니다.

## DNS 서버 추가

![네트워크 > 인터페이스 > 편집]({{"/assets/images/2021-12-11/2021-12-11-01-03.png"| relative_url}})

저는 DNS 서버 주소를 수동으로 넣지 않으면, 이후 도커를 설치할 때 도커 컨테이너의 네트워크 문제가 발생하더라고요. 그래서 미리 DNS 서버 주소를 넣어줬습니다.

**네트워크 > 인터페이스** 메뉴에 등록된 인터페이스를 (저의 경우엔 `eth0`이었어요) 편집합니다. `8.8.8.8`은 구글 DNS의 주소인데요, 다른 DNS 서버의 주소를 넣어도 괜찮아요.

## 초기 사용자 계정 ssh 권한 재할당

![User Management > 사용자 > 편집]({{"/assets/images/2021-12-11/2021-12-11-01-04.png"| relative_url}})

OMV를 설치한 이후에는 SSH로 접속이 되지 않게 되는데요. 왜냐하면, 설치 스크립트가 OMV를 설치하며 모든 사용자의 SSH 권한을 빼기 때문입니다. 이후에 SSH를 사용할 일이 많으니, 지금 미리 넣어주는 것이 좋겠죠.

**User Management > 사용자** 메뉴에서 데비안 설치 시 등록한 초기 사용자를 수정할 수 있어요. 이 사용자에게 **SSH** 그룹을 다시 선택하여 넣어주면 됩니다.

## Workbench 접속 포트 수정

![시스템 > Workbench]({{"/assets/images/2021-12-11/2021-12-11-01-05.png"| relative_url}})

OMV 워크벤치는 HTTP 기본 포트인 **80**번을 가지고 있어요. 이 포트는 나중에 역방향 프록시 용도로 써야 하니, 지금 워크벤치의 포트를 바꾸도록 하겠습니다.

**시스템 > Workbench** 메뉴에서 접속 포트를 바꿀 수 있습니다. 저는 **1580**번으로 바꿨어요.

이 설정을 적용하고 나면 앞으로 워크벤치에는 **http://나스_IP:1580**으로 접속해야 됩니다.

그러면 이제 기본적인 설정은 모두 완료됐습니다. 다음에는 기본적인 파일 공유 서비스를 설정해 볼 거예요.
