# linux system resource 모니터링 도구

btop과 glances 혼합 사용 권장

- htop : 시스템 리소스 모니터링 기본형, 각 process에 signal 전송 또는 renice 기능 제공
- iotop : DISK I/O 전용 모니터링
- glances : htop 대비 DISK I/O 정보 추가 모니터링 가능
- btop : 사용자 친화적 UI 제공, htop의 상위 호환(htop의 기능 대부분 지원 가능), 마우스 지원

## top 메모리 사용량(VIRT, RES, SHR) 이해

```text
VIRT (Virtual Memory)
  └─ RES (Resident Memory) ← 실제 물리 메모리 사용량
       ├─ SHR (Shared Memory) ← 공유 메모리
       └─ Private Memory ← 프로세스 전용 메모리

∴ RES = SHR + Private Memory
```

```text
VIRT ≥ RES ≥ SHR

RES = 실제 물리 메모리 사용량
Private Memory = RES - SHR
```

## 각 항목의 의미

| 항목 | 의미 | 설명 |
|------|------|------|
| **VIRT** | Virtual Memory | 할당된 가상 메모리 총량 (실제 사용 여부 무관), 메모리 파일 맵핑 포함 |
| **RES** | Resident Memory | ⚠️ 실제 물리 RAM에 상주하는 메모리 (SHR을 포함), 프로세서의 실제 메모리 사용량 |
| **SHR** | Shared Memory | 다른 프로세스와 공유 가능한 메모리. 공유 라이브러리(libc.so, libpthread.so 등 어플리케이션 이나 시스템 공유 라이브러리 모두 포함)와 실행 파일의 코드 세그먼트가 물리 메모리에 상주된 것을 의미하며, 여러 프로세스가 동일한 물리 메모리를 공유 |
| **Private** | (표시 안 됨) | ⚠️ RES - SHR = 단일 프로세스 전용 메모리 사용량 |

## 실제 예시

```bash
$ top

PID  USER     VIRT    RES    SHR  S  %CPU  %MEM
1234 user    500M    100M   30M  S   5.0   2.0

해석:
- VIRT = 500M: 가상 메모리 공간 (예약됨)
- RES  = 100M: 실제 물리 메모리에 로드됨
- SHR  = 30M:  그중 30M은 공유 메모리
- Private = 100M - 30M = 70M (프로세스 전용)
```

### 시각적 표현

```text
┌─────────────────────────────────────┐
│  VIRT (500M) - 가상 메모리          │
│  ┌───────────────────────────────┐  │
│  │ RES (100M) - 물리 메모리      │  │
│  │ ┌──────────┐  ┌─────────────┐ │  │
│  │ │ SHR (30M)│  │Private (70M)│ │  │
│  │ │  공유    │  │  전용       │ │  │
│  │ └──────────┘  └─────────────┘ │  │
│  └───────────────────────────────┘  │
│  나머지 400M은 스왑 또는 미사용     │
└─────────────────────────────────────┘
```

