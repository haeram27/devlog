# MicroCeph 설정

MicroCeph의 접속 포트는 각 서비스(대시보드, RGW 등)를 활성화할 때 옵션을 주거나, 설치 후 Ceph 내부 설정을 변경하는 방식으로 조정할 수 있습니다. [1]

```bash
```


## 주요 서비스 및 기본 포트

- [microceph command enable](https://canonical-microceph.readthedocs-hosted.com/stable/reference/commands/enable/)
- [microceph network configuration reference](https://docs.ceph.com/en/latest/rados/configuration/network-config-ref/)
- [ceph dash board](https://docs.ceph.com/en/latest/mgr/dashboard/)

MicroCeph의 활성화 가능한 서비스 및 기본 포트

- RGW (microceph enable rgw)
  - HTTP: 80
  - HTTPS: 443

- MON (모니터)
  - msgr2: 3300
  - msgr1: 6789

- OSD / MDS / MGR 데몬 통신
  - 동적 포트 범위: 6800-7568
  - 고정 1개가 아니라, 데몬이 이 범위에서 사용 가능한 포트를 점유합니다.

- Dashboard (Ceph mgr dashboard 모듈)
  - 기본 SSL: 8443
  - SSL 비활성 시: 8080

참고로 “MicroCeph 설치 직 후 전부 활성되는 것은 아니며, 어떤 서비스를 enable 했는지에 따라 실제로 열리는 포트가 달라집니다.

현재 노드에서 실제 열린 포트를 확인하려면 다음 2개가 가장 정확합니다.
1. `microceph status`
2. `sudo microceph.ceph mgr services`

## 1. 주요 서비스 포트 설정 방법

- S3/Object Gateway (RGW): `microceph enable rgw --port <포트번호>` 명령어로 활성화 시 포트를 지정합니다. 활성화된 RGW의 포트를 변경하려면 disable 후 재활성화합니다.

- 대시보드 (Dashboard): 기본 `8080/8443` 포트를 사용하며, `microceph.ceph config set` 명령어를 통해 `mgr/dashboard/server_port` (HTTP) 또는 `ssl_server_port` (HTTPS)를 수정하여 변경할 수 있습니다.

## 2. Ceph 기본 통신 포트

- MON: 3300(v2), 6789(v1) 포트를 사용합니다.
- OSD: 6800~7300 대역을 사용하며, 필요 시 ms_bind_port_min/max 설정으로 변경 가능합니다. [8, 9, 10, 11] 

## 3. 설정 확인

microceph.ceph config dump 또는 microceph status 명령어로 현재 적용된 포트와 서비스 상태를 확인할 수 있습니다.

대시보드에 접속하기 위한 관리자 계정 생성 방법이 필요하신가요? 혹은 외부 IP에서 접속할 수 있도록 바인딩 설정을 추가로 구성하고 싶으신가요?

[3] [https://canonical-microceph.readthedocs-hosted.com](https://canonical-microceph.readthedocs-hosted.com/stable/tutorial/get-started/)

[4] [https://canonical-microceph.readthedocs-hosted.com](https://canonical-microceph.readthedocs-hosted.com/v19.2.0-squid/reference/commands/enable/)


## 대시보드 접속 포트 변경

대시보드 포트 번호를 변경할 때는 서비스를 껐다 켤 필요 없이 명령어 입력 후 매니저(mgr) 데몬만 자동으로 재시작되도록 하면 즉시 반영됩니다.
MicroCeph(Ceph) 대시보드 포트를 변경하는 정확한 절차는 다음과 같습니다.

### 1. 포트 설정 변경

먼저 원하는 포트 번호(예: 9000)를 설정합니다.

```bash
# HTTPS 포트 변경 시
sudo microceph.ceph config set mgr mgr/dashboard/ssl_server_port 9000
# 만약 HTTP를 사용 중이라면
sudo microceph.ceph config set mgr mgr/dashboard/server_port 9000
```

### 2. 설정 반영을 위한 모듈 재시작

설정값을 바꾼 후, 대시보드 모듈을 껐다가 다시 켜주면 변경된 포트로 서비스가 다시 올라옵니다.

```bash
sudo microceph.ceph mgr module disable dashboard
sudo microceph.ceph mgr module enable dashboard
```

## 3. 변경 확인

포트가 정상적으로 변경되었는지 확인합니다.

```bash
sudo microceph.ceph mgr services
```

출력 결과에 `dashboard: https://<IP>:9000/` 과 같이 표시되면 성공

참고: 만약 microceph enable dashboard 명령어를 통해 처음 활성화할 때 포트를 지정하고 싶다면, 현재 MicroCeph 버전에서는 별도의 포트 옵션을 지원하지 않는 경우가 많아 위와 같이 설치 후 config set으로 변경하는 것이 가장 확실한 방법입니다.

## 대시보드 관리자 계정 관리

대시보드에서는 웹 화면으로 OSD 상태나 스토리지 사용량을 확인 가능

### 1. 관리자 계정 생성

MicroCeph(Ceph) 대시보드의 계정 설정은 ceph dashboard 명령어를 사용하여 관리합니다. 초기 설치 시에는 계정이 생성되어 있지 않으므로 아래 순서대로 진행해 주세요.

가장 먼저 관리자 권한(administrator)을 가진 사용자 아이디와 비밀번호를 생성합니다.

```bash
# 형식: sudo microceph.ceph dashboard ac-user-create <아이디> <비밀번호> administrator
sudo microceph.ceph dashboard ac-user-create admin mypassword administrator
```

- admin: 사용할 아이디
- mypassword: 사용할 비밀번호 (실제 사용하실 비밀번호로 바꾸세요)
- administrator: 부여할 역할(Role) 이름

### 2. 생성된 계정 확인

계정이 정상적으로 생성되었는지 리스트를 확인합니다.

```bash
sudo microceph.ceph dashboard ac-user-show
```

### 3. 비밀번호를 잊어버렸을 때 (변경 방법)

기존 계정의 비밀번호를 바꾸고 싶다면 다음 명령어를 사용합니다.

```bash
sudo microceph.ceph dashboard ac-user-set-password admin newpassword
```

### 4. 대시보드 접속 주소 확인

계정을 만들었다면 어느 주소로 접속해야 하는지 확인합니다.

```bash
sudo microceph.ceph mgr services
```

출력되는 URL(예: https://192.168.1)을 브라우저에 입력하고 방금 만든 계정으로 로그인하면 됩니다.

### ⚠️ 대시보드 접속 시 참고사항:

- SSL 인증서 경고: 로컬 테스트용은 자체 서명 인증서를 사용하므로 브라우저에서 "연결이 비공개로 설정되어 있지 않습니다"라는 경고가 뜰 수 있습니다. 이때 [고급] -> [접속하기(안전하지 않음)]를 클릭하여 진행하시면 됩니다.
- 로그인 불가 시: 대시보드 모듈이 정상 작동 중인지 sudo microceph.ceph mgr module ls 명령어로 dashboard 항목이 enabled 상태인지 확인해 보세요.

