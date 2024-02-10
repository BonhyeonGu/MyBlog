---
title: "Gateway에 OpenVPN 설치"
description: "다른 망에 설치된 Gateway를 사용하기 위한 VPN 설정에 대한 내용입니다."
date: 2023-05-15T17:22:49+09:00
lastmod: 2023-05-15T17:22:49+09:00
categories: ["리눅스", "서버"]
tags: ["certificate", "vpn", "openvpn", "gateway", "forward"]
image: "thum.png"
draft: false
---

## 서론

### 개요

라우팅, 방화벽, NAT 등의 조건이 걸린 Gateway를 경우에 따라서 다른 망의 호스트로 부터 접근하고 싶은 경우가 있습니다.  
클라이언트가 속한 망이 본인 권한이 아닐 경우 VPN을 Gateway에 설치하는 것이 이상적인 솔루션입니다.

물론 이것이 클라이언트가 속한 망의 정책에 위반되지 않는지 검토되어야 하며,  
라우팅은 네트워크 계층을, VPN은 네트워크와 전송 계층의 사이에 존재하는 서비스이기 때문에,  
정작 실제 망에 문제가 생기면 VPN 설정과 무관하게 됨을 이해해야 합니다.

### VPN의 UDP? TCP?

VPN도 네트워크 서비스인 만큼, 두 네트워크 프로토콜중에 하나를 골라야 합니다.  
하지만 궁금증이 생길 수 있습니다. 만약 UDP로 VPN을 연결하면 TCP서비스를 사용하지 못하는가?  
답은 '그렇지 않다.' 입니다. VPN에서 선택하는 프로토콜은 VPN 터널에서 사용할 패킷을 어떻게 전달하느냐 입니다.  
이 위에 올라갈 서비스와는 다른 것입니다.

하지만 이것은 신중히 선택되어야 합니다. 만약 VPN을 UDP로 설정한다면, 서비스가 TCP를 사용한들  
TCP 본연의 신뢰성을 잃거나, 서비스의 handshake를 VPN의 UDP가 손실시켜 본래의 속도보다 더 느릴 수 있습니다.

이번 예에서는 VPN을 UDP로 설정합니다. 해당 경우로 얻길 원하는 이점은 VPN으로 소요되는 시간을 최소화 하는 것입니다.


## 설치

설명되는 과정은 Ubunut 22.04 LTS 기준입니다. 기본 게이트웨이의 설정

```zsh
net.ipv4.ip_forward=1
```

과 같은 것은 마무리 되었다고 가정합니다.

### 인증서

인증서를 만들어줄 패키지를 설치하고, 인증서 설정이 포함된 디렉토리를 생성,  
그 후 인증서 메타데이터를 수정합니다.

```zsh
sudo apt-get update
sudo apt-get install easy-rsa openvpn
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
nano vars
```

vars의 내용물은 대부분 주석으로 처리되어 있는 내용일 경우 필자와 동일한 Easy-RAS 3.x 환경입니다.
만약 export 등의 키워드가 붙어있다면 2.x 임으로 다른 문서를 확인해주세요  
저는 다음과 같이 정의하겠습니다.

```zsh
set_var EASYRSA_REQ_COUNTRY    "KR"
set_var EASYRSA_REQ_PROVINCE   "Busan"
set_var EASYRSA_REQ_CITY       "Busan"
set_var EASYRSA_REQ_ORG        "Oklab"
set_var EASYRSA_REQ_EMAIL      "bonhyeon.gu@9bon.org"
set_var EASYRSA_REQ_OU         "graduate"
```

이후 필요한 파일을 출력하겠습니다. 인증, 인증서, 키, 서명등의 이론적 설명은 다음에 포스팅에 하도록 하겠습니다.  
클라이언트 이름을 h100으로 하는 예제입니다.

공개키 인프라 초기화
```zsh
./easyrsa init-pki
```

CA인증서 및 키 생성
```zsh
./easyrsa build-ca nopass
```

서버의 인증서 요청(CSR) 및 개인 키 생성
```zsh
./easyrsa gen-req server nopass
```

서버 CSR에 서명하여 서버 인증서 생성
```zsh
./easyrsa sign-req server server
```

