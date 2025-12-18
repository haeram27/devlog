# 리눅스 메모리 회수 로직

- 리눅스는 `kswapd` 와 `Direct Reclaim` 다음의 방법으로 메모리 관리를 수행
  - `kswapd`
    - 커널에 의해 운용되는 메모리 회수용 백그라운드 데몬(`ps`로 확인 가능)
    - `kswapd`는 시스템의 여유 메모리가 low 임계값에 도달하면 커널이 `kswapd` 스레드를 깨웁니다. 이 스레드는 여유 메모리가 다시 high 워터마크가 될 때까지 백그라운드에서 페이지를 정리하여 가용 메모리를 확보합니다. 사용자 프로세스 들은 이 과정 중에 멈추지 않고 계속 실행됩니다.
  - `Direct Reclaim`
    - 메모리 요청 프로세스가 직접 회수하는 방식
    - 프로세스가 메모리를 요청(malloc)했을 때 FREE 메모리가 min 임계값 이하이면, 커널은 메모리를 할당받으려던 프로세스의 할당 작업을 멈추고 메모리를 회수를 시작합니다. 이 과정이 끝날 때까지 해당 프로세스는 대기 상태(Blocking)가 되므로 애플리케이션의 성능이나 반응 속도가 급격히 떨어집니다.
    - 커널이 메모리 회수 시도에도 충분한 메모리를 확보 하지 못하면 `oom(out-of-memory) killer` 과정을 실행하여 우선 순위가 낮은 프로세스를 강제로 종료합니다.
  - `OOM Killer` 호출
    - Direct Reclaim을 시도했음에도 불구하고, 즉 커널이 가능한 모든 수단(페이지 회수, 압축 등)을 동원했음에도 필요한 가용 메모리를 확보하지 못했을 때 최후의 수단으로 OOM Killer를 호출
    - Direct Reclaim으로도 해결되지 않는 심각한 메모리 고갈 상태"일 때 트리거되는 메커니즘
    - /var/log/messages나 dmesg에서 OOM Killer 발생 로그 확인 가능

- 시스템 성능을 유지하기 위해서는 `kswapd`가 제때 동작하여 `Direct Reclaim`이 발생하지 않도록 유지하는 것이 중요
- 어플리케이션에서 주로 사용되는 메모리와 회수 방식은 다음과 같다
  - Anonymouse Cache : 힙/스택 등 프로세스가 실행 중에 동적으로 할당 받는 메모리
    - 회수 시점에 swap 영역으로 이동 가능
  - File Backed Cache : Disk I/O 성능 향샹용 캐시
    - File read/write시 `Page Cache` 방식에 의해 점유되므로 `Page Cache`라고 통칭 되기도 하지만 `Page cache`는 방식을 말하며 `File-backed Cache`이외에도 다른 목적의 Cache 또한 포함한다.
    - 회수 시점에 swap 되지 않으며(swap의 대상이 아님) Discard 됨
    - write 대기 상태의 page 영역인 `Dirty Page`는 회수 대상 아님
    - read only 목적의 page 영역은 회수 대상

## `kswapd` 와 `Direct Reclaim` 주요 차이점 비교

| 구분 | kswapd (Background Reclaim) | Direct Reclaim (Foreground Reclaim) |
| --- | --- | --- |
| 작동 방식 | 비동기 (Asynchronous) | 동기 (Synchronous) |
| 회수 트리거 주체 | 커널 스레드 (kswapd) | 메모리를 요청한 프로세스 자신 |
| 발생 시점 | 여유 메모리가 low 워터마크 이하일 때 | 여유 메모리가 min 워터마크 이하일 때 |
| 영향성 | 시스템 백그라운드에서 조용히 수행 (비차단) | 메모리 할당 중단 및 회수 완료까지 대기 (차단) |
| 성능 저하 | 거의 없음 (미리 여유 공간 확보) | 매우 큼 (프로세스 지연 발생) |

## `kswapd`

### 워터마크 기준

- 리눅스 커널은 메모리 관리를 위해 세 가지 주요 page 워터마크를 사용하며 값의 단위는 `page`임
  - High = 약 Min * 1.5 (또는 Min + (Min / 2))
    - `kswapd`에 의한 메모리 회수 중단 지점
    - FREE 메모리가 이 수치에 도달하면 `kswapd`가 메모리 회수를 중단
  - Low = 약 Min * 1.25 (또는 Min + (Min / 4))
    - `kswapd`에 의한 메모리 회수 시작 지점
    - FREE 메모리가 이 수치보다 낮아지면 커널의 배경 프로세스인 `kswapd`가 깨어나 메모리 회수를 시작
    - Min 값을 기준으로 distance를 연산
  - Min = `vm.min_free_kbytes` 커널 변수에 의해 결정된 값
    - `kswapd`만으로는 부족하여 `Direct Reclaim`이 강제로 발생하는 임계점

- `kswapd`가 참조하는 임계값 (min, low, high)는 `/proc/zoneinfo` 에서 확인

### `/proc/zoneinfo`

```bash
Node 0, zone   Normal
  pages free     1446382
        min      15451
        low      22904
        high     30357
        spanned  7602176
        present  7602176
        managed  7455511
        protection: (0, 0, 0, 0, 0)
      nr_free_pages 1446382
      nr_zone_inactive_anon 1517750
      nr_zone_active_anon 24440
      nr_zone_inactive_file 2392079
      nr_zone_active_file 1786313
      nr_zone_unevictable 0
      nr_zone_write_pending 0
```

- `Normal` 존은 대부분의 애플리케이션과 페이지 캐시가 할당되는 주 메모리 영역
- 값의 단위는 `page`
- page이 실제 사이즈는 다음 방식으로 확인
  - `getconf PAGESIZE`
    - 4096(byte)
  - `grep -i pagesize /proc/self/smaps`
    - KernelPageSize: 4 kB
- 각 항목 설명
  - `pages free` : 현재 이 존에 남아있는 순수 여유 메모리 페이지 수(1,446,382 page * 4kB = 5,785,528 kB).
  - `min` : 메모리 회수를 위한 최소 방어선. 여유 메모리가 이 값 아래로 떨어지면, 프로세스가 메모리를 요청했을 때 메모리 회수(Direct Reclaim)이 수행됨.
  - `low` : 백그라운드 회수 시작점. 여유 메모리가 이 값 아래로 떨어지면, 커널의 kswapd 프로세스가 깨어나 high 워터마크까지 메모리를 확보하기 시작. 즉, 페이지 캐시 회수가 시작되는 시점.
  - `high` : 백그라운드 회수 목표점, kswapd는 여유 메모리가 이 값에 도달하면 메모리 회수를 멈춤.
  - `managed` : 커널의 메모리 관리자가 실제로 관리하는 전체 페이지 수.
  - `nr_zone_inactive_anon` : 비활성 익명 페이지(Inactive Anonymous). 최근 사용되지 않은 프로세스 메모리(힙, 스택 등)로, 메모리 회수 시 Swap 대상 1순위.
  - `nr_zone_active_anon` : 활성 익명 페이지(Active Anonymous). 최근 사용된 프로세스 메모리.
  - `nr_zone_inactive_file` : 비활성 파일 페이지(Inactive File). 최근 사용되지 않은 페이지 캐시로, 메모리 회수 시 Drop 대상 1순위.
  - `nr_zone_active_file` : 활성 파일 페이지(Active File). 최근 사용된 페이지 캐시.
  - `nr_zone_unevictable` : mlock() 등으로 잠겨 있어 회수 불가능한 페이지.
  - `nr_zone_write_pending` : 디스크에 기록되기를 기다리는 Dirty 페이지.
