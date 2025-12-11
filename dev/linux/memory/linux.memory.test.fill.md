# linux 메모리 부하 테스트

## linux 메모리 할당 방법

Linux에서 테스트 목적으로 메모리를 채우는 방법

### stress-ng 사용 (권장)

```bash
# 설치
sudo apt-get install stress-ng

# 메모리 4GB 채우기 (60초간)
stress-ng --vm 1 --vm-bytes 4G --timeout 60s

# 메모리를 계속 채우기 (수동 종료까지)
stress-ng --vm 1 --vm-bytes 4G
```

## `dd (diskdump)` 명령어 사용

## `dd` 사용법

***4G 파일 생성하기***

```bash
dd if=/dev/zero of=/tmp/bigfile bs=1M count=4096
dd if=/dev/zero of=/tmp/bigfile bs=1G count=4  # bs 1M 보다 dump 속도 빠름
```

***`dd` 명령어***

- **Disk Dump** (또는 Data Description)
- 블록 단위로 데이터를 복사하는 저수준 유틸리티
- 파일 생성, 복사, 변환에 사용

***`if=/dev/zero` (Input File)***

- **입력 소스**: `/dev/zero`
- `/dev/zero`는 무한히 null 바이트(0x00)를 제공하는 특수 장치 파일
- 읽을 때마다 0으로 채워진 데이터를 반환
- `/dev/zero` 대신 0이 아닌 데이터를 출력하는 `/dev/urandom`을 사용할 수도 있으나 dump 속도가 2-3배 느림

```bash
# /dev/zero의 동작
cat /dev/zero | hexdump -C | head
# 00 00 00 00 00 00 00 00 ... 계속 0
```

***`of=/dev/shm/bigfile` (Output File)***

- **출력 대상**: `/dev/shm/bigfile`
- `/dev/shm`: tmpfs 파일시스템 (RAM을 filesystem으로 사용), 파일 생성시 메모리 항목중 `shared`로 할당
- `bigfile`: 생성될 파일 이름
- 물리 디스크가 아닌 **메모리에 파일 생성**

***`bs=1M` (Block Size)***

- **블록 크기**: 1 메가바이트
- 한 번에 읽고 쓰는 데이터 크기
- `1M` = 1 MiB = 1,048,576 bytes (1024 × 1024)

```bash
# 블록 크기 단위들
bs=1         # 1 byte
bs=1K        # 1 KiB = 1,024 bytes
bs=1M        # 1 MiB = 1,048,576 bytes
bs=1G        # 1 GiB = 1,073,741,824 bytes
```

***`count=5120` (Count)***

- **블록 개수**: 5120개
- `bs` 크기의 블록을 몇 개 복사할지 지정

### Shared Memory 채우기 - /dev/shm 사용 (tmpfs)

```bash
# 메모리 기반 파일시스템에 데이터 쓰기
dd if=/dev/zero of=/dev/shm/bigfile bs=10M count=512 # 5GB 파일 생성

# 확인
df -h /dev/shm
free -h
```

### Page Cache 채우기

```bash
# 큰 파일 생성 및 읽기로 page cache 채우기
dd if=/dev/zero of=/tmp/bigfile bs=1M count=4096  # 4GB 파일 생성
cat /tmp/bigfile > /dev/null  # page cache에 적재

# 메모리 상태 확인
free -h
```

## 메모리 상태 모니터링

### 실시간 메모리 사용량 확인

```bash
watch -n 1 free -h
```

### 상세 메모리 정보

```bash
cat /proc/meminfo
```

### 프로세스별 메모리 사용량

```bash
ps aux --sort=-%mem | head -10
```
