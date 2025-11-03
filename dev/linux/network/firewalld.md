
# firewalld

- firewalld는 zone 단위로 네트워크 방화벽 규칙(rule) 세트를 정의하여 NIC에 적용하는 방식으로 동작하는 방화벽 서비스 이다.
- zone은 네트워크 인터페이스에 적용되는 신뢰 수준별 방화벽 규칙 세트이다.
- 하나의 NIC는 하나의 zone에만 소속될 수 있다.
- zone에는 방화벽 규칙을 포함한다.
- 규칙은 src/dest의 address, port, 프로토콜 등을 사용하여 다양하게 정의할 수 있다.
- 알려진 네트워크 어플리케이션의 프로토콜과 포트 정보를 사전 정의하여 `service`라는 unit으로 규칙에 사용할 수 있다.

## terms

### zone

```bash
# firewall-cmd --zone=public --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens33
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

### service

네트워크 서비스를 제공하는 서비스(application)의 프로토콜과 포트 정보를 인식하기 쉬운 이름(주로 application 이름)으로 사전 정의해 두는 룰 unit

```bash
# firewall-cmd --info-service=http
http
  ports: 80/tcp
  protocols:
  source-ports:
  modules:
  destination:
  includes:
  helpers:
```

서비스가 정의된 룰 파일의 경로

```bash
# ls -l /usr/lib/firewalld/services/
...
-rw-r--r--. 1 root root 227 Feb  4  2025 dhcp.xml
-rw-r--r--. 1 root root 318 Feb  4  2025 dns-over-tls.xml
-rw-r--r--. 1 root root 346 Feb  4  2025 dns.xml
-rw-r--r--. 1 root root 212 Feb  4  2025 git.xml
-rw-r--r--. 1 root root 406 Feb  4  2025 gpsd.xml
-rw-r--r--. 1 root root 448 Feb  4  2025 https.xml
-rw-r--r--. 1 root root 353 Feb  4  2025 http.xml
...
```

```bash
# cat /usr/lib/firewalld/services/http.xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>WWW (HTTP)</short>
  <description>HTTP is the protocol used to serve Web pages...</description>
  <port protocol="tcp" port="80"/>
</service>
```

사전 정의된 서비스 룰이 아닌 사용자 정의 룰을 서비스로 등록하려면 위 경로에 사용자 정의 파일을 추가하고 firewalld가 설정 파일을 다시 로드 하도록 한다.

```bash
# 설정 적용
sudo firewall-cmd --reload

# 확인
sudo firewall-cmd --list-services
```

## firewalld 기본 설정

Rocky Linux 설치 시 firewalld의 기본 설정은 다음과 같습니다

**기본 Zone**: `public`

- 새로 설치된 시스템에서는 모든 네트워크 인터페이스가 기본적으로 `public` zone에 할당됩니다.

**public zone의 기본 허용 서비스**:

- **SSH (22/tcp)** - 원격 접속을 위해 기본적으로 열려있음
- **DHCPv6-client** - IPv6 주소 자동 할당용
- **Cockpit (9090/tcp)** - 웹 기반 관리 도구 (Rocky Linux 8 이상)

**차단되는 주요 서비스**:

- **HTTP (80/tcp)** ❌
- **HTTPS (443/tcp)** ❌
- 기타 대부분의 서비스

HTTP/HTTPS는 기본적으로 차단되어 있습니다. 이는 보안을 위한 설정으로, 필요한 서비스만 명시적으로 열도록 되어 있습니다.

## 현재 설정 확인 방법

```bash
# 기본 zone 확인
sudo firewall-cmd --get-default-zone

# public zone의 허용된 서비스 확인
sudo firewall-cmd --zone=public --list-all

# 또는
sudo firewall-cmd --list-all
```

## HTTP/HTTPS 허용 방법

웹 서버를 운영하신다면 다음과 같이 포트를 열 수 있습니다:

```bash
# HTTP 허용
sudo firewall-cmd --permanent --add-service=http

# HTTPS 허용
sudo firewall-cmd --permanent --add-service=https

# 설정 적용
sudo firewall-cmd --reload

