# Self-Signed Certificate 정리

## 개요

**Self-signed certificate**는 인증서의 `subject`(주체)와 `issuer`(발급자)가 동일한 인증서다.  
주로 로컬/테스트 환경에서 TLS를 빠르게 검증할 때 사용한다.

- 상용 환경: 서버 인증서는 보통 **공인/사설 CA가 서명**
- 테스트 환경: 서버 인증서를 **자체 서명**하거나, 내부 CA를 만들어 서명

> 참고: 운영 환경에서는 일반적으로 신뢰된 CA 체인을 사용해야 하며, self-signed 인증서는 클라이언트 신뢰 저장소에 직접 등록해야 한다.

## 핵심 개념

### 1) Self-signed 서버 인증서 (단일 인증서 방식)

- 서버 키로 서버 인증서를 스스로 서명
- 구성은 간단하지만 다수 시스템 관리가 불편할 수 있음

### 2) 내부 CA + 서버 인증서 (권장 테스트 방식)

- 내부 CA 인증서(`ca-cert.pem`)를 self-signed로 만들고
- 서버 인증서는 CA 키로 서명
- 여러 서버 인증서를 일관되게 관리하기 쉬움

## SAN과 CN

현대 TLS 클라이언트는 서버 이름 검증 시 **SAN(Subject Alternative Name)**을 사용한다.  
CN은 보조적 의미만 가지며, SAN이 없으면 검증 실패할 수 있다.

- 도메인으로 접속: `DNS:example.com` 형태 SAN 필요
- IP로 접속: `IP:192.168.1.10` 형태 SAN 필요

## OpenSSL 설치

### Ubuntu/Debian

```bash
sudo apt-get update
sudo apt-get install -y openssl
```

### CentOS/RHEL

```bash
sudo yum install -y openssl
```

## 방법 A: 서버용 self-signed 인증서 빠르게 생성

```bash
# 1) 서버 개인키 생성
openssl genrsa -out selfsign-key.pem 4096

# 2) 서버 인증서 생성 (자체 서명)
openssl req -x509 -new -key selfsign-key.pem -sha256 -days 3650 \
  -out selfsign-cert.pem \
  -subj "/C=KR/ST=Seoul/L=Seoul/O=Test/OU=Dev/CN=localhost" \
  -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

- `4096`: 키의 비트 수, genrsa 명령을 사용하므로 rsa 키의 비트 길이.
- `-x509`: CSR 출력 대신 즉시 X.509 인증서를 생성(자체 서명 흐름).
- `-new`: 새로운 인증서 요청(요청 객체)을 생성.
- `-key selfsign-key.pem`: 서명에 사용할 개인키 파일 지정.
- `-sha256`: 서명 해시 알고리즘으로 SHA-256 사용.
- `-days 3650`: 인증서 유효기간을 3650일로 설정.
- `-out selfsign-cert.pem`: 생성된 인증서 출력 파일 경로.
- `-subj ".../CN=localhost"`: 주체(Subject) DN 값을 비대화식으로 지정.
- `-addext "subjectAltName=..."`: SAN 확장 필드(DNS/IP 식별자) 추가.

## 방법 B: 내부 CA로 서버 인증서 서명

### 1) CA 개인키/인증서 생성

```bash
# CA 개인키
openssl genrsa -out ca-key.pem 4096

# CA 인증서 (self-signed)
openssl req -x509 -new -key ca-key.pem -sha256 -days 3650 \
  -out ca-cert.pem \
  -subj "/C=KR/ST=Seoul/L=Seoul/O=Test/OU=Dev/CN=sw-test-ca"
```

### 2) 서버 개인키/CSR 생성

> **CSR(Certificate Signing Request)**: 인증서 발급을 위해 CA에 제출하는 요청 파일(공개키 + 주체 정보 + 서명 정보 포함). CSR에 포함되는 서버 공개키는 입력 된 서버 개인키를 이용하여 생성된다. 

```bash
# 서버 개인키(server-key.pem) 생성
openssl genrsa -out server-key.pem 4096

# 서버 CSR (중요: 서버 개인키를 사용해야 함)
openssl req -new -key server-key.pem -out server-csr.pem \
  -subj "/C=KR/ST=Seoul/L=Seoul/O=Test/OU=Dev/CN=localhost" \
  -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

> 주의: CSR 생성 시 `-key ca-key.pem`을 쓰는 것은 잘못이다. 반드시 `server-key.pem`을 사용해야 한다.

### 3) CSR을 이용한 서버 인증서 생성(서명)

`server-csr.pem`(CSR) + `ca-cert.pem`(CA 인증서) + `ca-key.pem`(CA 개인키)를 사용해 서버 인증서(`server-cert.pem`)를 서명/발급한다.

- CSR ( server-csr.pem ): 발급 대상 정보/공개키 제공
- CA 인증서 ( ca-cert.pem ): 발급자(issuer = 여기서 CA) 정보 제공
- CA 개인키 ( ca-key.pem ): CA 개인키로 서버 인증서를 서명함

```bash
openssl x509 -req -in server-csr.pem -CA ca-cert.pem -CAkey ca-key.pem \
  -CAcreateserial -out server-cert.pem -days 825 -sha256 \
  -copy_extensions copy
```

## 생성 파일 관계

- `ca-key.pem`: CA 개인키
- `ca-cert.pem`: CA 인증서 (ca-key.pem으로 서명된 self-signed)
- `server-key.pem`: 서버 개인키
- `server-csr.pem`: 서버 CSR (server-key.pem 기반)
- `server-cert.pem`: 서버 인증서 (CA가 서명)

## 신뢰 설정 요약

클라이언트가 서버 인증서를 검증하려면, 서버 인증서를 서명한 **CA 인증서(`ca-cert.pem`)**를 클라이언트의 신뢰 저장소에 등록해야 한다.

- Linux(예: Debian/Ubuntu): `/usr/local/share/ca-certificates` + `update-ca-certificates`
- Java: truststore(`cacerts` 또는 사용자 truststore) 등록
- 브라우저/OS: OS 인증서 관리자에서 신뢰 루트로 등록

## certtool(gnutls) 예시

참고: <https://github.com/rsyslog/rsyslog-doc/blob/master/source/tutorials/tls.rst>

```bash
sudo yum install -y gnutls-utils rsyslog-gnutls
sudo mkdir -p /etc/rsyslog.d/keys
cd /etc/rsyslog.d/keys
```

### CA 키/인증서

```bash
certtool --generate-privkey --outfile ca-key.pem
chmod 400 ca-key.pem
certtool --generate-self-signed --load-privkey ca-key.pem --outfile ca.pem
```

### 서버 키/CSR/인증서

```bash
certtool --generate-privkey --outfile server-key.pem
certtool --generate-request --load-privkey server-key.pem --outfile request.pem
certtool --generate-certificate \
  --load-request request.pem \
  --outfile cert.pem \
  --load-ca-certificate ca.pem \
  --load-ca-privkey ca-key.pem
```

## 참고 자료

- <https://www.sslcert.co.kr/guides/Install/Apache-SSL-Certificate-Install>
- <https://cert.crosscert.com/wp-content/uploads/2019/02/Apache-SSL-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EC%84%A4%EC%B9%98-%EB%A7%A4%EB%89%B4%EC%96%BC.pdf>
