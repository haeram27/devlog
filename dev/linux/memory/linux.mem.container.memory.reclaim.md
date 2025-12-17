# Host vs Container 메모리 회수 구조

## 1. 메모리 회수 구조

### 전역 메모리 회수 시스템 (하나만 존재)

```
┌─────────────────────────────────────────────────────┐
│              Host Linux Kernel                      │
│                                                     │
│  ┌───────────────────────────────────────────────┐ │
│  │    Global Memory Reclaim System (하나!)      │ │
│  │    - kswapd (백그라운드 회수)                 │ │
│  │    - direct reclaim (즉시 회수)              │ │
│  └───────────────┬───────────────────────────────┘ │
│                  │                                  │
│                  ▼                                  │
│  "전체 시스템 메모리가 부족한가?"                    │
│           ├─ Yes → 전역 회수 시작                   │
│           └─ No → 아무 일도 안 함                   │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │ cgroup A │  │ cgroup B │  │ cgroup C │         │
│  │ limit:2G │  │ limit:4G │  │ limit:1G │         │
│  └──────────┘  └──────────┘  └──────────┘         │
│       ↑              ↑              ↑              │
│       └──────────────┴──────────────┘              │
│              "cgroup은 제약 조건일 뿐"              │
└─────────────────────────────────────────────────────┘
```

## 2. 메모리 회수 트리거 시나리오

### 시나리오 1: 전역 메모리 부족 (호스트 메모리 부족)
```bash
# 호스트 메모리: 16GB
# 사용 중: 15GB
# 여유: 1GB ← watermark 이하!

# 커널의 동작:
# → kswapd 깨어남
# → 전체 시스템에서 페이지 회수
# → cgroup 상관없이 모든 프로세스가 대상
```

```c
// 전역 메모리 회수 (커널 코드 단순화)

void kswapd_main_loop(void) {
    while (1) {
        // 1. 전체 시스템 메모리 확인
        if (global_memory_below_watermark()) {
            
            // 2. 시스템 전체에서 회수
            //    cgroup 경계 무시!
            reclaim_from_all_zones();
            
            // 3. 모든 cgroup의 페이지를 후보로
            scan_all_lru_lists();  // ← 전역 스캔!
        }
        
        sleep_interruptible();
    }
}
```

### 시나리오 2: 특정 cgroup 제한 도달 (컨테이너 제한)
```bash
# 호스트 메모리: 16GB
# 사용 중: 8GB
# 여유: 8GB ← 충분함!

# 하지만 컨테이너 A (limit: 2GB)
# 사용 중: 2GB ← 제한 도달!

# 커널의 동작:
# → 전역 회수는 시작 안 함 (호스트 메모리 충분)
# → cgroup A의 메모리 할당 프로세스가 직접 회수
# → cgroup A의 페이지만 회수
```

```c
// cgroup 제한 회수 (커널 코드 단순화)

void *malloc_with_memcg(size_t size, struct mem_cgroup *memcg) {
    // 1. 이 cgroup 사용량 확인
    if (memcg->usage >= memcg->limit) {
        
        // 2. 이 cgroup 범위에서만 회수
        //    전역 회수 시스템은 작동 안 함!
        try_to_free_mem_cgroup_pages(memcg);
        
        // 3. 여전히 부족하면 이 프로세스만 블로킹
        if (memcg->usage >= memcg->limit) {
            return NULL;  // or OOM
        }
    }
    
    return allocate_pages(size);
}
```

## 3. 실제 확인 - kswapd 모니터링