# 확인
sudo firewall-cmd --list-services
```

이렇게 하면 웹 트래픽이 정상적으로 수신됩니다.

## Zone이란

Zone은 **네트워크 인터페이스에 적용되는 신뢰 수준별 방화벽 규칙 세트**입니다. 네트워크 환경의 신뢰도에 따라 다른 보안 정책을 적용할 수 있습니다.

핵심 원칙
하나의 NIC는 오직 하나의 zone에만 소속될 수 있다.

## 주요 개념

- 각 네트워크 인터페이스(eth0, wlan0 등)는 하나의 zone에 할당됩니다
- Zone마다 허용/차단할 서비스와 포트를 개별적으로 설정할 수 있습니다
- 동일한 시스템에서 여러 zone을 동시에 사용 가능합니다

## 사전 정의된 Zone (신뢰도 순)

### 1. **trusted** (가장 신뢰)

- 모든 트래픽 허용
- 내부 네트워크나 완전히 신뢰할 수 있는 환경

### 2. **home**

- 가정 네트워크용
- 기본 허용: ssh, mdns, samba-client, dhcpv6-client
- 같은 네트워크의 다른 컴퓨터를 어느 정도 신뢰

### 3. **internal**

- 내부 네트워크용
- home과 유사하지만 회사 내부망 등에 사용
- 기본 허용: ssh, mdns, samba-client, dhcpv6-client

### 4. **work**

- 직장 네트워크용
- 기본 허용: ssh, dhcpv6-client, cockpit
- 동료들의 컴퓨터는 신뢰하지만 제한적

### 5. **public** (기본값)

- 공공 장소, 신뢰할 수 없는 네트워크
- 기본 허용: ssh, dhcpv6-client, cockpit (최소한의 서비스만)
- 새 설치 시 기본 zone

### 6. **external**

- 외부 네트워크, NAT 환경
- 마스커레이딩(masquerading) 활성화
- 기본 허용: ssh

### 7. **dmz**

- DMZ(비무장지대)용
- 공개 서버 배치 시 사용
- 기본 허용: ssh

### 8. **block**

- 들어오는 모든 연결 거부 (icmp-host-prohibited 응답)
- 나가는 연결만 허용

### 9. **drop** (가장 제한적)

- 들어오는 모든 패킷을 응답 없이 drop
- 나가는 연결만 허용
- 가장 엄격한 보안

## 실제 사용 예시

```bash
# 현재 활성 zone 확인
sudo firewall-cmd --get-active-zones

# 특정 인터페이스가 속한 zone 확인
sudo firewall-cmd --get-zone-of-interface=eth0

# 인터페이스를 다른 zone으로 변경
sudo firewall-cmd --zone=home --change-interface=eth0

# 각 zone의 설정 확인
sudo firewall-cmd --zone=home --list-all
sudo firewall-cmd --zone=public --list-all
```

## 실무 활용 시나리오

**시나리오 1**: 노트북 사용자

- 회사: eth0 → work zone
- 집: wlan0 → home zone
- 카페: wlan0 → public zone

**시나리오 2**: 서버 운영

- 외부 인터페이스(eth0): public 또는 dmz
- 내부 인터페이스(eth1): internal 또는 trusted

**시나리오 3**: 방화벽 라우터

- 외부망(eth0): external (NAT 활성화)
- 내부망(eth1): trusted

## Zone 변경 시 주의사항

```bash
# 임시 변경 (재부팅 시 초기화)
sudo firewall-cmd --zone=home --change-interface=eth0

# 영구 변경
sudo firewall-cmd --permanent --zone=home --change-interface=eth0
sudo firewall-cmd --reload
```

Zone 개념을 이해하면 환경에 따라 유연하게 방화벽 정책을 관리할 수 있습니다

## 하나의 NIC는 하나의 zone에만 소속

하나의 네트워크 인터페이스는 **오직 하나의 zone에만** 소속될 수 있습니다.

## 핵심 원칙

```
1개 인터페이스 = 1개 zone (1:1 관계)
```

## 이유

firewalld는 인터페이스별로 방화벽 규칙을 적용하는데, 하나의 인터페이스가 여러 zone에 속하면:

- 충돌하는 규칙이 발생할 수 있음
- 어떤 규칙을 우선 적용할지 불명확
- 보안 정책이 모호해짐

## 확인해보기

```bash
# eth0이 속한 zone 확인 (하나만 반환됨)
sudo firewall-cmd --get-zone-of-interface=eth0

# 다른 zone으로 변경하면 이전 zone에서는 제거됨
sudo firewall-cmd --zone=work --change-interface=eth0  # work로 이동
sudo firewall-cmd --zone=home --change-interface=eth0  # home으로 이동 (work에서 자동 제거)
```

## 여러 보안 정책이 필요한 경우 해결책

### 방법 1: 여러 인터페이스 사용 (권장)

```bash
# 각 인터페이스를 다른 zone에 할당
sudo firewall-cmd --zone=public --change-interface=eth0   # 외부 네트워크
sudo firewall-cmd --zone=internal --change-interface=eth1 # 내부 네트워크
sudo firewall-cmd --zone=dmz --change-interface=eth2      # DMZ 네트워크

