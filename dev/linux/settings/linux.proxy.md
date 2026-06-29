# linux proxy setting

- 네트워크에서 프록시의 기본 역할은 클라이언트(사용자 컴퓨터)와 서버(웹사이트 등) 사이에서 트래픽을 대신 전달(중계)해 주는 것입니다. 클라이언트가 외부 인터넷에 직접 연결하는 대신, 프록시 서버가 중간에서 요청을 받아 목적지 서버에 전달하고 그 결과를 다시 클라이언트에게 돌려줍니다

- 프록시는 L7(어플리케이션) 에서 동작합니다. OS의 프록시 설정을 사용할지는 어플리케이션에서의 선택이며 100% 애플리케이션(프로그램) 개발자의 선택이자 구현 방식에 달려 있습니다.

- 프록시 프로그램으로는 대표적으로 Squid가 있으며, 3128 포트를 사용합니다.

## proxy와 netfilter 비교

프록시(Proxy)와 넷필터(Netfilter)는 완전히 다른 계층(Layer)에서 동작하는 별개의 기술이며, 네트워크 패킷이 시스템을 통과할 때 OSI 7 Layer를 계층 구조에 따라 각 단계별로 처리됩니다.

| 구분 | 넷필터 (Netfilter) | 일반적인 프록시 (Proxy) |
|---|---|---|
| 작동 계층 | OSI 3계층(Network), 4계층(Transport) | OSI 7계층(Application) |
| 실행 공간 | 리눅스 커널 내부 (Kernel Space) | 사용자 프로그램 (User Space) |
| 주요 역할 | 방화벽(iptables 기반 차단), NAT(IP 변환) | 웹 캐싱, 데이터 변조, ACL 제어, 인증 |
| 데이터 인지 | 패킷의 출발지/목적지 IP, 포트만 인식 | HTTP URL, 쿠키, 로그인 세션 등 내용 인식 |

## linux proxy 설정 스크립트 (/etc/environment)

단, `/etc/environment` 파일은 wsl ubuntu에서는 적용이 되지않음

```bash
#!/bin/bash

PROXY_URL="http://proxy.internal.com:3128"
NO_PROXY_LIST="127.0.0.1,localhost,1.2.3.0/24,.internal.co.kr,internal.co.kr"

# 기존 항목 제거 후 새로 추가
sed -i '/^http_proxy=/d;/^https_proxy=/d;/^no_proxy=/d' /etc/environment

cat >> /etc/environment << EOF
http_proxy=${PROXY_URL}
https_proxy=${PROXY_URL}
no_proxy="${NO_PROXY_LIST}"
EOF
```

### `.internal.co.kr` 과 `internal.co.kr` 차이

http_proxy, https_proxy, no_proxy 같은 환경 변수는 POSIX 표준이나 RFC 같은 인터넷 표준에 규정된 규격이 아닙니다.

어플리케이션 별로 해당 환경 변수를 참조 할수 있지만, 값에 대한 해석 방법은 조금씩 차이가 있습니다.

1. `.internal.co.kr` (dot(.)으로 시작)

- 동작: internal.co.kr 도메인의 모든 하위 도메인(Subdomain)을 포함하여 프록시 우회
- 적용 범위: internal.co.kr뿐만 아니라 api.internal.co.kr, web.internal.co.kr 등 모든 하위 도메인이 프록시를 타지 않습니다
- 주의사항: 대부분의 리눅스 도구(curl 등)는 앞의 점(.)을 보고 도메인 단위로 판단하지만, 일부 도구(wget 등)는 문자열 자체로 인식하여 루트 도메인(internal.co.kr) 자체는 우회하지 못할 수 있습니다

2. `internal.co.kr` (dot(.) 없이 시작)

- 동작: 정확한 문자열(Exact Match)이 일치할 때만 프록시 우회
- 적용 범위: internal.co.kr 호스트에 대한 요청만 프록시를 우회합니다
- 하위 도메인 처리: sub.internal.co.kr과 같은 하위 도메인을 호출할 때는 프록시를 거치게 됩니다

#### 추천 설정 가이드

사용하는 프로그램(curl, wget, 도커 등)에 따라 proxy 환경 변수에 대한해석 방식에 차이가 있습니다. 하위 도메인을 포함해 해당 도메인 그룹 전체를 프록시 없이 접근하려면 두 가지 표현을 모두 쉼표로 구분해 등록하는 것이 가장 안전합니다.

```bash
export no_proxy="internal.co.kr,.internal.co.kr"
export NO_PROXY="internal.co.kr,.internal.co.kr"
```

## proxy on/off function

- ~/.bashrc

```bash
# --- Proxy Switch Script ---
# 본인의 프록시 서버 주소와 포트로 변경하세요
PROXY_SERVER="http://127.0.0.1:7890"

# 프록시 켜기 함수
function proxy_on() {
    export http_proxy="$PROXY_SERVER"
    export https_proxy="$PROXY_SERVER"
    export ftp_proxy="$PROXY_SERVER"
    export HTTP_PROXY="$PROXY_SERVER"
    export HTTPS_PROXY="$PROXY_SERVER"
    export FTP_PROXY="$PROXY_SERVER"
    echo -e "\033[1;32m[✔] Proxy has been enabled. ($PROXY_SERVER)\033[0m"
}

# 프록시 끄기 함수
function proxy_off() {
    unset http_proxy https_proxy ftp_proxy HTTP_PROXY HTTPS_PROXY FTP_PROXY
    echo -e "\033[1;31m[✘] Proxy has been disabled.\033[0m"
}

# 현재 프록시 상태 확인 함수
function proxy_status() {
    if [ -z "$http_proxy" ]; then
        echo -e "Proxy status: \033[1;31mOFF\033[0m"
    else
        echo -e "Proxy status: \033[1;32mON\033[0m ($http_proxy)"
    fi
}
# ---------------------------
```