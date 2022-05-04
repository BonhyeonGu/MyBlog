---
title: "리눅스 디스크 문제"
description: "검사, 오류해결등의 겪은일을 기반으로 한 정리입니다."
date: 2022-03-12T03:43:58+09:00
categories: ["리눅스"]
tags: ["fsck", "fstab"]
image: "thum.png"
draft: false
---
## 하드디스크의 문제

### 증상

저장, 삭제시 에러문구 : "Structure needs cleaning", "구조에 청소가 필요합니다"  
ls 시 : ??? ?? ???? 로 표시  


### 설명

불량, 노후화, 최악에는 소프트웨어의 오류에도 하드디스크에 치명적인 문제가 발생할 수 있습니다.  
제가 겪은 문제는 서버가 일종의 과부하를 받을 때 디렉토리 일부가 사라져 조회조차 하지 못하는 상황에  
놓이는 것이였습니다.

놀라서 umount하거나 reboot해서 돌아와보면 디렉토리가 다시 보였지만  
디렉토리를 포함한 파일의 일부가 열리지만 소실되었거나, 열리지도 않으며 청소가 필요하다는 경고가 나타났습니다.  
흥미로웠던 사실은 비교적 최근에 생성된 것만 그런 상태였습니다. 추정해보면 EXT파일시스템의 특징이  
여기서 보이는 것이 아닐까 생각했습니다.

현재는 과거의 사태에 높은 확률을 따라, 하드디스크의 불량이라고 생각하고 있지만  
정확한 원인을 알기는 어려운 상황입니다. 이런일이 발생한다면 어떻게 대처하고 판단해야하는지  
작성해 보겠습니다.

## 해결

### fsck

fsck는 리눅스에서 지원하는 Filesystem fix 명령입니다. 실행하면 체크를 하고, 문제가 생긴 항목을 불러준 뒤  
허락하면 교정을 시작합니다. 당연히 un mount 되어있는 상황이여야 합니다.

```bash
umount /dev/<장치명>
fsck -y /dev/<장치명>
```

Yes 또는 No로 물어보는 항목이 많고 대부분 오래 걸리므로 -y를 붙여 진행할 수 있습니다.

### badblocks

badblocks는 리눅스에서 기본적으로 하드디스크의 badblock을 점검해주는 명령어 입니다.  
어차피 읽기 전용모드로 작동되기 때문에 un mount 되어 있을 필요는 없으나  
항상 장치적 문제를 해결할 때는 un mount 하시는 것을 추천드리고 싶습니다.

```bash
umount /dev/<장치명>
badblocks -v -o out.txt /dev/<장치명>
```

-v는 자세하게 보여달라는 뜻이고 -o는 badblock list 를 file output으로 해달라는 뜻입니다.

### smartctl

smartctl은 기본 제공은 아니지만 자주 사용했었던 것입니다.  
하드디스크의 모니터링 기술 SMART(Self-Monitoring, Analysis, and Reporting Technology)을 사용하여  
점검하는 도구이며 아주 옛날의 하드디스크가 아닌 이상 대부분 장착되어 있습니다.

```bash
apt-get install smartmontools
umount /dev/<장치명>
smartctl -a /dev/<장치명>
```

자세히는 -a, 간단히는 -H, 정보만은 -i 옵션을 사용하여 진행 가능합니다.
