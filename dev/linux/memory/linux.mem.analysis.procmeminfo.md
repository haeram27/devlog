# linux memory 사용량 조회

```bash
# cat /proc/meminfo
MemTotal:       65683408 kB
MemFree:         2575020 kB
MemAvailable:   37871944 kB
Buffers:               4 kB
Cached:         36131228 kB
SwapCached:        12808 kB
Active:         21468144 kB
Inactive:       33136800 kB
Active(anon):    1207136 kB
Inactive(anon): 19408188 kB
Active(file):   20261008 kB
Inactive(file): 13728612 kB
Unevictable:     4715228 kB
Mlocked:         4715228 kB
SwapTotal:      25165816 kB
SwapFree:       24707320 kB
Dirty:            384436 kB
Writeback:           100 kB
AnonPages:      22725636 kB
Mapped:          2474172 kB
Shmem:           1974348 kB
KReclaimable:    1988604 kB
Slab:            2676932 kB
SReclaimable:    1988604 kB
SUnreclaim:       688328 kB
KernelStack:       62240 kB
PageTables:       145496 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    58007520 kB
Committed_AS:   46935396 kB
VmallocTotal:   34359738367 kB
VmallocUsed:           0 kB
VmallocChunk:          0 kB
Percpu:            21568 kB
HardwareCorrupted:     0 kB
AnonHugePages:  10928128 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
FileHugePages:         0 kB
FilePmdMapped:         0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
DirectMap4k:    18058684 kB
DirectMap2M:    49049600 kB
DirectMap1G:     2097152 kB
```

## /proc/meminfo 차이 vs /sys/fs/cgroup/memory/memory.stat

|항목|/proc/meminfo|/sys/fs/cgroup/memory/memory.stat|
|---|---|---|
|범위|시스템 전체|특정 cgroup (컨테이너)|
|단위|kB|Byte|
|관점|호스트 OS|컨테이너 내부|
|용도|전체 시스템 메모리 현황|컨테이너별 메모리 사용량|
|제한|물리 메모리 전체|cgroup limit 기준|

- /proc/meminfo
  - host OS의 메모리 사용량을 표시하므로 `host에서 조회`해야함
  - 표기 단위는 `kB`
  - 리눅스 커널이 관리하는 전체 물리적 메모리(RAM)의 총량, 사용 가능한 메모리, 버퍼, 캐시된 메모리 등 시스템 전반의 메모리 통계를 포함
  - 컨테이너 내부에서 이 파일을 읽어도 항상 호스트 시스템 전체의 메모리 정보를 표시하므로, 컨테이너에 할당된 메모리 제한을 확인하는 데는 유용하지 않음
- /sys/fs/cgroup/memory/memory.stat
  - cgroup(vm, 컨테이너)의 메모리 사용량을 표시하므로 `특정 cgroup 내에서 조회`해야함
  - 표기 단위는 `byte`
  - 리눅스의 cgroup 기능은 프로세스들이 사용할 수 있는 시스템 자원을 격리하고 제한하는 데 사용
  - 이 파일은 특정 cgroup 내에서 메모리가 어떻게 사용되고 있는지에 대한 상세한 통계(예: 활성/비활성 익명 메모리, 파일 캐시 등)를 제공
  - 컨테이너 환경에서는 해당 컨테이너(cgroup)에 적용된 실제 메모리 사용량과 제한을 파악하기 위해 이 파일의 정보를 활용

## `/proc/meminfo` 주요 항목

- MemTotal: 전체 메모리
- MemFree: 할당 되지 않은 메모리
- Cached: 디스크 IO 캐시
- Active(file): 최근 접근한 clean pages (Cached)
- InActive(file): 회수 대상인 clean pages (Cached)
- Dirty: 디스크에 아직 쓰이지 않은 page cache
- SwapTotal:  총 스왑 공간
- SwapFree: 사용 가능한 스왑 공간

## `/proc/meminfo` 항목 설명

### **기본 메모리 정보**

```bash
MemTotal:       16384000 kB   # 시스템의 총 물리 메모리(RAM) 용량
MemFree:         2048000 kB   # 완전히 사용되지 않은 메모리 (application에서 즉시 사용가능한 메모리)
MemAvailable:    6656000 kB   # 사용할 수 가능한 메모리 추정치 (가장 중요!)
```

**MemFree vs MemAvailable:**

- MemFree는 application에서 요청 즉시 사용가능한 메모리
- MemAvailable는 MemFree에 더하여 Reclaimable 메모리를 포함
- Reclaimable Memory는 OS에서 회수 가능한 메모리를 의미하며, application에서 메모리 사용 요청시 즉시 사용할 수 없고 OS가 회수 과정을 거쳐야함(시간이 오래 걸림)
- MemAvailable이 Reclaimable(Cache 등)만 남아있다면 application의 메모리 할당 요청에 대해 시스템은 설정에 따라 다르지만 대게 Reclaimable 회수보다 `oom killer`를 우선 호출 함

