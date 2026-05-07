# ubuntu microceph 설치 (PoC, 테스트 환경)

Ceph의 기본 설정(데이터 3중 복제) 때문에 디스크 1개만 추가하면 '경고(HEALTH_WARN)' 상태가 뜨게 됩니다.
이를 해결하고 디스크(OSD) 1개만으로 HEALTH_OK를 만드는 가장 빠른 방법은 다음과 같습니다.

## 테스트용 microceph 설치 스크립트

- 단일 노드, 단일 loopback 디스크, 단일 pool 구성

```bash
#/usr/bin/env bash

sudo snap install microceph
sudo microceph cluster bootstrap
sudo microceph disk add loop,4G,1
sudo ceph config set global osd_pool_default_size 1
sudo ceph config set global osd_pool_default_min_size 1
sudo ceph osd pool set device_health_metrics size 1
sudo ceph status

# 풀의 min_size/size 조정 (단일 노드 테스트용, )
sudo ceph osd pool set rbd min_size 1
sudo ceph osd pool set rbd size 1

# (선택) 기존 RGW가 이미 떠 있으면 먼저 비활성화
sudo microceph disable rgw

# RGW for object storage 활성화 (HTTP 포트 지정)
sudo microceph enable rgw --port 2929

# MDS for CephFS 활성화
sudo microceph enable mds
```

## snap 버전 확인 및 설치

```bash
# snap 버전 확인 (설치되어 있다면 버전 정보 출력)
snap --version

# 만약 설치되어 있지 않다면
sudo apt update
sudo apt install snapd
```

## MicroCeph 설치 전 검토 사항

## 1. MicroCeph 설치 및 클러스터 초기화

먼저 최소한의 관리를 위해 초기화를 수행합니다. (이 과정 자체가 관리 데몬을 띄우는 과정이라 생략은 불가능합니다.)

```bash
sudo snap install microceph
sudo microceph cluster bootstrap
```

## 2. 가상 디스크(Loopback) 1개만 추가

명령어에서 개수를 1로 지정합니다.

```bash
sudo microceph disk add loop,4G,1
```

참고: 디스크 축소 설정이 아닌 기본 설정으로 사용하고자 하는 경우 - loopback 파일 3개 할당 필요

```bash
sudo microceph disk add loop,4G,3  # 4GB 가상 디스크 3개 생성
```

## 3. 단일 디스크를 위한 설정 변경 (중요)

주의: 디스크 3개로 할당을 했다면 이 과정은 필요하지 않습니다.

Ceph는 기본적으로 "데이터를 3군데에 복사해야 성공"이라고 판단합니다. 디스크가 1개뿐이면 "복사본을 만들 곳이 부족하다"며 에러를 띄웁니다. 이를 무시하고 디스크 1개에 최적화되도록 설정을 강제 변경해야 합니다.

```bash
# 기본 복제본 개수를 1개로 변경

sudo ceph config set global osd_pool_default_size 1
sudo ceph config set global osd_pool_default_min_size 1

# 이미 생성된 기본 풀(pool)이 있다면 해당 풀의 복제 설정도 변경
# (보통 초기에는 풀이 없지만, 안전을 위해 실행)

sudo ceph osd pool set device_health_metrics size 1
```

## 4. 상태 확인

이제 잠시 기다린 후 상태를 확인하면 디스크 1개만으로도 정상 상태가 됩니다.

```bash
sudo ceph status
sudo ceph -s
```

## 작업 요약

   1. Loopback 파일 1개 추가(가상 디스크)
   2. Ceph의 복제본 설정(Replication Size)을 1로 변경 - Ceph의 object HEALTH_OK 판단을 위함

이렇게 설정하면 로컬 PC에서 자원을 거의 먹지 않는 가장 가벼운 Ceph PoC 환경이 완성됩니다.
