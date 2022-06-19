---
title: "[OMV6 on HC4] ODROID-HC4 OLED 활용"
excerpt: 고작 시계나 IP 주소 정도를 보려고 만 원을 더 주고 OLED 모델을 산 건 아니겠죠.
categories:
  - OMV6 on HC4
sidebar:
  - nav: omv6_nas
tags:
  - ODROID
  - ODROID-HC4
  - ODROID-HC4 with OLED
---

> **[NOTE]**  
> 이 글은 [**ODROID-HC4 with OLED**](https://www.hardkernel.com/ko/shop/odroid-hc4-oled/){: target="_blank"}만을 위한 글입니다.  
> OLED가 없는데 굳이 이 글을 따라 할 필요는 없겠죠.

---

# 11,600원을 더 내고 OLED를 샀는데

![ODROID-HC4 with OLED]({{"/assets/images/2021-12-18/2021-12-18-01-01.png"| relative_url}})

오드로이드 HC4는 두 종류의 제품이 있습니다. 84,700원의 기본형 제품과 96,300원의 OLED 포함 제품이 있죠. 저는 H2도 그렇고 이런 액세서리를 대부분 사는 편인데요. 그래서 HC4도 당연히 OLED 모델로 샀습니다. ~~그래도 리모컨은 사지 말걸~~ 그리고 OLED를 활용해 보려고 했는데요.

## 안 깔려

오드로이드 위키에는 [**OLED 시계 샘플 패키지**](https://wiki.odroid.com/odroid-hc4/application_note/oled){: target="_blank"} 설치법을 안내하고 있는데요. 데비안 APT 패키지로 설치법이 매우 간단한 편이긴 하지만, **데비안 11에서 정상 설치가 안 됩니다.**

오드로이드 HC4는 공식적으로 우분투 20.04를 지원하고, 데비안 11은 출시된 지 얼마 되지 않아 그럴 수도 있겠다는 생각은 들었는데요. 그럼 만원을 투자한 가치가 없는데요... 직접 OLED를 활용하는 프로그램을 만들기도 쉽지 않고요.

하지만 세상에는, 아니 깃허브에는 모든 코드가 있습니다.

# rpardini/sys-oled-hc4 설치하기

깃허브를 찾아보니 [**rpardini/sys-oled-hc4**](https://github.com/rpardini/sys-oled-hc4){: target="_blank"}라는 저장소가 있더라고요. 이 저장소는 [**kobol-io/sys-oled**](https://github.com/kobol-io/sys-oled){: target="_blank"} 저장소를 포크 해서 HC4에 맞게 수정한 것 같네요. 원본 저장소는 [**Helios4**](https://kobol.io/helios4/){: target="_blank"}라는 오픈 소스 나스 하드웨어의 I2C OLED 디스플레이에 사용하기 위한 코드인 것 같습니다. ~~이것도 사 봐야겠다~~

아무튼, 이걸 설치해봅시다.

## 사전 준비

이 저장소의 코드를 가져오기 위해 **git** 패키지를 설치해야 해요. HC3에 SSH로 접속하여 아래 명령어를 입력하면 됩니다. OMV에 APT 패키지를 관리하는 플러그인을 사용하여 설치할 수도 있는데요, 이 글은 번외적인 느낌이 많으므로 여기서는 SSH를 통한 설치법을 안내합니다.

```
$ sudo apt update
$ sudo apt install git
```

그럼 이제 `git` 명령어로 깃허브의 코드를 받아올 수 있습니다.

## sys-oled-hc4 설치

이제부터는 sys-oled-hc4 저장소의 설치법을 그대로 따라가면 됩니다. 우선, 깃허브의 소스를 가져옵니다.

```
$ git clone https://github.com/rpardini/sys-oled-hc4.git
```

그리고 내려받은 저장소의 폴더에 들어가 설치 스크립트를 실행하면 돼요.

```
$ cd sys-oled-hc4
sys-oled-hc4$ sudo ./install.sh
```

설치가 완료되면 OLED에 아래와 같은 화면이 표시되기 시작할 거예요.

```
ld: 25% T: 40C up: 1:23
------------------------------
mem: 155M / 3.6G - 8%
sd: 1.6G / 914.4G - 0%
eth0: xxx.xxx.xxx.xxx
```

상당히 많은 정보가 표시되는 것을 볼 수 있어요. 샘플 패키지의 시계보다는 훨씬 유용한 것 같습니다.

그런데 상태 표시가 너무 긴 것 같네요. 저는 즉각적으로 바뀌는 것을 보고 싶은데 말이죠. 그리고 디스크 용량은 1번 하드디스크만 표시되고 있는 데다가, 이름도 `sd`라고 잘못 나와 있습니다.

물론 이 부분을 수정할 수 있어요.

## sys-oled-hc4 설정 변경

설정을 수정하는 방법 역시 설명서에 잘 나와 있네요. 아래 명령어를 입력하여 설정 파일을 수정하면 됩니다.

```
sudo nano /etc/sys-oled.conf
```

그럼 이제 **nano**라고 하는 텍스트 에디터가 실행되며 설정 파일을 수정할 수 있게 되는데요. 저는 이 부분만 원하는 대로 바꿨어요.

```
# Status refresh interval
refresh = 10
```

이 부분은 OLED의 상태를 어느 주기로 갱신할지를 설정하는 부분입니다. 기본값은 10초로 되어 있네요.

```
# Status refresh interval
refresh = 1
```

저는 이걸 1초로 바꾸었어요. 너무 짧으면 화면 갱신에 과도한 리소스가 사용될 것 같지만, 1초 정도는 괜찮지 않을까요?

```
# Storage Device 1
# Device name
storage1_name = sd

# Device mount path
storage1_path = /

# Storage Device 2
;storage2_name= md0
;storage2_path= /mnt/md0
```

이 부분은 용량을 표시할 폴더의 경로와 이름을 설정하는 곳이에요. 1번 디스크의 위치는 `/`가 맞긴 하는데 여기는 이름을 바꿔야겠네요.

2번 디스크 부분은 `;` 기호로 설정이 비활성화되어 있는데요. 해당 기호를 지우고 앞선 글에서 설정한 2번 디스크의 심볼릭 링크 경로를 넣어줄 거예요.

```
# Storage Device 1
# Device name
storage1_name = hdd1

# Device mount path
storage1_path = /

# Storage Device 2
storage2_name= hdd2
storage2_path= /archive
```

그럼 저는 이렇게 되겠죠. 이제 `Ctrl + O`를 눌러 설정을 저장하고, `Ctrl + X`를 눌러 에디터에서 빠져나오면 됩니다.

하지만 바로 적용이 되지 않죠. HC4를 재부팅 하거나, 아래 명령을 입력하여 서비스를 재시작해 줍니다.

```
$ sudo service sys-oled restart
```

그럼 이제 이렇게 표시되기 시작할 거예요.

```
ld: 25% T: 40C up: 1:23
------------------------------
mem: 155M / 3.6G - 8%
hdd1: 1.6G / 914.4G - 0%
hdd2: 44K / 292.3G - 0%
eth0: xxx.xxx.xxx.xxx
```

원하는 대로 잘 나왔군요. 이제는 OMV에 접속하지 않고도 HC4의 상태를 알 수 있겠네요.

---

# 그리고 남은 이야기

몇 가지 아쉬운 점이 있다면, 표시하는 정보량이 너무 많아 글씨가 작다는 점입니다. 헬리오스4의 OLED 패널은 얼마나 큰지 궁금해지는 부분이네요.

그리고 eth0 랜에 할당된 IP는 저한테는 크게 중요한 정보는 아닌데요. 전송 속도를 표시하는 것도 좋은 생각일 것 같습니다.

물론, 오픈 소스이기 때문에 원한다면 코드를 직접 수정하여 원하는 내용으로 바꿀 수 있습니다. 전송 속도 표시는 한 번 시도해 볼 만 할 것 같아요.
