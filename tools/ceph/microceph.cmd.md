# microceph 명령어

microceph 명령들은 크게 2종류입니다.

1. MicroCeph 관리용 래퍼
2. Ceph 원본 CLI(또는 그 별칭)

각 명령의 역할은 아래와 같습니다.

- microceph
  - MicroCeph 전용 관리 명령.
  - 클러스터 초기화(init), 노드 조인, 서비스 enable/disable(mon/mgr/mds/rgw), 디스크 추가 같은 “배포/운영 래퍼” 역할.
  - mon/mgr은 클러스터 초기화(`microcepth init`)시 자동 활성화(단, microcepth를 설치하면 init도 자동 완료됨)되자만 rgw(object storage)/mds(cephFS)는 기능 필요시 별도로 활성화 해야한다

- microceph.ceph
  - Ceph 메인 CLI.
  - Ceph CLI를 MicroCeph 컨텍스트에서 실행하는 이름.
  - 클러스터 상태 조회, 설정 변경, 데몬/풀/CRUSH/헬스 확인 등 전반 작업.
- ceph
  - microceph.ceph 의 별칭

- microceph.ceph-authtool
  - Ceph 인증 키링/키 생성·조회·수정 도구.
  - 클라이언트/데몬 키링 작업 시 사용.

- microceph.ceph-bluestore-tool
  - BlueStore(OSD 백엔드) 저수준 점검/복구/검사 도구.
  - OSD 장애 분석이나 오프라인 유지보수에 사용(주의해서 사용).

- microceph.rados
  - RADOS 저수준 오브젝트 조작 CLI.
  - 풀/오브젝트 직접 조회, put/get, 벤치마크 등.

- rados (alias microceph.rados)
  - microceph.rados와 동일한 별칭.

- microceph.radosgw-admin
  - RGW(Object Gateway) 관리 CLI.
  - 사용자, 버킷, realm/zone/zonegroup, quota, 메타데이터 관리.

- microceph.rbd
  - RBD(Rados Block Storage) 관리 CLI.
  - 이미지 생성/삭제, 스냅샷, clone, map/unmap 등.

- radosgw-admin
  - 보통 microceph.radosgw-admin과 동일 계열의 RGW 관리 CLI(환경에 따라 alias 또는 동일 바이너리 경로).

실무적으로는 이렇게 기억하면 편합니다.
1. 클러스터 운영: microceph, ceph
2. 오브젝트 저수준: rados
3. 오브젝트 게이트웨이 관리: radosgw-admin
4. 블록 볼륨: rbd
5. 인증/OSD 내부 복구: ceph-authtool, ceph-bluestore-tool (신중 사용)

## 자주 사용하는 명령어 예제

1. microceph (클러스터 운영 래퍼)
- 클러스터 상태 확인: microceph status
- 단일 노드 초기화: sudo microceph init
- 디스크 추가: sudo microceph disk add /dev/sdb
- RGW 활성화(기본 80/443): sudo microceph enable rgw
- 특정 포트로 RGW 활성화: sudo microceph enable rgw --port 7480 --ssl-port 7443

2. ceph / microceph.ceph (Ceph 메인 관리)
- 헬스 확인: sudo microceph.ceph -s
- 상세 헬스: sudo microceph.ceph health detail
- 서비스 엔드포인트 확인(대시보드 포함): sudo microceph.ceph mgr services
- 설정 조회: sudo microceph.ceph config dump
- 풀 목록 확인: sudo microceph.ceph osd lspools

3. microceph.rados 또는 rados (오브젝트 저수준)
- 벤치마크 쓰기: sudo microceph.rados bench -p mypool 10 write
- 오브젝트 업로드: echo hello | sudo microceph.rados -p mypool put obj1 -
- 오브젝트 다운로드: sudo microceph.rados -p mypool get obj1 ./obj1.out
- 오브젝트 목록: sudo microceph.rados -p mypool ls
- 오브젝트 삭제: sudo microceph.rados -p mypool rm obj1

4. microceph.rbd (블록 스토리지 이미지)
- 이미지 생성: sudo microceph.rbd create mypool/vm-disk --size 10G
- 이미지 목록: sudo microceph.rbd ls mypool
- 이미지 정보: sudo microceph.rbd info mypool/vm-disk
- 스냅샷 생성: sudo microceph.rbd snap create mypool/vm-disk@snap1
- 스냅샷 롤백: sudo microceph.rbd snap rollback mypool/vm-disk@snap1

5. microceph.radosgw-admin 또는 radosgw-admin (RGW 관리)
- 사용자 생성: sudo microceph.radosgw-admin user create --uid=test --display-name="Test User"
- 사용자 조회: sudo microceph.radosgw-admin user info --uid=test
- 버킷 목록: sudo microceph.radosgw-admin bucket list
- 버킷 통계: sudo microceph.radosgw-admin bucket stats --bucket=mybucket
- 사용자 쿼터 설정: sudo microceph.radosgw-admin quota set --quota-scope=user --uid=test --max-size=10G --enabled=true

