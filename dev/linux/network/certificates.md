# 인증서 (certificates) 설치

1. 서버 인증서 (Server Certificate)
보통 우리가 말하는 SSL/TLS 인증서가 바로 서버 인증서입니다.

* 인증 대상: 웹사이트, 도메인, 또는 특정 IP를 가진 '서버 장비' 자체입니다.
* 주요 용도: 사용자가 올바른 웹사이트(예: google.com)에 접속했는지 서버의 신원을 보증하고, 브라우저와 서버 간의 통신 데이터를 암호화합니다.
* 인증서의 정보: 인증서 안에 해당 서버의 도메인 이름(Common Name / Subject Alternative Name)이 기록되어 있습니다.

## 설치 위치

- `/usr/share/ca-certificates`
- `/etc/certificates`

| 구분 | /usr/share/ca-certificates | /etc/certificates (또는 /etc/ssl) |
|---|---|---|
| 성격 | 시스템 기본 제공 (Read-Only 권장) | 사용자 정의 (Read-Write 관리) |
| 주요 내용물 | 전 세계 공인 CA 루트 인증서 목록 | 내 서버의 인증서, 비밀키, 사설 CA 인증서 |
| 주 목적 | 외부 웹사이트/서비스를 검증(신뢰)할 때 | 내 서버를 외부에 증명(인증)할 때 |
| 업데이트 방식 | OS 패키지 업데이트 (apt, yum 등) | 시스템 관리자가 수동으로 갱신 및 관리 |

### 1. /usr/share/ca-certificates (시스템 공인 인증서)

- 출처: 운영체제(OS) 패키지 매니저가 공식적으로 배포하고 관리합니다.
- 용도: 전 세계적으로 신뢰받는 공인 인증기관(CA)들의 루트(Root) 인증서가 모여 있는 곳입니다.
- 특징:
  - 사용자가 이 디렉토리의 파일을 직접 수정하거나 추가하지 않습니다.
    - ca-certificates 패키지가 업데이트될 때마다 자동으로 내용이 갱신됩니다.
    - 웹 브라우징(curl, wget 등) 시 외부 보안 사이트(HTTPS)를 신뢰할 수 있는지 검증할 때 이 곳의 인증서를 기반으로 확인합니다.

### 2. /etc/certificates (사용자/애플리케이션 맞춤 인증서)

- 출처: 시스템 관리자(사용자)가 직접 생성하거나 외부에서 가져온 인증서입니다.
- 용도: 특정 서비스(예: Apache, Nginx, VPN 등)를 구동하기 위한 서버 본인의 인증서와 비밀키를 보관하는 곳입니다.
- 특징:
  - 내부 사설 CA 인증서나 Let's Encrypt 등으로 발급받은 본인 서버의 도메인 인증서를 주로 저장합니다.
    - OS 업데이트의 영향을 받지 않으며, 관리자가 수동으로 파일을 관리합니다.
    - 시스템에 따라 /etc/ssl/certs 또는 /etc/pki 등으로 경로가 조금씩 다를 수 있습니다.

## 사설 인증서 등록

사설 인증서(또는 사설 CA 인증서)를 시스템에 등록하여 curl이나 wget 같은 도구들이 이를 신뢰하게 만들려면 시스템 신뢰 저장소 경로에 넣고 업데이트 명령어를 실행해야 합니다.

### 1. Ubuntu / Debian 계열

Ubuntu나 Debian에서는 `/usr/local/share/ca-certificates/` 경로를 사용합니다.

#### 별도 사설인증서 디렉토리를 추가하는 경우(추후 사설 인증서 구분 쉬움)

```bash
# 사설 인증서 저장 경로 생성
sudo mkdir /usr/share/ca-certificates/yourca

# 인증서 복사 (확장자는 반드시 .crt여야 합니다. .pem인 경우 확장자를 바꾸세요)
sudo cp your-ca.crt /usr/local/share/ca-certificates/yourca

# 시스템 CA 신뢰 저장소(trust store=시스템 전체 인증서 묶음 파일=/etc/ssl/certs/ca-certificates.crt) 업데이트

sudo dpkg-reconfigure ca-certificates
```

#### 시스템 인증서 경로에 인증서 파일을 추가 하는 경우(추후 사설 인증서 구분 어려움)

```bash
sudo cp your-ca.crt /usr/local/share/ca-certificates
sudo update-ca-certificates
```

- 성공하면 1 added, 0 removed;와 같은 메시지가 출력됩니다.

### 2. RHEL 계열(CentOS / RHEL / Rocky / Fedora)

레드햇 계열에서는 `/etc/pki/ca-trust/source/anchors/` 경로를 사용합니다.

```bash
# 인증서 복사
sudo cp your-ca.crt /etc/pki/ca-trust/source/anchors/

# 시스템 CA 신뢰 저장소(trust store) 활성화
sudo update-ca-trust enable

# 시스템 CA 신뢰 저장소(trust store) 업데이트
# /etc/pki/ca-trust/source/anchors 를 /etc/pki/tls/certs 에 반영
sudo update-ca-trust extract


# 설치된 cert 확인
cat /etc/pki/tls/certs/ca-bundle.crt
cat /etc/pki/tls/certs/ca-bundle.trust.crt
```
