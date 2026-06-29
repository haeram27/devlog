# ubuntu network 설정

- ubuntu는 기본적으로 NIC 설정 및 사용에 `netplan`을 사용함 (rocky의 경우 NetworkManager)
- ubuntu 설치 중 입력된 network 설정 정보는 `/etc/netplan/00-installer-config.yaml` 파일에 저장됨
- 단, 별도로 NetworkManager를 설치하면 netplan이 NetworkManger를 참조 하는 형식으로 사용이 가능함

## NetworkManger 설치 시

NetworkManager를 설치하면 netplain 설정이 다음과 같이 자동 설정 됨

```bash
$ cat /etc/netplan/01-network-manager-all.yaml
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
```

## netplan 설정

```bash
$ cat /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      addresses:
      - 192.168.57.50/24
      match:
        macaddress: 00:00:00:00:00:00
      nameservers:
        addresses:
          - 1.1.1.1
          - 2.2.2.2
          - 8.8.8.8
        search: []
      routes:
      - to: default
        via: 192.168.57.1
      set-name: ens33
    ens34:
      accept-ra: true
      dhcp4: true
  version: 2
```
- `ens33` - ipv4 static 설정
  - addresses : nic에 할당될 ipv4 주소
  - nameservers(array) : dns 서버 설정
  - routes(array) : gateway 설정
- `ens34` - dhcp 사용 설정
  - accept-ra : IPv6 환경에서 RA(Router Advertisement)를 수신할지 여부를 설정하는 옵션


## proxy 설정

### 1. 터미널 세션에 임시 설정 (현재 세션만)
터미널에서 명령어를 실행할 때만 임시로 사용하려면 export 명령어를 사용합니다.

```bash
export http_proxy="http://아이피:포트"
export https_proxy="http://아이피:포트"# 인증이 필요한 경우
export http_proxy="http://ID:Password@아이피:포트"
```

### 2. 시스템 전체 프록시 설정 (영구 적용) [4] 
시스템 전체의 환경 변수에 프록시를 등록하려면 /etc/environment 파일을 수정합니다.

```bash
$ sudo vi /etc/environment
http_proxy="http://proxy-ip:port"
https_proxy="http://proxy-ip:port"
no_proxy="localhost,127.0.0.1"
$ source /etc/environment # 임시 적용, 재부팅 추천
```

### 3. APT 패키지 매니저 전용 설정
apt는 기본적으로 system proxy(http_proxy 환경변수)를 참조 한다.
하지만 apt만 별도로 사용해야 하는 프록시가 있다면 다음과 같이 설정이 가능하다.

```bash
$ sudo vi /etc/apt/apt.conf.d/95proxy
Acquire::http::Proxy "http://아이피:포트/";
Acquire::https::Proxy "http://아이피:포트/";
```
설정 후 별도로 시스템 재부팅 등은 필요 없음
apt는 명령 호출시 마다 이 설정 파일을 참조하여 실행함

## CA 인증서 설치

```bash
sudo cp your-certificate.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

## ca 인증서와 서버 인증서


CA 인증서는 CA 자신의 공개키를 담고 있음
서버 인증서는 서버용 SSL/TLS 인증서를 말하며 CA 발급함 

### 1. CA 인증서 vs 서버 인증서 차이

- 서버 인증서 (Server Certificate): 웹 서버가 클라이언트(사용자)에게 제시하는 인증서입니다. 여기에는 해당 서버의 공개키와 도메인 정보가 들어 있으며, CA의 디지털 서명이 찍혀 있습니다.
- CA 인증서 (CA Certificate): 서버 인증서가 진짜인지 확인하기 위해 사용하는 '도장 원본'과 같습니다. 여기에는 CA의 공개키가 들어 있습니다 

### 2. 인증 과정 (신뢰의 사슬)
   1. 서버의 제시: 사용자가 웹사이트에 접속하면 서버는 자신의 서버 인증서를 보냅니다.
   2. 클라이언트의 검증: 사용자의 컴퓨터(Ubuntu 등)는 이미 가지고 있는 CA 인증서 속의 공개키를 꺼냅니다.
   3. 서명 확인: 이 CA 공개키로 서버 인증서에 찍힌 CA의 서명을 해독해 봅니다. 해독이 잘 된다면 "이 서버 인증서는 내가 믿는 CA가 보증한 것이 맞구나"라고 판단하여 신뢰하게 됩니다.

### 요약 비교

| 구분 | 서버 인증서 (SSL/TLS) | CA 인증서 (Root/Intermediate) |
|---|---|---|
| 포함된 공개키 | 웹 서버의 공개키 | CA 기관의 공개키 |
| 주요 역할 | 서버의 신원 증명 및 암호화 통신 | 서버 인증서의 위조 여부 검증 |
| 저장 위치 | 웹 서버 설정 파일에 등록 | 운영체제/브라우저의 신뢰 저장소 |

