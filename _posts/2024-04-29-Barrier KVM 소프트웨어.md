---
title: "[Ubuntu 24.04 LTS] Barrier KVM 소프트웨어"
excerpt: 이번에는 리눅스를 서버로, 윈도우를 클라이언트로 설정해 볼게요.
categories:
  - 우분투 24.04 LTS
sidebar:
  - nav: ubuntu_24_04_lts
tags:
  - barrier
  - KM switch
  - KVM switch
  - Linux
  - Ubuntu
  - Ubuntu Desktop
---

> **[NOTE]**  
> **Barrier KVM** 무슨 서비스인지 궁금하세요?
> 그렇다면 [**이전 시리즈의 글**](/2022-06-25/Barrier-KVM-소프트웨어)을 확인하세요!

2년 전과 달라진 점이라면, Snap에서 Barrier 패키지가 내려갔다는 점이 있습니다. 그래서 이전 글을 그대로 따라할 수가 없었어요.

그래서 APT에서 패키지를 받아 설치해 보겠습니다. 그리고 이번엔 리눅스를 서버로, 윈도우를 클라이언트로 설정할 거예요.

---

# 준비 사항

배리어로 연결할 PC들은 모두 같은 네트워크 내에 있어야 합니다. 아래와 같은 조건을 만족하면 될 거예요.

- 서버가 될 PC (리눅스) 의 IP가 고정되어 있어야 해요. 공유기의 DHCP 설정에서 해당 PC를 수동 등록해 주세요. 그렇지 않으면 IP가 바뀔 때마다 배리어 연결이 끊어지게 됩니다.
- 두 PC가 같은 네트워크에 연결되어 있어야 해요. 일반적으로 같은 공유기에 연결된 상태라면 신경 쓸 필요가 없습니다.

# Barrier Linux 서버 설치

```
$ sudo apt update
$ sudo apt install barrier
```

이번엔 배리어를 APT를 통해서 설치합니다. 우분투 24.04에서는 Final 버전인 2.4.0을 제공하고 있어요.

설치 후에 **메뉴 > 보조 프로그램**에 **Barrier**가 생긴 것을 볼 수 있어요. 들어가서 첫 실행을 해 봅시다.

## Barrier Linux 서버 설정

![Barrier 설정 - 언어 선택]({{"/assets/images/2024-04-29/2024-04-29-01-01.png"| relative_url}})

언어를 선택할 수 있습니다. 배리어는 한 번 설치한 후에는 인터페이스를 볼 일이 거의 없지만, 그래도 **한국어**가 편하겠죠?

![Barrier 설정 - 서버/클라이언트 선택]({{"/assets/images/2024-04-29/2024-04-29-01-02.png"| relative_url}})

이번에는 리눅스를 서버로 설정할게요.

그럼 이제 홈 화면이 나오는데요, **Barrier > Change Settings (F4)**에 들어가서 기본 설정을 할게요.

![Barrier 설정 - 설정]({{"/assets/images/2024-04-29/2024-04-29-01-03.png"| relative_url}})

**화면 이름(R)**을 기억해 두세요. 저는 `Banyazavi-LinuxPC`이라고 정했습니다. (사실 기본값인 PC 이름이에요)

**Minimize to System Tray**, **Hide on startup**을 선택해서 부팅시 팝업이 계속 뜨고, 작업 표시줄에 남아 공간을 차지하지 않도록 해 줍니다.  

그런데 정작 리눅스 배리어에서 **Start Barrier on startup** 옵션은 제대로 동작하지 않네요. 그래서 우리는 이 옵션을 끄고, 직접 시작 프로그램에 등록하도록 하겠습니다.

**Enable SSL**, **Require client certificate**은 보안 연결을 위한 설정인데요. 일단 선택 후에 이것에 대한 추가 설정은 뒤에서 다시 하겠습니다.

이제 설정 화면을 닫고 홈 화면에서 **서버 설정(C)**를 눌러 화면 배치를 해야 하는데요.

![Barrier 설정 - 서버 설정]({{"/assets/images/2024-04-29/2024-04-29-01-04.png"| relative_url}})

