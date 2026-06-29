# HTTP/S API 보안 기법 정리

이 문서는 HTTP/S 기반 API 서비스에서 사용할 수 있는 주요 보안 기법을 한 곳에 정리한 것입니다.

- 목표는 "어떤 기법을 어디에 써야 하는가"를 빠르게 판단하는 것입니다.
- 각 기법마다 용도, 부족한 점, 부족한 점을 보완하는 기술을 함께 적었습니다.
- 하나의 기법만으로 충분한 경우는 드물며, 보통 여러 계층의 방어를 조합해야 합니다.

## 1. 전체 요약

아래는 HTTP/S API 서비스에서 실무적으로 고려할 수 있는 주요 보안 기법 목록과 한 줄 설명입니다.

| 기법 | 한 줄 설명 |
| --- | --- |
| HTTPS(TLS) | 전송 구간 암호화와 서버 인증으로 기본 보안 채널을 만듭니다. |
| mTLS | 서버뿐 아니라 클라이언트 인증서까지 검증해 상호 인증합니다. |
| HSTS | 브라우저가 항상 HTTPS로만 접속하도록 강제합니다. |
| Certificate Pinning | 특정 인증서 또는 공개키만 신뢰하게 해 MITM 위험을 줄입니다. |
| HMAC 서명 | 공유 비밀키로 요청 무결성과 송신자 진위를 검증합니다. |
| Digital Signature | 비대칭키 서명으로 무결성과 송신자 증명을 제공합니다. |
| JWT/JWS | 서명된 토큰으로 인증 정보를 전달합니다. |
| JWE | 토큰 내용 자체를 암호화해 민감 정보를 숨깁니다. |
| OAuth 2.0 | 액세스 토큰 기반 위임 권한 부여 프레임워크입니다. |
| OpenID Connect | OAuth 2.0 위에 사용자 인증 계층을 추가합니다. |
| API Key | 간단한 호출자 식별과 접근 제어에 쓰는 정적 키입니다. |
| Session Cookie | 브라우저 기반 상태 유지 인증에 사용합니다. |
| Nonce | 요청 1회성을 강제해 재전송 공격을 줄입니다. |
| Timestamp | 요청 유효 시간을 제한해 오래된 요청 재사용을 막습니다. |
| Sequence Number | 요청 순서와 중복 여부를 추적해 재전송과 순서 뒤바뀜을 방지합니다. |
| Idempotency Key | 같은 비즈니스 요청의 중복 처리를 막습니다. |
| Delivery ID / Request ID | 이미 처리한 요청인지 식별하는 고유 ID입니다. |
| CSRF Token | 브라우저에서 사용자의 의도 없는 요청 전송을 막습니다. |
| SameSite Cookie | 교차 사이트 요청에 쿠키가 자동 포함되는 범위를 제한합니다. |
| CORS | 브라우저가 허용된 출처에서만 API를 호출하게 제한합니다. |
| Origin / Referer 검증 | 브라우저 요청의 출처를 추가로 확인합니다. |
| Input Validation | 잘못된 형식과 악성 입력을 초기에 차단합니다. |
| Output Encoding | 응답 데이터가 클라이언트에서 스크립트로 해석되는 일을 줄입니다. |
| Schema Validation | JSON/XML 구조를 계약대로 강제합니다. |
| Rate Limiting | 과도한 요청 빈도를 제한해 남용과 DoS를 줄입니다. |
| Quota / Throttling | 사용자나 클라이언트별 사용량과 속도를 제어합니다. |
| WAF | 알려진 웹 공격 패턴을 앞단에서 차단합니다. |
| IP Allowlist | 허용된 네트워크나 파트너 IP만 접속시킵니다. |
| RBAC | 역할 단위로 API 권한을 부여합니다. |
| ABAC / Policy Engine | 속성과 정책 기반으로 세밀한 인가를 수행합니다. |
| Scope | 토큰이 허용하는 API 범위를 좁힙니다. |
| Secret Rotation | 유출 시 피해를 줄이기 위해 비밀값을 주기적으로 교체합니다. |
| KMS / HSM / Secret Manager | 키와 비밀값을 안전하게 저장하고 관리합니다. |
| Payload Encryption | TLS 외에 본문 자체를 추가 암호화합니다. |
| Field-level Encryption | 민감 필드만 별도로 암호화합니다. |
| Audit Logging | 누가 무엇을 호출했는지 추적 가능하게 남깁니다. |
| Security Monitoring / SIEM | 이상 징후와 공격 패턴을 탐지합니다. |
| Retry with Backoff | 장애 시 재시도 폭주를 줄여 보안 장치 우회를 어렵게 합니다. |
| Circuit Breaker / Backpressure | 과부하 상황에서 서비스 붕괴를 막습니다. |
| Dependency / Patch Management | 라이브러리와 런타임 취약점을 줄입니다. |
| Security Headers | 브라우저 보안 동작을 강화하는 헤더 세트입니다. |

