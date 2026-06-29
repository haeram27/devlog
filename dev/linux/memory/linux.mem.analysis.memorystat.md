# linux memory 사용량 조회

```bash
# docker exec my-app cat /sys/fs/cgroup/memory/memory.stat
cache 9102508032
rss 3039137792
rss_huge 1254096896
shmem 0
mapped_file 356352
dirty 0
writeback 0
swap 4096
pgpgin 6216562
pgpgout 3557864
pgfault 3587940
pgmajfault 18
inactive_anon 3039084544
active_anon 20480
inactive_file 1579216896
active_file 7523291136
unevictable 0
hierarchical_memory_limit 9223372036854771712
hierarchical_memsw_limit 9223372036854771712
total_cache 9102508032
total_rss 3039137792
total_rss_huge 1254096896
total_shmem 0
total_mapped_file 356352
total_dirty 0
total_writeback 0
total_swap 4096
total_pgpgin 6216562
total_pgpgout 3557864
total_pgfault 3587940
total_pgmajfault 18
total_inactive_anon 3039084544
total_active_anon 20480
total_inactive_file 1579216896
total_active_file 7523291136
total_unevictable 0
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

## 주요 항목

## 항목 설명

- **cache**
  - Page cache 메모리 (파일 시스템 캐시). 파일 읽기/쓰기 시 커널이 캐싱하는 메모리입니다. 최근 Docker 이슈에서 경험하신 것처럼, 파일 작업 후 여기에 많이 쌓일 수 있습니다.

- **rss**
  - Resident Set Size. 프로세스가 실제로 사용 중인 익명 메모리(anonymous memory)입니다. 힙, 스택 등이 포함됩니다.

- **rss_huge**
  - Huge pages로 할당된 RSS 메모리

- **mapped_file**
  - 메모리 맵된 파일의 크기 (mmap으로 매핑된 파일)

- **dirty**
  - 디스크에 아직 쓰이지 않은 dirty page cache

- **writeback**
  - 현재 디스크로 쓰이고 있는 중인 페이지

- **swap**
  - 스왑 영역으로 이동된 메모리 양

- **inactive_anon / active_anon**
  - 비활성/활성 익명 메모리. LRU 리스트에서 최근 사용 여부에 따라 구분됩니다.

- **inactive_file / active_file**
  - 비활성/활성 파일 캐시. 메모리 부족 시 inactive_file이 먼저 회수됩니다.

### 페이지 폴트 카운터

- **pgfault**
  - Minor page fault 횟수 (메모리에는 있지만 페이지 테이블에 매핑되지 않은 경우)

- **pgmajfault**
  - Major page fault 횟수 (디스크에서 읽어와야 하는 경우)

- **pgpgin / pgpgout**
  - 페이지 단위로 읽어온/내보낸 횟수

### 기타

- **unevictable**
  - 회수 불가능한 메모리 (mlock으로 잠긴 메모리 등)

- **hierarchical_memory_limit**
  - 이 cgroup과 상위 cgroup들의 메모리 제한 중 가장 작은 값

- **total_***
  - 하위 cgroup들을 포함한 계층적 합계

