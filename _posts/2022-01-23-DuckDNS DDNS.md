---
title: "[OMV6 on HC4] DuckDNS DDNS"
excerpt: 외부에서 나스에 접속하려면 도메인 하나쯤은 가지고 있는 게 좋겠죠?
categories:
  - OMV6 on HC4
sidebar:
  - nav: omv6_nas
tags:
  - DDNS
  - DuckDNS
  - NAS
  - OMV6
  - openmediavault
---

**DDNS**<sup>Dynamic Domain Name System</sup>란 유동 IP에 대해 도메인을 제공해 주는 서비스를 말합니다. 나스가 속해 있는 공유기의 외부 IP에 대한 알파벳 주소를 만들어 주는 거예요. 이 과정이 끝나면 우리는 `xxx.duckdns.org`라는 주소로 집 밖에서 나스에 접속할 수 있습니다.

물론 아직 공유기 설정에서 외부 접속을 허용하지도 않았고, 외부 접속이 가능한 서비스도 설치한 것이 없으니 결과를 바로 확인해 볼 수는 없겠죠.

---

# Duck DNS

![Duck DNS 소개]({{"/assets/images/2022-01-23/2022-01-23-01-01.png"| relative_url}})

[**Duck DNS**](https://www.duckdns.org/){: target="_blank"}는 유명한 무료 DDNS 서비스 중 하나입니다. 그리고 2차 서브도메인에 대한 지원도 하므로, 나중에 역방향 프록시를 통해 도메인을 잘게 쪼갤 때도 유용해요. 그리고 나스를 통한 IP 갱신도 쉬운 편이고요. 우리는 이 서비스를 통해 DDNS를 설정해 보도록 할 거예요.

# Duck DNS 서브도메인 등록

![Duck DNS 홈]({{"/assets/images/2022-01-23/2022-01-23-01-02.png"| relative_url}})

**Duck DNS**의 주소는 [https://www.duckdns.org/](https://www.duckdns.org/){: target="_blank"}이에요. 이 사이트에는 다섯 가지의 로그인 옵션이 있어요. 사용하고 싶은 것을 눌러 로그인하면 됩니다. 저는 구글 계정으로 로그인했어요.

![Duck DNS 초기 화면]({{"/assets/images/2022-01-23/2022-01-23-01-03.png"| relative_url}})

로그인하면 위와 같은 화면이 나옵니다. 원래는 **token** 옆에 이상한 문자열이 있는데요. **이 값은 보안상 중요하기 때문에 노출하면 안 됩니다!** 그래서 저도 가렸어요.

서브도메인을 하나 등록해 봅시다. `http://` 와 `.duckdns.org` 사이에 사용하고 싶은 서브도메인을 입력하고 **add domain** 버튼을 누릅니다. 누구나 Duck DNS의 서브도메인을 등록할 수 있으므로, 너무 유명한 것은 등록이 되지 않을 수도 있어요. 저는 `banyazavi`라는 서브도메인을 등록해 보기로 했습니다.

![Duck DNS 도메인 등록]({{"/assets/images/2022-01-23/2022-01-23-01-04.png"| relative_url}})

등록에 성공하면 위와 같은 화면이 떠요. **current ip**에는 현재 PC (또는 공유기) 의 외부 IP 주소가 기본으로 적혀 있는데요, 스크린샷에서는 해당 부분을 가려두었어요.

Duck DNS에 서브도메인 등록은 이게 끝이에요. 이제, 나스에서 IP를 주기적으로 갱신해 주도록 설정하기만 하면 됩니다.

# DDNS IP 갱신 요청 자동화

Duck DNS에서는 [**다양한 환경별 DDNS 갱신 방법**](https://www.duckdns.org/install.jsp){: target="_blank"}을 안내하고 있는데요. 우리는 **freenas**의 갱신 방법을 응용하여 적용해 볼 거예요.

## curl 설치 확인

![서비스 > Apt Tool > Packages (curl 설치 확인)]({{"/assets/images/2022-01-23/2022-01-23-01-05.png"| relative_url}})

우선 `curl`이 설치되어 있어야 합니다. 그래서 저번 시간에 [**Apt Tool을 이용한 curl 설치**](/2022-01-16/Apt-Tool-패키지-관리자-GUI)를 배웠죠. 정상적으로 설치하였다면 **서비스 > Apt Tool > Packages** 메뉴에서 curl의 **설치됨** 항목이 **Yes**로 표시되어 있을 거예요. 아니라면 이전 글을 진행하여 curl을 설치하고 돌아오면 됩니다.

## 예약된 작업 등록

![시스템 > Scheduled Tasks]({{"/assets/images/2022-01-23/2022-01-23-01-06.png"| relative_url}})

그리고 **시스템 > Scheduled Tasks** 메뉴에 접속합니다. 기존에 아무 작업도 등록하지 않았다면 그림과 같이 비어 있을 거예요. 상단의 **생성** 버튼을 눌러 작업을 추가합니다.

![시스템 > Scheduled Tasks > 생성]({{"/assets/images/2022-01-23/2022-01-23-01-07.png"| relative_url}})

Duck DNS에 IP 갱신을 요청하는 주기는 5분이에요. 이 주기로 작업을 생성하려면 아래와 같이 시간을 설정해 줍니다.

- 실행 시간: 유효 기간
- 분: 5 (매 N분마다)
- 시간: *
- 달의 하루: *
- 달: *
- 요일: *

사용자는 `nobody` 또는 `root`로 하면 됩니다. `root`에는 너무 많은 권한이 있어 위험할 수 있으니, 저는 `nobody`를 주었어요.

명령이 매우 중요한데요. `/usr/bin/curl http://www.duckdns.org/update/` + 서브도메인 주소 + `/` + token 값을 넣으면 됩니다. 저는 `banyazavi` 서브도메인에 대한 갱신 작업을 만들고 있기 때문에 `/usr/bin/curl http://www.duckdns.org/update/banyazavi/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`이 되겠죠.

![시스템 > Scheduled Tasks (설정 후)]({{"/assets/images/2022-01-23/2022-01-23-01-08.png"| relative_url}})

작업을 저장하고 나면 목록이 이렇게 바뀌어 있을 거예요. 방금 생성한 작업의 **일정**이 `*/5 * * * *`이면 시간 설정이 잘 된 거예요. 이제 공유기의 IP가 바뀌어도 5분 안에 DDNS 갱신이 이루어지게 됩니다.

# 갱신 테스트

하지만 과연 잘 될까 싶죠? 테스트를 해 봅시다!

![Duck DNS 임의 IP 갱신]({{"/assets/images/2022-01-23/2022-01-23-01-09.png"| relative_url}})

일단, Duck DNS 사이트에서 `current ip` 부분에 임의의 IP를 넣고 **update ip** 버튼을 누릅니다. 저는 `8.8.8.8`이라는 IP로 업데이트를 해 보았어요.

![Duck DNS 임의 IP 갱신 후]({{"/assets/images/2022-01-23/2022-01-23-01-10.png"| relative_url}})

업데이트가 잘 되면 이제 `banyazavi.duckdns.org` 도메인은 `8.8.8.8`이라는 IP를 보고 있겠죠. 그리고 5분 내에 예약된 작업에 의해 다시 원래의 공유기 외부 IP로 바뀌게 될 거예요.

5분을 기다리고 난 후에 Duck DNS의 서브도메인 목록을 봐도 괜찮지만...

![시스템 > Scheduled Tasks (실행)]({{"/assets/images/2022-01-23/2022-01-23-01-11.png"| relative_url}})

작업을 기다리지 않고 직접 실행하는 방법도 있습니다. 다시 **시스템 > Scheduled Tasks**로 들어가서 아까 만든 작업을 선택하고, 상단의 **실행** 버튼을 누릅니다.

![Run scheduled task]({{"/assets/images/2022-01-23/2022-01-23-01-12.png"| relative_url}})

그러면 이런 창이 뜨게 되는데요. **시작** 버튼을 누르면 지정한 명령을 바로 실행시킬 수 있어요. 정상적으로 실행되었다면 위와 같은 메시지가 출력되고, 마지막에 `OK`라는 문구를 볼 수 있을 거예요.

![Duck DNS (갱신 후)]({{"/assets/images/2022-01-23/2022-01-23-01-13.png"| relative_url}})

그리고 다시 Duck DNS에 들어가 보면, 갱신된 IP가 적용되어 있는 것을 볼 수 있습니다. 스크린샷에서는 갱신된 IP를 가려 놓아서 확인이 되지 않는데요. 아무튼 잘 갱신되었어요!

---

# 그리고 남은 이야기

서론에서 얘기했듯이, 외부 접속이 가능한 서비스가 아직 없으므로 이 도메인을 이용해 할 수 있는 것은 별로 없어요.

하지만 이 다음 글부터는 외부 접속을 유용하게 사용할 수 있습니다.
