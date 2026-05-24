# HMAC 검증

HMAC 검증은 웹훅 보안의 기본입니다. 핵심은 아래 2가지를 확인하는 것입니다.

1. 인증: 이 요청이 정말 정상 발신자(예: GitHub/Bitbucket)에서 왔는가
2. 무결성: 전송 중 본문이 한 글자라도 바뀌지 않았는가

현재 코드에서도 같은 흐름을 사용합니다. 서명 헤더를 읽고, HMAC 계산 및 비교로 검증합니다.

## 무엇을 검증하나

검증에 필요한 입력값은 아래 3가지입니다.

1. 공유 비밀값 Secret
2. 원본 요청 본문 Raw Body
3. 요청 헤더의 서명값 Signature Header(예: sha256=...)

판정 기준은 간단합니다.

- 송신자가 보낸 서명값 signature와
- 수신자가 같은 Secret, 같은 Raw Body로 재계산한 expected가
- 상수 시간 비교에서 일치하면 통과, 아니면 거부합니다.

## 검증 흐름

검증은 "송신자 서명 생성"과 "수신자 재계산 비교"의 2단계로 진행됩니다.

1. 송신자(발신 시스템)
- Raw Body와 Secret으로 아래 값을 계산합니다.

$$
signature = HMAC_{SHA256}(secret, raw\_body)
$$

- 계산한 signature를 요청 헤더(예: sha256=<hex>)에 담아 전송합니다.

2. 수신자(검증 서버)
- 수신한 Raw Body를 그대로 사용해 아래 값을 다시 계산합니다.

$$
expected = HMAC_{SHA256}(secret, raw\_body)
$$

- 헤더의 signature와 expected를 상수 시간 비교로 대조합니다.
- 일치하면 통과, 다르면 거부합니다.

## HMAC과 SHA-256은 무엇인가

1. SHA-256
- SHA-256은 해시 함수입니다.
- 임의 길이 입력을 받아 고정 길이 256비트(32바이트) 해시값을 만듭니다.
- 같은 입력이면 항상 같은 출력이 나오며, 입력이 조금만 바뀌어도 결과가 크게 달라집니다.
- 다만 SHA-256만으로는 "누가 보냈는지"를 증명할 수 없습니다.

2. HMAC
- HMAC은 Keyed-Hash Message Authentication Code의 약자입니다.
- 해시 함수에 비밀키(Secret)를 결합해, 메시지 인증과 무결성 검증을 수행합니다.
- 즉, "메시지 + 비밀키"를 함께 사용해 서명을 만듭니다.
- 비밀키를 모르면 올바른 HMAC 값을 만들 수 없습니다.

## 둘의 관계(HMAC-SHA256)

1. 관계 요약
- HMAC은 구성 방식(알고리즘 틀)이고, SHA-256은 그 안에서 쓰는 해시 함수입니다.
- 따라서 HMAC-SHA256은 HMAC을 SHA-256으로 구현한 것을 의미합니다.

2. 왜 sha256=... 형태를 쓰나
- 헤더의 sha256= 접두사는 "이 서명이 SHA-256 기반 HMAC으로 계산되었다"는 알고리즘 힌트입니다.
- 서버는 같은 알고리즘(HMAC-SHA256), 같은 Secret, 같은 Raw Body로 다시 계산해 비교합니다.

3. 자주 헷갈리는 점
- SHA256(body)는 체크섬에 가깝고 인증 기능이 없습니다.
- HMAC-SHA256(secret, body)는 인증 + 무결성 검증을 제공합니다.
- SHA256(secret + body) 같은 임의 조합보다 표준 HMAC 구성을 사용해야 합니다.

