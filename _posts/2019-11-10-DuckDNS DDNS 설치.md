---
title: "[OMV4 NAS] DuckDNS DDNS 설치"
excerpt: 언제까지 IP를 입력해서 서비스를 사용할 수는 없습니다.
key: post-2019-11-10-01
categories:
  - OMV4 자작 NAS 구축
sidebar:
  - nav: omv4_nas
tags:
  - DDNS
  - DuckDNS
  - NAS
  - openmediavault
---

**DDNS**<sup>Dynamic DNS</sup>란 유동 IP에 대해 **DNS**<sup>Domain Name System</sup>를 제공하는 방법입니다. DDNS 서버에 주기적으로 IP 갱신 요청을 보내, 설령 해당 도메인의 IP가 변경되었더라도 빠르게 이를 반영하는 방식으로 동작합니다.

---

# 사실 이건 OMV에서 제공하지 않는다

사실 이 글을 주제에 넣는 게 맞는지 고민을 했습니다. 왜냐하면, 이건 OMV에서 지원하는 서비스가 아니거든요. 하지만 외부 접속이 필요한 시점에서 도메인은 꼭 필요한 것이 되므로, 나스의 설치를 위해서라면 꼭 필요한 과정이라고 생각해서 넣었습니다.

그리고 굳이 이 글의 방법으로 DDNS를 설치할 필요도 없는데요.

# 공유기에서 DDNS 서비스를 지원한다

사실 DDNS는 홈 네트워크를 외부에서 쉽게 접속하려는 방법이고, 이 방법을 위해서는 24시간 동작하는 컴퓨팅 장비가 필요하므로, 이 조건을 충족하는 장비인 공유기에서도 DDNS를 지원합니다.

ipTIME의 경우 **iptime.org**와 **ipdisk.co.kr**를 제공하고 있고, ASUS의 경우 **asuscomm.com** 이름으로 제공하며, 그 외 다른 유명 DDNS 연결도 지원하고 있어요.

하지만 우리 역시 24시간 돌아가는 나스가 있으니! ~~공유기별 방법 다 안내하기도 귀찮고~~ 나스를 통하여 설치하는 방법을 소개하겠습니다.

# Duck DNS

![Duck DNS 소개]({{"/assets/images/2019-11-10/2019-11-10-01-01.png"| relative_url}})

[**Duck DNS**](https://www.duckdns.org/){: target="_blank"}는 무료 DDNS 서비스 중 하나입니다.

다른 DDNS 서비스도 있지만, Duck DNS도 꽤 안정적인 서비스이고 서브도메인에 대한 연결도 지원하고 있으므로 나스용으로는 꽤 괜찮은 것 같아요. 그리고 DDNS 갱신도 쉬운 편이에요.

우선, 가입 후 도메인 등록부터 해 봅시다.

# Duck DNS에 도메인 등록

![Duck DNS 등록 화면]({{"/assets/images/2019-11-10/2019-11-10-01-02.png"| relative_url}})

소셜 로그인으로 로그인하고 나면 위와 같은 화면이 뜹니다. **토큰은 개인정보로 DDNS 갱신에 필요하니 외부에 노출하는 일이 없도록 주의합니다.** ~~누가 대신 갱신해 버릴 수도~~

그리고 하고 싶은 도메인을 적은 뒤 **add domain** 버튼을 누르면, 중복 검사가 완료된 후에,

![Duck DNS 등록 완료]({{"/assets/images/2019-11-10/2019-11-10-01-03.png"| relative_url}})

이렇게 도메인이 등록됩니다!

이 작업을 하는 컴퓨터와 나스가 같은 공유기 아래 있다면, **update ip** 버튼을 눌러 DDNS와 IP를 연동시킬 수 있어요. 그러므로 앞으로 5분마다 이 사이트에 접속하여 저 버튼을 눌러주면 됩니다 :) ~~다음 글은 5분마다 갱신 버튼을 누를 불침번을 정하는 방법입니다.~~

5분마다 이걸 눌러줄 수는 없으니, 나스가 자동으로 갱신하게 해야겠죠.

# NAS에 자동 갱신 설정하기

![Duck DNS Install]({{"/assets/images/2019-11-10/2019-11-10-01-04.png"| relative_url}})

사실 Duck DNS에서 시스템별 자동 갱신 방법을 제공하고 있어요. [**Duck DNS Install**](https://www.duckdns.org/install.jsp){: target="_blank"}

이 방법 그대로 따라 해 봅시다. 우선 `ssh`에 접속해서,

```
$ sudo mkdir -p /root/duckdns
$ sudo vi /root/duckdns/duck.sh
```

root의 홈 디렉토리 아래에 **duckdns** 폴더를 만들고, 그 아래 **duck.sh** 파일을 만들어 편집합니다. 그리고 파일에 다음과 같이 내용을 넣으세요.

```
echo url="https://www.duckdns.org/update?domains={사용 도메인}&token={할당받은 토큰}&ip=" | curl -k -o ~/duckdns/duck.log -K -
```

그리고 root 유저가 실행할 수 있도록 권한을 조정합니다.

```
$ sudo chmod 700 /root/duckdns/duck.sh
```

그리고 **시스템 > 예약된 작업**에서 이 스크립트를 추가합니다.

![Duck DNS 갱신 스크립트 예약]({{"/assets/images/2019-11-10/2019-11-10-01-05.png"| relative_url}})

스크립트가 한번 실행되고 나면 등록된 DDNS 주소로 나스에 접속할 수 있습니다.

혹시 공유기의 설정 페이지가 뜬다면 공유기의 포트포워딩 설정을 수정하고, 아예 연결되지 않는다면 Duck DNS 사이트에 접속하여 공인 IP가 정확하게 등록되었는지 확인해 봅시다.

---

# 그리고 사실 남은 이야기

원래 갱신 스크립트를 생성하고 이를 예약 실행하는 복잡한 방법만 알고 있었는데, 이번에 글을 쓰기 위해 알아보다가 그만...

**freenas용 설치 방법을 활용한 간단 설정법을 알게 되었어요.**

방법은 매우 간단합니다. 다음의 스크립트를 예약된 작업의 명령에 추가시켜 주면 됩니다. 프리나스와는 `curl` 명령의 위치가 다르므로 `/usr/local/bin/curl`만 `/usr/bin/curl`로 바꾸어 줍니다. 혹시 `curl`이 설치되지 않았으면 `$ sudo apt-get install curl` 명령을 통해 `curl`을 설치해 주세요.

```
/usr/bin/curl http://www.duckdns.org/update/{사용 도메인}/{할당받은 토큰}
```

그리고 아래와 같이 설정해 주면...

![간단한 Duck DNS 갱신 스크립트 예약]({{"/assets/images/2019-11-10/2019-11-10-01-06.png"| relative_url}})

되네요. ~~앞에 쓴 글 다 날아가는 소리~~

저도 지금은 이 방법으로 쓰고 있습니다.
