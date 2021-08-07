---
title: ODROID-XU4 - OMV 설치 후 파티션 용량 늘리기
excerpt: 16GB 이상의 SD카드나 eMMC를 샀는데, OMV 설치 후 8GB밖에 쓸 수 없다면 본전 생각이 나겠죠.
key: post-2019-09-28-01
tags:
  - ODROID
  - ODROID-XU4
  - openmediavault
  - SBC
---

# 나는 16GB eMMC를 샀는데 왜

![OMV 초기 파일 시스템 화면]({{"/assets/images/2019-09-28/2019-09-28-01-01.png"| relative_url}})

**ODROID XU4**에 **OMV4**를 설치 후 파일 시스템 항목에 들어가면, 위와 같이 7GB 정도의 용량만 잡혀 있는 것을 볼 수 있습니다. 처음에는 **SWAP 파티션** (가상 메모리 비슷한 것) 이 아닐까 하는 생각도 해 보았지만, eMMC 용량의 절반을 잡는 것은 너무하잖아요.

그래서 찾아보니, 이미지 자체가 8GB를 기준으로 만들어져 있다고 합니다.

# 왜 용량이 이것밖에 안 되지?

임베디드 장치에 OS를 설치, 즉 플래싱하는 방법은 마치 윈도우 복원과 비슷합니다. OS 이미지 배포 작업자가 어떤 신묘한 방법으로 해당 임베디드 장치에 대하여 설치작업을 진행한 후, 그 상태의 설치 상황을 디스크째로 이미지로 만드는 방식인데요. 임베디드 장비는 스펙이 모두 같으므로 이러한 방법이 가능해요. 애당초 바이오스와 같은 시스템 초기 환경을 구성하는 방법이 없는 임베디드 장비는 OS를 배포할 수단이 많지가 않습니다.

그러므로 대부분의 임베디드 장비는 설치 기본 ID와 비밀번호 등이 고정되어 있고, 이미지 배포 시 이를 같이 알려주는 방식으로 되어있습니다. 예를 들면 라즈베리 파이의 리눅스 이미지는 ID = "pi", 비밀번호 = "raspberry"죠.

따라서 `.iso` CD 이미지로 설치하는 데스크탑 시스템에서는 이 방법이 의미가 없습니다. 대신에 OMV의 경우 전문가 설치 메뉴를 통하지 않으면, 설치 관리자가 **SWAP 파티션** 등을 고려하여 임의로 파티션을 나누므로 이것이 마음에 들지 않을 수 있습니다. 이때는 전문가 설치 메뉴를 통해 설치하여 조정하거나, Debian을 직접 설치한 후 설치 스크립트를 통해 OMV를 설치하면 됩니다.

# 준비물

- OMV가 설치된 ODROID XU4. `.img` 형식의 플래싱 이미지로 설치하는 다른 임베디드 장치도 가능합니다.
- 일단 OMV 웹 패널에 접속하여 **서비스 > SSH** 메뉴에서 **'루트 로그인을 허용합니다'**를 체크합니다. 또는 장비에 모니터와 키보드를 꽂아 직접 할 수도 있어요.
- OMV 초기 설정은 ID = **'root'**, 비밀번호 = **'openmediavault'**입니다. SSH에 최초 접속 시 바로 비밀번호 변경을 진행하게 됩니다.

# 대상 디스크 찾기

`fdisk` 명령을 통해 OMV가 설치된 디스크를 찾습니다. `dd`로 OS 이미지 플래싱하셨던 분이면 어떤 내용인지 알 거예요.

## 명령

```
root@odroidxu4:~# fdisk -l
```

## 결과

```
...
Disk /dev/mmcblk0: 14.7 GiB, 15758000128 bytes, 30777344 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x01bb5fb4

Device         Boot    Start      End  Sectors  Size Id Type
/dev/mmcblk0p1          8192   139263   131072   64M 83 Linux
/dev/mmcblk0p2        139264 15500000 15360737  7.3G 83 Linux
/dev/mmcblk0p3      15500001 30469567 14969567  7.1G 83 Linux
...
```

여기서 기억해야 할 내용은 OMV가 설치된 디스크의 경로가 `/dev/mmcblk0`이라는 것과, 파티션을 조절해야 하는 파티션이 **'2번 파티션'**, 시작 섹터 번호가 **'139264'**번이라는 점입니다.

# 파티션 조정 실행

`fdisk`는 원래 파티션을 관리하는 프로그램입니다. 아래 명령을 사용하여 파티션 조정을 실행합니다. 위에서 확인한 값에 따라 일부 내용이 변경될 수 있는 점 참고하세요.

## fdisk 실행

```
root@odroidxu4:~# fdisk /dev/mmcblk0

Welcome to fdisk (util-linux 2.29.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): 
...
```

`fdisk`에 잘 접속하였다면 다음 화면을 볼 수 있습니다. 잘 선택되었는지 확신이 서지 않는다면 먼저 `p` 명령을 입력하여 `fdisk -l`에서 확인한 디스크의 내용과 같게 출력되는지 확인합시다.

## 3번, 2번 파티션 삭제

```
...
Command (m for help): d
Partition number (1-3, default 3): 3

Partition 3 has been deleted.

Command (m for help): d
Partition number (1,2, default 2): 2

Partition 2 has been deleted.

Command (m for help):
...
```