6. microceph.ceph-authtool (키링/인증)
- 새 키링 생성: sudo microceph.ceph-authtool /tmp/client.demo.keyring --create-keyring
- 엔티티/키 추가: sudo microceph.ceph-authtool /tmp/client.demo.keyring --name client.demo --gen-key
- 키링 내용 확인: sudo microceph.ceph-authtool /tmp/client.demo.keyring --list

7. microceph.ceph-bluestore-tool (OSD 내부 점검, 주의)
- BlueStore 라벨 보기: sudo microceph.ceph-bluestore-tool show-label --dev /dev/sdb
- FSCK 점검(오프라인 권장): sudo microceph.ceph-bluestore-tool fsck --path /var/snap/microceph/common/data/osd/ceph-0
- QFSCK 점검: sudo microceph.ceph-bluestore-tool qfsck --path /var/snap/microceph/common/data/osd/ceph-0

참고:
- 운영 중인 OSD에 bluestore-tool을 바로 쓰는 건 위험할 수 있어서, 유지보수 창구간/중지 상태에서 수행하는 것을 권장합니다.
- 대부분 환경에서는 ceph, rados, radosgw-admin이 microceph.* 명령으로 연결된 alias라서, 둘 중 편한 이름을 써도 됩니다.

## 초기 구축 10분 점검 커맨드 세트

```bash
# 1) MicroCeph 기본 상태
microceph status

# 2) Ceph 전체 상태 요약
sudo microceph.ceph -s

# 3) 헬스 상세(경고/에러 원인 확인)
sudo microceph.ceph health detail

# 4) MON quorum 확인
sudo microceph.ceph quorum_status --format json-pretty

# 5) OSD 트리/업다운 확인
sudo microceph.ceph osd tree

# 6) 실제 서비스 엔드포인트 확인 (dashboard/rgw 등)
sudo microceph.ceph mgr services

# 7) 매니저 모듈 상태 확인 (dashboard/prometheus 등)
sudo microceph.ceph mgr module ls

# 8) 풀 목록 확인
sudo microceph.ceph osd lspools

# 9) 용량/DF 확인
sudo microceph.ceph df

# 10) 현재 리슨 포트 확인 (네트워크 검증)
sudo ss -lntp | grep -E ':(80|443|3300|6789|6800|8443|8080)\b'
```

빠른 판정 기준:
1. `HEALTH_OK`면 1차 통과
2. `mgr services`에 dashboard/rgw URL이 기대값으로 보이면 통과
3. `osd tree`에서 OSD가 `up/in`이면 통과

## 단일 노드 기준 이상 징후 체크리스트(경고별 조치 커맨드)

1. HEALTH_WARN / HEALTH_ERR
- 확인
sudo microceph.ceph -s
sudo microceph.ceph health detail
- 조치
  - 원인 문구 기준으로 아래 항목별 조치 수행 후 재확인:
sudo microceph.ceph -s

2. OSD_DOWN 또는 OSD가 out 상태
- 확인
sudo microceph.ceph osd tree
sudo microceph.ceph osd stat
sudo systemctl status snap.microceph.daemon-osd.*
- 조치
  - OSD 프로세스 재기동:
sudo systemctl restart snap.microceph.daemon-osd.*
  - 재확인:
sudo microceph.ceph osd tree

3. MON quorum 문제 (quorum 없음/축소)
- 확인
sudo microceph.ceph quorum_status --format json-pretty
sudo systemctl status snap.microceph.daemon-mon.*
- 조치
  - MON 재기동:
sudo systemctl restart snap.microceph.daemon-mon.*
  - 재확인:
sudo microceph.ceph quorum_status --format json-pretty

4. MGR 비활성 또는 대시보드 접속 불가
- 확인
sudo microceph.ceph mgr stat
sudo microceph.ceph mgr services
sudo microceph.ceph mgr module ls
- 조치
  - MGR 재기동:
sudo systemctl restart snap.microceph.daemon-mgr.*
  - 대시보드 모듈 재적용:
sudo microceph.ceph mgr module disable dashboard
sudo microceph.ceph mgr module enable dashboard
  - 재확인:
sudo microceph.ceph mgr services

5. FULL/NEARFULL (용량 임계치)
- 확인
sudo microceph.ceph df
sudo microceph.ceph osd df tree
- 조치
  - 불필요 데이터 정리 또는 디스크 증설:
sudo microceph disk add /dev/sdX
  - 재균형 대기 후 확인:
sudo microceph.ceph -s

6. PG_DEGRADED / PG_UNDERSIZED / stuck PG
- 확인
sudo microceph.ceph -s
sudo microceph.ceph pg stat
sudo microceph.ceph pg dump_stuck inactive
sudo microceph.ceph pg dump_stuck unclean
- 조치
  - OSD/네트워크/용량 이슈 먼저 해결 후 회복 대기:
watch -n 2 sudo microceph.ceph -s

7. Slow ops / blocked requests
- 확인
sudo microceph.ceph -s
sudo microceph.ceph health detail
sudo microceph.ceph osd perf
- 조치
  - 디스크/CPU/메모리 병목 확인:
iostat -x 1 5
top
메모리 압박 시 불필요 프로세스 정리 후 OSD 재기동 고려

8. Clock skew 경고
- 확인
timedatectl status
sudo microceph.ceph health detail
- 조치
  - NTP 동기화 활성:
sudo timedatectl set-ntp true
  - 재확인:
timedatectl status

9. RGW 비정상 (S3 접근 실패)
- 확인
sudo microceph.ceph orch ps | grep rgw
sudo microceph.ceph mgr services
sudo ss -lntp | grep -E ':80|:443|:7480|:7443'
- 조치
  - RGW 재활성화(필요 시 포트 명시):
sudo microceph disable rgw
sudo microceph enable rgw --port 80 --ssl-port 443

10.  포트/방화벽 이슈
- 확인
sudo ss -lntp | grep -E ':3300|:6789|:6800|:8443|:8080|:80|:443'
sudo ufw status verbose
- 조치
  - 필요 포트 허용:
sudo ufw allow 3300/tcp
sudo ufw allow 6789/tcp
sudo ufw allow 6800:7568/tcp
sudo ufw allow 8443/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

11.  설정 반영 여부 최종 검증
- 확인
microceph status
sudo microceph.ceph -s
sudo microceph.ceph health detail
sudo microceph.ceph mgr services

## One-Time Health Check Script (Copy/Paste)

The script below performs read-only status checks for a single-node environment.

```bash
#!/usr/bin/env bash
set -u

if ! command -v microceph >/dev/null 2>&1; then
  echo "[ERROR] microceph command not found."
  exit 1
fi

if ! command -v sudo >/dev/null 2>&1; then
  echo "[ERROR] sudo command not found."
  exit 1
fi

run() {
  echo
  echo "=================================================="
  echo "[CHECK] $1"
  echo "--------------------------------------------------"
  shift
  "$@" || echo "[WARN] Failed: $*"
}

run "MicroCeph status" microceph status
run "Ceph summary" sudo microceph.ceph -s
run "Ceph health detail" sudo microceph.ceph health detail
run "MON quorum status" sudo microceph.ceph quorum_status --format json-pretty
run "OSD tree" sudo microceph.ceph osd tree
run "MGR service endpoints" sudo microceph.ceph mgr services
run "MGR module status" sudo microceph.ceph mgr module ls
run "Pool list" sudo microceph.ceph osd lspools
run "Capacity/DF" sudo microceph.ceph df

echo
echo "=================================================="
echo "[CHECK] Listening status of key ports"
echo "--------------------------------------------------"
sudo ss -lntp | grep -E ':(80|443|3300|6789|6800|8443|8080)\b' || \
  echo "[WARN] No listening entries found for key ports."

echo
echo "=================================================="
echo "[DONE] Check completed"
echo "- HEALTH_OK check: sudo microceph.ceph -s"
echo "- Service URL check: sudo microceph.ceph mgr services"
echo "=================================================="
```

How to run

```bash
chmod +x ./microceph-check.sh
./microceph-check.sh
```

Quick run (create file + execute)

```bash
cat > ./microceph-check.sh <<'EOF'
#!/usr/bin/env bash
set -u

if ! command -v microceph >/dev/null 2>&1; then
  echo "[ERROR] microceph command not found."
  exit 1
fi

if ! command -v sudo >/dev/null 2>&1; then
  echo "[ERROR] sudo command not found."
  exit 1
fi

run() {
  echo
  echo "=================================================="
  echo "[CHECK] $1"
  echo "--------------------------------------------------"
  shift
  "$@" || echo "[WARN] Failed: $*"
}

run "MicroCeph status" microceph status
run "Ceph summary" sudo microceph.ceph -s
run "Ceph health detail" sudo microceph.ceph health detail
run "MON quorum status" sudo microceph.ceph quorum_status --format json-pretty
run "OSD tree" sudo microceph.ceph osd tree
run "MGR service endpoints" sudo microceph.ceph mgr services
run "MGR module status" sudo microceph.ceph mgr module ls
run "Pool list" sudo microceph.ceph osd lspools
run "Capacity/DF" sudo microceph.ceph df

echo
echo "=================================================="
echo "[CHECK] Listening status of key ports"
echo "--------------------------------------------------"
sudo ss -lntp | grep -E ':(80|443|3300|6789|6800|8443|8080)\b' || \
  echo "[WARN] No listening entries found for key ports."

echo
echo "=================================================="
echo "[DONE] Check completed"
echo "- HEALTH_OK check: sudo microceph.ceph -s"
echo "- Service URL check: sudo microceph.ceph mgr services"
echo "=================================================="
EOF

chmod +x ./microceph-check.sh
./microceph-check.sh
```