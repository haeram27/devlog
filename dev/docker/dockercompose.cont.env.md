# Docker Compose의 Container 환경 변수

이 문서는 docker compose 실행 시 Container 내부 환경 변수를 명시하는 방법을 설명한다.

## compose 환경 변수와 container 환경 변수

docker compose에는 언급되는 환경 변수에는 두 가지 타입이 있다.

* ***compose 환경 변수***
  * docker-compose.yaml 파일내에서 참조 가능한 환경이며 변수 yaml 파일을 구성하는데 사용된다.
  * ***compose 환경 변수는 컨테이너 내부의 환경변수로 전파되지 않는다.***
  * 지정 방법
    * docker compose 명령에 전달 되는 쉘 환경 변수 (`export` 또는 compose 명령 라인에 함께 명시)
    * `docker-compose.yaml`과 같은 디렉토리에 존재하는 `.env` 파일 (`--env_file` 옵션)
  * .env 파일은 docker compose 명령 실행시 단 하나의 파일만 참조 가능하다.
  * 자동 참조 되는 .env 파일보다 `--env_file` 옵션으로 지정한 파일의 우선 순위가 높다.
* ***container 환경 변수***
  * docker compose에 의해 실행되는 컨테이너 내부에 생성되는 환경 변수
  * 지정 방법
    * docker-compose.yaml 파일 내 `env_file` 또는 `environment` 필드에 명시
    * 중복키의 경우 `environment` 필드가 `env_file` 필드 보다 우선순위가 높다.
    * `env_file` 필드에는 여러 파일이 명시 가능하다.

## 우선 순위 (중복 키에 대한 최종 값)

1. `environment` 필드
1. `env_file` 필드
   * 다중 파일 정의시 위에서 아래로 override 방식으로 최종 값 적용

---

## env_file vs environment 차이

| 항목 | `env_file` | `environment` |
| --- | --- | --- |
| 입력 방식 | 외부 파일 지정 | YAML 안에 직접 key-value 작성 |
| 가독성 | 외부로 분리되어 관리 쉬움 | 한눈에 보이지만 길어지면 복잡 |
| COMPOSE 환경변수 참조 | 지원하지 않음 | `${VAR}` 형식으로 확장 참조 지원 |
| 파일 포맷 요구 | `KEY=VALUE` | YAML 리스트 또는 딕셔너리 |

예:

```yaml
environment:
  - VAR1=value1
  - VAR2=${HOSTNAME}
```

---

## env_file 필드

`docker compose` (또는 `docker-compose`) 명령에서 `env_file` 옵션에 지정하는 파일은 **간단한 `KEY=VALUE` 형식의 환경변수 정의 파일**입니다. 이는 `.env` 파일과 거의 동일한 형식을 따릅니다.

### 1. 기본 형식

`env_file`로 지정하는 파일은 아래처럼 구성됩니다:

```ini
# .env.app
APP_ENV=production
DB_HOST=db.example.com
DB_PORT=5432
DEBUG=false
```

windows에서 사용되던 ini 설정파일의 형식과 유사

* 라인 별로 `KEY=VALUE` 형식의 아이템을 지정 가능
* `VALUE`는 반드시 string literal이어야 하며 `${VAR}`과 같은 확장 참조 표현 불가능

#### 형식 규칙

| 항목 | 설명 |
| --- | --- |
| 형식    | `KEY=VALUE` (양쪽에 공백 없어야 함)                         |
| 주석    | `#` 으로 시작하는 라인은 무시됨                                |
| 따옴표   | `VALUE`에 공백, 특수문자가 있으면 `"..."` 또는 `'...'`로 감쌀 수 있음 |
| 변수 확장 | 지원하지 않음 (다른 변수 참조 불가)                              |

예:

```bash
NAME="App Server"
MESSAGE='Hello, world!'
```

---

### 2. env_file 사용 예

```yaml
services:
  app:
    image: myapp
    env_file:
      - .env.app
```

* `.env.app`에 정의된 모든 환경변수들이 컨테이너 내부에 주입됩니다.
* 이는 컨테이너의 `ENV`로 전달되며, `environment:`와 유사하게 작동합니다.

---

### 3. env_file에 사용 불가한 예시

```env
APP_ENV=prod
PORT=${APP_PORT}   # 지원 안 됨 (변수 확장 안 됨)
```

* `${APP_PORT}`와 같은  **변수 참조는 작동하지 않습니다**.
* 값은 반드시 리터럴 문자열이어야 합니다.

---

#### env_file 요약

| 항목     | 설명                    |
| ------ | --------------------- |
| 지원 형식  | `KEY=VALUE`, 한 줄당 하나  |
| 주석 처리  | `#` 사용 가능             |
| 변수 확장  | 미지원 (`${VAR}` 안 됨)    |
| 따옴표 사용 | 가능 (`"..."`, `'...'`) |
| 사용 목적  | 컨테이너 내부에 환경변수 주입      |

주의:
`env_file` 필드 파일은 container 환경 변수 지정 용이이고  
`.env` 파일은 compose 환경 변수 지정 용도 이다.
