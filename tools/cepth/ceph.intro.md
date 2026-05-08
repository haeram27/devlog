# ceph

대규모 분산 스토리지 시스템으로, 분산 클러스터 위에서 Object storage를 구현해 object, block, file 레벨의 인터페이스를 제공하는 스토리지 솔루션

## 구성 요소 및 용어

- Monitors(MON)
  - 클러스터 전체 상태가 저장된 Map 정보(Monitor, Manager, OSD, MDS Map)를 기반으로 클러스터의 상태를 관리하고 동기화한다. 
  - 각각의 구성요소들은 데몬으로 실행되는데, heartbeat를 통해 여러 데몬들(MON, MGR, OSD, MSD)의 상태를 감시한다.

- Managers(MGR)
  - Storage utilizaion, performance, system load와 같은 클러스터 상태 관련 메트릭을 수집하고 자체 대시보드를 생성하여 모니터링을 진행한다.
  - Prometheus로 메트릭을 제공하여 연동할 수 있으며, Grafana / Alertmanager를 통해 모니터링 시스템을 별도로 구축할 수도 있다.
  - OSD 추가/삭제 등의 작업들을 통해 클러스터 운영하고 관리한다.

- Object Storage Daemons(OSD)
  - 실제 데이터를 저장하고 복제, 복구 그리고 분산 기능을 수행하여 데이터를 관리하는 디스크들의 묶음이다.

- Metadata Servers(MDS)
  - Ceph file system을 사용할 때만 필요한 구성요소로 파일 이름, 접근 권한, 소유자 등의 메타 데이터를 저장한다.

## 장점 및 단점

- 장점
  - 높은 확장성 → 수 천대의 노드까지 수평 확장이 가능
  - 고가용성 제공 & 무중단 운영 → 데이터를 여러 OSD에 복제하고 분산 저장하여 OSD에 장애가 발생해도 자가 복구가 가능
  - 다양한 스토리지 인터페이스 지원 → block, file, object 스토리지를 단일 시스템에서 지원
  - 오픈소스 기반

- 단점
  - 디스크 I/O 및 네트워크 성능에 민감하고 운영 난이도가 높음
  - Cluster 초기 구축 단계에서의 설정 혹은 OSD 노드 증가 시에 튜닝이 복잡함
  - 소규모 환경에서는 비효율적임

## Ceph storage type

RADOS(Reliable Autonomic Distributed Object Store)는 ceph의 저장소 계층으로 객체 단위의 파일/블록 데이터를 분산 저장하고 복구하는 역할을 수행하는 핵심 엔진이다.

- 데이터 쓰기 흐름

  - Ceph client는 ceph의 주요 스토리지 타입(block, file, object)을 기반으로 RADOS 라이브러리(LIBRADOS)를 통해 RADOS에 접근한다.
  - 데이터는 PG(Placement Group)에 먼저 매핑되고 CRUSH 알고리즘을 통해 PG를 어떤 OSD에 저장할지 결정하며 실제 데이터는 OSD를 통해 디스크에 저장된다.

### RBD (RADOS Block Device)

- 가상 디스크 장치 제공

- RADOS 기반의 ceph가 제공하는 고성능 분산 블록 스토리지로 ceph cluster 위에 가상 블록 디바이스를 생성하여 내부적으로는 데이터를 객체로 저장하지만, 외부에서는 디스크처럼 인식할 수 있게 해준다.

- 블록 디바이스란 장치 이미지(Image) 파일을 기반으로 생성된 `가상 디스크 디바이스`
  - 이미지(Image)가 곧 디바이스의 본체입니다 Ceph 클러스터 내부에 저장된 하나의 거대한 데이터 파일(이미지)이 존재합니다.이 이미지를 운영체제(OS)가 인식할 수 있도록 연결(매핑)하면, OS는 이를 실제 하드디스크인 `/dev/sdb`나 `/dev/rbd0` 같은 블록 디바이스로 인식하게 됩니다.
  - 가상 머신(VM)과 유사한 개념
    - 이미지: ISO 파일이나 VHD 파일처럼 저장소 풀(Pool) 안에 저장된 데이터 덩어리입니다.
    - 블록 디바이스: 그 이미지를 컴퓨터에 '꽂아서' 실제 읽고 쓸 수 있는 상태가 된 장치입니다.

