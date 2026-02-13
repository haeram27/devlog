# Linux OS Timezone 설정 확인

linux application은 local timezone 정보가 필요할 때 `TZ` 환경변수 또는 OS의 localtime 설정파일(`/etc/localtime`, `/etc/timezone`)을 참조한다.

linux application은 보통 다음의 우선 순위(높음->낮음)로 timezone을 참조한다.
단, 어플리케이션 별로 차이가 있을 수 있다.

1. TZ 환경변수
1. OS의 localtime 설정파일
   - /etc/localtime (systemd base linux - Debian/Ubuntu/Rocky) - /usr/share/zoneinfo 경로 이하의 timezone 설정 파일에 링크
   - /etc/timezone (initd base linux - Alpine)

위 둘 모두 설정이 되지 않은 경우, 모든 application은 기본적으로 UTC로 동작한다.
TZ나 OS localtime 설정 파일이 비어있거나 기본값이면 glibc/musl가 **UTC**로 동작합니다.

## TZ 환경 변수

```bash
$ echo $TZ
Asia/Seoul
```

## /etc/localtime

```bash
$ readlink /etc/localtime
../usr/share/zoneinfo/Asia/Seoul

$ cat /etc/localtime
TZif2UTCTZif2UTC
UTC0
```

## date

TZ 환경변수 > /etc/localtime 우선 순위로 참조

```bash
$ date
Tue Oct 14 00:51:15 UTC 2025
```

## timedatectl (systemd base linux)

`/etc/localtime` 파일을 소스로 하여 timezone 정보 설정 및 관리

```bash
$ timedatectl
Local time: Tue 2025-10-14 09:52:52 KST
Universal time: Tue 2025-10-14 00:52:52 UTC
RTC time: Tue 2025-10-14 00:52:52
Time zone: Asia/Seoul (KST, +0900)
System clock synchronized: yes
NTP service: active
RTC in local TZ: no
```
