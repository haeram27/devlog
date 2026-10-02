# SAN(Subject Alternative Name) 설명과 사용 예시

## 1) SAN이란 무엇인가

SAN(Subject Alternative Name)은 TLS/X.509 인증서의 표준 확장 필드다.  
인증서가 어떤 도메인/호스트명/IP에 대해 유효한지를 정의한다.

**TLS 클라이언트는 서버 인증서를 검증할 때, 접속한 대상 서버의 hostname 또는 IP가 서버 인증서의 SAN 필드에 명시된 이름에 포함되어 있는지 확인**한다.

- 포함되어 있으면: 이름 검증 통과 가능
- 포함되어 있지 않으면: 이름 불일치 경고/실패 가능

즉, SAN는 HTTPS, SMTP, Kafka 등 TLS를 쓰는 서비스 전반에서 공통으로 사용된다.

## 2) SAN과 ACL(접속 허용)의 차이

SAN은 **인증서의 이름 검증 범위**를 정의하는 값이다.  
방화벽/보안그룹처럼 접속 자체를 허용/차단하는 ACL 기능이 아니다.

- SAN: "이 인증서가 어떤 이름/IP에 유효한가?"
- ACL/방화벽: "누가 어떤 포트로 접속 가능한가?"

## 3) 서버 어플리케이션 SANS 설정 예제

### Mailpit에서 SAN 사용 예시

```bash
MP_SMTP_TLS_CERT='sans:localhost,127.0.0.1,172.16.100.100'
MP_SMTP_TLS_KEY='sans:localhost,127.0.0.1,172.16.100.100'
MP_SMTP_REQUIRE_TLS=true
```

- `sans:`는 Mailpit 에서 self-singed 인증서 생성 및 SANS 값 설정을 쉽게 지원하기 위한 특수 문법이다. `sans:` 구문 자체가 어떤 표준은 아니고, **SAN이라는 표준 필드를 설정하기 위한 Mailpit에서의 편의 문법**이다.
- 위 예시는 self-signed 인증서 생성 시 SAN에 `localhost`, `127.0.0.1`, `172.16.100.100`을 포함한다.
- `MP_SMTP_REQUIRE_TLS=true`는 TLS 없는 SMTP를 거부한다.

### Nginx에서 SAN 사용 예시

Nginx는 `sans:` 문법을 직접 쓰지 않는다.  
대신 SAN이 포함된 인증서를 `ssl_certificate`로 로드해 사용한다.

```nginx
server {
    listen 443 ssl;
    server_name app.example.com;

    ssl_certificate     /etc/nginx/ssl/app.example.com.fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/app.example.com.key;
}
```

핵심은 인증서 파일 내부 SAN에 `app.example.com`(또는 필요한 DNS/IP)이 포함되어 있어야 한다는 점이다.

### Kafka에서 SAN 사용 예시

Kafka도 TLS hostname verification을 사용하므로 브로커 인증서 SAN이 중요하다.

- 브로커 접속 주소가 `broker1.example.com`이면 SAN에 `DNS:broker1.example.com` 필요
- IP로 접속하면 SAN에 해당 `IP:<address>` 필요

OpenSSL CSR 예시:

```ini
[ req ]
distinguished_name = dn
req_extensions = req_ext

[ dn ]
CN = broker1.example.com

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = broker1.example.com
IP.1 = 10.0.0.21
```
