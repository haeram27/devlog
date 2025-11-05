# 환경 설정 파일(.env) 병합 및 배

## 1. 환경별 설정 분리 (`docker-compose.override.yaml`)

### 기본 Compose 구조

* `docker-compose.yaml`: 공통 설정
* `docker-compose.override.yaml`: 개발/테스트용 설정 자동 병합

*예제:*

```yaml
# docker-compose.override.yaml
services:
  web:
    ports:
      - "8080:80"
    environment:
      - DEBUG=true
```

* `docker-compose up` 시 자동으로 .override.yaml 파일이 자동으로 overwrite 적용됨
* 프로덕션에서는 override 파일 없이 실행해야 함 (사용할 환경 변수 파일만 -f로 지정)

### 명시적으로 파일 지정

```bash
docker-compose -f docker-compose.yaml -f docker-compose.prod.yaml up
```

* -f 옵션 다중 정의시 **뒤에 지정된 .yaml 파일이 override 방식으로 적용됨**

---

## 요약 (추가 포인트)

| 항목 | 설명 |
| --- | --- |
| 다중 env 병합 | \`cat .env.base .env.dev \| xargs export\` |
| 파일 override | `-f` 옵션 또는 `override.yaml` 사용 |
| 환경 확인     | `docker-compose config` 로 확인 |
| `.env` 보안   | Git에 커밋하지 않도록 `.gitignore` 등록 |
| 자동화 도구   | `make`, `direnv`, `dotenv-cli`, `docker compose --env-file` |

좋은 질문입니다.
`docker-compose.override.yaml` 파일은 기본적으로 **자동으로 적용**되기 때문에, 이를 **제외하고 프로덕션(기본 구성만)으로 실행**하려면 **명시적으로 사용할 Compose 파일만 지정**해야 합니다.

---

## 방법: `-f` 옵션으로 기본 파일만 명시

```bash
docker-compose -f docker-compose.yaml up
```

또는 Compose V2 (`docker compose` 사용 시):

```bash
docker compose -f docker-compose.yaml up
```

이렇게 하면:

* `docker-compose.yaml` **파일만 사용됨**
* `docker-compose.override.yaml` **무시됨**
* 결과적으로 **프로덕션과 같은 최소 구성으로 실행**할 수 있음

---

## 실무 팁: 명시적 파일 지정으로 환경 분리

| 환경 | 명령 |
| --- | --- |
| **개발(기본 + override)** | `docker-compose up` (자동 병합) |
| **프로덕션(override 제외)** | `docker-compose -f docker-compose.yaml up` |
| **테스트(별도 설정 사용)** | `docker-compose -f docker-compose.yaml -f docker-compose.test.yaml up` |

---

## 권장 구조 예시

```bash
.
├── docker-compose.yaml # 공통/기본 설정
├── docker-compose.override.yaml # 개발 환경 (자동 적용)
├── docker-compose.prod.yaml # 프로덕션 설정 (선택 적용)
```

### 프로덕션 실행 예:

```bash
docker-compose -f docker-compose.yaml -f docker-compose.prod.yaml up
```

---

## 정리

> `docker-compose.override.yaml` 없이 실행하려면?
 **`-f docker-compose.yaml`** 명시해서 실행하세요.
이렇게 하면 **override.yaml은 완전히 무시되고**, 프로덕션 전용 설정이나 베이스 설정만 반영됩니다.
