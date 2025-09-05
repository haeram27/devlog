# CEF (Common Event Format)

**CEF(Common Event Format)**은 ArcSight(옛 HP, Micro Focus, 현재 OpenText 소유) SIEM 등에서 사용하는 표준화된 이벤트(로그) 형식입니다. 보안 이벤트, 시스템 이벤트 등을 통일된 방식으로 표현하여 SIEM과 같은 솔루션이 로그를 자동으로 파싱하고, 분석 및 모니터링을 쉽게 할 수 있도록 돕습니다.

CEF 메시지는 흔히 **Syslog 메시지** 형태로 전송되는데, RFC3164/RFC5424 등 Syslog 표준 헤더 뒤에 `CEF:0|...` 구조로 이어집니다. 즉, Syslog 프로토콜을 이용하여 네트워크로 전달할 때, “본문”에 CEF 형식의 문자열이 들어가게 됩니다.

---

## 1. CEF 형식 개요

**CEF 헤더 구조**는 다음과 같습니다.

```text
CEF:Version|Device Vendor|Device Product|Device Version|Signature ID|Name|Severity|Extension
```

- `CEF:Version`  
  - 보통 `CEF:0` 형태
- `Device Vendor`  
  - 벤더 이름(예: `Microsoft`, `Apache`, `MyCompany`)
- `Device Product`  
  - 제품/애플리케이션 이름(예: `Windows`, `Tomcat`, `CustomWebApp`)
- `Device Version`  
  - 제품 버전(예: `10.0`, `9.0`)
- `Signature ID`  
  - 이벤트 ID, 이벤트 유형 식별자(예: `1001`, `AUTH_SUCCESS`)
- `Name`  
  - 이벤트 이름/설명(예: `Login success`)
- `Severity`  
  - 이벤트 심각도(숫자 혹은 `Low`, `Medium`, `High`, `Critical` 등으로 표현)

이후 **Extension** 부분에 Key=Value 쌍을 빈칸으로 구분하여 적어줍니다.  
예: `src=10.0.0.1 suser=admin msg="User admin logged in successfully"`

### 전체 예시 (순수 CEF 포맷)

```text
CEF:0|MyCompany|MyProduct|1.2.3|1001|User Login|Low|src=10.0.0.1 suser=admin msg="Login success"
```

---

## 2. Syslog + CEF 결합 예시

Syslog 표준 헤더는 보통 `<PRI>`(우선순위) + `TIMESTAMP` + `HOSTNAME` + `TAG` 등의 정보를 갖습니다(RFC3164 기준).  
예: `<13>Apr 11 14:34:56 MyHost MyApp: ...`

### RFC3164 스타일

아래는 “우선순위가 13, 날짜는 Apr 11 14:34:56, 호스트 명은 MyHost, 프로세스 명은 MyApp, 그리고 본문이 CEF 형식”인 예시입니다.

```text
<13>Apr 11 14:34:56 MyHost MyApp: CEF:0|MyCompany|MyProduct|1.0|100|Authentication succeeded|Low|src=10.0.0.1 suser=admin msg="login success"
```

- `<13>`: Syslog 우선순위(PRI) 필드 예시  
- `Apr 11 14:34:56`: 타임스탬프  
- `MyHost`: 호스트 이름  
- `MyApp:` 프로세스나 태그 등  
- 이후 **본문**에 `CEF:0|...` 형식의 문자열이 들어감

### RFC5424 스타일

RFC5424는 좀 더 구조화된 포맷을 제공하지만, 핵심 개념은 유사합니다. 다음은 예시입니다.

```text
<14>1 2025-04-11T14:34:56.789Z MyHost MyApp 12345 ID47 [exampleSDID@32473 iut="3" eventSource="Application" eventID="1011"] CEF:0|MyCompany|MyProduct|1.0|100|Authentication succeeded|Low|src=10.0.0.1 suser=admin msg="login success"
```

---

## 3. CEF에서 자주 쓰이는 Extension 필드 예시

| 필드    | 의미                                        |
|--------|---------------------------------------------|
| `src`   | 소스 IP (source IP)                         |
| `dst`   | 목적지 IP (destination IP)                   |
| `suser` | 소스 사용자명 (source user)                 |
| `duser` | 목적지 사용자명 (destination user)           |
| `srcPort` | 소스 포트                                |
| `dstPort` | 목적지 포트                              |
| `msg`   | 추가 메시지(설명)                           |
| `deviceFacility` | 이벤트가 발생한 분야(예: auth)      |
| `requestURL` | 요청된 URL                             |
| `requestMethod` | 요청 메서드(GET, POST 등)           |
| `cat`   | 이벤트 카테고리(Category)                   |