## 2. 계층별로 보는 큰 그림

HTTP/S API 보안은 보통 아래 계층을 함께 봐야 합니다.

- 전송 계층 보호: HTTPS, mTLS, HSTS, 인증서 검증
- 요청 진위와 무결성 보호: HMAC, 디지털 서명, JWT/JWS, nonce, timestamp
- 인증과 인가: OAuth 2.0, OIDC, API Key, Session, RBAC, ABAC, Scope
- 브라우저 및 웹 보안: CSRF, SameSite, CORS, Origin 검증, 보안 헤더
- 남용 및 운영 방어: Rate Limit, WAF, IP Allowlist, Audit Log, Monitoring
- 비밀값 및 데이터 보호: Secret Rotation, KMS/HSM, Payload Encryption, Field-level Encryption

## 3. 전송 계층 보안

### 3.1 HTTPS(TLS)

용도

- 클라이언트와 서버 사이 전송 구간을 암호화합니다.
- 서버 인증서를 검증해 사용자가 진짜 서버에 연결했는지 확인합니다.

부족한 점

- TLS는 전송 중 보호만 담당합니다.
- API 호출 권한, 메시지 재사용, 비즈니스 로직 남용은 막지 못합니다.
- TLS 종료 지점 이후 내부 구간은 별도 보호가 필요할 수 있습니다.

보완 기술

- 인증/인가: OAuth 2.0, API Key, Session
- 요청 무결성: HMAC, 디지털 서명
- 재전송 방지: nonce, timestamp, idempotency key
- 내부 구간 보호: 서비스 메시 TLS, 내부망 정책, payload encryption

### 3.2 mTLS

용도

- 서버와 클라이언트가 서로 인증서를 검증합니다.
- 서버 대 서버 통신, 파트너 연동, 내부 서비스 간 호출에서 강력한 호출자 인증 수단이 됩니다.

부족한 점

- 인증서 발급, 배포, 폐기, 갱신 운영 비용이 큽니다.
- 사용자 단위 권한 모델에는 직접 대응하지 못합니다.
- 인증서만으로는 요청별 세부 권한이나 재전송 방지를 처리하지 못합니다.

보완 기술

- 세부 인가: OAuth Scope, RBAC, ABAC
- 요청 보호: HMAC, nonce, timestamp
- 운영 자동화: PKI 자동화, 인증서 로테이션, service mesh

### 3.3 HSTS

용도

- 브라우저가 해당 도메인에 HTTP로 접속하지 못하게 하고 HTTPS만 사용하게 합니다.

부족한 점

- 브라우저 환경에 주로 의미가 있습니다.
- 서버 간 API 호출이나 비브라우저 클라이언트에는 직접 효과가 작습니다.

보완 기술

- API 클라이언트 설정 강제, TLS 정책, 리다이렉트 금지, 인증서 검증 강화

### 3.4 Certificate Pinning

용도

- 특정 인증서 또는 공개키만 신뢰하게 해 잘못된 CA 체인이나 MITM 위험을 줄입니다.

부족한 점

- 인증서 교체 시 장애 위험이 큽니다.
- 운영 실수 시 클라이언트가 모두 접속 불가 상태가 될 수 있습니다.

보완 기술

- 백업 핀 준비, 점진적 배포, 짧은 TTL, 자동화된 인증서 갱신 절차

## 4. 요청 진위와 무결성 보안

### 4.1 HMAC

용도

- 공유 Secret으로 요청 본문이나 특정 헤더를 서명해 무결성과 송신자 진위를 검증합니다.
- 웹훅, 파트너 서버 간 호출에서 자주 사용합니다.

부족한 점

- Secret을 공유해야 하므로 키 유출 시 양쪽 모두 영향이 큽니다.
- 부인 방지를 제공하지 못합니다.
- HMAC만으로는 재전송 공격을 막지 못합니다.

보완 기술

