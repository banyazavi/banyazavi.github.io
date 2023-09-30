---
title: ODROID-H2용 ESXi 6.7 설치 이미지 만들기
excerpt: 오드로이드 H2에 여러 가지 OS를 설치하고 싶으면 따라 해 봅시다.
tags:
  - ESXi
  - Hypervisor
  - ODROID
  - ODROID-H2
---

# ESXi란?

VMware에서 제품 개요를 가져왔어요.

> VMware ESXi: 맞춤형 베어메탈 하이퍼바이저  
> 물리적 서버에 바로 설치되는 강력한 베어메탈 하이퍼바이저에 대해 알아보십시오. VMware ESXi는 기반 리소스에 대한 직접 액세스와 제어가 가능하므로 하드웨어를 효과적으로 파티셔닝하여 애플리케이션을 통합하고 비용을 절감할 수 있습니다. 또한 업계 최고의 효율적인 아키텍처로서 안정성과 성능, 지원에 대한 표준이 되고 있습니다.  
> \- [VMWare ESXi 제품 개요](https://www.vmware.com/kr/products/esxi-and-esx.html){: target="_blank"}

**VMWare ESXi**는 VMWare에서 만든 **베어메탈 하이퍼바이저**<sup>Bare Metal Hypervisor</sup>입니다. 그러니까 호스트 하이퍼바이저와는 달리 하드웨어 위에 가상화 환경이 바로 설치되는... 어렵네요.

간단하게 설명하자면, 컴퓨터를 켜면 바로 VMWare (가상 머신 관리 도구) 가 실행되는 시스템을 의미합니다. 호스트 운영체제 (윈도우나 우분투 리눅스 같은) 없이 바로 설치되기 때문에, 호스트 운영체제 구동에 필요한 자원 낭비를 하지 않아 최적의 성능을 뽑아낼 수 있어요.

그렇다고 전문적인 서버도 아닌 오드로이드 H2에 대단한 성능을 바랄 수는 없겠죠.

# ODROID H2에 ESXi 설치하기

베어메탈 하이퍼바이저가 여러 종류가 있겠지만, 홈서버 레벨에서 ESXi가 주로 추천되는 이유는 **공짜**이기 때문일 겁니다. VMWare에 회원가입을 하면 ESXi 설치파일과 라이선스를 무료로 받을 수 있거든요.

그렇다고 무턱대고 파일을 받아 실제로 설치를 진행하다 보면 잘 안 되는데, 그건 오드로이드 H2의 NVMe와 NIC 드라이버가 기본 포함되어 있지 않기 때문입니다.

다행히도 오드로이드 포럼에 설치법이 공유되어 있어, 이 글에서는 그 설치법을 번역하여 소개해 보도록 하겠습니다.

[# Complete guide for running ESXi 6.7.0 update 03 on Odroid H2 with NVMe SSD](https://forum.odroid.com/viewtopic.php?t=32854#p266578){: target="_blank"}

# 준비물

- ODROID H2
- NVMe SSD (NVMe를 지원하도록 드라이버를 설치하는 것이므로 꼭 장착되어 있을 필요는 없습니다)
- Windows 10이 설치된 컴퓨터
- 2개의 USB 3.0 메모리 (하나는 설치 이미지가 담긴 USB, 나머지 하나는 ESXi가 설치될 USB) - USB에 ESXi를 설치할 계획이 아니면 1개만 있어도 됩니다.

그리고 다음 파일들도 준비합니다.

- VMWare에서 제공하는 ESXi 6.7.0 update-from-esxi6.7-6.7_update03. 로그인 후, 여기에서 검색하여 받을 수 있습니다. [https://my.vmware.com/group/vmware/patch#search](https://my.vmware.com/group/vmware/patch#search){: target="_blank"}
- Realtek 8168/8111/8411/8118 기반 NIC 드라이버 [https://vibsdepot.v-front.de/wiki/index.php/Net55-r8168
](https://vibsdepot.v-front.de/wiki/index.php/Net55-r8168){: target="_blank"}
- NVMe 드라이버 [https://hostupdate.vmware.com/software/ ... 169922.vib](https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/esx/vmw/vib20/nvme/VMW_bootbank_nvme_1.2.1.34-1vmw.670.0.0.8169922.vib){: target="_blank"} 이 링크는 누르면 이상한 페이지가 나올 텐데, 링크를 **우클릭 -> 다른 이름으로 링크 저장**을 클릭하여 내려받도록 합니다.
- Rufus (`.iso` 파일을 USB에 설치하기 위한 프로그램) [https://rufus.ie/](https://rufus.ie/){: target="_blank"}

그리고 이 파일들을 한 폴더에 몰아넣고, **CMD** 환경에서 접근하기 좋은 위치에 둡니다. `C:\` 또는 `D:\` 같은 곳이 찾기 쉬울 거예요.

# 드라이버 통합된 설치 이미지 만들기

우선 윈도우 명령 프롬프트를 **'관리자 권한으로 실행'**합니다. 그리고 `cd` 명령으로 다운 받아둔 폴더가 있는 곳으로 이동합시다. 저는 `C:\ESXi`에 파일을 모아놨어요.

## PowerShell 실행

```
c:\>cd ESXi

c:\ESXi>powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

새로운 크로스 플랫폼 PowerShell 사용 https://aka.ms/pscore6

PS C:\ESXi>
...
```

PowerShell 실행 환경으로 전환합니다. 이제 커맨드라인 앞에 **'PS'**가 붙어 나오는 것을 볼 수 있습니다. 사실 바로 PowerShell을 실행시켜 사용할 수도 있는데요, 일단 원문의 가이드를 충실히 따라가도록 합시다.

## PowerShell 스크립트 실행 제한 해제

```
...
PS C:\ESXi> Set-ExecutionPolicy Unrestricted
Set-ExecutionPolicy : Windows PowerShell에서 실행 정책을 업데이트했지만 좀 더 구체적인 범위에서 정의된 정책에 의해 설정
이 재정의되었습니다. 재정의로 인해 셸은 현재 유효 실행 정책인 RemoteSigned을(를) 유지합니다. 실행 정책 설정을 보려면 "G
et-ExecutionPolicy -List"를 입력하십시오. 자세한 내용은 "Get-Help Set-ExecutionPolicy"를 참조하십시오.
위치 줄:1 문자:1
+ Set-ExecutionPolicy Unrestricted
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (:) [Set-ExecutionPolicy], SecurityException
    + FullyQualifiedErrorId : ExecutionPolicyOverride,Microsoft.PowerShell.Commands.SetExecutionPolicyCommand
PS C:\ESXi>
...
```

커스텀 ESXi 이미지 생성 중 스크립트 실행이 필요하므로, 실행 정책을 수정하는 과정입니다.

## VMWare CLI 모듈 설치

```
...
PS C:\ESXi> Install-Module -Name VMware.PowerCLI

신뢰할 수 없는 리포지토리
신뢰할 수 없는 리포지토리에서 모듈을 설치하는 중입니다. 이 리포지토리를 신뢰하는 경우 Set-PSRepository cmdlet을
실행하여 InstallationPolicy 값을 변경하십시오. 'PSGallery'에서 모듈을 설치하시겠습니까?
[Y] 예(Y)  [A] 모두 예(A)  [N] 아니요(N)  [L] 모두 아니요(L)  [S] 일시 중단(S)  [?] 도움말 (기본값은 "N"): A
PS C:\ESXi>
...
```

PowerShell용 VMWare 도구 설치를 위해 모듈을 설치합니다. 신뢰할 수 없다고 경고가 뜨는데, 윈도우 입장에서는 신뢰할 수 없는 게 맞고 우리는 필요한 것도 맞으니 (...) 그냥 진행합니다.

## 이미지 생성을 위한 파일 추가하기

```
...
PS C:\ESXi> Add-EsxSoftwareDepot .\net55-r8168-8.045a-napi-offline_bundle.zip

Depot Url
---------
zip:C:\ESXi\net55-r8168-8.045a-napi-offline_bundle.zip?index.xml


PS C:\ESXi> Add-EsxSoftwareDepot .\update-from-esxi6.7-6.7_update03.zip

Depot Url
---------
zip:C:\ESXi\update-from-esxi6.7-6.7_update03.zip?index.xml


PS C:\ESXi> Get-EsxSoftwarePackage -PackageUrl .\VMW_bootbank_nvme_1.2.1.34-1vmw.670.0.0.8169922.vib

Name                     Version                        Vendor     Creation Date
----                     -------                        ------     -------------
nvme                     1.2.1.34-1vmw.670.0.0.8169922  VMW        2018-04-03 오...


PS C:\ESXi>
...
```

다운 받아둔 파일과 NVMe 드라이버용 스크립트를 실행합니다. 파일명과 경로가 맞는지 다시 한번 확인하세요!

## ESXi 설치 정보 구성하기

```
...
PS C:\ESXi> New-EsxImageProfile -cloneprofile ESXi-6.7.0-20190802001-standard -Name ODroidH2 -Vendor Custom

Name                           Vendor          Last Modified   Acceptance Level
----                           ------          -------------   ----------------
ODroidH2                       Custom          2019-08-08 ...  PartnerSupported


PS C:\ESXi> Set-EsxImageProfile -ImageProfile "ODroidH2" -AcceptanceLevel CommunitySupported

Name                           Vendor          Last Modified   Acceptance Level
----                           ------          -------------   ----------------
ODroidH2                       Custom          2019-09-29 ...  CommunitySupported


PS C:\ESXi> Add-EsxSoftwarePackage -ImageProfile "ODroidH2" -SoftwarePackage "nvme 1.2.1.34-1vmw.670.0.0.8169922"

Name                           Vendor          Last Modified   Acceptance Level
----                           ------          -------------   ----------------
ODroidH2                       Custom          2019-09-29 ...  CommunitySupported


PS C:\ESXi> Add-EsxSoftwarePackage -ImageProfile "ODroidH2" -SoftwarePackage net55-r8168

Name                           Vendor          Last Modified   Acceptance Level
----                           ------          -------------   ----------------
ODroidH2                       Custom          2019-09-29 ...  CommunitySupported


PS C:\ESXi>
...
```

설치된 이미지의 프로파일을 작성합니다. 요약하자면 ODroidH2라는 이미지를 생성하고, 아까 추가한 드라이버들을 추가하는 과정이에요. 프로파일을 생성할 때 사용하는 **'Name'**과 **'Vendor'** 부분은 후에 ESXi 관리 패널의 **'호스트 > 구성'** 부분에 적히게 됩니다.

필요하다면 다른 이름과 벤더명으로 작성할 수도 있어요. 대신에 아래 내용도 그것에 맞게 바꿔야 합니다.

## .iso 파일 생성하기

```
...
PS C:\ESXi> Export-EsxImageProfile -ImageProfile "ODroidH2" -ExportToIso -FilePath .\ODroidH2.iso
PS C:\ESXi>
```

잘 실행되었으면 설치파일들을 저장한 폴더에 **'ODroidH2.iso'** 이름으로 파일이 생성되어 있게 됩니다.

---

# 그리고 남은 이야기

이 글을 쓴 이후에 [**Proxmox VE**](https://www.proxmox.com/en/proxmox-ve){: target="_blank"}이라는 같은 다른 무료 하이퍼바이저도 설치해봤습니다.

프록스목스 VE의 경우는 Debian 위에 올라가므로, 오드로이드 H2의 모든 확장 포트들을 이용할 수 있는 장점이 있습니다. (I2C LCD 모듈을 사용하거나, eMMC 슬롯도 사용할 수 있어요) 대신에 한국어가 지원되지 않고, 가상머신 위 도커 설정에 오류가 발생하는 문제가 있더라고요. 한 일주일 설정해 보다가 포기했습니다.

후에 뭔가 알려드릴 만큼 알게 되면 프록스목스 VE에 관한 내용도 작성해야겠네요.
