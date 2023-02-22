---
title: "zsh, oh-my-zsh!"
description: "자꾸 찾아보게 되는, 우분투 기준 주로 사용하는 쉘 환경 구축법입니다."
date: 2021-12-24T16:11:32+09:00
categories: ["리눅스"]
tags: ["git", "zsh"]
image: "thum.png"
draft: false
---

## zsh

### 설명
기능이 다양해서 유행중인 bash 쉘을 대체하는 쉘입니다.

### 설치
```bash
apt-get install zsh
chsh -s /usr/bin/zsh
```
쉘은 유저에 귀속되므로 **권한에 주의해야 합니다.**
또한 **작은 따옴표가 아닙니다.**

재시작 하면 설치가 완료됩니다.
```bash
echo $SHELL
```
로 확인 할 수 있습니다.

## oh-my-zsh!

### 설명
zsh를 설정할 수 있는 편리한 콘피그들을 제공해주는 프레임워크입니다.

### 설치

```bash
curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh
```
해당 설치법은 http://ohmyz.sh/ 에서 제공하는 방법입니다. 문제가 생길경우 이곳에서 편집되었는지 확인할 수 있습니다.

### 설정

```bash
vim ~/.zshrc
```

에서 제가 주로 사용하는 것은

ZSH_THEME="agnoster"

입니다.

또한 agnoster 테마를 이용하기 위해서는 다른 글꼴이 필요합니다.
적당한 디렉토리에
```bash
git clone https://github.com/powerline/fonts.git
cd fonts
./install.sh
```

또는 apt 패키지 관리자로 설치 가능합니다.

```bash
apt-get install fonts-powerline
```

만약 wsl2와 같은 윈도우 환경이라면 수동으로 설치해야합니다.
https://github.com/powerline/fonts.git 의 릴리즈에 들어가 압축파일을 받은 뒤
수동 설치를 진행하고 터미널->속성에서 글꼴을 변경할 수 있습니다.

## 기타

### 업데이트 충돌 오류

여러가지 수정하고 oh-my-zsh 업데이트를 받을 시 git 오류가 발생하는 것을 확인했습니다.

```bash
cd ~/.oh-my-zsh
```

후
```bash
git status
git add
git commit -m
```
등을 이용하여
```bash
omz update
```
로 수동 업데이트를 진행하여 해결했습니다.