# 확인
sudo firewall-cmd --get-active-zones
# 출력:
# public
#   interfaces: eth0
# internal
#   interfaces: eth1
# dmz
#   interfaces: eth2
```

### 방법 2: VLAN 사용

```bash
# 하나의 물리 인터페이스를 여러 VLAN으로 분할
sudo firewall-cmd --zone=public --change-interface=eth0.100   # VLAN 100
sudo firewall-cmd --zone=internal --change-interface=eth0.200 # VLAN 200
sudo firewall-cmd --zone=dmz --change-interface=eth0.300      # VLAN 300
```

### 방법 3: Rich Rule로 세밀한 제어

하나의 zone 내에서 IP나 서비스별로 다른 규칙 적용:

```bash
# public zone에서 세밀한 제어
sudo firewall-cmd --zone=public --add-service=ssh

# 특정 IP에서만 SSH 허용
sudo firewall-cmd --permanent --zone=public --add-rich-rule='
  rule family="ipv4"
  source address="192.168.1.100/32"
  service name="ssh"
  accept'

# 특정 IP 대역은 모든 트래픽 허용
sudo firewall-cmd --permanent --zone=public --add-rich-rule='
  rule family="ipv4"
  source address="10.0.0.0/8"
  accept'

# 특정 IP는 차단
sudo firewall-cmd --permanent --zone=public --add-rich-rule='
  rule family="ipv4"
  source address="1.2.3.4/32"
  reject'
```

### 방법 4: 소스 기반 Zone (고급)

인터페이스 대신 소스 IP 주소를 zone에 할당:

```bash
# 특정 IP 대역을 trusted zone에 추가
sudo firewall-cmd --permanent --zone=trusted --add-source=192.168.1.0/24

# 다른 IP 대역을 internal zone에 추가
sudo firewall-cmd --permanent --zone=internal --add-source=10.0.0.0/8

# 확인
sudo firewall-cmd --get-active-zones
# 출력:
# trusted
#   sources: 192.168.1.0/24
# internal
#   sources: 10.0.0.0/8
# public
#   interfaces: eth0
```

이 방식으로 하나의 인터페이스(eth0)를 통해 들어오는 트래픽을:

- 소스 IP가 192.168.1.0/24 → trusted zone 규칙 적용
- 소스 IP가 10.0.0.0/8 → internal zone 규칙 적용
- 나머지 → public zone 규칙 적용

## 실전 예시

### 시나리오: 웹서버가 하나의 인터페이스로 다양한 클라이언트 처리

```bash
# eth0을 public zone에 할당 (기본)
sudo firewall-cmd --zone=public --change-interface=eth0

# 내부 관리자 IP는 더 많은 접근 허용
sudo firewall-cmd --permanent --zone=public --add-rich-rule='
  rule family="ipv4"
  source address="192.168.100.0/24"
  service name="ssh"
  accept'

sudo firewall-cmd --permanent --zone=public --add-rich-rule='
  rule family="ipv4"
  source address="192.168.100.0/24"
  port port="3306" protocol="tcp"
  accept'

# 일반 사용자는 웹만 접근 가능
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https

sudo firewall-cmd --reload
```

## 우선순위 규칙

동일한 인터페이스에서 소스 기반 zone과 인터페이스 기반 zone이 겹칠 때:

```text
1. 소스 기반 zone (가장 높은 우선순위)
2. 인터페이스 기반 zone
3. 기본 zone
```

예시:

```bash
sudo firewall-cmd --zone=public --change-interface=eth0
sudo firewall-cmd --zone=trusted --add-source=192.168.1.50/32

# 192.168.1.50에서 들어오는 트래픽 → trusted zone 적용
# 다른 IP에서 들어오는 트래픽 → public zone 적용
```

## 요약

| 질문 | 답변 |
|------|------|
| 하나의 인터페이스가 여러 zone에 속할 수 있나? | ❌ 불가능 (1:1 관계) |
| 여러 보안 정책이 필요하다면? | ✅ 여러 인터페이스 사용 |
| 물리 인터페이스가 하나뿐이라면? | ✅ VLAN 또는 소스 기반 zone 활용 |
| 하나의 zone 내에서 세밀한 제어? | ✅ Rich Rule 사용 |

**결론**: 하나의 인터페이스는 하나의 zone에만 속하지만, 소스 IP 기반 zone과 rich rule을 활용하면 유연한 방화벽 정책을 구현할 수 있습니다!

## firewalld 비활성화

다수의 서비스를 테스트 하는 용도의 호스트에서는 firewalld를 비활성화 하는 것이 편리하다.

```bash
sudo systemctl status firewalld
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo systemctl mask firewalld
```