- 키 관리: Secret Rotation, Secret Manager, KMS
- 재전송 방지: nonce, timestamp, sequence number, delivery id, idempotency key
- 전송 보호: TLS

### 4.2 Digital Signature

용도

- 개인키로 서명하고 공개키로 검증해 무결성과 송신자 증명을 제공합니다.
- 공유 Secret을 나누기 싫은 파트너 연동에 적합합니다.

부족한 점

- 키쌍, 공개키 배포, 키 폐기 관리가 복잡합니다.
- 서명 자체만으로 재전송을 막지 못합니다.

보완 기술

- 키 배포 체계: PKI, JWKS
- 재전송 방지: nonce, timestamp
- 전송 보호: TLS

### 4.3 JWT/JWS

용도

- 서명된 토큰 안에 주체, 권한, 만료 시각 등의 클레임을 담아 인증과 인가 전달에 사용합니다.

부족한 점

- 토큰 탈취 시 만료 전까지 재사용될 수 있습니다.
- 발급 후 즉시 회수가 까다롭습니다.
- 클레임을 과도하게 넣으면 정보 노출 범위가 커집니다.

보완 기술

- 짧은 만료 시간, refresh token, revocation 전략
- HTTPS 강제, secure storage
- scope 최소화, audience 검증, issuer 검증

### 4.4 JWE

용도

- 토큰 내용 자체를 암호화해 중간에 내용을 읽지 못하게 합니다.

부족한 점

- 키 관리와 디버깅이 더 복잡합니다.
- 암호화해도 재전송과 권한 남용 자체는 막지 못합니다.

보완 기술

- 서명과 병행: JWS + JWE
- 키 관리: KMS, JWKS
- 재전송 방지: nonce, 짧은 만료 시간

### 4.5 Nonce

용도

- 요청마다 한 번만 쓰는 난수를 붙여 같은 요청이 다시 수용되지 않게 합니다.

부족한 점

- 서버가 사용한 nonce를 저장하고 재사용 여부를 확인해야 합니다.
- 저장소 일관성, TTL, 분산 환경 동기화 비용이 있습니다.

보완 기술

- Redis 같은 짧은 TTL 저장소
- timestamp와 함께 사용해 저장 기간 제한
- request signature와 함께 묶어서 검증

### 4.6 Timestamp

용도

- 요청 생성 시각을 포함해 허용 시간 창 내 요청만 받습니다.

부족한 점

- 서버와 클라이언트의 시계 차이에 민감합니다.
- 허용 시간 창 내 재전송은 여전히 가능할 수 있습니다.

보완 기술

- nonce와 함께 사용
- 짧은 허용 윈도우 적용
- NTP 동기화, clock skew 보정

### 4.7 Sequence Number

용도

- 요청 순서 번호를 부여해 중복과 역순 재전송을 탐지합니다.

부족한 점

- 세션별 상태 저장이 필요합니다.
- 병렬 요청이나 멀티노드 환경에서 관리가 복잡합니다.

보완 기술

- 세션 범위 정의
- 재정렬 허용 정책
- nonce 또는 delivery id와 조합

### 4.8 Idempotency Key

용도

- 같은 비즈니스 요청을 중복 처리하지 않도록 합니다.
- 결제, 주문 생성, 송금 같은 쓰기 요청에 특히 중요합니다.

부족한 점

- 보안 기법이라기보다 중복 실행 방지 기법입니다.
- 키 범위, 저장 기간, 응답 재사용 정책을 잘못 정하면 부작용이 있습니다.

보완 기술

- nonce, timestamp와 결합
- 요청 본문 해시와 함께 저장
- 적절한 TTL과 상태 코드 정책 수립

### 4.9 Delivery ID / Request ID

용도

- 이미 처리한 요청인지 식별합니다.
- 웹훅 이벤트 중복 수신 방지에 유용합니다.

부족한 점

- 고유 ID가 없거나 공격자가 임의 생성할 수 있으면 한계가 있습니다.
- ID만 보고 요청 내용의 변조 여부까지 판단할 수는 없습니다.

보완 기술

- HMAC 또는 디지털 서명과 함께 사용
- TTL 저장소와 중복 기록 유지
- 파트너 시스템과 ID 생성 규칙 합의

## 5. 인증(Authentication)과 인가(Authorization)

### 5.1 API Key

용도

- 간단한 서버 간 호출 식별, 내부 도구 호출, 낮은 위험도의 파트너 API 보호에 사용합니다.

