# Linux 메모리 Pressure 관련 커널 변수

메모리 pressure를 제어하는 주요 커널 파라미터들입니다:

## 1. **vm.min_free_kbytes**

```bash
# 확인
cat /proc/sys/vm/min_free_kbytes

# 설정 (512MB 예시)
sysctl -w vm.min_free_kbytes=524288
```

- **역할**: 시스템이 항상 유지할 최소 여유 메모리 (KB)
- **영향**: 이 값에 가까워지면 커널이 메모리 회수 시작
- **기본값**: 총 메모리의 약 0.4~1%

## 2. **vm.watermark_scale_factor**

```bash
# 확인
cat /proc/sys/vm/watermark_scale_factor

# 설정 (기본값 10)
sysctl -w vm.watermark_scale_factor=10
```

- **역할**: watermark 간격 조정 (1~1000)
- **영향**: 값이 클수록 메모리 회수를 더 일찍 시작
- **계산**: watermark_distance = 총메모리 * scale_factor / 10000

## 3. **vm.vfs_cache_pressure**

```bash
# 확인
cat /proc/sys/vm/vfs_cache_pressure

# 설정
sysctl -w vm.vfs_cache_pressure=100
```

- **역할**: **page cache와 inode/dentry cache 회수 비율**
- **값 의미**:
  - `100`: 기본값 (균형)
  - `0`: inode/dentry 거의 회수 안 함
  - `200`: inode/dentry 적극적으로 회수
- **당신의 케이스**: page cache를 더 적극적으로 회수하려면 이 값을 높일 수 있음

## 4. **vm.swappiness**

```bash
# 확인
cat /proc/sys/vm/swappiness

# 설정
sysctl -w vm.swappiness=10

```
- **역할**: 스왑 사용 경향 (0~100)
- **값 의미**:
  - `0`: 스왑 최소화 (page cache 우선 회수)
  - `60`: 기본값
  - `100`: 스왑 적극 사용

## 5. **Memory Watermarks**

Linux는 3단계 watermark로 메모리 pressure 판단:

```
high watermark ━━━━━━━━━━━  정상 상태
                 ▼
                 여유 영역
                 ▼
low watermark  ━━━━━━━━━━━  kswapd 시작 (백그라운드 회수)
                 ▼
                 위험 영역
                 ▼
min watermark  ━━━━━━━━━━━  direct reclaim (동기 회수)
```

## 6. **PSI (Pressure Stall Information)** - 커널 4.20+

```bash
# 메모리 pressure 확인
cat /proc/pressure/memory

# 출력 예시:
# some avg10=0.00 avg60=0.00 avg300=0.00 total=12345
# full avg10=0.00 avg60=0.00 avg300=0.00 total=6789
```

## 당신의 Docker + Page Cache 케이스

이전 논의했던 page cache 문제 해결을 위한 설정:

```bash
# 1. page cache를 더 적극적으로 회수
sysctl -w vm.vfs_cache_pressure=150

# 2. 메모리 회수를 더 일찍 시작
sysctl -w vm.watermark_scale_factor=15

# 3. 스왑 대신 page cache 회수 우선
sysctl -w vm.swappiness=10

# 영구 적용
echo "vm.vfs_cache_pressure=150" >> /etc/sysctl.conf
echo "vm.watermark_scale_factor=15" >> /etc/sysctl.conf
echo "vm.swappiness=10" >> /etc/sysctl.conf
```

## 모니터링

```bash
# 현재 메모리 상태
cat /proc/meminfo | grep -E 'MemFree|Cached|Active|Inactive'

# watermark 확인
cat /proc/zoneinfo | grep -A 10 "Node 0, zone"

# 메모리 pressure 실시간 모니터링
watch -n 1 'cat /proc/pressure/memory'
```

## 주의사항

- **너무 공격적인 설정**은 성능 저하 유발
- **프로덕션 환경**에서는 점진적으로 조정하며 테스트
- **컨테이너 환경**에서는 호스트 레벨 설정이 모든 컨테이너에 영향

이 설정들이 page cache 회수 동작에 직접적인 영향을 미칩니다!