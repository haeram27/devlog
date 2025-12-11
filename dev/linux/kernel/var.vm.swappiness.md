# vm.swappiness

## vm.swappiness 개요

`vm.swappiness`는 커널 변수이며, 커널이 메모리 압박 상황에서 **익명 페이지(anonymous pages)**를 스왑하는 것과 **페이지 캐시(page cache)**를 회수하는 것 사이의 균형을 조절하는 파라미터

```bash
# 현재 값 확인
cat /proc/sys/vm/swappiness
# 또는
sysctl vm.swappiness

# 일반적인 기본값: 60
```

## 값의 범위와 의미

```bash
vm.swappiness = 0 ~ 200
```

### 값별 동작 방식

#### 0 (스왑 최소화)

```bash
sysctl -w vm.swappiness=0
```

- 스왑을 **거의 사용하지 않음**
- 메모리 부족 시 page cache를 먼저 회수
- OOM(Out of Memory) killer가 더 쉽게 발동될 수 있음
- ⚠️ **주의**: 완전히 스왑을 비활성화하는 것은 아님!

#### 1 (스왑 극도로 회피)

```bash
sysctl -w vm.swappiness=1
```

- 스왑을 극도로 회피
- 메모리 압박이 매우 심할 때만 스왑 사용
- **데이터베이스 서버**에서 자주 사용

#### 10 (스왑 회피)

```bash
sysctl -w vm.swappiness=10
```

- 스왑을 많이 회피하지만 0보다는 유연
- page cache 우선 회수
- **성능 중시 서버**에 권장

#### 60 (기본값, 균형)

```bash
sysctl -w vm.swappiness=60
```

- Linux의 기본값
- page cache와 스왑 사이의 **균형잡힌 접근**
- 데스크톱 환경에 적합

#### 100 (적극적 스왑)

```bash
sysctl -w vm.swappiness=100
```

- page cache와 익명 메모리를 동등하게 취급
- 스왑을 더 적극적으로 사용
- **메모리가 부족한 환경**에서 유용

#### 200 (스왑 선호)

```bash
sysctl -w vm.swappiness=200
```

- 스왑을 매우 선호
- page cache보다 익명 메모리를 더 적극적으로 스왑아웃
- 특수한 워크로드에서만 사용

## 내부 동작 원리

### 메모리 회수 알고리즘

커널이 메모리를 회수할 때 두 가지 선택:

```text
1. Page Cache 회수 (파일 캐시)
   - 디스크에 이미 백업이 있음
   - 나중에 다시 읽으면 됨
   - 비교적 빠름

2. Anonymous Pages 스왑아웃 (프로세스 메모리)
   - 스왑 공간에 써야 함
   - I/O 비용 발생
   - 나중에 스왑인 필요
   - 비교적 느림
```

### Swappiness 계산식

커널 내부에서 사용하는 간단화된 공식:

```text
swap_tendency = mapped_ratio/2 + distress + vm_swappiness

mapped_ratio: 매핑된 페이지 비율
distress: 메모리 압박 정도 (0-100)
vm_swappiness: 설정값 (0-200)
```

**높은 swap_tendency → 스왑 선호**
**낮은 swap_tendency → page cache 회수 선호**

## 실제 동작 예시

### 시나리오 1: vm.swappiness=60 (기본값)

```bash
# 초기 상태
free -h
              total        used        free      shared  buff/cache   available
Mem:            32G         10G         15G         1.0G           7G          20G
Swap:           8G          0B          8G

# 메모리 압박 발생 (대용량 파일 복사)
cp large_file.iso /tmp/

# 결과
free -h
              total        used        free      shared  buff/cache   available
Mem:            32G         10G          2G         1.0G          20G          15G
Swap:           8G        500M        7.5G
#                           ↑ 일부 프로세스 메모리가 스왑됨
#                                               ↑ page cache 크게 증가
```

**동작:**

- page cache가 크게 증가 (파일 읽기)
- 일부 사용하지 않는 프로세스 메모리가 스왑됨
- **균형잡힌 접근**

