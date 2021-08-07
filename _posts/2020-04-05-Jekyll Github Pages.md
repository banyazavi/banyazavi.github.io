---
title: "[Docker for a NAS] Jekyll Github Pages"
excerpt: 이 블로그를 만들어봅시다!
key: post-2020-04-05-01
categories:
  - Docker for a NAS
sidebar:
  - nav: docker_nas
tags:
  - Docker
  - Docker Compose
  - Github Pages
  - Jekyll
  - Portainer
---

> **[NOTE]**  
> 이 서비스는 **x86-64** 아키텍처에만 설치할 수 있습니다.

---

사실 제 블로그는 [**Github Pages**](https://github.com/banyazavi/banyazavi.github.io){: target="_blank"}를 통해 운영하고 있습니다. 공짜로 호스팅해 주는 서비스가 있는데 굳이 직접 호스팅 할 필요가 없죠.

그런데도 저는 오늘 소개할 **Jekyll** 서비스도 굉장히 유용하게 쓰고 있습니다. 바로 이 블로그의 **작성 중 페이지**로써 말이죠. 작성 중인 글을 깃허브에 올려서 작업 결과를 확인하기에는 시간이 조금 오래 걸려서, 내부에 서버를 만들어 직접 올려 확인하고 완성본만을 깃허브 페이지에 올리고 있습니다. 매우 편해요. 수정 후 F5만 누르면 확인할 수 있으니까요.

또는 페이지 소스를 공개하기 싫은 분들에게도 유용합니다. 저도 스터디 사이트는 도커에 직접 올려 사용하고 있어요.

---

# Github Pages

![Github Pages 로고]({{"/assets/images/2020-04-05/2020-04-05-01-01.png"| relative_url}})  
[**Github Pages 공식 홈페이지**](https://pages.github.com/){: target="_blank"}

[**Github**](https://github.com/){: target="_blank"}는 현재 가장 유명한 원격 코드 저장소 서비스입니다. 공개된 저장소에 대해서는 과금을 하지 않기 때문에, 많은 오픈 소스 프로젝트가 깃허브를 통하여 공유되고 있죠.

그리고 깃허브에서 특정 방법으로 생성된 저장소에 대해 제공하는 웹 사이트 호스팅이 바로 이 **Github Pages**입니다. 깃허브라는 개발자 친화적인 방법 (이라기보단 비 개발자 비 친화적인 방법) 으로 서비스가 제공되고 있어, 개발자의 개발 블로그나 크리에이터의 포트폴리오 사이트를 주로 호스팅하고 있는 것 같네요.

중요한 것은 이어서 소개할 **Jekyll**이에요. 깃허브 페이지는 사이트 빌더로 지킬을 사용하고 있거든요.

# Jekyll

![Jekyll 로고]({{"/assets/images/2020-04-05/2020-04-05-01-02.png"| relative_url}})  
[**Jekyll 공식 홈페이지**](https://jekyllrb.com/){: target="_blank"}

**Jekyll**은 정적 웹 사이트 변환기입니다. 특정 파일과 구성으로 파일을 배치해 두면, 해당 구조를 분석하여 웹 사이트 형식으로 바꿔줍니다. 그리고 이 경로를 웹 서버로 호스팅하여 사이트를 운영할 수 있죠.

정적 사이트 빌더이기 때문에 기존의 블로그 서비스와는 큰 차이점이 있습니다. **글쓰기** 버튼 같은 게 있는 게 아니고, 사이트 소스를 직접 편집하여 페이지를 작성해야 한다는 거예요. 페이지 작성 방식으로 Markdown을 지원해줘서 다행이지, 아니었으면 웹 초창기의 사이트 만들기처럼 보였을 것 같네요.

소개를 가장한 잡담은 여기까지 하고, 이제부터 지킬을 사용하여 깃허브 페이지와 같은 웹 사이트를 만들어보도록 하겠습니다.

# 사전 작업

이번에는 호스팅할 사이트의 폴더만 구성하면 됩니다. 맘에 드는 테마를 구한 상황에서 시작합니다. 저는 [**Minimal Mistakes**](https://mademistakes.com/work/minimal-mistakes-jekyll-theme/){: target="_blank"}라는 테마가 마음에 들었어요.

## 방법 1. 사이트 기본 구성 폴더를 복사하여 구성

대부분의 테마가 깃허브를 통해 파일을 제공하고 있어요. [**Minimal Mistakes**](https://github.com/mmistakes/minimal-mistakes){: target="_blank"} 역시 깃허브에 파일이 있죠. 페이지를 잘 둘러보면 아래와 같은 메뉴가 있을 겁니다.

![Minimal Mistakes - Clone or download]({{"/assets/images/2020-04-05/2020-04-05-01-03.png"| relative_url}})

**Download ZIP** 버튼을 눌러 프로젝트를 받아줍시다. 받은 다음에 나스 내 적당한 곳에 압축을 풀어 주세요. 저는 늘 그렇듯이 `/docker` 폴더 안에 풀었습니다. 소스가 담겨 있는 폴더의 이름을 바꿔도 좋아요.

## 방법 2. `git clone`을 사용하여 구성

`git` 명령을 사용할 줄 아는 분은 더 편하게 구성할 수 있습니다. 위 그림에서 제공되는 URL을 통해 프로젝트를 클론하면 되요.

```
$ mkdir -p /docker/github-blog
$ cd /docker/github-blog
$ git clone https://github.com/mmistakes/minimal-mistakes.git
```

그러면 `/docker/github-blog` 폴더 아래 프로젝트의 이름으로 폴더가 만들어지게 됩니다. 이 폴더 역시 이름을 바꿔도 좋습니다.

그리고 이게 끝입니다. `_config.yml` **파일 수정과 콘텐츠 구성**은 직접 해야 하지만요...

# 도커 컨테이너 생성

이제는 익숙하죠? 아래의 **docker-compose** 파일 내용을 포테이너의 **Add Stack > Web editor** 아래의 공간에 복사하여 붙여넣습니다.

```
version: "2"
services:
  jekyll:
    image: jekyll/jekyll:3.8.5
    ports:
      - 4000:4000
    command: jekyll serve
    volumes:
      - /docker/github-blog/minimal-mistakes:/srv/jekyll
    environment:
      - LANG=ko_KR.UTF-8
      - LANGUAGE=ko_KR
      - TZ=Asia/Seoul
      - LC_ALL=ko_KR.UTF-8
    restart: unless-stopped
```

`volumes`의 `/docker/github-blog/minimal-mistakes` 경로는 여러분의 사이트 소스 경로입니다. 해당 폴더의 최상위에 `_config.yml` 파일이 있다면 거기가 맞아요.

다른 서비스 또한 마찬가지지만, 여러 사이트를 동시에 운영하려면 `ports` 설정의 `4000:4000` 앞부분을 바꿔주면 됩니다. `3999:4000` 이렇게 적어주면 이 사이트는 **3999**번 포트로 접속할 수 있어요.

# 서비스 동작 확인

최초 셋업할 때는 의존 패키지들의 설치에 많은 시간이 필요합니다. 포테이너에서 해당 컨테이너의 **Logs**를 확인하시면 진행 상황을 알 수 있어요. 혹시 오류가 발생할 수도 있으니 로그를 지켜보는 것이 좋겠네요.

구성이 완료되었다면 **http://나스_IP:4000**에 접속해 봅시다. 위에서 포트를 바꿨다면 거기에 맞게 바꿔 접속해 주세요.

![Minimal Mistake 초기 홈페이지]({{"/assets/images/2020-04-05/2020-04-05-01-04.png"| relative_url}})

Minimal Mistake의 기본 페이지가 나오네요. 다른 테마를 선택하셨거나, 사용하던 깃허브 페이지를 가져온 분은 거기에 맞는 화면이 뜰 거예요.

이후에 페이지를 수정하면 거의 즉시 수정사항이 반영됩니다. 단, `_comfig.yml` 파일의 수정은 컨테이너를 재시작해야 적용된다는 점에 주의하세요.

---

# 그리고 남은 이야기

간혹 컨테이너를 구성해도 사이트가 정상적으로 만들어지지 않을 때가 있습니다. `.yml` 구성 파일의 문법 오류로 파싱에 실패했거나, 이 서비스에서 지원하지 않는 플러그인이 있기 때문일 수도 있습니다. 전자의 경우 컨테이너 로그를 확인하여 디버깅해야 하고, 후자의 경우에는 지원되는 테마를 구해야 해요.

눈썰미가 좋은 분은 왜 지킬 **3.8.5** 버전을 사용하는지 궁금할 수도 있습니다. 큰 이유는 없고요, [**깃허브 페이지의 제공 버전**](https://pages.github.com/versions/){: target="_blank"}을 그대로 가져온 거예요. 만약 이 글을 작성된 지 오래 뒤에 읽는 분은 해당 페이지에서 지킬 버전을 확인하여 적용해 주세요. 깃허브 페이지와 무관하게 사이트를 구성하거나, 최신 버전으로 사용하려는 분은 다른 버전을 사용해도 좋습니다.
