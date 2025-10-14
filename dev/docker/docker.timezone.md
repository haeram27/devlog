# Docker Container에서 OS의 기본 timezone 설정 값

* **대부분의 공식 Docker 베이스 이미지(Alpine, Debian/Ubuntu 등)는 아무 설정도 하지 않으면 기본 타임존이 UTC**입니다.
* 단, “Docker가 정한다”기보다 **컨테이너 안 OS/라이브러리 설정이 기본값(대개 UTC)** 으로 잡힙니다.
* **호스트의 타임존이 Docker Container로 자동 상속되지 않습니다.**

## 왜 그런가?

* 컨테이너의 OS 로컬 타임존은 일반 linux와 마찬가지로 `TZ` 환경변수 또는 OS의 localtime 설정파일(`/etc/localtime`, `/etc/timezone`)으로 결정됩니다.
  * 일반적으로 linux applcation은 TZ 환경변수 > `/etc/localtime` 파일의 우선순위로 timezone을 참조하지만 모든 경우가 같은 것은 아니며, application별로 참조하는 소스가 다를 수 있다.
  * OS의 localtime 설정파일
    * `/etc/localtime`
      * systemd를 사용하는 배포판(Debian/Ubuntu/Rocky 등)의 경우 
      * `/etc/localtime`은 `/usr/share/zoneinfo` 경로의 특정 timezone 파일 항목에 링크되어 있음
    * `/etc/timezone`
      * initd를 사용하는 배포판(Alpine 등)에서 사용
* TZ나 OS localtime 설정 파일이 비어있거나 기본값이면 glibc/musl가 **UTC**로 동작합니다.
* 공식 이미지 기본값:
  * **Alpine**: initd 시스템, `/etc/timezone` 기본 `UTC`
  * **Debian/Ubuntu**: systemd 시스템, tzdata 미설치 상태면 `/etc/localtime`이 UTC 링크거나 미설정 → 사실상 UTC

## 현재 컨테이너의 타임존 확인

```bash
date                          # 현재 시간과 TZ 표시
readlink -f /etc/localtime    # 어떤 타임존 파일을 가리키는지
test -f /etc/timezone && cat /etc/timezone
echo $TZ
```

## 원하는 타임존(예: Asia/Seoul)으로 설정하는 방법

### 가벼운 방법(컨테이너 실행 시)

```bash
docker run -e TZ=Asia/Seoul ...
# 또는 호스트의 타임존/로컬타임 파일 바인드 마운트
docker run -v /etc/localtime:/etc/localtime:ro -v /etc/timezone:/etc/timezone:ro ...
```

### 이미지에 영구 반영(Dockerfile)

* **Alpine**

    ```dockerfile
    FROM alpine:3.20
    RUN apk add --no-cache tzdata \
        && ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime \
        && echo "Asia/Seoul" > /etc/timezone
    ```

* **Debian/Ubuntu**

    ```dockerfile
    FROM debian:bookworm-slim
    RUN apt-get update && apt-get install -y --no-install-recommends tzdata \
        && ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime \
        && echo "Asia/Seoul" > /etc/timezone \
        && rm -rf /var/lib/apt/lists/*
    ```

## 자주 겪는 포인트

* **JVM/Node/Python 등 런타임의 기본 타임존**도 결국 OS 설정이나 `TZ` 환경변수에 의존합니다. 컨테이너 OS가 UTC면 애플리케이션도 기본은 UTC로 잡힙니다.
* **로그 타임스탬프**가 UTC로 보이면 위 방식 중 하나로 타임존을 명시하세요.

요약: 별도 설정이 없으면 거의 항상 **UTC**입니다. 필요 시 `TZ` 환경변수, `/etc/localtime` 또는 `/etc/timezone` 설정, 혹은 tzdata 설치로 명시적으로 지정하세요.