부족한 점

- 정적 비밀값이라 유출 시 바로 악용됩니다.
- 사용자 단위 권한 모델이나 세밀한 위임에 약합니다.
- 키만으로는 요청 무결성이나 재전송 방지가 되지 않습니다.

보완 기술

- HTTPS 필수, IP Allowlist, Rate Limit
- HMAC 서명과 결합
- 키 로테이션, 키별 Scope, 사용량 모니터링

### 5.2 Session Cookie

용도

- 브라우저 로그인 세션을 유지합니다.
- 서버가 세션 상태를 관리할 수 있어 즉시 무효화가 쉽습니다.

부족한 점

- CSRF, 세션 탈취, 쿠키 노출 위험이 있습니다.
- 수평 확장 시 세션 저장소 관리가 필요합니다.

보완 기술

- HttpOnly, Secure, SameSite 쿠키
- CSRF Token
- 세션 고정 방지, 짧은 만료 시간, 세션 저장소 보호

### 5.3 OAuth 2.0

용도

- 액세스 토큰으로 권한을 위임합니다.
- 써드파티 앱, 모바일 앱, SPA, 서버 간 호출에서 표준적으로 사용합니다.

부족한 점

- 프로토콜이 복잡해 잘못 구현하면 취약해집니다.
- 토큰 유출, 과도한 scope, 잘못된 redirect URI가 위험합니다.

보완 기술

- PKCE, 짧은 access token 수명, refresh token 보호
- scope 최소화, audience 제한
- token introspection 또는 revocation 정책

### 5.4 OpenID Connect

용도

- OAuth 2.0 위에서 사용자 인증 결과를 표준화합니다.

부족한 점

- 인증과 API 인가를 혼동하기 쉽습니다.
- ID token을 API access token처럼 잘못 쓰는 실수가 흔합니다.

보완 기술

- ID token과 access token 역할 분리
- nonce, state 검증
- issuer, audience, signature 검증

### 5.5 RBAC

용도

- 역할 단위로 접근 권한을 관리합니다.

부족한 점

- 세밀한 조건이 많아질수록 역할 폭발이 생깁니다.

보완 기술

- ABAC, 정책 엔진, scope 세분화

### 5.6 ABAC / Policy Engine

용도

- 사용자, 리소스, 환경 속성에 따라 동적으로 인가합니다.

부족한 점

- 정책 복잡도가 높고 디버깅이 어렵습니다.

보완 기술

- 정책 테스트 자동화
- 공통 정책 저장소와 감사 로그
- RBAC와 혼합 모델 적용

### 5.7 Scope

용도

- 토큰이 접근 가능한 API 범위를 좁힙니다.

부족한 점

- 너무 넓거나 추상적인 scope 설계는 실효성이 떨어집니다.

보완 기술

- 리소스 단위 권한, audience 분리, RBAC/ABAC와 병행

## 6. 브라우저 기반 API 보안

### 6.1 CSRF Token

용도

- 사용자가 로그인된 브라우저에서 의도하지 않은 상태 변경 요청이 전송되는 것을 막습니다.

부족한 점

- 브라우저 쿠키 기반 세션 모델에 주로 해당합니다.
- 토큰 저장/전달 구현이 틀리면 효과가 없습니다.

보완 기술

- SameSite Cookie
- Origin/Referer 검증
- 상태 변경 메서드 분리

### 6.2 SameSite Cookie

용도

- 교차 사이트 요청에 쿠키가 자동 전송되는 범위를 제한합니다.

부족한 점

- 일부 레거시 흐름이나 외부 로그인 연동과 충돌할 수 있습니다.

보완 기술

- CSRF Token, Secure, HttpOnly
- 필요 시 SameSite=None + Secure 조합을 제한적으로 사용

### 6.3 CORS

용도

- 브라우저에서 어떤 Origin이 API를 호출할 수 있는지 제한합니다.

부족한 점

- 브라우저 보안 정책일 뿐 서버 인증 수단이 아닙니다.
- 서버 간 호출, curl, 봇 호출은 막지 못합니다.

보완 기술

- 인증/인가 기법 병행
- 엄격한 Origin 화이트리스트
- credential 사용 여부 최소화

### 6.4 Origin / Referer 검증

용도

- 요청이 어느 웹 페이지에서 시작되었는지 추가 확인합니다.

부족한 점