### 전역 vs cgroup 회수 구분
```bash
#!/bin/bash

echo "=== 메모리 회수 시스템 관찰 ==="

# kswapd 프로세스 확인 (전역 회수 담당)
echo "kswapd 프로세스:"
ps aux | grep kswapd | grep -v grep
echo ""

# kswapd 활동 모니터링
echo "=== 시나리오 1: 호스트 메모리 부족 시 ==="
echo "대량 메모리 할당 (호스트 메모리 압박)..."

# 호스트에서 직접 메모리 할당 (컨테이너 없이)
stress-ng --vm 1 --vm-bytes 10G --timeout 30s &
STRESS_PID=$!

# kswapd CPU 사용률 관찰
echo "kswapd 활동 (5초 간격):"
for i in {1..6}; do
    sleep 5
    # kswapd CPU 사용률
    ps -p $(pgrep kswapd) -o %cpu,stat,cmd 2>/dev/null || echo "kswapd not running"
    
    # 전역 메모리 상태
    free -h | grep Mem
    
    # 전역 회수 통계
    grep -E "pgpgin|pgpgout|pgscan|pgsteal" /proc/vmstat | head -4
    echo "---"
done

wait $STRESS_PID

echo ""
echo "=== 시나리오 2: 컨테이너 제한 도달 시 ==="
echo "컨테이너 생성 (1GB 제한)..."

# 호스트는 여유 있지만 컨테이너만 제한
docker run -d --name cgroup_limit --memory=1g ubuntu sleep infinity

# 컨테이너에서 메모리 할당
docker exec cgroup_limit bash -c '
apt-get update && apt-get install -y stress-ng
stress-ng --vm 1 --vm-bytes 950M --timeout 30s
' &

echo "kswapd 활동 (5초 간격):"
for i in {1..6}; do
    sleep 5
    # kswapd는 활동 안 함! (호스트 메모리 충분)
    ps -p $(pgrep kswapd) -o %cpu,stat,cmd 2>/dev/null
    
    # cgroup 회수 통계
    CONTAINER_ID=$(docker inspect -f '{{.Id}}' cgroup_limit)
    echo "Container memory:"
    cat /sys/fs/cgroup/memory/docker/$CONTAINER_ID/memory.stat | grep -E "pgpgin|pgpgout|pgscan"
    echo "---"
done

docker stop cgroup_limit
docker rm cgroup_limit
```

### 예상 출력
```
=== 시나리오 1: 호스트 메모리 부족 시 ===
대량 메모리 할당 (호스트 메모리 압박)...

kswapd 활동:
%CPU STAT CMD
 85.2 R    [kswapd0]     ← kswapd가 활발히 작동!
              total        used        free
Mem:           16Gi        15Gi       800Mi  ← 호스트 메모리 부족
pgscan_kswapd_normal 1250000  ← 전역 스캔 많음
pgsteal_kswapd_normal 980000  ← 전역 회수 많음
---

=== 시나리오 2: 컨테이너 제한 도달 시 ===
컨테이너 생성 (1GB 제한)...

kswapd 활동:
%CPU STAT CMD
  0.0 S    [kswapd0]     ← kswapd는 대기 상태!
              total        used        free
Mem:           16Gi        8Gi        8Gi   ← 호스트 메모리 충분

Container memory:
pgscan_direct 125000      ← direct reclaim만 발생
                            (프로세스가 직접 회수)
pgsteal_direct 98000
pgscan_kswapd 0           ← kswapd 활동 없음!
```

## 4. Direct Reclaim vs Background Reclaim

### 두 가지 회수 메커니즘
```
┌──────────────────────────────────────────┐
│  1. Background Reclaim (kswapd)          │
│     - 전역 메모리 부족 시                 │
│     - 백그라운드 프로세스                 │
│     - 모든 cgroup에서 회수               │
│     - 프로세스 블로킹 없음                │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│  2. Direct Reclaim                       │
│     - cgroup 제한 도달 시                │
│     - 할당 요청한 프로세스가 직접         │
│     - 해당 cgroup에서만 회수             │
│     - 프로세스 블로킹됨!                  │
└──────────────────────────────────────────┘
```

