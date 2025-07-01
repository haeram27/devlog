# 환경 설정 파일(.env) 병합 적용

- docker-compose.yaml 파일과 달리 docker-compose 자체에서 .env 파일을 병합하는 기능은 지원되지 않는다.
- docker-compose 명령의 --env-file 옵션도 중복 해서 사용할 수 없다.
- --env-file 옵션은 1회만 명시 가능하다.

그러므로 여러 .env 파일을 병합해서 사용하려면 쉘 환경에서 병합하여 docker-compose 명령에 환경변수 형태로 전달(주입)해야 한다.

---

## 1. `.env` 병합 자동화 (`dotenv` 도구 또는 `make`)

Docker Compose는 기본적으로 **하나의 `.env` 파일**만 읽지만, **여러 환경 파일을 병합하고 주입**하고 싶을 때는 다음 방식이 자주 사용됩니다.

### 예: `.env.base`, `.env.dev`, `.env.prod` 구조

```bash
# 병합 및 주입
export $(cat .env.base .env.dev | grep -v '^#' | xargs)
docker-compose up
```

* `grep -v '^#'` → 주석 라인 제거
* `xargs` → `key=value` 들을 export 형태로 변환

### Makefile로 자동화

```makefile
run-dev:
	export $(cat .env.base .env.dev | grep -v '^#' | xargs) && \
	docker-compose up
```

---

## 2. Compose 환경변수 값 확인 (`config` 명령)

```bash
docker-compose config
```

* `${VAR}` 가 실제 값으로 치환된 최종 YAML을 출력해 줍니다
* 디버깅이나 배포 전 확인할 때 유용

---

## 3. `.env` 외 다른 확장자 사용 (`--env-file`)

Compose V2에서는 기본 `.env` 외의 파일도 명시 가능:

```bash
docker compose --env-file .env.prod up
```

* `.env.prod` 같은 사용자 지정 환경 파일을 직접 지정 가능

---

## 4. 환경 변수 보안 관리 (예: `.env` Git 제외)

실무에서는 `.env`에 민감 정보가 담기므로 반드시 `.gitignore`에 추가:

```
# .gitignore
.env
.env.*
```

대신 `.env.example` 같은 템플릿 파일을 공유

---

## 요약 (추가 포인트)

| 항목 | 설명 |
| --- | --- |
| 다중 env 병합 | \`cat .env.base .env.dev \| xargs export\` |
| 환경 확인     | `docker-compose config` 로 확인 |
| `.env` 보안   | Git에 커밋하지 않도록 `.gitignore` 등록 |
| 자동화 도구   | `make`, `direnv`, `dotenv-cli`, `docker compose --env-file` |

