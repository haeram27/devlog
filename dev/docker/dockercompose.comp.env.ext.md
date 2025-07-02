# Docker Compose에서 사용 가능한 compose 환경변수 확장 표현들

Docker Compose (`docker-compose` 및 `Compose V2`)는 `.env` 파일 또는 셸 환경에 정의된 변수들을 docker-compose.yml 파일 내에서 `${...}` 형태로 참조하며, **일부 Bash 스타일의 확장 문법도 지원**합니다.

다음은 Compose에서 **지원되는 확장 표현**입니다:

## 요약표

| 표현                | 의미                       |
| ----------------- | ------------------------ |
| `${VAR}`          | VAR 값 사용                 |
| `${VAR:-default}` | unset 또는 빈 문자열이면 default |
| `${VAR-default}`  | unset이면 default          |
| `${VAR:?err}`     | unset이면 에러 출력            |
| `${VAR:+alt}`     | 설정되어 있으면 alt 사용          |

---

### 🔹 1. `${VAR}`

* `VAR`의 값을 그대로 사용
* 없으면 에러 발생 (또는 빈 문자열로 해석)

---

### 🔹 2. `${VAR:-default}`

* `VAR`가 선언되지 않았거나 빈 문자열이면 `"default"` 사용
* **Compose에서 매우 자주 쓰임**

---

### 🔹 3. `${VAR-default}`

* `VAR`가 **선언되지 않았을 때만** `"default"` 사용 (빈 문자열은 그대로 사용)

---

### 🔹 4. `${VAR:?error}`

* `VAR`가 선언되지 않으면 **오류 메시지를 출력하고 종료**
* 예:

  ```yaml
  image: ${REQUIRED_VAR:?You must set REQUIRED_VAR}
  ```

---

### 🔹 5. `${VAR:+alt}`

* `VAR`가 설정되어 있으면 `"alt"` 사용
* 설정되지 않았거나 비어 있으면 아무 것도 출력하지 않음

---

## 사용 예제 (docker-compose.yaml)

```yaml
services:
  app:
    image: ${APP_IMAGE:-myapp:latest}
    environment:
      - ENV=${APP_ENV:-development}
      - PASSWORD=${APP_PASSWORD:?APP_PASSWORD must be set!}
    volumes:
      - ${DATA_DIR:-./data}:/app/data
```

---

## 주의 사항

1. **중첩된 변수 참조**는 지원되지 않음

   ```yaml
   # ${${SOME_VAR}} — 지원되지 않음
   ```

2. **산술 연산, 패턴 치환, 배열 등 고급 Bash 문법은 지원되지 않음**

3. `.env` 파일의 변수는 **기본 환경변수보다 우선순위가 낮음**
   (쉘에서 `export VAR=...` 하고 실행하면 우선 적용됨)