이외에도 CEF에서 정의한 확장 키들은 매우 많습니다. 또한 사용자 정의 Key=Value도 허용됩니다.

---

## 4. 실무에서의 활용

- **로그 수집 에이전트 또는 애플리케이션**에서 CEF 포맷 문자열을 생성한 뒤, Syslog 전송(UDP/TCP/TLS)으로 SIEM(ArcSight, Splunk 등)에 전달합니다.
- **로그 프레임워크(Log4j, Logback 등)**를 활용하는 경우, 패턴 레이아웃 등을 설정해 CEF 형태의 문자열을 만들거나, 별도의 CEF 전용 Appender를 사용하기도 합니다.
- **OS 수준(예: Linux rsyslog, syslog-ng)**에서 템플릿을 활용해 기존 로그를 CEF로 변환하여 SIEM으로 전송하기도 합니다.

---

## 5. 간단한 CEF Syslog 메시지 예시 코드 (Log4j)

아래는 Log4j 2에서 간단히 “CEF 포맷” 문자열을 Syslog로 보내는 예시(아이디어)를 보여줍니다.

```xml
<Configuration status="WARN">
  <Appenders>
    <Syslog name="SyslogAppender" format="RFC3164" host="logs.example.com" port="514" protocol="UDP">
      <PatternLayout pattern="%d{ISO8601} CEF:0|MyCompany|MyService|1.0|1001|Login success|Low|src=%X{clientIp} suser=%X{username} msg=%m%n" />
    </Syslog>
  </Appenders>

  <Loggers>
    <Root level="info">
      <AppenderRef ref="SyslogAppender"/>
    </Root>
  </Loggers>
</Configuration>
```

- `%X{clientIp}`, `%X{username}`처럼 Mapped Diagnostic Context(MDC)에 담긴 정보를 활용해 CEF 확장 필드에 매핑.
- 메시지 `%m`을 `msg=`로 매핑.

이렇게 하면 로그를 남길 때 `logger.info("정상 로그인");` 등으로 호출하면, Syslog 서버로 전송되는 메시지 본문이 CEF 형식으로 만들어집니다.

---

## 요약

- **CEF(Common Event Format)**은 보안/이벤트 로그를 표준화하여 SIEM에서 효과적으로 처리할 수 있게 해주는 포맷입니다.
- **Syslog**는 로그 전송 프로토콜(및 표준 메시지 포맷)이며, 실제로 SIEM 등에 데이터를 보낼 때 보통 CEF를 “Syslog 메시지의 본문”에 넣습니다.
- 기본 형태는 `CEF:0|벤더|제품|버전|시그니처ID|이벤트명|심각도|key=value key=value ...` 이고, syslog 헤더 뒤에 붙여 전송합니다.
- 실무에서는 rsyslog, syslog-ng, Log4j/Logback, 또는 타사 에이전트를 통해 로그를 전송하고, SIEM(ArcSight 등)에서 이를 파싱해 모니터링/분석을 수행합니다.

Syslog(Syslog 표준 메시지 포맷 + 프로토콜)과 CEF(Common Event Format)는 각각 용도와 구조가 다르지만, 실무에서는 **Syslog 프로토콜**을 통해 **CEF 형식** 메시지를 전송하는 것이 일반적입니다. 다음은 두 가지를 비교한 요약입니다.

---

## 1. 개념 및 역할

- **Syslog**  
  - **표준 로그 전송 프로토콜**이자 **메시지 형식**입니다.  
  - 메시지에 우선순위(PRI), 타임스탬프, 호스트명, 프로세스/태그, 그리고 실제 로그 내용이 포함됩니다.  
  - RFC3164(오래된 표준)와 RFC5424(새로운 표준)로 나뉩니다.  
  - 일반 시스템/애플리케이션 로그를 중앙 서버로 전송, 저장, 모니터링하기 위한 가장 전통적·보편적인 방식입니다.

- **CEF (Common Event Format)**  
  - **보안 이벤트/침입탐지/감사로그**를 표준화하기 위한 포맷(ArcSight SIEM에서 시작)입니다.  
  - 보안 로그를 일정한 형태(`CEF:0|Vendor|Product|Version|SignatureID|Name|Severity|Extension`)로 정의해, SIEM이 자동으로 파싱·분석하기 쉽게 해줍니다.  
  - Syslog, 파일, 기타 프로토콜을 통해 전송 가능하지만, 주로 Syslog 프로토콜로 전송해 SIEM에 적재됩니다.

---

## 2. 구조 비교