### 시나리오 2: vm.swappiness=10

```bash
sysctl -w vm.swappiness=10

# 같은 작업 수행
cp large_file.iso /tmp/

# 결과
free -h
              total        used        free      shared  buff/cache   available
Mem:            32G         10G          1G         1.0G          21G          14G
Swap:           8G         50M        7.95G
#                           ↑ 스왑 사용 최소화
#                                               ↑ page cache 더 크게 증가
```

**동작:**

- page cache를 최대한 확보
- 스왑은 극도로 회피
- 프로세스 메모리는 RAM에 유지

### 시나리오 3: vm.swappiness=0

```bash
sysctl -w vm.swappiness=0

# 메모리 부족 상황
stress-ng --vm 8 --vm-bytes 4G

# 결과
# OOM killer 발동 가능성 높음
# dmesg | tail
# Out of memory: Kill process 12345 (stress-ng) score 800
```

## 워크로드별 권장 설정

### 1. 데이터베이스 서버 (MySQL, PostgreSQL, Redis)

```bash
# 권장: 1 또는 10
sysctl -w vm.swappiness=1

# /etc/sysctl.conf에 영구 설정
echo "vm.swappiness=1" >> /etc/sysctl.conf
sysctl -p
```

**이유:**

- DB는 자체 캐싱 메커니즘 보유
- 스왑되면 성능 급격히 저하
- 메모리에 상주해야 함

### 2. 웹 서버 (Nginx, Apache)

```bash
# 권장: 10
sysctl -w vm.swappiness=10
```

**이유:**
- 빠른 응답 시간 필요
- page cache도 중요 (정적 파일)
- 균형잡힌 접근

### 3. Java 애플리케이션 (당신의 경우)

```bash
# 권장: 10
sysctl -w vm.swappiness=10
```

**이유:**

- JVM 힙이 스왑되면 GC 성능 저하
- 파일 I/O도 많음 (청크 업로드)
- page cache도 필요

### 4. 데스크톱 / 개발 환경

```bash
# 권장: 60 (기본값 유지)
```

**이유:**

- 다양한 애플리케이션 실행
- 균형잡힌 메모리 관리

### 5. 파일 서버 / NAS

```bash
# 권장: 10-20
sysctl -w vm.swappiness=10
```

**이유:**

- page cache가 매우 중요
- 프로세스 메모리는 상대적으로 적음

### 6. 컨테이너 호스트 (Docker, K8s)

```bash
# 권장: 10
sysctl -w vm.swappiness=10
```

**이유:**

- 많은 컨테이너 실행
- 각 컨테이너의 성능 중요
- 스왑 사용 시 전체 성능 저하

## 설정 방법

### 임시 설정 (재부팅 시 초기화)

```bash
# 방법 1
sysctl -w vm.swappiness=10

# 방법 2
echo 10 > /proc/sys/vm/swappiness

# 확인
cat /proc/sys/vm/swappiness
```

### 영구 설정

```bash
# /etc/sysctl.conf 편집
sudo nano /etc/sysctl.conf

# 다음 라인 추가
vm.swappiness=10

# 저장 후 적용
sudo sysctl -p

# 또는 개별 파일로 관리
echo "vm.swappiness=10" | sudo tee /etc/sysctl.d/99-swappiness.conf
sudo sysctl -p /etc/sysctl.d/99-swappiness.conf
```

### Docker 컨테이너에서

```bash
# 컨테이너는 호스트의 swappiness 설정을 따름
# 호스트에서 설정 필요

# 호스트에서
sudo sysctl -w vm.swappiness=10

# 컨테이너 실행 시 스왑 제한
docker run --memory-swap=4g --memory=4g myapp
```

## 실시간 모니터링

### 스왑 사용량 확인

