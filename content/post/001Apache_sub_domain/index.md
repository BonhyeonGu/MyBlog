---
title: "아파치 가상호스트 설정"
description: "서브도메인으로 가상호스트 설정을 적용하는 방법입니다."
date: 2021-09-02T19:32:24+09:00
categories: ["리눅스", "서버"]
tags: ["apache", "virtualhost", "domain"]
image: "thum.png"
draft: false
---
## 서론

웹 서버 데몬은 한 호스트를 여러 호스트처럼 쓸 수 있는 **가상 호스트(VirtualHost)** 를 대부분 지원하고 있습니다. 포트번호, 서브도메인, 출발지 주소등의 규칙을 적용시킵니다.
이번엔 Debian과 Apache로 서브 도메인에 따른 규칙으로 가상 호스트를 구현해 보도록 하겠습니다.
마지막엔 cloud.9bon.org와 9bon.org가 똑같은 장치에 구성되어있음에 불구하고 서로 다른 내용을 보여주는 것이 목표입니다.

## 아파치 준비

아파치가 설치되어있지 않으면 패키지 관리자로 쉽게 설치 가능합니다.

```bash
apt-get update && apt-get install apache2
```

또한 앞으로의 과정 중 재시작을 하고 싶을땐 다음과 같습니다.
```bash
service apache2 restart
```
당연히 재시작 말고도 start, stop, **reload**, status를 사용할 수 있습니다.

 

## 아파치 설정

 데비안 10기준 Apache2에서는 설정파일이 모두 모듈화 되어있으며 추가 설정 또한 부분화하길 권장하고 있습니다. 
```bash
cd /etc/apache2/sites-available/
```
아래처럼 설정파일을 <추가할 사이트 도메인>.conf 으로 생성하면 됩니다.
```bash
nano a.com.conf
```

```apacheConf
<VirtualHost *:80>
    ServerName  a.com
    DocumentRoot /var/www/a.com

    <Directory />
        AllowOverride All
        Options All -Indexes
        Require all granted
    </Directory>
</VirtualHost>
```
요약하면 80번 포트로 연결되는 가상 호스트이며 도메인이 a.com일때 /var/www/a.com의 내용을 보여라는 뜻입니다.
그대로  www.a.com또한 작성하면 됩니다.

```bash
nano www.a.com
```
```ApacheConf
<VirtualHost *:80>
    ServerName  www.a.com
    DocumentRoot /var/www/www.a.com
    
    <Directory />
        AllowOverride All
        Options All -Indexes
        Require all granted
    </Directory>
</VirtualHost>
```

작성을 마치면 
```bash
a2ensite a.com
a2ensite www.a.com
```

잘못 입력했다면
```bash
a2dissite <설정 파일을 기반한 주소>
```
할 수 있습니다.
마지막으로 reload 하면 완료입니다.

```bash
service apache2 reload
```
참고로 아파치는 사이트 설정을 바꾸는 것에는 reload를 모듈을 추가 해제할 때는 restart 하기를 추천하고 있습니다.
