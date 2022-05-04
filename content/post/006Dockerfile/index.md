
---
title: "도커 파일"
description: "도커에서 컨테이너를 생성해 주는 도커 파일의 간단한 예제입니다."
date: 2022-01-02T02:21:44+09:00
categories: ["도커"]
tags: ["container", "docker-compose", "image"]
image: "thum.png"
draft: false
---

## 간단한 형식

### 운영체제

```docker
#원본이 될 이미지, https://hub.docker.com/ 참고
#버전을 기록하지 않으면 최신 버전을 사용하게 된다.
FROM ubuntu:16.04

#작성자 정보, 필수는 아님, 옛날에는 MAINTAINER을 사용했으나 대체됨
LABEL email="tayasriel@gmail.com"
LABEL name="BonhyeonGu"

#언어 설정, ENV 자체는 도커 이미지 환경변수를 조작한다.
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

#명령어, 물음이 오는것을 피해야 진행이 가능하다.
RUN apt-get update -y && \
 apt-get install -y nano
 
#노출될 포트, 주의할 점은 이게 포트포워딩을 해주는 것이 아님
EXPOSE 5000, 5001
```

### 인터프리터

```docker
FROM ubuntu
LABEL email="tayasriel@gmail.com"
LABEL name="BonhyeonGu"

RUN  apt-get update -y && \
 apt-get install -y python-pip python-dev

#호스트 경로 .의 파일들을 컨테이너의 경로 /app으로 복사시킨다.
COPY  . /app

#컨테이너가 실행되면 활성화 될 경로, cd /app 과 동일하다.
WORKDIR  /app
RUN  pip install -r requirements.txt

#명령어 실행, RUN과 다르게 컨테이너가 활성화 될 때 마다 실행된다.
ENTRYPOINT  [ "python" ]

#명령어 실행, ENTRYPOINT 와 비슷하지만, 만약 도커 컨테이너 실행과 함께
#인수가 제공되면 그것으로 대체된다.
CMD  [ "app.py" ]

## 즉 이 형태는 컨테이너 start와 함께, 파이썬이 실행되고 컨테이너가 stop되는 형식
```
## 도커 컴포즈, 도커 파일의 조합

### 안내

이런 방법으로 컴공에 새로 입문하시는 분들께 개발환경을 배포할 수 있을 것 같습니다.

### 파이썬 인터프리터
 
도커파일
```docker
FROM ubuntu
LABEL email="tayasriel@gmail.com"
LABEL name="BonhyeonGu"

RUN  apt-get update -y && \
 apt-get install -y python3-pip python3-dev

#mkdir 이 필요할 수 있다.
WORKDIR  /app
RUN pip install --no-cache-dir --upgrade pip && \
    pip install numpy
    #경우에 따라서 pip 모두 업그레이드 받는건 피해야겠다. 꽤 오래걸림

#명령어 실행, RUN과 다르게 컨테이너가 활성화 될 때 마다 실행된다.
ENTRYPOINT  [ "python3" ]

#명령어 실행, ENTRYPOINT 와 비슷하지만, 만약 도커 컨테이너 실행과 함께
#인수가 제공되면 그것으로 대체된다.
CMD  [ "app.py" ]

## 즉 이 형태는 컨테이너 start와 함께, 파이썬이 실행되고 컨테이너가 stop되는 형식
```

컴포즈
```docker
version: "3"
services:
 test1:
  build: .
  volumes:
   - .:/app
```

이후
```docker
docker-compose up && docker-compose rm -f
```
로 쉬운 실행/삭제 가능

### 파이썬 리모트 전용

도커파일 
```docker
FROM ubuntu
LABEL email="tayasriel@gmail.com"
LABEL name="BonhyeonGu"

#언어 설정, ENV 자체는 도커 이미지 환경변수를 조작한다.
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
RUN  apt-get update -y && \
 apt-get install -y python3-pip python3-dev

#vscode remote시 초기 경로를 root로 잡는다.
WORKDIR  /root
RUN pip install --no-cache-dir --upgrade pip && \
    pip install numpy
```

컴포즈
```docker
version: "3"
services:
 test2:
  #stdin_open: true # docker run -i
  tty: true        # docker run -t
  build: .
  volumes:
   - .:/root
```

이후
```docker
docker-compose up -d
code .
그리고 리모트, 다쓰면 이후로
docker-compose down
```
