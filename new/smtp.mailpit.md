# Mailpit으로 SMTP 서버 실행하기

Mailpit은 개발/테스트 환경에서 SMTP 수신을 확인할 때 유용한 경량 메일 서버다.  
아래 예시는 Docker 컨테이너 기준으로 정리했다.

## 1) 기본 실행 (TLS 없음, 인증 없음)

```bash
docker run --rm -d --name mailpit \
  -p 1025:1025 \
  -p 8025:8025 \
  axllent/mailpit:latest
```

| 포트 | 용도 |
|---|---|
| `1025` | SMTP 서버 포트 (애플리케이션이 메일 전송 시 접속) |
| `8025` | Mailpit 웹 UI 대시보드 |

### 웹 UI 접속이 방화벽으로 막힌 경우 (SSH 포트포워딩)

```bash
ssh -L 8025:localhost:8025 user@sshserver
```

로컬 브라우저에서 `http://localhost:8025`로 접속한다.

## 2) SMTP TLS(SSL) 활성화

### 2-1) 테스트용 self-signed 인증서 자동 생성 (`sans:`)

`sans:` 설정은 별도 인증서/키 파일 없이 Mailpit이 컨테이너 내부에서 self-signed 인증서를 자동 생성한다.

```bash
docker run --rm -d \
  --name mailpit \
  -p 1025:1025 \
  -p 8025:8025 \
  -e MP_SMTP_TLS_CERT='sans:localhost,127.0.0.1,172.16.100.100' \
  -e MP_SMTP_TLS_KEY='sans:localhost,127.0.0.1,172.16.100.100' \
  -e MP_SMTP_REQUIRE_TLS=true \
  axllent/mailpit:latest
```

- `MP_SMTP_REQUIRE_TLS=true` : TLS 없는 SMTP는 거부
- `sans:localhost,127.0.0.1,172.16.100.100` : self-signed 인증서의 SAN(Subject Alternative Name) 목록
  - 클라이언트가 `localhost`, `127.0.0.1`, `172.16.100.100`으로 접속할 때 인증서 이름 검증을 통과할 수 있도록 사용
  - 누구나 TCP 접속은 가능하지만, 클라이언트가 TLS 검증을 엄격히 하면 SAN에 없는 이름/IP로는 인증서 불일치로 실패할 수 있음

### 2-2) 실제 인증서/개인키 파일 사용

서버 인증서(`cert.pem`)와 개인키(`key.pem`)를 마운트해 TLS를 활성화할 수 있다.

```bash
-v /path/cert.pem:/certs/cert.pem:ro \
-v /path/key.pem:/certs/key.pem:ro \
-e MP_SMTP_TLS_CERT=/certs/cert.pem \
-e MP_SMTP_TLS_KEY=/certs/key.pem
```

## 3) SMTP 인증(user/password) 설정

아래 예시는 TLS 없이 SMTP 인증만 적용한다.

```bash
docker run --rm -d \
  --name mailpit \
  -p 1025:1025 \
  -p 8025:8025 \
  -e MP_SMTP_AUTH_ALLOW_INSECURE=true \
  -e MP_SMTP_AUTH_ACCEPT_ANY=false \
  -e MP_SMTP_AUTH_USER=myuser \
  -e MP_SMTP_AUTH_PASS=mypassword \
  axllent/mailpit:latest
```

- `MP_SMTP_AUTH_ALLOW_INSECURE=true` : 평문(비TLS) 인증도 허용
- `MP_SMTP_AUTH_USER/PASS`  + `MP_SMTP_AUTH_ACCEPT_ANY=false` : 지정 계정만 허용

## 4) SMTP 인증 + TLS 설정

아래 예시는 TLS와 SMTP 인증을 함께 적용한다.

```bash
docker run --rm -d \
  --name mailpit \
  -p 1025:1025 \
  -p 8025:8025 \
  -e MP_SMTP_TLS_CERT='sans:localhost,127.0.0.1,172.16.100.100' \
  -e MP_SMTP_TLS_KEY='sans:localhost,127.0.0.1,172.16.100.100' \
  -e MP_SMTP_REQUIRE_TLS=true \
  -e MP_SMTP_AUTH_ALLOW_INSECURE=false \
  -e MP_SMTP_AUTH_ACCEPT_ANY=false \
  -e MP_SMTP_AUTH_USER=myuser \
  -e MP_SMTP_AUTH_PASS=mypassword \
  axllent/mailpit:latest
```

- `MP_SMTP_AUTH_ALLOW_INSECURE=false` : 평문(비TLS) 인증 불가, `REQUIRE_TLS=true` 시 권장

## 참고

- 테스트 자동화 환경에서는 `--rm` 옵션으로 컨테이너 종료 시 자동 삭제되게 유지하면 편리하다.
- 사내망 환경에서는 방화벽/프록시 정책에 따라 8025 접근 및 SMTP 포트 접근 가능 여부를 먼저 확인한다.