### 실제 커널 흐름
```c
// 메모리 할당 요청

void *allocate_pages(size_t nr_pages) {
    struct mem_cgroup *memcg = current->memcg;
    
    // 1. cgroup 제한 확인
    if (memcg && memcg_over_limit(memcg)) {
        // ← Direct Reclaim (이 프로세스가 블로킹됨)
        try_to_free_mem_cgroup_pages(memcg);
        
        if (still_over_limit(memcg)) {
            return NULL;  // 실패
        }
    }
    
    // 2. 전역 메모리 확인
    if (global_memory_low()) {
        // ← Direct Reclaim (전역)
        //    또는 kswapd 깨우기
        if (very_low) {
            __perform_reclaim();  // 직접 회수
        } else {
            wakeup_kswapd();      // kswapd 깨우기
        }
    }
    
    // 3. 실제 할당
    return get_free_pages(nr_pages);
}
```

## 5. 전역 스캔 시 cgroup 고려

### kswapd의 전역 스캔
```c
// kswapd가 전역 회수할 때

unsigned long kswapd_shrink_zone(struct zone *zone) {
    unsigned long nr_reclaimed = 0;
    
    // 전역 LRU 리스트 순회
    for_each_lruvec(lruvec, zone) {
        
        // 각 lruvec은 cgroup에 속할 수 있음
        struct mem_cgroup *memcg = lruvec_memcg(lruvec);
        
        // cgroup의 swappiness 고려
        unsigned long swappiness = memcg ? 
            memcg->swappiness : vm_swappiness;
        
        // 회수 수행
        nr_reclaimed += shrink_lruvec(lruvec, swappiness);
    }
    
    return nr_reclaimed;
}
```

### 실제 동작 예시
```bash
# 호스트 메모리 부족 시

# kswapd가 모든 페이지를 후보로 스캔:
# - cgroup A의 페이지들 (swappiness=10 적용)
# - cgroup B의 페이지들 (swappiness=60 적용)  
# - cgroup에 속하지 않은 페이지들 (vm.swappiness=60 적용)
#
# → 하나의 스캔 과정에서 cgroup별 설정을 각각 적용
```

## 6. 실제 확인 - vmstat 분석

### 전역 회수 통계
```bash
# 전역 회수 이벤트 (모든 프로세스 대상)
cat /proc/vmstat | grep -E "pgscan_kswapd|pgsteal_kswapd"

# pgscan_kswapd_normal: kswapd가 스캔한 페이지 수
# pgsteal_kswapd_normal: kswapd가 회수한 페이지 수

# cgroup별 회수 통계
CONTAINER_ID=$(docker inspect -f '{{.Id}}' mycontainer)
cat /sys/fs/cgroup/memory/docker/$CONTAINER_ID/memory.stat | grep pgscan

# pgscan_direct: 이 cgroup에서 direct reclaim으로 스캔
# pgscan_kswapd: 이 cgroup에서 kswapd로 스캔 (전역 회수 시)
```

