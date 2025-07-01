# Docker Compose의  `.env` 파일

## 1. `.env` 파일의 위치 및 역할

* `.env` 파일은 **Docker Compose가 자동으로 읽는 기본 환경 설정 파일**입니다.
* **docker-compose.yaml** 파일과 **같은 디렉토리에 있어야** 자동으로 인식됩니다.
* 이 파일에 정의된 변수는 `${VAR}` 형태로 docker-compose.yaml에서 참조할 수 있습니다.

```dotenv
# .env
APP_IMAGE=myapp:1.0
APP_PORT=8080
```

---

## 2. 환경변수 우선순위

Docker Compose는 환경 변수를 아래 **우선순위 순서**대로 해석합니다 (위가 우선):

| 우선순위 | 설명 |
| --- | --- |
| (1) 셸 환경 변수 (`export VAR=...`)  | docker-compose 명령 실행 쉘에서 설정한 값 |
| (2) `.env` 파일 | Compose와 같은 디렉토리의 .env |
| (3) docker-compose.yaml의 하드코딩 값 | 직접 값 지정 (예: `image: nginx:latest`) |
| (4) Dockerfile 내 `ENV` | 컨테이너 이미지 생성시 지정한 기본값 |

> 즉, **`.env` 파일보다 쉘에서 `export VAR=...` 한 값이 우선합니다.**

---

## 3. `.env` 파일 병합 / 다중 파일 사용

### 기본적으로 `.env` 파일은 **하나만 사용됩니다.**

Compose는 자동으로 **현재 경로의 `.env` 파일 하나만 읽습니다.**

### 다중 `.env` 파일 병합하려면?

#### 방법 1: `env_file` 키 사용 (단, 이건 **컨테이너 안 환경변수 용도** 입니다)

```yaml
services:
  web:
    image: nginx
    env_file:
      - .env
      - .env.prod
```

* 이는 `.env` 파일처럼 Compose 파일 내 `${VAR}`를 대체하는 것이 아니라,
* **컨테이너의 `ENV`로 들어가는 환경 변수들**을 설정합니다.

#### 방법 2: Bash로 병합 후 실행

```bash
export $(cat .env .env.prod | grep -v '^#' | xargs)
docker-compose up
```

* 이 방법은 **`.env`를 병합해서 셸 환경변수로 주입**한 다음 Compose를 실행

---

## 4. 환경변수 주입 방식 (Compose 내에서)

Docker Compose는 변수 주입을 세 가지 방식으로 지원합니다:

### ① 파일 내 `${VAR}` 확장

```yaml
services:
  app:
    image: ${APP_IMAGE}
```

* Compose가 시작될 때 **셸 + .env의 변수로 대체**

---

### ② `environment:` 섹션에서 직접 주입

```yaml
services:
  app:
    environment:
      - VAR1=value1
      - VAR2=${EXTERNAL_VAR}
```

* 환경변수 직접 지정 또는 외부 변수 참조 가능
* 컨테이너 내부의 `ENV`로 설정됨

---

### ③ `env_file:` 로 외부 파일 불러오기

```yaml
services:
  app:
    env_file:
      - .env
      - custom.env
```

* 해당 파일 내 key=value 들을 **컨테이너 환경변수로 주입**
* 명시된 순서의 위에서 아래로 override 방식으로 적용됨 

---

## 우선순위 예시

```bash
export APP_IMAGE=prod-image:latest
```

```dotenv
# .env
APP_IMAGE=dev-image:v1
```

```yaml
# docker-compose.yaml
services:
  app:
    image: ${APP_IMAGE}
```

▶ 결과적으로 사용되는 이미지:

```
prod-image:latest
```

왜? → **셸 환경변수(export)가 .env보다 우선**

---

## 요약

| 항목 | 설명 |
| --- | --- |
| `.env` 파일 위치 | `docker-compose.yaml`와 같은 디렉토리에 있어야 자동 인식 |
| 변수 우선순위 | `셸 환경 > .env > compose 하드코딩 > Dockerfile ENV` |
| 병합 방식 | Compose 자체는 `.env` 하나만 읽음. 병합하려면 셸에서 export로 처리해야 함 |
| 주입 방법 | `${VAR}` 치환, `environment:`, `env_file:` 세 가지 방식 있음 |
