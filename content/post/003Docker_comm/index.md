---
title: "도커 명령어와 예제"
description: "도커의 기본적인 명령어와 편하게 볼 수 있는 예제들입니다."
date: 2021-10-03T01:40:54+09:00
categories: ["도커"]
tags: ["image","nextcloud", "mssql", "mysql"]
image: "thum.png"
draft: false
---

## 기본

### 컨테이너 조회
```Docker
docker ps -a
```
| 첨자 | 뜻 |
|--|--|
| -a | 꺼져있는 것을 포함하여 모두 |

### 컨테이너 시작, 종료
```Docker
docker apache start
```
| 이후 내용| 뜻 |
|--|--|
| start | 실행 |
| restart| 재시작 |
| stop | 중지 |
| status| 상태확인 |

## 컨테이너 생성/수정

### 일반적인 경우
```Docker
docker container run --name nextcloud -d -v /media/ext1/Docker/cloud/:/var/www/html/data -p 1234:80 nextcloud
```
| 첨자 | 뜻 |
|--|--|
| -run | 생성 후 바로 실행함 |
| --name | 생성될 컨테이너의 이름 지정 |
| -d | 백그라운드(데몬 프로세스)로 실행 |
| -v (복수가능) | 저장공간 맵핑/마운트 컨테이너 외부(호스트) : 컨테이너 내부|
| -p (복수가능) | 포트 맵핑/포워딩 컨테이너 외부(호스트) : 컨테이너 내부 |
*이후 내용엔 이미지 파일 이름*

### 환경변수 설정
```Docker
docker container run --name test_DB_YJ -d -p 49152:22 -p 1234:3306 -e MYSQL_ROOT_PASSWORD=12345 mariadb --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=12345" -p 1234:1433 --name rpadb -d mcr.microsoft.com/mssql/server:2019-latest
```

| 첨자 | 뜻 |
|--|--|
| -e| 컨테이너 내부 환경변수 수정 |


## 실행

### 컨테이너 내부 쉘실행

```Docker
docker exec -it nextcloud bash
```
| 첨자 | 뜻 |
|--|--|
| -i | 상호작용 할 것, 입력 출력이 가능해짐 |
| -t | 쉘과 같이 사용 될 것 |
*이후 내용엔 실행할 명령어*

### 컨테이너 내부 권한으로 명령 실행
```docker
docker exec -u www-data nextcloud php occ files:scan --all
docker exec -u 0 -it rpadb /bin/bash
```

| 첨자 및 매게변수 | 뜻 |
|--|--|
| -u | 해당 유저의 권한으로 |
*이후 내용엔 실행할 명령어*

