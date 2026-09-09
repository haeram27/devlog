# Linux FHS (Filesystem Hierarchy Standard)

FHS는 리눅스/유닉스 계열 시스템에서 디렉토리 구조와 각 경로의 용도를 표준화한 규격입니다.  
운영체제, 패키지 매니저, 애플리케이션이 서로 충돌 없이 공존하도록 기준을 제공합니다.

참고: <https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html>

## 왜 중요한가

- 운영 자동화(백업, 배포, 모니터링) 규칙을 일관되게 만들 수 있음
- 패키지 설치 경로가 표준화되어 충돌과 중복을 줄임
- 장애 대응 시 로그/설정/실행파일 위치를 빠르게 추적 가능

## 핵심 디렉토리 요약

| 경로 | 용도 | 예시 |
|---|---|---|
| `/` | 루트 파일시스템 최상위 | 모든 경로의 시작점 |
| `/bin` | 기본 사용자 명령(부팅/복구에 필요) | `ls`, `cp`, `cat` |
| `/sbin` | 시스템 관리 명령 | `fsck`, `ip`, `mount` |
| `/etc` | 시스템 설정 파일 | `/etc/ssh/sshd_config` |
| `/usr` | 사용자 공간 프로그램/라이브러리/문서 | `/usr/bin`, `/usr/lib` |
| `/var` | 가변 데이터(로그, 스풀, 캐시) | `/var/log`, `/var/lib` |
| `/tmp` | 임시 파일(재부팅 시 정리될 수 있음) | 앱 임시 작업 파일 |
| `/home` | 일반 사용자 홈 디렉토리 | `/home/devuser` |
| `/root` | root 사용자 홈 디렉토리 | `/root/.bashrc` |
| `/opt` | 서드파티/벤더 제공 추가 소프트웨어 | `/opt/myapp` |
| `/srv` | 서비스 데이터(사이트/FTP 등) | `/srv/www`, `/srv/git` |
| `/run` | 부팅 이후 런타임 상태 데이터(PID, 소켓) | `/run/nginx.pid` |
| `/dev` | 디바이스 파일 | `/dev/sda`, `/dev/null` |
| `/proc` | 커널/프로세스 가상 파일시스템 | `/proc/cpuinfo` |
| `/sys` | 커널 디바이스/드라이버 정보 | `/sys/class/net` |
| `/boot` | 부트로더/커널 이미지 | `vmlinuz`, `initramfs` |

## /opt vs /usr vs /usr/local

실무에서 가장 많이 헷갈리는 경로입니다.

| 경로 | 주 용도 | 관리 주체 | 권장 상황 |
|---|---|---|---|
| `/usr` | 배포판 기본 패키지 구성요소 | OS 패키지 매니저(`apt`, `dnf`, `yum`) | 운영체제 표준 패키지 설치 |
| `/usr/local` | 로컬 관리자가 직접 설치한 소프트웨어 | 시스템 관리자(수동 설치/컴파일) | 소스 빌드 설치, OS 패키지와 분리 운영 |
| `/opt` | 벤더/서드파티 독립 패키지 | 벤더 설치 스크립트 또는 관리자 | 독립 디렉토리 단위 배포/업그레이드/롤백이 필요한 상용/독립 앱 |

### 선택 기준 (실무)

1. **OS 패키지 생태계에 맞출 것**이면 `/usr` 계열(패키지 매니저) 사용
2. **직접 빌드/수동 설치**면 `/usr/local` 우선
3. **벤더 제품 또는 앱 전체를 한 디렉토리로 관리**하려면 `/opt/<app>` 사용
4. 컨테이너 이미지에서는 단순성을 위해 `/app` 같은 커스텀 경로도 자주 쓰지만, 호스트 운영 표준 문서에는 FHS 기준을 명시하는 것이 좋음

## 상태 데이터 배치 원칙

- 설정: `/etc`
- 실행 바이너리: `/usr/bin` 또는 `/usr/local/bin`
- 라이브러리: `/usr/lib` 또는 `/usr/local/lib`
- 로그: `/var/log/<app>`
- 영속 데이터(DB, 인덱스, 업로드): `/var/lib/<app>` 또는 `/srv/<service>`
- 캐시: `/var/cache/<app>`
- PID/소켓/런타임: `/run/<app>`
- 임시 파일: `/tmp` 또는 `/var/tmp`  
  - `/tmp`: 단기 임시, 재부팅 시 정리될 수 있음  
  - `/var/tmp`: 더 오래 유지될 수 있는 임시 데이터

## 배포 예시

### 예시 1) 패키지 매니저 기반 설치

- 바이너리: `/usr/bin/myapp`
- 설정: `/etc/myapp/myapp.yaml`
- 로그: `/var/log/myapp/`
- 데이터: `/var/lib/myapp/`
- 서비스 유닛: `/usr/lib/systemd/system/myapp.service` (배포판별 차이 가능)

### 예시 2) 벤더 번들 설치 (/opt)

- 앱 홈: `/opt/myapp/`
- 실행 파일: `/opt/myapp/bin/myapp`
- 설정 심볼릭 링크: `/etc/myapp -> /opt/myapp/conf`
- 로그: `/var/log/myapp/`
- 데이터: `/var/lib/myapp/`

## 운영 시 주의점

- 앱 로그를 실행 디렉토리(`./logs`)에 남기지 말고 `/var/log`로 분리
- 설정 파일을 바이너리 경로와 분리해 배포/업그레이드 시 덮어쓰기 리스크 감소
- 데이터 디렉토리 권한(소유자/그룹/umask) 명확화
- 백업/복구 정책을 `/etc`, `/var/lib`, `/srv` 중심으로 설계

## 요약

- FHS는 리눅스 시스템의 파일 배치를 표준화하는 기준이다.
- `/usr`는 OS 패키지 영역, `/usr/local`은 로컬 수동 설치 영역, `/opt`는 독립 벤더/서드파티 앱 영역으로 구분한다.
- 운영 안정성을 위해 **설정(`/etc`) / 로그(`/var/log`) / 데이터(`/var/lib` 또는 `/srv`)**를 분리하는 패턴을 유지하는 것이 핵심이다.
