---
title: "휴고 서버에 직접 구동시키기"
description: "브류를 설정한 다음 휴고를 설치하고 구동하는 방법입니다."
date: 2021-09-01T14:21:27+10:23
categories: ["리눅스"]
tags: ["homebrew", "hugo"]
image: "thum.png"
draft: false
---

## 주의사항

### 2022.03.20 

하드디스크가 문제가 생겼었고, 교환 받은 디스크도 종종 긁히는 소리가 갑자기 생기는데 Hugo를 직접 호스팅 하기 전까지 이런적은 없었습니다.  
hugo server 로 24시간 작동시키는것이 다른 문제가 있는지 의심하는 중입니다. 주의하시길 바랍니다.

## 서론

학습했던 것들을 정리하기 위해 정적 페이지를 알아보다 HUGO와 Github page를 사용한 대중적인 게시글들을 확인할 수 있었습니다.
Github page를 사용하면 관리하기 편리할 것 같지만 호스팅 경험을 위하여 예전부터 동작 중인 개인 서버를 통해 가동해 보겠습니다.

## Homebrew 설치

2021년 08월 29일 기준 데비안 10.10 버전의 apt 패키지 관리자에서는 최신 버전의 hugo를 지원하지 않고 있습니다.
사용하고 싶은 테마가 최신버전의 hugo에서만 지원하고 있기 때문에 수동으로 받아 설치하거나 다른 패키지 관리자를 사용하는 것이 방법일 것입니다. 여기서는 Mac OS 에서 주로 사용하는 Homebrew를 사용했습니다.


```bash
sudo apt-get install build-essential curl file git 
sh -c "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install.sh)"
```
여기서 brew는 설치, 설치후 사용할 때 root의 권한으로 동작시킬 수 없습니다. 

설치가 완료되었다면 자기가 사용하는 쉘에 설치된 경로를 포함시켜야 합니다.
데비안 기준 기본 쉘은 bash입니다.
```bash
nano ~/.bashrc
```
또는 사람들이 주로 사용하는 zsh는
```bash
nano ~/.zshrc
```
를 사용해 아래를 추가해주고
```bash
export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"
export MANPATH="/home/linuxbrew/.linuxbrew/share/man:$MANPATH"
export INFOPATH="/home/linuxbrew/.linuxbrew/share/info:$INFOPATH"
```
저장 후 자신의 쉘에 맞게
```bash
source ~/.bashrc
```
또는
```bash
source ~/.zshrc
```
를 실행시켜 주면 적용되어 바로 사용 할 수 있습니다.

만약 저처럼 알기 힘든 오류를 출력하여 진행 할 수 없을 때는 아래와 같은 방법으로 해결방법을 찾을 수 있습니다.
```bash
brew doctor
```

저 같은 경우에는 해당 명령어를 추천했으며 올바르게 동작했습니다.

```bash
rm -rf /usr/local/Cellar /usr/local/.git && brew cleanup
```

모든 것이 완료되었다면 아래처럼 테스트 해볼 수 있습니다.

```bash
brew update
brew install hello
hello
brew remove hello
```

## HUGO 설치 및 실행

```bash
brew install hugo
```
후
```bash
hugo version
```
하면 올바르게 최신 버전이 설치되었음을 확인 할 수 있습니다.
이후 원하는 디렉토리에 들어가 새로운 사이트를 생성하고

```bash
hugo new site docs.9bon.org
```
원하는 테마를 받아 올 수 있습니다.
```bash
cd themes
https://github.com/CaiJimmy/hugo-theme-stack.git
```
시험삼아 간단한 테스트를 하기위해 설정파일을 조작할 것인데
기본으로 생기는 설정파일이 config.toml인것에 반해 제작자 가이드는 config.yaml에 맞추어져 있습니다. 편의상인 것으로 추측됩니다.
원래 있는 config.toml을 치우고 config.yaml을 만들어주면 yaml타입으로 설정할 수 있습니다.

```bash
cd .. / mv ./config.toml ./__config.toml 나노 config.yaml
```
 




이후에 설정파일은 테마 제작자 깃허브 https://github.com/CaiJimmy/hugo-theme-stack 를 참고하여 작성 했습니다.

작성이 완료되었다면 해당 명령어로 로컬 서버를 동작시켜 사이트 테스트를 할 수 있습니다.
```bash
hugo server
```

하지만 /tmp/hugo_cache/ 쪽에서 권한 문제가 생겼다는 에러를 확인할 수 있었습니다.
또한 brew로 설치한 hugo는 root에 존재하지 않기 때문에 root권한으로 실행할 수 없습니다.
따라서 해당 디렉토리 이하의 권한을 user권한으로 맞춰 줄 필요가 있습니다. 이 작업은 당연히 보안상에 문제가 없는지 검토가 이루어져야 합니다.
```bash
sudo chown -R < 사용자 이름 > : < 사용자 이름 > /tmp/hugo_cache/
```
 


위의 작업까지 완료하고 작성한 포스트를 작성 후 함께 Github에 올리고 뿌리기 전용 저장소까지 완성하면 git page로 호스팅이 가능합니다. 방법은 쉽게 인터넷에서 찾아볼 수 있습니다.

## hugo server 상세 명령어

hugo server를 외부에 제한없이 오픈하려면 다음과 같은 명령어를 사용해야 합니다.
```bash
hugo server --baseURL "docs.9bon.org" --bind "0.0.0.0"
```

이때 baseURL을 추가하지 않으면 config.yaml에 따라 적용되는 것으로 보였습니다.
hugo server의 기본 포트는 1313이기에 docs.9bon.org:1313으로 활성화 됩니다.

만약 포트번호를 바꾸고 싶다면
```bash
hugo server --port 2222 --baseURL "docs.9bon.org" --bind "0.0.0.0"
```
와 같은 방법으로 포트 번호를 바꿀 수 있습니다.

만약 저처럼 해당 장치에 웹서버가 동작하고 있어 80포트는 사용할 수 없지만 URL에 포트번호가 찍혀 접속하는 것도 흉해서 싫다면 웹서버의 **프록시**를 사용해야 합니다.
프록시의 구축은 이후에 설명하겠습니다.

만약 장치에 **리버스 프록시**가 적용된 웹서버가 존재하기 때문에 클라이언트는 접속을 80포트로 진행하게 될 예정이지만, 서버 내에선 다른 포트로 동작해야 한다면
명령어에 오픈 주소와 포트를 붙이지 말라는 내용을 추가해야 합니다.

```bash
hugo server --port 2222 --appendPort=false --baseURL "docs.9bon.org" --bind "0.0.0.0"
```
이렇게 하고 웹서버의 설정도 맞춘다면
서버 내에선 2222포트로 동작하며 클라이언트는 80포트가 생략된 docs.9bon.org의 주소로 접속해야만 합니다.