이 화면은 연결할 PC들의 화면이 어떻게 위치할지를 설정하는 곳이에요. 저는 윈도우 PC가 리눅스 PC의 왼쪽에 존재하는 상황이에요.

오른쪽 위의 모니터 그림을 드래그 앤 드롭으로 리눅스 PC 옆에 배치하면 **이름없음** 이라는 기본값으로 배치가 되는데요. 이 아이콘을 더블클릭해서 상세 설정으로 들어간 뒤, **화면 이름(N)**을 변경해 줄 수 있습니다. 저는 `Banyazavi-WindowsPC`라고 이름을 정했어요.

이렇게 설정을 하면 위 화면 같이 배치가 될 거예요.

## Barrier 시작 프로그램 등록

자, 이제 동작하지 않는 **Start Barrier on startup** 옵션 대신 직접 시작 프로그램에 배리어를 등록해 봅시다.

**메뉴 > 기본 설정**에 **시작 프로그램**이 있어요. 이 항목에서 **추가 (A)**를 선택하면 아래와 같은 화면이 나올 거예요.

![시작 프로그램 추가]({{"/assets/images/2024-04-29/2024-04-29-01-05.png"| relative_url}})

위와 같이 시작 프로그램을 등록해 줍니다. **명령(M)** 부분이 중요한데요. 혹시 APT를 통해서 배리어를 설치한 것이 아니라면, 아래 명령을 통해 배리어의 실행 위치를 확인할 필요가 있어요.

```
$ which barrier
/usr/bin/barrier
```

그럼 이제 배리어 리눅스 서버는 준비가 완료되었습니다. 재부팅 후 다시 로그온하여 트레이에 배리어가 정상적으로 실행되었는지 확인해 봅시다.

# Barrier Windows 클라이언트 설치

