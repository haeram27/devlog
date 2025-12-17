# 리눅스 메모리 회수 로직

리눅스의 메모리 회수
시스템 성능을 유지하기 위해서는 kswapd가 제때 동작하여 Direct Reclaim이 발생하지 않도록 유지하는 것이 중요

## kswapd 와 `Direct Reclaim`

1. 주요 차이점 비교

| 구분 | kswapd (Background Reclaim) | Direct Reclaim (Foreground Reclaim) |
|---|---|---|
| 작동 방식 | 비동기 (Asynchronous) | 동기 (Synchronous) |
| 주체 | 커널 스레드 (kswapd) | 메모리를 요청한 프로세스 자신 |
| 발생 시점 | 여유 메모리가 low 워터마크 이하일 때 | 여유 메모리가 min 워터마크 이하일 때 |
| 영향성 | 시스템 백그라운드에서 조용히 수행 (비차단) | 메모리 할당 중단 및 회수 완료까지 대기 (차단) |
| 성능 저하 |거의 없음 (미리 여유 공간 확보)|매우 큼 (프로세스 지연 발생) |

2. 세부 설명

- kswapd (백그라운드 회수): 시스템의 여유 메모리가 low 임계값에 도달하면 커널이 kswapd 스레드를 깨웁니다. 이 스레드는 여유 메모리가 다시 high 워터마크가 될 때까지 백그라운드에서 페이지를 정리하여 가용 메모리를 확보합니다. 사용자 프로세스는 이 과정 중에 멈추지 않고 계속 실행됩니다.
- Direct Reclaim (직접 회수): 프로세스가 메모리를 요청했을 때 free 메모리 부족으로 min 임계값 이하이면, 커널은 메모리를 할당받으려던 프로세스의 할당 작업을 멈추고 메모리를 회수를 시작합니다. 이 과정이 끝날 때까지 해당 프로세스는 대기 상태(Blocking)가 되므로 애플리케이션의 성능이나 반응 속도가 급격히 떨어집니다.

1. 워터마크 기준
리눅스 커널은 메모리 관리를 위해 세 가지 주요 워터마크를 사용합니다:

- High: 메모리가 충분한 상태입니다.
- Low: kswapd가 깨어나 메모리 회수를 시작하는 지점입니다.
- Min: kswapd만으로는 부족하여 Direct Reclaim이 강제로 발생하는 임계점입니다.


kswapd가 참조하는 임계값 (min, low, high)는 /proc/zoneinfo 에서 확인 가능

`/proc/zoneinfo`

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

- 값의 단위는 page
- page이 실제 사이즈는 다음 방식으로 확인
  - `getconf PAGESIZE`
    - 4096(byte)
  - `grep -i pagesize /proc/self/smaps`
    - KernelPageSize: 4 kB