- 모든 요청에 항상 신뢰 가능하게 존재하지는 않습니다.
- 프라이버시 정책, 프록시, 브라우저 설정에 영향받을 수 있습니다.

보완 기술

- CSRF Token과 병행
- SameSite Cookie와 병행

### 6.5 Security Headers

용도

- CSP, X-Frame-Options, X-Content-Type-Options 같은 헤더로 브라우저 공격면을 줄입니다.

부족한 점

- 브라우저 환경 바깥의 API 보안 문제는 해결하지 못합니다.

보완 기술

- 입력/출력 검증
- 세션 보안, CORS, CSRF 방어와 결합

## 7. 입력 검증과 데이터 보호

### 7.1 Input Validation

용도

- 예상하지 않은 형식, 길이, 타입, 문자셋, 범위의 입력을 차단합니다.

부족한 점

- 인증과 인가를 대체하지 못합니다.
- 검증 누락 시 우회가 생깁니다.

보완 기술

- Schema Validation
- 파라미터 바인딩 안전화
- 출력 인코딩, prepared statement

### 7.2 Schema Validation

용도

- JSON Schema, OpenAPI Schema, XML Schema 등으로 요청 구조를 강제합니다.

부족한 점

- 비즈니스 규칙까지 모두 표현하지는 못합니다.

보완 기술

- 도메인 검증 로직
- 버전별 계약 관리

### 7.3 Output Encoding

용도

- 응답이 UI에서 표시될 때 XSS나 컨텍스트 혼동을 줄입니다.

부족한 점

- API 응답만으로는 항상 충분하지 않고 소비자 측 처리도 중요합니다.

보완 기술

- CSP, Content-Type 명시, 템플릿 escaping

### 7.4 Payload Encryption

용도

- TLS 종료 이후에도 본문 내용을 숨겨야 할 때 추가 암호화를 적용합니다.

부족한 점

- 키 배포와 운영이 복잡합니다.
- 검색, 로깅, 디버깅, 라우팅이 어려워집니다.

보완 기술

- 필요한 필드만 암호화
- KMS/HSM 연동
- 메타데이터와 본문 암호문 분리

### 7.5 Field-level Encryption

용도

- 주민번호, 계좌번호, 카드번호 같은 일부 필드만 별도로 암호화합니다.

부족한 점

- 정렬, 검색, 부분 일치 같은 기능이 제한됩니다.

보완 기술

- 토큰화
- 별도 인덱스 전략
- KMS 기반 키 분리

## 8. 남용 방지와 경계 방어

### 8.1 Rate Limiting

용도

- 초당 요청 수를 제한해 무차별 대입, 스팸, 과부하를 줄입니다.

부족한 점

- 분산 공격이나 대규모 봇넷에는 단독으로 부족합니다.
- 정상 대량 트래픽과 악성 트래픽 구분이 어렵습니다.

보완 기술

- WAF, IP Reputation, CAPTCHA, 사용자/토큰 단위 제한
- 백오프와 circuit breaker

### 8.2 Quota / Throttling

용도

- 분당, 일별, 월별 사용량과 순간 속도를 제어합니다.

부족한 점

- 즉시 공격 차단보다는 자원 관리 성격이 강합니다.

보완 기술

- Rate Limit, anomaly detection, 계약 플랜 기반 정책

### 8.3 WAF

용도

- SQL Injection, XSS, 알려진 악성 패턴을 프록시 앞단에서 차단합니다.

부족한 점

- 비즈니스 로직 공격, 정상처럼 보이는 권한 남용에는 약합니다.
- 오탐과 미탐 관리가 필요합니다.

보완 기술

- 입력 검증, 인가 정책, 애플리케이션 로그 분석
- 룰 튜닝, rate limit, bot detection

### 8.4 IP Allowlist

용도

- 특정 네트워크나 파트너 IP에서만 접속하게 제한합니다.

부족한 점

- NAT, 프록시, 클라우드 환경에서 IP가 자주 변할 수 있습니다.
- 내부 또는 허용 IP 탈취 상황은 막지 못합니다.

보완 기술

- mTLS, HMAC, VPN, private link

### 8.5 Retry with Backoff / Circuit Breaker / Backpressure

용도

- 장애 시 무한 재시도와 폭주를 줄여 보안 장치 우회와 자원 고갈을 방지합니다.

부족한 점

- 인증이나 인가를 제공하는 기법은 아닙니다.

보완 기술