[**Barrier 최신 릴리즈 다운로드**](https://github.com/debauchee/barrier/releases){: target="_blank"}

배리어의 설치 파일은 배리어 깃허브의 **Release** 페이지에서 받을 수 있어요. 위 링크를 들어가면 **Assets** 항목에서 설치 파일을 찾을 수 있습니다.

![Barrier 설치 파일]({{"/assets/images/2024-04-29/2024-04-29-01-06.png"| relative_url}})

배리어의 마지막 버전은 **2.4.0**입니다. 이 버전 이후로는 개발이 되고 있지 않은 것 같네요.

`exe` 확장자를 가진 윈도우용 설치 파일로 설치를 진행합니다. 설치가 완료되면, **Launch Barrier**를 선택하여 바로 프로그램을 설정해 봅시다.

## Barrier Windows 클라이언트 설정

![Barrier 설정 - 언어 선택]({{"/assets/images/2024-04-29/2024-04-29-01-07.png"| relative_url}})

언어는 서버와 마찬가지로 **한국어**가 편할 것 같네요.

![Barrier 설정 - 서버/클라이언트 선택]({{"/assets/images/2024-04-29/2024-04-29-01-08.png"| relative_url}})

이번에는 윈도우를 클라이언트로 설정하기로 했으니, **Client**를 선택하고 넘어갑니다.

![Barrier 설정 - Bonjour 설치]({{"/assets/images/2024-04-29/2024-04-29-01-09.png"| relative_url}})

[**Bonjour**](https://ko.wikipedia.org/wiki/봉주르_(소프트웨어)){: target="_blank"}를 설치하면 이후에 **Auto config**를 통해 현재 네트워크상의 배리어 서버를 편하게 찾아 연결할 수 있다는데요. 추가적인 프로그램 설치가 필요하기도 하고, 저희는 서버의 IP를 직접 기입하여 연결할 계획이므로 굳이 설치할 필요는 없어요.

그럼 이제 홈 화면이 보일 거예요. 서버와 마찬가지로 **Barrier > Change Settings (F4)**에 들어가서 기본 설정부터 하겠습니다.

![Barrier 설정 - 설정]({{"/assets/images/2024-04-29/2024-04-29-01-10.png"| relative_url}})

기본적으로는 리눅스의 설정과 비슷합니다. **Elevate**라는 속성이 하나 생긴 것을 제외하면요.

**Elevate**는 **사용자 계정 컨트롤** (이 앱이 디바이스를 변경할 수 있도록 허용하시겠어요?) 이 발생했을 때 배리어 서버 PC에서 이것에 접근할 수 있는지를 결정합니다. **Always**가 아니라면 이 때마다 마우스가 서버 PC의 중앙으로 튕기게 됩니다. 엄격한 보안 상황이 아니라면, **Always**가 제일 편할 거예요.

자, 이제 설정을 완료하고 서버/클라이언트 양쪽의 홈 화면에서 **시작(S)**을 누르면, 안 될 거예요. ~~?!~~ 대부분은 SSL 인증서 오류가 발생하고 있기 때문이에요.

# SSL 인증서 없음 문제 해결

서버/클라이언트 양쪽에서 배리어를 시작한 후 배리어 설정 화면의 **Barrier > Show Log (F2)** 메뉴를 열어 연결 로그를 보면, 보통은 아래 유형의 오류가 발생하고 있을 거예요.

```
# Linux
ERROR: ssl certificate doesn't exist: /home/banyazavi/.local/share/barrier/SSL/Barrier.pem

# Windows
ERROR: ssl certificate doesn't exist: C:\Users\Banyazavi\AppData\Local\Barrier\SSL\Barrier.pem
```

이 오류는 **SSL을 활성화하는데 필요한 인증서가 없다**는 뜻이에요. 이 문제에 대한 해결법은 [**Debian 위키의 Barrier 문서**](https://wiki.debian.org/Barrier#Securing_the_communications){: target="_blank"}에서 잘 알려 주고 있습니다. 따라해 봅시다.

```
$ cd ~/바탕화면
$ openssl req -x509 -nodes -days 365 -subj /CN=Barrier -newkey rsa:4096 -keyout Barrier.pem -out Barrier.pem
```

이 명령을 실행하고 나면, 바탕화면에 `Barrier.pem` 이라는 파일이 생기게 돼요. 이 파일을 SSL 인증서 파일이 존재하지 않는다는 경로에 복사해 넣어 주면 됩니다. 저는 `/home/banyazavi/.local/share/barrier/SSL/Barrier.pem` 경로에 인증서 파일을 넣어 주면 되겠네요.

그리고 이 파일을 윈도우도 마찬가지로 필요한 경로에 붙여 넣습니다. 원칙적으로는 SSL 인증서를 하나 더 만들어 각각 써야 하지만, 개인 사용 수준에서는 이렇게 해도 상관 없을거예요.

그리고 다시 배리어의 **시작(S)** 버튼을 누르면, 서버와 클라이언트에 각각 아래와 같은 화면이 뜨며 연결이 될 거예요.

![Barrier 연결 - 서버 인증]({{"/assets/images/2024-04-29/2024-04-29-01-11.png"| relative_url}})

![Barrier 연결 - 클라이언트 인증]({{"/assets/images/2024-04-29/2024-04-29-01-12.png"| relative_url}})

(핑거프린트 정보는 가렸어요)

**Yes**를 눌러 서로의 인증서를 허용하면, 이제 배리어가 의도한 대로 잘 동작하는 것을 볼 수 있을 거예요!

---

# 그리고 남은 이야기

글 중간에 언급했다시피, 현재 배리어 프로젝트는 개발이 종료된 것 같아요. 그래서 지금은 배리어에서 다시 포크된 프로젝트들이 개발되고 있고, 저는 이 중에서 [**Input Leap**](https://github.com/input-leap/input-leap){: target="_blank"}라는 프로젝트를 소개받기도 했습니다.

하지만 이 프로젝트는 아직 한창 개발중인 상황인 것 같네요. 해당 프로젝트의 깃허브 README에서 안내하듯이 **v3.0.0**이 출시된 후에야 상용으로 사용할 수 있을 것이라고 합니다. 그런 이유로 APT에 패키지가 등록되어 있지 않아 설치 과정이 복잡한 편이고요.

이 프로젝트가 정식 출시되면 다시 소개하도록 하겠습니다. 배리어의 버그를 수정하고 기능을 개선해서 나온다면 정말 좋겠네요.