- 생성된 블록 디바이스(RBD 이미지)는 VM, 컨테이너, DB 등에 마운트하여 사용할 수도 있고 RBD 이미지를 다수의 객체로 분산 저장하여 병렬적으로 빠르게 읽고 쓰기가 가능하다.

- 추가로 mirroring 기능을 통한 RBD 이미지의 비동기 복제(snapshot 기반)와 resizing 기능, 그리고 iSCSI 게이트웨이를 통한 RBD 이미지 외부 전송 기능을 통해 높은 확장성과 유연성을 제공한다.

- RBD Mirroring 기능 같은 경우, 서로 다른 ceph cluster가 구축되어 있어야 하고, iSCSI 같은 경우엔 ceph-iscsi 게이트웨이를 통해 지원된다.

### CephFS (Ceph File System)

- File System storage 제공: OS에서 디렉토리로 mount할 수 있는 저장소를 제공
- CephFS는 POSIX와 호환되는 분산 파일 시스템으로 클라이언트는 CephFS를 통해 파일을 폴더 단위로 계층적으로 관리할 수 있다. 다만, 폴더 계층 정보를 저장해야 하기 위해 ceph cluster 상에 MDS가 반드시 포함되어야 한다.

### RGW (RADOS Gateway)

- object storage: object(파일 등, 바이너리)를 저장할 수 있는 저장소
- RGW는 Amazon S3와 OpenStack Swift API와 호환되는 객체 스토리지이다. RGW는 HTTP 서버로 동작하며 클라이언트의 API 요청을 처리하고 버킷/객체 단위로 데이터를 저장하고 관리한다.

## Pool type & value

Ceph의 pool은 object 데이터들의 묶음을 저장하는 논리적인 공간으로, ceph에서는 사용자 파일을 저장하는 data라는 이름의 pool 혹은 시스템 로그를 저장하는 log라는 이름의 pool과 같이 데이터의 특성에 따라 pool을 나눠 효율적인 데이터 저장 및 관리가 가능하다.

### Replicated pool

Replicated pool은 CephFS, RBD, RGW 등 거의 모든 스토리지 타입에서 기본값으로 사용되는 데이터 저장 방식으로 동일한 복사본을 서로 다른 N개의 OSD에 분산 저장하는 방식으로 동작한다. → 데이터 전체를 복제

복제본 수가 증가할 수록 스토리지 사용량이 증가하는 단점이 있지만 보통 복제본 수를 3으로 사용하여 높은 IOPS 성능을 바탕으로 하나 또는 두 개의 OSD에서 장애가 발생해도 빠르게 데이터를 복구할 수 있다.

### EC Pool(Erasure Coded Pool)

EC pool은 데이터를 K개로 분할하고 M개의 패리티 블록을 추가로 생성하여 총 K+M개의 블록을 서로 다른 OSD에 저장하는 방식이다. → 데이터를 분할하고 패리티 블록 추가 저장

M개의 디스크에서 장애가 발생해도 복구할 수 있게 하는 데이터 보호 방식으로 replicated pool에 비해 스토리지 공간을 30 ~ 70% 절감된 수준으로 활용할 수 있지만 쓰기 성능이나 인코딩/디코딩 연산으로 인한 CPU 성능 저하, 그리고 객체 크기가 작을수록 오버헤드가 크다는 단점이 존재한다.

## 참고

- [ceph storage arch - intel arch for cloud](https://www.slideshare.net/slideshow/ceph-open-source-storage-software-optimizations-on-intel-architecture-for-cloud-workloads/47719746?utm_source=clipboard_share_button&utm_campaign=slideshare_make_sharing_viral_v2&utm_variation=variant&utm_medium=share#slide8)

- [ceph object storage arch - 2015 ceph berlin meetup](https://www.slideshare.net/slideshow/ceph-object-storage-at-spreadshirt-july-2015-ceph-berlin-meetup/51002195?utm_source=clipboard_share_button&utm_campaign=slideshare_make_sharing_viral_v2&utm_variation=variant&utm_medium=share#slide7)