```text
MemAvailable = MemFree + Reclaimable Memory
```

Reclaimable Memory:

- Clean Page Cache (파일 캐시)
- Reclaimable Slab (커널 캐시)
- 기타 축출 가능한 메모리

MemAvailable = MemFree + Inactive(file) + Active(file) - Low watermark

**메모리 사용 관련 커널 파라미터:**

- vm.panic_on_oom: 이 값을 1로 설정하면 OOM killer를 실행하는 대신 시스템 전체 커널 패닉(재부팅)을 유발함. 이는 OOM killer가 특정 프로세스를 죽이는 것보다 시스템의 즉각적인 재시작을 선호할 때 사용됨
- vm.overcommit_memory 및 vm.overcommit_ratio:
  - vm.overcommit_memory를 2로 설정하면 커널은 설정된 비율(vm.overcommit_ratio)의 메모리만 할당하며, 이보다 많은 메모리 요청 시 즉시 할당 실패(OOM 아님)를 반환할 수 있음. 이는 OOM killer가 호출될 가능성을 줄이지만, 메모리 할당 실패로 인해 프로그램이 예기치 않게 종료될 수 있음
- vm.swappiness: 스왑 사용 빈도를 조절하는 값입니다 (기본값 60, 범위 0-100). 이 값을 100으로 높이면 커널이 스왑을 더 적극적으로 사용하여 물리 메모리를 더 일찍 확보하려고 시도하지만, 이는 메모리 회수 동작을 더 빠르게 할 뿐 OOM killer 우선순위를 높이지는 않음
- vm.min_free_kbytes: 커널이 항상 유지하려는 최소한의 여유 물리 메모리 양을 설정합니다. 이 값을 높게 설정하면 메모리 부족 상황을 더 일찍 감지하고 메모리 회수를 더 빠르게 시작할 수 있지만, OOM killer 호출 시점을 직접 앞당기지는 않음

### **버퍼와 캐시**

```bash
Buffers:  파일 시스템 메타데이터를 위한 버퍼 캐시
Cached:   파일 내용을 위한 페이지 캐시 (디스크 I/O 캐싱) (중요!)
SwapCached:  스왑에서 다시 메모리로 불러왔지만 스왑에도 남아있는 메모리
```

`Cached` 항목은 어플리케이션이 파일에 대해 read/write할 때, 디스크에 쓰기 전에 메모리상에 데이터를 상주 시킨 것을 의미, `한번 참조된 데이터는 다시 참조될 가능 성이 높음`의 캐싱원칙에 따라 OS가 메모리상에 저장 한 것

***`Cached`는 Memory
대용량 파일을 

**의미:**

- Linux는 남는 메모리를 캐시로 사용 (성능 향상)
- 메모리 부족 시 자동으로 해제되므로 "사용 가능한" 메모리로 간주

### **활성/비활성 메모리**

```bash
Active:          최근에 사용된 메모리 (회수 우선순위 낮음)
Inactive:        최근에 사용되지 않은 메모리 (회수 우선순위 높음)

Active(anon):    활성 익명 메모리 (프로세스 힙, 스택 등)
Inactive(anon):  비활성 익명 메모리
Active(file):    활성 파일 캐시
Inactive(file):  비활성 파일 캐시
```

**분류:**

- **anon (anonymous)**: 파일과 연결되지 않은 메모리 (프로세스 메모리)
- **file**: 파일과 연결된 메모리 (페이지 캐시)

### **스왑 메모리**

```bash
SwapTotal:  총 스왑 공간
SwapFree:   사용 가능한 스왑 공간
```

**계산:**

```bash
사용 중인 스왑 = SwapTotal - SwapFree
```

### **더티 메모리**

```bash
Dirty:      디스크에 아직 기록되지 않은 수정된 페이지
Writeback:  현재 디스크에 쓰고 있는 중인 페이지
```

**중요성:**

- Dirty가 높으면 → 디스크 I/O 대기 중
- 시스템 크래시 시 Dirty 데이터 손실 가능

### **매핑된 메모리**

```bash
Mapped:   mmap()으로 매핑된 파일 (공유 라이브러리 등)
Shmem:    공유 메모리 (tmpfs, IPC 등)
```

### **슬랩 (커널 객체 캐시)**

```bash
Slab:          커널이 사용하는 캐시 메모리 총합
SReclaimable:  회수 가능한 슬랩
SUnreclaim:    회수 불가능한 슬랩
```

**의미:**

- 커널이 자주 사용하는 객체를 캐싱
- SReclaimable은 메모리 부족 시 회수 가능

### **커널 메모리**

