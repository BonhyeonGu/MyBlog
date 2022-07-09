---
title: "ServerLog"
description:
date: 2021-10-03T23:21:22+09:00
draft: false
---
## v01 (2017~2020.11)

### 사진
![1 세대](page/serverLog/01.png)
<img src = "page/serverLog/01-01.png" width="50%" height="50%">

### 제원
| 총 금액 | ~60,000원 |
|--|--|
| 기종 | 라즈베리파이3 |
| OS | 라즈비안 |
| CPU | 4Core 1.2GHz |
| RAM | 1GB |
| HDD | **외장**-1TB |

ARM 아키텍처 환경은 많은 단점이 있었습니다.  
지원하지 않는것이 많았으며 높은 레벨의 루틴을 시도하기 어려웠습니다.

### 서비스
| 최종 가동 서비스 | 데몬  |
|---|--|
| WEB + CLOUD| APACHE + PYDIO |
| FTP | VSFTPD |
| DB | MYSQL |
| SHARE DIRECTORY | SAMBA |

웹 클라우드 또한 PYDIO와 OwnCloud 등 여러가지 소스를 시도했었습니다. 

## v02 (2020.11~가동중)

### 사진
![2 세대](page/serverLog/02.png)

### 제원
| 총 금액 | ~200,000원 |
|--|--|
| MB | ASROCK J5040-ITX |
| OS | Debian |
| CPU | **MB 결합**-4Core 2~3.2GHz |
| RAM | 8GB |
| HDD | 2TB |
| SYSFAN | NOCTUA120mm |

오랫동안 계획된 시스템입니다.  
CPU가 결합된 상태에서 출고되는 저전력이지만 데스크탑 아키텍처인 보드를 구해 기반을 맞췄습니다.

### 서비스
**도커 컨테이너 내에서 작동되는 서비스는 cn_이라 표기**
| 서비스 | 상세 |
|---|--|
| cn_WEB1 | APACHE_Main |
| cn_WEB2 | HUGO |
| cn_WEB3 | NextCloud|
| FTP | VSFTPD |
| cn_DB1 | MARIADB |
| cn_DB2 | MSSQL |

### 관리 기록

| 날짜 | 내용 | 사유 |
|--|--|--|
| 2021.07.25 | 방화벽 추가 설정 | 불특정한 잦은 핑 수신 |
| 2021.08.06 | SYSFAN 추가 | 발열문제로 인한 쓰로틀링, 여름이라 유난히 그런듯  |
| 2021.10.12 | 컨테이너 정리 | 체계적으로 분류하여 관리를 용이하게 함  |