4. HMAC-SHA256(secret, body)와 SHA256(secret + body)의 차이
- 공통점: 둘 다 Secret과 body를 입력으로 해시 값을 만듭니다.
- 차이점: HMAC은 키를 안전하게 다루도록 설계된 표준 구조(내부/외부 해시 2단계)를 사용합니다.
- 내부/외부 해시 2단계란 아래 흐름을 뜻합니다.
	1) inner = SHA256((K' xor ipad) || body)
	2) mac = SHA256((K' xor opad) || inner)
	여기서 K'는 블록 크기에 맞게 정규화한 키, ipad/opad는 고정 패딩 상수입니다.
- 핵심은 "키를 앞에 한 번 붙이는 방식"이 아니라, 키를 두 번 다른 형태로 섞어 해시한다는 점입니다.
- 이 구조 덕분에 단순 SHA256(secret + body) 대비 알려진 해시 조합 취약성에 더 강하고, 표준(RFC 2104)으로 오래 검증되었습니다.
- 실무에서는 이 수식을 직접 구현하기보다 언어 표준 라이브러리의 HMAC 구현을 그대로 사용하는 것이 안전합니다.
- SHA256(secret + body)는 단순 문자열 결합 후 해시하는 방식이라, 표준 인증 방식으로 권장되지 않습니다.
- HMAC은 다양한 공격 모델을 고려해 검증된 방식이며, 라이브러리로 일관되게 구현할 수 있습니다.

```mermaid
flowchart LR
	subgraph S[송신자]
		K1[Secret K]
		B1[Raw Body]
		I1[K' xor ipad]
		O1[K' xor opad]
		H1[inner = SHA256 I1 || Body]
		H2[mac = SHA256 O1 || inner]
		SIG[Signature Header: sha256=mac]
		K1 --> I1
		K1 --> O1
		I1 --> H1
		B1 --> H1
		H1 --> H2
		O1 --> H2
		H2 --> SIG
	end

	SIG --> REQ[HTTP Request]
	B1 --> REQ

	subgraph R[수신자]
		K2[Secret K]
		B2[Received Raw Body]
		RSIG[Received Signature]
		I2[K' xor ipad]
		O2[K' xor opad]
		RH1[inner' = SHA256 I2 || Body]
		EXP[expected = SHA256 O2 || inner']
		CMP{constant-time compare\nReceived Signature vs expected}
		OK[일치: 통과]
		NO[불일치: 거부]
		K2 --> I2
		K2 --> O2
		I2 --> RH1
		B2 --> RH1
		RH1 --> EXP
		O2 --> EXP
		RSIG --> CMP
		EXP --> CMP
		CMP -->|true| OK
		CMP -->|false| NO
	end

	REQ --> B2
	REQ --> RSIG
```

5. HMAC이 무엇인가(한 줄 정의)
- HMAC은 "비밀키를 이용해 메시지의 인증과 무결성을 검증"하기 위한 표준 메시지 인증 코드(MAC) 방식입니다.
- 즉, 단순 해시가 아니라 "같은 Secret을 아는 양측만 같은 결과를 만들 수 있게" 설계된 알고리즘 절차입니다.

## HMAC 검증 절차(송신자/수신자)

HMAC은 단순 해시 값 자체가 아니라, 송신자와 수신자가 같은 Secret을 공유하고 같은 계산 규칙으로 메시지를 검증하는 절차입니다.

1. 사전 준비(양측 공통)
- 송신자와 수신자는 같은 Secret을 안전하게 공유합니다.
- 사용할 알고리즘(예: HMAC-SHA256)과 서명 헤더 이름(예: X-Hub-Signature-256)을 합의합니다.

2. 송신자가 하는 일(서명 생성)
- 전송할 Raw Body 바이트를 확정합니다.
- Secret과 Raw Body로 HMAC-SHA256 값을 계산합니다.
- 계산한 값을 보통 sha256=<hex> 형태로 서명 헤더에 넣습니다.
- Raw Body와 서명 헤더를 함께 전송합니다.

3. 수신자가 하는 일(서명 검증)
- 요청에서 Raw Body를 그대로 읽습니다.
- 헤더에서 서명값을 추출하고 형식을 검사합니다.
- 서버에 저장된 같은 Secret으로 expected = HMAC-SHA256(secret, raw_body)를 다시 계산합니다.
- 헤더 서명과 expected를 상수 시간 비교로 대조합니다.
- 일치하면 통과, 불일치하면 거부합니다.

4. 왜 Raw Body 그대로가 중요한가
- JSON 파싱 후 재직렬화, 공백/개행 변화, 인코딩 변화가 생기면 바이트 단위가 달라집니다.
- HMAC은 바이트가 하나만 달라도 결과가 달라지므로, 반드시 원문 바이트 그대로 계산해야 합니다.

5. 절차 요약
- 송신자: Secret으로 body 서명 생성 후 header에 담아 전송
- 수신자: 같은 Secret으로 재계산 후 상수 시간 비교
- 이 과정을 통해 인증(발신자 진위)과 무결성(변조 여부)을 함께 확인합니다.

## 자주 묻는 질문(FAQ)

1. HMAC은 수신자가 비밀키와 body를 다시 hash해서 request 메시지를 검증하는 것인가?
- 네, 개념적으로 맞습니다.
- 수신자는 Raw Body 그대로와 동일한 Secret으로 HMAC-SHA256 값을 다시 계산합니다.
- 그리고 요청 헤더의 서명값과 상수 시간 비교(hmac.Equal 등)로 대조합니다.
- 값이 같으면 인증(Secret을 아는 발신자 가능성)과 무결성(전송 중 변조 없음)을 확인한 것으로 봅니다.

2. 주의할 점
- 단순 hash(SHA256(body)) 검증과 HMAC 검증은 다릅니다.
- HMAC은 Secret을 포함하므로 인증 성질이 생깁니다.
- 다만 HMAC만으로 재전송(Replay) 공격은 막지 못하므로 delivery id, timestamp, nonce 같은 추가 검증이 필요합니다.

## HMAC 검증 목적

1. 인증(Authentication): Secret을 아는 쪽만 올바른 서명을 만들 수 있으므로 발신자 진위를 확인합니다.
2. 무결성(Integrity): 바디가 변조되면 HMAC이 달라져 즉시 탐지할 수 있습니다.
3. 신뢰 경계 확립: 내부 후속 로직(큐 적재, 빌드 트리거 등)을 안전한 입력에만 수행합니다.

## 방어 가능한 공격

1. 위조 웹훅 전송
- 공격자가 임의 JSON을 보내도 Secret이 없으면 유효 서명을 만들 수 없습니다.

2. 전송 중 페이로드 변조(MITM 변조)
- 프록시/중간자/네트워크 경로에서 body가 바뀌면 서명 불일치로 차단됩니다.

3. 단순 페이로드 오염/인코딩 변형
- 원문 바이트가 달라지면 실패하므로 데이터 손상을 탐지할 수 있습니다.

4. 타이밍 기반 비교 공격(부분)
- 상수 시간 비교를 쓰면 서명 문자열 비교 시 정보 누설 가능성을 줄일 수 있습니다.
- 현재 코드의 hmac.Equal 사용은 이 점에서 올바른 방향입니다.

## 방어하지 못하는 것

1. 재전송 공격(Replay)
- 캡처한 정상 요청을 그대로 다시 보내면 서명은 여전히 유효할 수 있습니다.
- 대응: delivery id 중복 차단, timestamp 윈도우 검사, nonce 저장

2. Secret 유출
- Secret이 탈취되면 공격자도 정상 서명을 만들 수 있습니다.
- 대응: Secret 로테이션, 안전 저장, 최소 권한

3. 대용량/고빈도 DoS
- 서명 검증 자체가 트래픽 폭주를 막지는 못합니다.
- 대응: rate limit, IP allowlist, 큐 백프레셔, WAF

4. 발신자 시스템 자체가 침해된 경우
- 공격자가 발신 시스템 권한을 가진 경우는 별도 신뢰 문제입니다.

## 실무 체크리스트

1. 반드시 Raw Body 그대로 HMAC 계산
2. JSON 파싱/정규화 전에 검증
3. 헤더 포맷 검사(예: sha256= 접두사, 알고리즘 파싱)
4. 상수 시간 비교 사용
5. 실패 사유는 로그에 남기되 Secret/원문 민감정보는 마스킹
6. 재전송 방지 로직 추가(특히 production)
7. Secret 주기적 교체