- Rate limit, queue, load shedding, idempotency key

## 9. 비밀값과 키 관리

### 9.1 Secret Rotation

용도

- Secret, API Key, 서명 키를 주기적으로 교체해 유출 피해를 줄입니다.

부족한 점

- 양쪽 시스템이 동시에 전환하지 못하면 장애가 납니다.

보완 기술

- 이중 키 기간 운영
- 키 버전 명시
- 자동 배포와 롤백 절차

### 9.2 KMS / HSM / Secret Manager

용도

- 키와 비밀값을 코드나 설정 파일 밖에서 안전하게 저장하고 접근을 통제합니다.

부족한 점

- 비용과 운영 복잡성이 증가합니다.
- 잘못된 권한 설정 시 오히려 중앙집중형 위험점이 될 수 있습니다.

보완 기술

- 최소 권한, 감사 로그, 자동 로테이션
- 애플리케이션별 분리 저장

### 9.3 Dependency / Patch Management

용도

- 프레임워크, 라이브러리, 런타임, 프록시의 알려진 취약점을 줄입니다.

부족한 점

- 패치만으로 설계 취약점이나 운영 실수를 해결할 수는 없습니다.
- 업그레이드 호환성 문제가 생길 수 있습니다.

보완 기술

- SBOM, 취약점 스캔, 점진 배포
- 지원 종료 버전 사용 금지
- 보안 공지 모니터링과 정기 점검

## 10. 추적과 탐지

### 10.1 Audit Logging

용도

- 누가, 언제, 어떤 토큰이나 키로, 어떤 API를 호출했는지 남깁니다.

부족한 점

- 사후 대응 중심이라 공격 자체를 실시간 차단하지는 못합니다.
- 민감정보를 과도하게 남기면 2차 유출 위험이 생깁니다.

보완 기술

- 민감정보 마스킹
- request id, user id, client id, 결과 코드 기록
- SIEM 연동

### 10.2 Security Monitoring / SIEM

용도

- 비정상 호출 패턴, 토큰 오남용, 반복 실패, 지리적 이상을 탐지합니다.

부족한 점

- 로그 품질과 룰 품질이 낮으면 실효성이 떨어집니다.

보완 기술

- 구조화 로그, 경보 튜닝, 자동 차단 연계

## 11. 무엇을 함께 써야 하나

### 11.1 외부 공개 REST API

권장 조합은 아래와 같습니다.

- HTTPS(TLS)
- OAuth 2.0 또는 API Key
- Scope + RBAC/ABAC
- Rate Limiting + WAF
- Audit Logging + Monitoring
- 입력 검증 + Schema Validation

### 11.2 서버 간 파트너 API

권장 조합은 아래와 같습니다.

- HTTPS(TLS)
- mTLS 또는 HMAC 또는 디지털 서명
- nonce + timestamp + request id
- idempotency key
- IP Allowlist
- Secret Rotation 또는 인증서 로테이션

### 11.3 웹훅 수신 API

권장 조합은 아래와 같습니다.

- HTTPS(TLS)
- HMAC 서명 검증
- Raw Body 기준 검증
- delivery id 중복 차단
- timestamp 또는 nonce 검증
- 비동기 처리, 재시도 정책, audit log

### 11.4 브라우저 세션 기반 API

권장 조합은 아래와 같습니다.

- HTTPS(TLS)
- Session Cookie + HttpOnly + Secure + SameSite
- CSRF Token
- CORS 최소 허용
- CSP 등 보안 헤더

## 12. 정리

HTTP/S API 보안은 한 가지 기술로 끝나지 않습니다.

- TLS는 전송 구간 보호입니다.
- 인증은 호출 주체를 확인합니다.
- 인가는 무엇을 할 수 있는지 제한합니다.
- HMAC, 서명, nonce, timestamp는 요청 자체를 보호합니다.
- rate limit, WAF, monitoring은 남용과 공격을 줄입니다.
- secret rotation, KMS, audit log는 운영 보안을 지탱합니다.

실무에서는 아래 원칙이 중요합니다.

1. 전송 보호, 인증, 인가, 재전송 방지, 남용 방지를 분리해서 설계할 것
2. 브라우저 API와 서버 간 API를 같은 방식으로 보지 말 것
3. 이 기술이 막는 것과 막지 못하는 것을 항상 같이 볼 것
4. 단일 기법보다 계층형 조합을 기본값으로 삼을 것