### 비교 실험
```python
#!/usr/bin/env python3
import subprocess
import time

def get_global_stats():
    """전역 vmstat"""
    result = subprocess.run(['cat', '/proc/vmstat'], 
                          capture_output=True, text=True)
    stats = {}
    for line in result.stdout.split('\n'):
        if line:
            key, value = line.split()
            stats[key] = int(value)
    return stats

def get_cgroup_stats(container_name):
    """컨테이너 cgroup 통계"""
    cmd = f"docker inspect -f '{{{{.Id}}}}' {container_name}"
    cid = subprocess.run(cmd, shell=True, 
                        capture_output=True, text=True).stdout.strip()
    
    path = f"/sys/fs/cgroup/memory/docker/{cid}/memory.stat"
    try:
        with open(path) as f:
            stats = {}
            for line in f:
                key, value = line.split()
                stats[key] = int(value)
            return stats
    except FileNotFoundError:
        return None

def test_global_pressure():
    """전역 메모리 압박 테스트"""
    print("=== 전역 메모리 압박 테스트 ===")
    
    # 초기 통계
    before_global = get_global_stats()
    
    # 호스트에서 직접 메모리 할당
    print("호스트 메모리 할당 중...")
    proc = subprocess.Popen(
        ['stress-ng', '--vm', '1', '--vm-bytes', '8G', '--timeout', '20s'],
        stdout=subprocess.DEVNULL,
        stderr=subprocess.DEVNULL
    )
    
    time.sleep(15)
    
    # 최종 통계
    after_global = get_global_stats()
    
    # kswapd 활동
    kswapd_scan = after_global.get('pgscan_kswapd_normal', 0) - \
                  before_global.get('pgscan_kswapd_normal', 0)
    
    print(f"kswapd 스캔: {kswapd_scan} pages")
    print(f"→ 전역 회수 시스템 작동!\n")
    
    proc.wait()

def test_cgroup_pressure():
    """cgroup 메모리 압박 테스트"""
    print("=== cgroup 메모리 압박 테스트 ===")
    
    # 컨테이너 시작
    subprocess.run(['docker', 'run', '-d', '--name', 'cgtest',
                   '--memory', '1g', 'ubuntu', 'sleep', 'infinity'],
                  stdout=subprocess.DEVNULL)
    
    time.sleep(2)
    
    # 초기 통계
    before_global = get_global_stats()
    before_cgroup = get_cgroup_stats('cgtest')
    
    # 컨테이너에서 메모리 할당
    print("컨테이너 메모리 할당 중...")
    subprocess.run(['docker', 'exec', 'cgtest', 'bash', '-c',
                   'apt-get update && apt-get install -y stress-ng && '
                   'stress-ng --vm 1 --vm-bytes 950M --timeout 20s'],
                  stdout=subprocess.DEVNULL,
                  stderr=subprocess.DEVNULL)
    
    time.sleep(2)
    
    # 최종 통계
    after_global = get_global_stats()
    after_cgroup = get_cgroup_stats('cgtest')
    
    # kswapd 활동 (전역)
    kswapd_scan = after_global.get('pgscan_kswapd_normal', 0) - \
                  before_global.get('pgscan_kswapd_normal', 0)
    
    # direct reclaim (cgroup)
    if before_cgroup and after_cgroup:
        direct_scan = after_cgroup.get('pgscan_direct', 0) - \
                     before_cgroup.get('pgscan_direct', 0)
        
        print(f"kswapd 스캔 (전역): {kswapd_scan} pages")
        print(f"direct 스캔 (cgroup): {direct_scan} pages")
        print(f"→ cgroup 자체 회수만 작동!\n")
    
    # 정리
    subprocess.run(['docker', 'stop', 'cgtest'], 
                  stdout=subprocess.DEVNULL)
    subprocess.run(['docker', 'rm', 'cgtest'],
                  stdout=subprocess.DEVNULL)

if __name__ == '__main__':
    test_global_pressure()
    time.sleep(5)
    test_cgroup_pressure()
```

### 예상 출력
```
=== 전역 메모리 압박 테스트 ===
호스트 메모리 할당 중...
kswapd 스캔: 2048576 pages
→ 전역 회수 시스템 작동!

=== cgroup 메모리 압박 테스트 ===
컨테이너 메모리 할당 중...
kswapd 스캔 (전역): 0 pages       ← kswapd 작동 안 함!
direct 스캔 (cgroup): 245760 pages ← direct reclaim만 작동
→ cgroup 자체 회수만 작동!
```

## 7. 정리 다이어그램

