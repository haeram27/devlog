# Reverse Proxy를 이용한 단일 포트(443) 기반 다중 웹 애플리케이션 서비스 구성

리버스 프록시 전용/중심 솔루션이 여러가지 있다.

대표적으로:

- HAProxy: L4/L7 프록시·로드밸런서에 매우 강함, 고성능/안정성
- Envoy Proxy: 서비스 메쉬/마이크로서비스 환경에 강함 (동적 라우팅, observability)
- Traefik: Docker/Kubernetes 연동이 쉬워서 자동 라우팅/자동 인증서에 강함
- Caddy: 설정이 간단하고 HTTPS 자동화(ACME)가 매우 편함
- Apache HTTP Server (mod_proxy): 기존 Apache 운영 환경이면 자연스럽게 선택 가능
- Kong / APISIX / Tyk: 리버스 프록시 + API Gateway 기능(인증, rate limit, plugin)

단순 웹서비스면 Caddy/Traefik, 전통적 고성능 LB면 HAProxy, 클라우드 네이티브/서비스메쉬면 Envoy, API 정책 중심이면 Kong/APISIX가 잘 맞습니다.

L4/L7 LB 및 proxy 기능이 주라면 `haproxy` 사용을 가장 추천 한다.

## 개요

서버에 여러 개의 웹 애플리케이션이 설치되어 있는 경우 일반적으로 각 애플리케이션은 서로 다른 포트를 사용한다.

예를 들어 다음과 같은 환경을 가정한다.

| 애플리케이션 | 내부 포트 |
|-------------|----------|
| A App | 8080 |
| B App | 9090 |
| C App | 5000 |

일반적인 구성에서는 각 포트에 대해 방화벽 예외 처리가 필요하다.

```text
1.1.1.1:8080
1.1.1.1:9090
1.1.1.1:5000
```

이 방식은 관리해야 할 포트가 많아지고 보안 정책 적용이 복잡해지는 문제가 있다.

---

## 해결 방안

서버 앞단에 Reverse Proxy(NGINX, Apache, IIS 등)를 배치하고 외부에서는 HTTPS(443) 포트만 개방한다.

```text
클라이언트
     |
     | HTTPS(443)
     ▼
+------------------+
| Reverse Proxy    |
| (NGINX/IIS 등)   |
+------------------+
     |
     +----> A App (8080)
     |
     +----> B App (9090)
     |
     +----> C App (5000)
```

이 경우 방화벽에서는 443 포트만 허용하면 된다.

---

## DNS 또는 Hosts 설정

클라이언트에서 다음과 같이 DNS 또는 Hosts 파일을 구성한다.

### Hosts 파일 예시

```text
1.1.1.1 a.myserver.com
1.1.1.1 b.myserver.com
1.1.1.1 c.myserver.com
```

모든 도메인은 동일한 서버 IP를 가리킨다.

---

## 동작 원리

브라우저는 HTTPS 요청 시 대상 도메인명을 서버에 함께 전달한다.

예를 들어 사용자가 다음 URL에 접속하면

```text
https://a.myserver.com
```

브라우저는 다음과 같은 Host 정보를 전송한다.

```http
Host: a.myserver.com
```

Reverse Proxy는 이 Host 값을 확인하여 적절한 백엔드 애플리케이션으로 요청을 전달한다.

---

## 요청 흐름

### A 애플리케이션 접속

```text
https://a.myserver.com
        |
        ▼
Reverse Proxy
        |
        ▼
127.0.0.1:8080
```

### B 애플리케이션 접속

```text
https://b.myserver.com
        |
        ▼
Reverse Proxy
        |
        ▼
127.0.0.1:9090
```

### C 애플리케이션 접속

```text
https://c.myserver.com
        |
        ▼
Reverse Proxy
        |
        ▼
127.0.0.1:5000
```

---

## NGINX 설정 예시

```nginx
server {
    listen 443 ssl;
    server_name a.myserver.com;
    ssl_certificate     /etc/nginx/ssl/a.myserver.com.fullchain.crt;
    ssl_certificate_key /etc/nginx/ssl/a.myserver.com.key;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}

server {
    listen 443 ssl;
    server_name b.myserver.com;
    ssl_certificate     /etc/nginx/ssl/b.myserver.com.fullchain.crt;
    ssl_certificate_key /etc/nginx/ssl/b.myserver.com.key;

    location / {
        proxy_pass http://127.0.0.1:9090;
    }
}

server {
    listen 443 ssl;
    server_name c.myserver.com;
    ssl_certificate     /etc/nginx/ssl/c.myserver.com.fullchain.crt;
    ssl_certificate_key /etc/nginx/ssl/c.myserver.com.key;

    location / {
        proxy_pass http://127.0.0.1:5000;
    }
}
```

### SNI + SSL 인증서 선택 동작