| 항목              | Syslog (RFC3164/5424)                                                               | CEF                                                                                                      |
|------------------|-------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| **포맷/구조**      | `<PRI> TIMESTAMP HOST TAG: MESSAGE` 형태 (RFC3164 기준)<br>RFC5424는 구조화된 헤더(Version, AppName 등) 추가 | `CEF:Version|Device Vendor|Device Product|Device Version|Signature ID|Name|Severity|Extension`<br>Extension: Key=Value 쌍을 공백으로 구분 |
| **용도**          | 일반 시스템·애플리케이션 로그 전송 (OS, 네트워크 장비, 소프트웨어 등에서 폭넓게 사용) | 보안·침입탐지·이벤트 분석을 위한 SIEM 솔루션 연동용 포맷 (ArcSight, Splunk 등) |
| **프로토콜**      | 일반적으로 UDP/514, TCP/514, 혹은 TLS를 이용해 전송 | 자체적으로 프로토콜을 규정하지 않고, 보통 Syslog(또는 파일)를 통해 전달 |
| **메시지 예시**    | RFC3164: `<13>Apr 11 14:34:56 MyHost MyApp: Service started` | `CEF:0|MyCompany|MyProduct|1.0|1001|Login success|Low|src=10.0.0.1 suser=admin msg="User login"`          |
| **확장성**        | Syslog 헤더는 고정 필드가 많지 않아 단순<br>RFC5424는 Structured Data 필드를 통해 확장 가능 | Extension 필드에 원하는 Key=Value 쌍을 추가하여 사용자 정의 확장 가능 |
| **분석 및 파싱**   | Syslog 파서는 일반 문자열 기반이지만, 구조가 단순하여 기본 필드 분리 정도를 제공 | CEF는 SIEM에서 표준화된 필드(`src`, `dst`, `suser`, `duser`, `msg` 등)를 인식하므로 보안 이벤트 분석에 특화됨 |

---

## 3. 상호 관계

- **보통 “Syslog”로 전송하면서, 메시지 본문이 “CEF”** 인 형태가 많습니다.  
  - 즉, Syslog 헤더(+우선순위, 타임스탬프 등) 뒤에 `CEF:0|...` 구조가 따라오는 방식입니다.  
  - SIEM은 Syslog 포맷을 통해 전달된 메시지에서 본문을 파싱해 “CEF” 이벤트로 인식하게 됩니다.

예시 (RFC3164 스타일):

```text
<13>Apr 11 14:34:56 MyHost MySecurityApp: CEF:0|MyCompany|MyProduct|1.2.3|1001|Access Granted|Low|src=10.0.0.1 suser=admin msg="User logged in"
```

---

## 4. 사용 시 고려사항

1. **일반 로그 vs. 보안 이벤트**  
   - Syslog는 모든 종류의 로그(시스템 이벤트, 애플리케이션 로그 등)에 쓸 수 있음.  
   - CEF는 보안 이벤트나 중요한 감사로그를 구조화해 SIEM과 연동하는 용도에 특화.

2. **구현 난이도**  
   - Syslog 전송은 rsyslog, syslog-ng, Log4j/Logback(Syslog Appender) 등으로 쉽게 구현 가능.  
   - CEF는 포맷을 직접 만들거나, CEF 전용 라이브러리/플러그인을 사용해야 하는 경우도 있음.

3. **SIEM 연동**  
   - SIEM(ArcSight, Splunk 등)은 CEF를 자동 인식하여 필드별로 파싱·분석·시각화 지원.  
   - 일반 Syslog도 수집·분석할 수 있지만, 특정 이벤트 필드를 구체적으로 사용하려면 CEF/LEEF 등 표준 포맷을 권장.

4. **성능 및 확장성**  
   - Syslog는 UDP 기반일 경우 전송 부하가 적지만 신뢰성이 떨어질 수 있음.  
   - CEF를 TCP 또는 TLS로 전송할 경우, 보안성과 신뢰성이 개선되나 시스템 부하가 증가할 수 있음.

---

## 요약

- **Syslog**: 로그를 전송·저장·분석하기 위한 범용 표준 프로토콜 및 메시지 포맷  
- **CEF**: 주로 **보안 이벤트**를 구조적으로 표현해 **SIEM**에서 손쉽게 파싱·분석하도록 설계된 포맷  
- 두 기술은 배타적 개념이 아니라, **Syslog 포맷(헤더) + CEF 본문** 형태로 결합해 사용하는 경우가 많습니다.  
- 일반 로그는 Syslog만으로 충분하지만, **보안 이벤트/침입 탐지 로그** 등을 SIEM에 보내야 한다면 **CEF** 포맷이 권장됩니다.