### 실제 메모리 회수 구조
```
                    메모리 할당 요청
                          ↓
            ┌─────────────┴─────────────┐
            │                           │
    cgroup 제한 확인          전역 메모리 확인
            │                           │
    ┌───────┴───────┐          ┌────────┴────────┐
    │ 제한 초과?    │          │ 메모리 부족?    │
    └───┬───────┬───┘          └────┬────────┬───┘
       Yes     No                  Yes      No
        │       │                   │        │
        ↓       │                   ↓        │
  Direct Reclaim│             kswapd 깨우기  │
  (이 cgroup만) │             (전역 회수)    │
        │       │                   │        │
        └───┬───┴───────────────────┴────────┘
            │
            ↓
        메모리 할당
```

### cgroup별 독립 vs 전역 통합
```
❌ 잘못된 이해:
┌──────────┐  ┌──────────┐  ┌──────────┐
│ cgroup A │  │ cgroup B │  │ cgroup C │
│          │  │          │  │          │
│ kswapd A │  │ kswapd B │  │ kswapd C │ ← 각각 회수 시스템?
│ reclaim A│  │ reclaim B│  │ reclaim C│
└──────────┘  └──────────┘  └──────────┘

✅ 올바른 이해:
┌─────────────────────────────────────────┐
│       하나의 전역 회수 시스템            │
│       - kswapd (하나)                   │
│       - direct reclaim (프로세스별)     │
└─────────┬───────────────────────────────┘
          │
          ├→ cgroup A 페이지 (swappiness=10)
          ├→ cgroup B 페이지 (swappiness=60)
          └→ cgroup C 페이지 (swappiness=30)
             ↑
        회수 시 각 설정 적용
```

## 8. 핵심 정리

### 질문에 대한 답

> "커널은 cgroup 단위로 메모리 회수 시스템이 동작하나?"

**아닙니다!**

**실제 동작:**

1. **전역 회수 시스템 (하나)**
   - kswapd: 하나의 백그라운드 프로세스
   - 호스트 메모리 부족 시 작동
   - 모든 cgroup의 페이지를 대상으로 스캔

2. **Direct Reclaim (프로세스별)**
   - cgroup 제한 도달 시만 작동
   - 할당 요청한 프로세스가 직접 수행
   - 해당 cgroup 범위에서만 회수

3. **cgroup의 역할**
   - 독립적인 회수 시스템이 아님
   - 회수 시 적용할 정책만 제공 (swappiness)
   - 회수 범위를 제한 (이 cgroup의 페이지만)

### 비유
```
cgroup은 "구역"이고, 커널은 "청소부"

❌ 틀린 이해:
각 구역마다 청소부가 따로 있다

✅ 올바른 이해:
청소부는 하나인데:
- 전체 건물이 더러우면 → 모든 구역 청소
- 특정 구역만 더러우면 → 그 구역만 청소
- 청소 방식은 구역별 규칙(swappiness)에 따름
```

**결론: 메모리 회수 시스템은 전역으로 하나만 존재하며, cgroup은 회수 범위와 정책을 제한하는 제약조건일 뿐입니다!**


## PageCache

### Anonymous Pages vs File-backed Pages

```txt
┌─────────────────────────────────────────────────────┐
│              프로세스 메모리                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────────────────────────────────────-─┐      │
│  │  Anonymous Pages (익명 페이지)            │      │
│  │  - malloc(), new로 할당한 메모리          │      │
│  │  - 스택, 힙                               │      │
│  │  - 디스크 백업 없음                       │      │
│  │  → Swap으로 내보냄 ✅                    │      │
│  └─────────────────────────────────────────-─┘      │
│                                                     │
│  ┌────────────────────────────────────────-──┐      │
│  │  File-backed Pages (파일 백업 페이지)     │      │
│  │  - mmap()으로 매핑한 파일                 │      │
│  │  - 실행 파일 코드 영역                    │      │
│  │  - 공유 라이브러리                        │      │
│  │  - Page Cache                             │      │
│  │  - 디스크 백업 있음                       │      │
│  │  → 그냥 버림 (evict) ❌ Swap 안 씀       │      │
│  └────────────────────────────────────────-──┘      │
└─────────────────────────────────────────────────────┘
```