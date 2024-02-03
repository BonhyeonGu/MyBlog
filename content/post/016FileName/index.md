---
title: "윈도우 대소문자 파일 이름 구별"
description: "대소문자 파일 이름을 구별 하기 위한 명령어를 작성했습니다."
date: 2023-01-10T17:42:59+09:00
lastmod: 2023-01-10T17:42:59+09:00
categories: ["윈도우"]
tags: ["filename", "uppercase", "lowercase"]
image: "thum.png"
draft: false
---

## 서론

윈도우는 파일 이름에서 대문자, 소문자를 구별하지 않습니다.  
당연히 프로그래밍 언어, 라이브리를 사용할 때에도 마찬가지입니다.

정확히는 NTFS와 같은 윈도우 파일시스템은 대소문자를 구별하지만 OS에서 허용하지 않게 되어있습니다.  
따라서 자신의 프로그램을 위해 아래의 명령어를 고려할 수 있습니다.

## CMD

```powershell
fsutil file setCaseSensitiveInfo <경로> enable
fsutil file setCaseSensitiveInfo <경로> disable
```