```bash
KernelStack:   커널 스택 메모리
PageTables:    페이지 테이블 메모리 (가상→물리 주소 매핑)
VmallocTotal:  가상 메모리 총 할당 가능량 (32비트: 제한적, 64비트: 거의 무한)
VmallocUsed:   사용 중인 vmalloc 메모리
VmallocChunk:  가장 큰 연속된 vmalloc 청크
```

### **투명 대형 페이지 (THP)**

```bash
AnonHugePages:   익명 대형 페이지 (2MB 페이지)
ShmemHugePages:  공유 메모리 대형 페이지
ShmemPmdMapped:  PMD로 매핑된 공유 메모리
```

### **파일 관련**

```bash
FilePages:      파일로 백업된 페이지 캐시
FilePmdMapped:  PMD로 매핑된 파일 페이지
```

### **락 메모리**

```bash
Unevictable:  회수 불가능한 메모리 (mlock 등)
Mlocked:      mlock()으로 잠긴 메모리
```

### **하드웨어 손상**

```bash
HardwareCorrupted:  하드웨어 오류로 사용 불가능한 메모리
```

### **커밋 메모리**

```bash
CommitLimit:  시스템이 할당할 수 있는 최대 메모리 (물리 메모리 + 스왑)
Committed_AS: 현재 할당(커밋)된 메모리 총량
```

**의미:**

- Committed_AS > CommitLimit 이면 OOM(Out of Memory) 위험

### **Percpu 메모리**

```bash
Percpu:   CPU별 할당된 메모리
```

### **CMA (Contiguous Memory Allocator)**

```bash
CmaTotal: 연속 메모리 할당자 총량
CmaFree:  사용 가능한 CMA 메모리
```

## 실용적인 항목 조합

### **실제 메모리 부족 여부 확인**

```bash
# 사용 가능한 메모리 확인 (가장 중요)
cat /proc/meminfo | grep MemAvailable

# 메모리 사용률 계산
awk '/MemTotal/ {total=$2} /MemAvailable/ {avail=$2} END {print "사용률: " (100 - avail/total*100) "%"}' /proc/meminfo
```

### **스왑 사용 확인**

```bash
# 스왑 사용량
cat /proc/meminfo | grep -E 'SwapTotal|SwapFree'

# 스왑 사용률
awk '/SwapTotal/ {total=$2} /SwapFree/ {free=$2} END {if (total > 0) print "스왑 사용률: " ((total-free)/total*100) "%"; else print "스왑 없음"}' /proc/meminfo
```

### **캐시 메모리 확인**

```bash
# 버퍼 + 캐시
cat /proc/meminfo | grep -E 'Buffers|Cached' | grep -v SwapCached
```

### **커널 메모리 사용량**

```bash
# 커널이 사용하는 메모리
cat /proc/meminfo | grep -E 'Slab|KernelStack|PageTables'
```

## 항목별 요약표

| 항목 | 의미 | 회수 가능 |
|------|------|----------|
| **MemAvailable** | 실제 사용 가능 메모리 | N/A |
| **MemFree** | 완전히 빈 메모리 | N/A |
| **Buffers** | 파일 시스템 버퍼 | ✅ |
| **Cached** | 파일 페이지 캐시 | ✅ |
| **Active** | 최근 사용 메모리 | 부분적 |
| **Inactive** | 미사용 메모리 | ✅ 우선 |
| **SwapTotal/Free** | 스왑 공간 | N/A |
| **Dirty** | 디스크 미기록 | ❌ |
| **Slab** | 커널 캐시 | 부분적 |
| **Committed_AS** | 할당된 메모리 | N/A |

## 전체 조회 스크립트

```bash
#!/bin/bash
# meminfo_analysis.sh

echo "=== memoinfo analysis ==="

# 총 메모리
total=$(awk '/MemTotal/ {print $2}' /proc/meminfo)
available=$(awk '/MemAvailable/ {print $2}' /proc/meminfo)
free=$(awk '/MemFree/ {print $2}' /proc/meminfo)
cached=$(awk '/^Cached/ {print $2}' /proc/meminfo)
buffers=$(awk '/Buffers/ {print $2}' /proc/meminfo)

echo "total: $((total/1024)) MB"
echo "available: $((available/1024)) MB"
echo "free: $((free/1024)) MB"
echo "cache: $((cached/1024)) MB"
echo "buffer: $((buffers/1024)) MB"

# 사용률
used=$((total - available))
usage=$((used * 100 / total))
echo "memmory usage: ${usage}%"

# 스왑
swap_total=$(awk '/SwapTotal/ {print $2}' /proc/meminfo)
swap_free=$(awk '/SwapFree/ {print $2}' /proc/meminfo)

if [ $swap_total -gt 0 ]; then
    swap_used=$((swap_total - swap_free))
    swap_usage=$((swap_used * 100 / swap_total))
    echo "swap usage: ${swap_usage}%"
else
    echo "no free swap"
fi
```