클라이언트는 TLS 핸드셰이크 단계에서 SNI(접속 도메인명)를 전송하고, NGINX는 `server_name`과 SNI를 매칭해 해당 `server` 블록의 인증서를 선택한다.

예를 들어:

- `a.myserver.com` 접속 시 `a.myserver.com`용 인증서/키 사용
- `b.myserver.com` 접속 시 `b.myserver.com`용 인증서/키 사용
- `c.myserver.com` 접속 시 `c.myserver.com`용 인증서/키 사용

### `ssl_certificate`와 `ssl_certificate_key`의 의미

- `ssl_certificate`: **서버 인증서 체인 파일**
  - 서버의 **공개키(public key)**가 포함된 X.509 인증서
  - 보통 서버 인증서 + 중간 인증서(chain)를 함께 둔 `fullchain` 파일 사용
- `ssl_certificate_key`: **서버 개인키(private key) 파일**
  - 위 인증서의 공개키와 쌍을 이루는 개인키
  - 외부에 노출되면 안 되며 파일 권한을 엄격히 제한해야 함

정리하면, 공개키는 인증서(`ssl_certificate`) 안에 있고, 개인키는 `ssl_certificate_key`에 별도 저장된다.

### 기본 서버(default server) 권장 설정

SNI가 매칭되지 않는 요청을 위한 기본 블록을 두는 것이 안전하다.

```nginx
server {
    listen 443 ssl default_server;
    server_name _;
    ssl_certificate     /etc/nginx/ssl/default.crt;
    ssl_certificate_key /etc/nginx/ssl/default.key;
    return 444;
}
```

---

## 구성의 장점

### 보안 강화

외부에 공개되는 포트를 최소화할 수 있다.

```text
기존
 ├─ 443
 ├─ 8080
 ├─ 9090
 └─ 5000

개선 후
 └─ 443
```

### 방화벽 정책 단순화

방화벽 예외 처리를 443 포트 하나만 관리하면 된다.

### SSL 인증서 관리 용이

모든 HTTPS 트래픽을 Reverse Proxy에서 처리할 수 있다.

### 서비스 확장 용이

새로운 애플리케이션 추가 시 방화벽 변경 없이 도메인과 프록시 설정만 추가하면 된다.

```text
d.myserver.com -> 7000
e.myserver.com -> 8000
```

---

## 실제 구성 예시

```text
                      Internet
                          |
                          |
                    TCP 443
                          |
                          ▼
                  +---------------+
                  |   Firewall    |
                  +---------------+
                          |
                          ▼
                +-------------------+
                |   Reverse Proxy   |
                |     NGINX/IIS     |
                +-------------------+
                   |      |      |
                   |      |      |
                   ▼      ▼      ▼
            8080(A) 9090(B) 5000(C)
```

### DNS 또는 Hosts

```text
a.myserver.com -> 1.1.1.1
b.myserver.com -> 1.1.1.1
c.myserver.com -> 1.1.1.1
```

### 라우팅 규칙

```text
a.myserver.com -> localhost:8080
b.myserver.com -> localhost:9090
c.myserver.com -> localhost:5000
```

---

## 핵심 기술 용어

### Reverse Proxy

클라이언트의 요청을 대신 받아 내부 서버 또는 애플리케이션으로 전달하는 프록시 서버.

### Virtual Host

동일한 IP와 동일한 포트(예: 443)에서 요청된 도메인명에 따라 서로 다른 서비스로 연결하는 기능.

### Host Header

브라우저가 HTTP 요청 시 전달하는 도메인 정보.

```http
Host: a.myserver.com
```

Reverse Proxy는 이 값을 기준으로 어떤 애플리케이션에 요청을 전달할지 결정한다.

### SNI (Server Name Indication)

HTTPS 연결 시 클라이언트가 접속하려는 도메인명을 TLS 핸드셰이크 단계에서 전달하는 기능.

이를 통해 하나의 IP 주소와 하나의 443 포트에서도 여러 HTTPS 사이트를 운영할 수 있다.

---

## 최종 정리

이 구성은 다음과 같은 특징을 가진다.

- 외부에는 443 포트만 공개
- 모든 웹 애플리케이션은 내부 포트 사용
- DNS 또는 Hosts 파일에서 모든 도메인을 동일 IP로 설정
- Reverse Proxy가 도메인명(Host Header/SNI)을 기준으로 적절한 애플리케이션에 전달
- 새로운 서비스 추가 시 방화벽 변경 없이 Reverse Proxy 설정만 추가

```text
https://a.myserver.com ──► Reverse Proxy ──► localhost:8080
https://b.myserver.com ──► Reverse Proxy ──► localhost:9090
https://c.myserver.com ──► Reverse Proxy ──► localhost:5000
```

즉, 하나의 서버 IP와 하나의 공개 포트(443)만으로 여러 웹 애플리케이션을 서비스할 수 있으며, 이를 "Host 기반 Virtual Host + Reverse Proxy 구성"이라고 한다.