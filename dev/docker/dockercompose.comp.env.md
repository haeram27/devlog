# Docker Compose의  `.env` 파일 (Compose 환경 변수)

## compose 환경 변수와 container 환경 변수

docker compose에는 언급되는 환경 변수에는 두 가지 타입이 있다.

* ***compose 환경 변수***
  * docker-compose.yaml 파일내에서 참조 가능한 환경이며 변수 yaml 파일을 구성하는데 사용된다.
  * ***compose 환경 변수는 컨테이너 내부의 환경변수로 전파되지 않는다.***
  * 지정 방법
    * docker compose 명령에 전달 되는 쉘 환경 변수 (export 또는 명령 라인에 함께 명시)
    * docker-compose.yaml과 같은 디렉토리에 존재하는 .env 파일 (--env_file 옵션)
  * .env 파일은 docker compose 명령 실행시 단 하나의 파일만 참조 가능하다.
  * 자동 참조 되는 .env 파일보다 --env_file 옵션으로 지정한 파일의 우선 순위가 높다.
* ***container 환경 변수***
  * docker compose로 실행되는 컨테이너 내부에 생성되는 환경 변수
  * 지정 방법
    * docker-compose.yaml 파일 내 env_file 또는 environment 필드에 명시
    * 중복키의 경우 environment 필드가 env_file 필드 보다 우선순위가 높다.
    * env_file 필드에는 여러 파일이 명시 가능하다.

## `.env` 파일의 위치 및 역할

* `.env` 파일은 **Docker Compose가 자동으로 읽는 기본 환경 설정 파일**입니다.
* docker compose 명령 실행시 Compose는 자동으로 **현재 실행 경로의 `.env` 파일 하나만 읽습니다.**
* **docker-compose.yaml** 파일과 **같은 디렉토리에 있어야** 자동으로 인식됩니다.
* 이 파일에 정의된 변수는 `${VAR}` 형태로 docker-compose.yaml에서 참조할 수 있습니다.

```dotenv
# .env
APP_IMAGE=myapp:1.0
APP_PORT=8080
```

---

## `.env` 파일 병합 / 다중 파일 사용

* docker-compose.yaml 파일과 달리 docker-compose 자체에서 .env 파일을 병합하는 기능은 지원되지 않는다.
* docker-compose 명령의 --env-file 옵션도 중복 해서 사용할 수 없다.

그러므로 여러 .env 파일을 병합해서 사용하려면 쉘 환경에서 병합하여 docker-compose 명령에 환경변수 형태로 전달(주입)해야 한다.

### Bash로 병합 후 실행

병합은 보통 개발/상용 환경 등을 구분하여 compose를 실행하는 경우 자주 사용된다.

다음 예는 `.env.base`, `.env.dev`, `.env.prod` 세 가지 파일 구조로 실행 환경 별로 dev와 prod를 구분하여 사용하는 경우를 상정한다.

#### 예: dev 환경 변수 병합 적용

```bash
# 병합 및 주입
export $(cat .env.base .env.dev | grep -v '^#' | xargs)
docker-compose up
```

#### 예: prodd 환경 변수 병합 적용

```bash
export $(cat .env.base .env.prod | grep -v '^#' | xargs)
docker-compose up
```

* `grep -v '^#'` → 주석 라인 제거
* `xargs` → `key=value` 들을 export 형태로 변환

## 병합 자동화 (`dotenv` 도구 또는 `make`)

Makefile로 자동화

```makefile
run-dev:
 export $(cat .env.base .env.dev | grep -v '^#' | xargs) && \
 docker-compose up
```

---

## Compose 환경변수 값 확인 (`config` 명령)

```bash
docker-compose config
```

* `${VAR}` 가 실제 값으로 치환된 `최종 YAML`을 출력해 줍니다
* yaml 출력 이외에 배포 등의 액션은 실행되지 않습니다
* 디버깅이나 배포 전 확인할 때 유용

---

## 기본 `.env` 파일 이외에 다른 이름의 설정 파일을 지정하여 사용(`--env-file`)

compose 명령어의 --env-file 옵션을 사용한다.

```bash
docker compose --env-file .env.prod up
```

* `.env.prod` 같은 사용자 지정 환경 파일을 직접 지정 가능

---

## 환경 변수 보안 관리 (예: `.env` Git 제외)

실무에서는 `.env`에 민감 정보가 담기므로 반드시 `.gitignore`에 추가:

```text
# .gitignore
.env
.env.*
```

대신 `.env.example` 같은 템플릿 파일을 공유