3번 파티션과 2번 파티션을 삭제합니다.

## 새 파티션 생성

```
...
Command (m for help): n
Partition type
    p   primary (1 primary, 0 extended, 3 free)
    e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (2048-30777343, default 2048): 139264
Last sector, +sectors or +size{K,M,G,T,P} (139264-30777343, default 30777343): 30777343

Created a new partition 2 of type 'Linux' and of size 14.6 GiB.
Partition #2 contains a btrfs signature.

Do you want to remove the signature? [Y]es/[N]o: N

Command (m for help):
,,,
```

Primary 타입의 새 파티션 2번을 만듭니다.

첫 번째 섹터는 아까 확인한 숫자 **'139264'**를 입력합니다. 기본값인 2048을 입력하면 **'1번 파티션'** (부트 파티션) 앞쪽의 매우 작은 영역을 선택하는 것이 됩니다. 조심하세요!

마지막 섹터는 **'default'** 값을 사용합니다. 즉, 가장 큰 용량을 지정한다는 뜻이에요.

그러면 새 파티션을 만들었다는 말과 함께 시그니처를 삭제하겠냐고 물어보는데, 파티션 포맷이 변할 것은 아니므로 **'N'**을 선택하면 됩니다.

## 파티션 설정 정보 기록

```
...
Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Re-reading the partition table failed.: Device or resource busy

The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or kpartx(8).

root@odroidxu4:~#
...
```

그리고 이제 **'W'**를 눌러 파티션을 기록해주면, 뭔가를 하라는 메시지와 함께 `fdisk`가 종료됩니다. 그러니까, 더 할 게 남았다는 거죠.

## btrfs 파티션 조정 명령 실행

```
...
root@odroidxu4:~# btrfs filesystem resize max /
Resize '/' of 'max'
root@odroidxu4:~#
...
```

명령 설명하기 전에 잠깐. fdisk 명령의 마지막에 `partprobe` 명령이나 `kpartx` 명령을 사용하라고 적혀있는데, 실제로 실행해 보면 `partprobe` 명령은 오류가 나고, `kpartx` 명령은 없다고 나오네요. 그래서 재부팅 후 적용해 보기로 했습니다.

오드로이드 XU 용 OMV 이미지에 사용된 **btrfs** 형식의 파티션은 `fdisk` 이후에 다음과 같은 절차가 필요한 것 같습니다. 위 명령은 루트 경로 (/) 에 대해 최대 용량 (max) 으로 조절하라는 의미입니다.

## 동기화, 그리고 재부팅

```
root@odroidxu4:~# sync
root@odroidxu4:~# reboot
```

왠지 이런 작업 후에는 꼭 해야 할 것 같은 `sync`를 한번 하고요, `reboot` 명령으로 재부팅을 합니다. 그리고 다시 OMV 웹 패널의 파일 시스템 항목을 보면...

# 쨘!

![파티션 확장 후 파일 시스템 화면]({{"/assets/images/2019-09-28/2019-09-28-01-02.png"| relative_url}})

잘 적용된 것을 볼 수 있습니다.

---

# 그리고 남은 이야기, 요약

임베디드 장비이므로 초기 파티션 설정이 똑같다는 점과, `fdisk`의 default 설정이 잘 되어있다는 점을 고려하면, 위의 과정을 다음과 같은 명령어들로만 요약할 수 있습니다.

물론, 어떤 의미인지는 알고 하는 게 좋겠죠.

```
root@odroidxu4:~# fdisk /dev/mmcblk0

Command (m for help): d
Partition number (1-3, default 3): [enter]

Command (m for help): d
Partition number (1,2, default 2): [enter]

Command (m for help): n
Select (default p): p
Partition number (2-4, default 2): [enter]
First sector (2048-30777343, default 2048): 139264
Last sector, +sectors or +size{K,M,G,T,P} (139264-30777343, default 30777343): [enter]
Do you want to remove the signature? [Y]es/[N]o: N

Command (m for help): w

root@odroidxu4:~# btrfs filesystem resize max /

root@odroidxu4:~# sync
root@odroidxu4:~# reboot
```

# 2019. 12. 9 추가

openmediavault의 이미지 다운로드 페이지에 가 보니, 12월 9일 자로 이런 내용과 함께 SBC용 이미지들이 거의 다 지워져 있더라고요.

> The pre-install images have been deprecated in favor of using this guide - [https://forum.openmediavault.org/index.php/Thread/28789-Installing-OMV5-on-Raspberry-PI-s-Armbian-Supported-SBC-s/](https://forum.openmediavault.org/index.php/Thread/28789-Installing-OMV5-on-Raspberry-PI-s-Armbian-Supported-SBC-s/){: target="_blank"}
>
> Source: readme.txt, updated 2019-12-09

대략, ARM 아키텍처의 SBC는 앞으로 ARM 용 Debian (Raspbian이나 Armbian) 을 설치 후, 스크립트를 통하여 OMV를 설치하라는 내용이네요.

따라서 이 글도 거의 무의미해지게 되었네요. Armbian을 통해서 OMV 설치하는 방법을 준비해야겠어요.