클라이언트의 인증서 요청(CSR) 및 개인 키 생성
```zsh
./easyrsa gen-req h100 nopass
```

클라이언트 CSR에 서명하여 클라이언트 인증서 생성
```zsh
./easyrsa sign-req client client1
```

DH 파라미터 생성
```zsh
./easyrsa gen-dh
```

CRL(인증서 폐지 목록) 생성
```zsh
./easyrsa gen-crl
```

작업의 편리함을 위하여 생성된 파일을 openvpn 디렉토리에 복사하도록 하겠습니다.

```zsh
sudo cp ./pki/ca.crt /etc/openvpn/server/
sudo cp ./pki/issued/server.crt /etc/openvpn/server/
sudo cp ./pki/private/server.key /etc/openvpn/server/
sudo cp ./pki/dh.pem /etc/openvpn/server/
sudo cp ./pki/crl.pem /etc/openvpn/server/
```

### OpenVPN 서버 설정

MitM(중간자공격)을 방지하기 위해 TLS-Auth를 설정할 수 있습니다. 이를 위해서는 추가적인 키 생성이 필요합니다.

```zsh
cd /etc/openvpn/server/
openvpn --genkey --secret ta.key
```

다음으로 */etc/openvpn/server.conf* 를 생성합니다.

```zsh
port 1194
proto udp
dev tun
ca server/ca.crt
cert server/server.crt
key server/server.key
dh server/dh.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
tls-auth server/ta.key 0
cipher AES-256-CBC
auth SHA256
compress lz4-v2
push "compress lz4-v2"
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3
crl-verify server/crl.pem
client-config-dir /etc/openvpn/ccd
```

다음으로 */etc/openvpn/ccd/* 디렉토리를 만들고 */etc/openvpn/ccd/h100* 를 생성합니다. 이 작업은  
클라이언트의 정보를 지정하는 것입니다. h100이라는 이름도 클라이언트 인증서의 CN을 따라갑니다.  
앞에서 *./easyrsa gen-req h100 nopass* 의 명령과 관련이 있습니다.

해당 파일에는 다음과 같은 고정 아이피 주소와 같은 내용을 넣을 수 있습니다.

```zsh
ifconfig-push 10.8.0.2 255.255.255.0
```

이제 재시작 하여 설정을 적용시켜야 합니다.

```zsh
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
```

### 클라이언트 설정

지금부터는 클라이언트(H100)의 설정입니다. 클라이언트에도 패키지를 설치해 줍니다.

```zsh
sudo apt-get update
sudo apt-get install openvpn
```

다음으로 */etc/openvpn/client/* 에 앞에서 만들었던 해당 파일을 옮겨주어야 합니다.  
당연히 인증서와 키를 옮기는 과정은 주의가 필요합니다.

- ca.crt
- h100.crt
- h100.key
- ta.key

그 후 다음 설정파일을 */etc/openvpn/client.conf* 에 작성하겠습니다.  
vpn.server.com 은 VPN 서버의 주소가 들어가야 합니다.

```zsh
client
dev tun
proto udp
remote vpn.server.com 1194
resolv-retry infinite
nobind
user nobody
group nogroup
persist-key
persist-tun
ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/h100.crt
key /etc/openvpn/client/h100.key
tls-auth /etc/openvpn/client/ta.key 1
cipher AES-256-CBC
auth SHA256
verb 3
```

마지막으로 다음의 명령어로 설정 변경을 적용시킵니다.

```zsh
sudo systemctl restart openvpn@client
```

그 후 *ifconfig* 그리고 *ping* 등으로 부여된 주소 및 연결을 확인할 수 있습니다.


## NAT 설정 참고

만약 Gateway에서 VPN 연결 대상으로 넘겨주기 위해서는 게이트웨이에 다음과 같은 예를 사용할 수 있습니다.

```zsh
sudo iptables -t nat -A PREROUTING -p tcp --dport [외부에서 들어올 포트번호] -j DNAT --to-destination [VPN주소]:22
sudo iptables -A FORWARD -p tcp -d [VPN주소] --dport [외부에서 들어올 포트번호] -j ACCEPT
```