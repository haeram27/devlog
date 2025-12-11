# Linux Memory Pressure

Memory pressure는 **시스템의 가용 메모리가 부족해지는 상태**를 의미합니다. 커널이 메모리를 회수(reclaim)해야 하는 압박을 받는 정도를 나타냅니다.

## 핵심 개념

```
메모리 상태:

높은 여유 메모리 ──────────  No Pressure (압박 없음)
       ↓
       ↓  페이지 할당 요청 증가
       ↓
Low Pressure ─────────────  kswapd 백그라운드 회수 시작
       ↓
       ↓  메모리 계속 감소
       ↓
Medium Pressure ──────────  더 적극적인 회수
       ↓
       ↓  메모리 부족 심화
       ↓
High Pressure ────────────  Direct reclaim (프로세스가 직접 대기)
       ↓
       ↓  최악의 경우
       ↓
OOM Killer ───────────────  프로세스 강제 종료
```

## Memory Pressure의 단계

### 1. **No Pressure (정상 상태)**

```bash
free -h
              total    used    free    shared  buff/cache   available
Mem:           16Gi    2.0Gi   10.0Gi    100Mi      4.0Gi       13.0Gi
```

- 충분한 free 메모리 존재
- page cache도 여유롭게 유지
- 메모리 할당이 즉시 이루어짐

### 2. **Low Pressure (가벼운 압박)**

```bash
# free 메모리가 low watermark에 근접
free -h
              total    used    free    shared  buff/cache   available
Mem:           16Gi    8.0Gi   1.0Gi    200Mi      7.0Gi        6.0Gi
```

- **kswapd** 데몬이 백그라운드에서 활성화
- page cache, inactive pages를 회수 시작
- 애플리케이션은 아직 영향 없음

### 3. **Medium Pressure (중간 압박)**

- kswapd가 더 적극적으로 메모리 회수
- dirty pages를 디스크에 기록
- 일부 프로세스가 메모리 할당 시 약간의 지연 경험

### 4. **High Pressure (높은 압박)**

```bash
# free 메모리가 min watermark 근처
free -h
              total    used    free    shared  buff/cache   available
Mem:           16Gi   14.0Gi   200Mi    300Mi      1.8Gi       1.0Gi
```

- **Direct reclaim** 발생
- 메모리를 요청하는 프로세스가 직접 메모리 회수 작업 수행
- 애플리케이션 성능 저하 시작
- I/O 대기 시간 증가

## Pressure의 측정 방법

### 1. **PSI (Pressure Stall Information)** - 커널 4.20+

```bash
cat /proc/pressure/memory
```

출력 예시:

```
some avg10=5.23 avg60=3.41 avg300=1.87 total=123456789
full avg10=1.45 avg60=0.89 avg300=0.34 total=45678901
```

**의미**:

- `some`: 최소 1개 task가 메모리 회수를 기다림 (%)
- `full`: 모든 non-idle task가 메모리를 기다림 (%)
- `avg10/60/300`: 최근 10초/60초/300초 평균
- `total`: 누적 stall time (microseconds)

### 2. **vmstat으로 확인**

```bash
vmstat 1
```

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 512000  50000 400000    0    0     0    10  500 1000  5  2 93  0  0
```

**주목할 필드**:

- `si`, `so`: swap in/out (값이 있으면 높은 pressure)
- `wa`: I/O wait (높으면 메모리 회수 중)
- `b`: blocked processes (메모리 대기 중)

### 3. **/proc/meminfo**

```bash
cat /proc/meminfo | grep -E 'MemFree|MemAvailable|Cached|Dirty|Writeback'
```

## 당신의 Docker 케이스와의 연관성

이전에 논의했던 **page cache가 회수되지 않는 문제**:

```text
호스트 메모리: 128GB
사용 중: 50GB
Page cache: 60GB
Free: 18GB

Docker 컨테이너 limit: 8GB
컨테이너 실제 사용: 7.5GB (limit 근접)
```

### 왜 page cache가 회수 안 되었나?

```text
호스트 관점:
- free 메모리 = 18GB (충분함)
- Memory Pressure = LOW/NONE
- 커널 판단: "여유 메모리가 있으니 page cache 유지하자"

컨테이너 관점:
- 컨테이너 limit 근접 (7.5GB/8GB)
- 하지만 cgroup limit는 page cache 회수를 직접 트리거하지 않음
- Page cache는 "호스트 레벨" 메모리로 간주됨
```

### Memory Pressure가 발생하는 조건

```bash
# Low watermark 계산 예시 (16GB 시스템)
min_free_kbytes = 65536  # 64MB
low_watermark = min_free_kbytes + (min_free_kbytes * watermark_scale_factor / 10000)

# free 메모리가 low_watermark 이하일 때만 pressure 발생
```

## 실무 적용

### Page Cache를 강제로 회수시키려면

#### 시스템 레벨 설정

```bash
# watermark를 높여서 더 일찍 회수 시작
sysctl -w vm.watermark_scale_factor=50
sysctl -w vm.vfs_cache_pressure=200
```

#### 애플리케이션 레벨 (posix_advise)

posix_advise() 강제성 없음, 커널이 page_cache를 삭제하는지 보장 불가

```java
// posix_fadvise로 page cache 명시적 해제
libc.posix_fadvise(fd, 0, 0, POSIX_FADV_DONTNEED);
```

#### 강제 회수 (비추천, 테스트용)

```bash
sync
echo 3 > /proc/sys/vm/drop_caches  # 모든 캐시 버림
```

## 모니터링 스크립트

```bash
#!/bin/bash
# memory-pressure-monitor.sh

while true; do
    echo "=== $(date) ==="
    
    # PSI 확인
    echo "Memory Pressure:"
    cat /proc/pressure/memory
    
    # 메모리 상태
    echo -e "\nMemory Status:"
    free -h | grep Mem
    
    # kswapd 활동
    echo -e "\nkswapd activity:"
    ps aux | grep kswapd | grep -v grep
    
    sleep 5
done
```

Memory pressure는 Linux 메모리 관리의 핵심이며, 당신의 page cache 문제는 "pressure가 충분히 높지 않아서" 발생한 것이었습니다!