```bash
# 기본 확인
free -h

# 실시간 모니터링
watch -n 1 'free -h && echo && cat /proc/sys/vm/swappiness'

# 상세 정보
cat /proc/meminfo | grep -E "Swap|Cached"

# 프로세스별 스왑 사용량
for pid in $(ls /proc | grep -E '^[0-9]+$'); do
    [ -f /proc/$pid/status ] && \
    swap=$(grep VmSwap /proc/$pid/status 2>/dev/null | awk '{print $2}')
    [ ! -z "$swap" ] && [ "$swap" != "0" ] && \
    echo "PID $pid: $swap kB ($(ps -p $pid -o comm=))"
done | sort -k3 -nr | head -10
```

### 스왑 이벤트 모니터링

```bash
# vmstat으로 스왑 in/out 확인
vmstat 1

# si (swap in): 스왑에서 메모리로
# so (swap out): 메모리에서 스왑으로

# 예시 출력:
# procs -----------memory---------- ---swap-- -----io----
#  r  b   swpd   free   buff  cache   si   so    bi    bo
#  0  0 524288 1048576  10240 5242880  0    0     0     0
#  1  0 524288 1048576  10240 5242880  0  128     0   256  ← 스왑아웃 발생
```

## Swappiness 0의 특별한 케이스

### 커널 버전별 차이

**커널 3.5 이전:**

```bash
vm.swappiness=0
# 완전히 스왑 비활성화 (매우 위험!)
```

**커널 3.5 이후:**

```bash
vm.swappiness=0
# 스왑을 회피하지만, 긴급 상황에서는 사용
# OOM killer 발동 전 마지막 수단으로 스왑 사용 가능
```

### 권장사항

```bash
# 스왑을 최소화하려면
vm.swappiness=1  # 0보다 안전
```

## 당신의 Java 애플리케이션 최적화

### 현재 상황 분석

```bash
# 파일 청크 업로드 → page cache 많이 사용
# Java 애플리케이션 → JVM 힙 메모리 중요
```

### 권장 설정

```bash
# 1. swappiness 설정
sudo sysctl -w vm.swappiness=10

# 2. 영구 적용
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf

# 3. JVM 옵션 추가
java -Xmx4g -Xms4g \
     -XX:+AlwaysPreTouch \  # 힙을 미리 할당
     -jar your-app.jar

# 4. posix_fadvise 사용 (이미 구현하신 것)
# POSIX_FADV_DONTNEED로 page cache 관리
```

### 모니터링 스크립트

```bash
#!/bin/bash
# monitor_memory.sh

while true; do
    clear
    echo "=== Memory Status ==="
    free -h
    echo ""
    echo "=== Swappiness ==="
    cat /proc/sys/vm/swappiness
    echo ""
    echo "=== Java Process Memory ==="
    ps aux | grep java | grep -v grep
    echo ""
    echo "=== Swap Activity (last 5 sec) ==="
    vmstat 1 5 | tail -1
    sleep 5
done
```

## 추가 관련 커널 파라미터

```bash
# page cache 관리
vm.vfs_cache_pressure=50     # 기본값 100, 낮을수록 cache 유지

# dirty page 관리
vm.dirty_ratio=10            # 전체 메모리의 10%
vm.dirty_background_ratio=5  # 백그라운드 플러시 시작

# OOM killer 조정
vm.overcommit_memory=1       # 과할당 허용
vm.panic_on_oom=0            # OOM 시 패닉 안함

# 설정 예시
sudo tee /etc/sysctl.d/99-memory-tuning.conf <<EOF
vm.swappiness=10
vm.vfs_cache_pressure=50
vm.dirty_ratio=10
vm.dirty_background_ratio=5
EOF

sudo sysctl -p /etc/sysctl.d/99-memory-tuning.conf
```

## 요약

| Swappiness | 사용 케이스 | 특징 |
|------------|-------------|------|
| 0 | ❌ 비권장 | OOM 위험 높음 |
| 1 | 데이터베이스 | 스왑 극도로 회피 |
| 10 | 웹서버, Java 앱 | **균형잡힌 성능 (권장)** |
| 60 | 데스크톱, 범용 | 기본값 |
| 100+ | 메모리 부족 환경 | 스왑 적극 사용 |
