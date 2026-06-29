# microceph 설치 후 서비스 활성화

MicroCeph 환경에서 활성화하여 사용할 수 있는 핵심 서비스는 크게 세 가지입니다.
Ceph는 기본적으로 통합 스토리지이므로 동일한 클러스터 위에서 다음 서비스들을 동시에 운영할 수 있습니다.

## 1. RBD (Ceph Block Device) - 블록 장치(가상 디스크)

- 용도: 리눅스 서버에 가상 하드디스크(블록 스토리지)를 연결하는 방식입니다.
- 특징: 가상 머신(VM)이나 데이터베이스처럼 빠른 입출력이 필요한 서비스에 적합합니다.
- 활성화: 이미 풀 생성 방법(ceph osd pool create rbd)을 알고 계시므로, 바로 이미지를 만들어 마운트할 수 있습니다.

```bash
# 1. rbd라는 이름의 풀 생성 (뒤의 32는 PG 숫자입니다)
sudo ceph osd pool create rbd 32

# 2. 이 풀을 RBD 용도로 사용하겠다고 선언 (초기화)
sudo ceph osd pool application enable rbd rbd

# 3. RBD 풀 내부에 실제 가상 디스크(이미지) 생성 (예: 2GB 크기)
sudo rbd create my-disk --size 2048 --pool rbd

# 3. 풀 목록 확인
sudo ceph osd lspools

# 4. 생성한 이미지를 /dev/rbd0 장치로 연결
sudo rbd map my-disk --pool rbd

# 5. 연결된 장치 확인
lsblk | grep rbd
```

## 2. RGW (RADOS Gateway) — 오브젝트 스토리지

- 용도: AWS S3와 호환되는 API를 제공하는 스토리지입니다.(풀 자동 생성됨).
- 특징: HTTP를 통해 파일을 업로드/다운로드하며, 웹 서비스의 이미지 저장소나 백업 용도로 주로 쓰입니다.
- 활성화 (MicroCeph): 서비스 활성화 시 시스템이 필요한 풀들을 한꺼번에 생성

```bash
# RGW 데몬 실행(default port 8080)
sudo microceph enable rgw --port 8555

# RGW 서비스가 살아있는지 확인
curl http://localhost:8555
# (XML 형태의 응답이 나오면 정상) <?xml version="1.0" encoding="UTF-8"?><ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/"><Owner><ID>anonymous</ID></Owner><Buckets></Buckets></ListAllMyBucketsResult>%
```

curl 사용하여 localhost 사용시 프록시 접속 오류 메세지가 나타난다면, localhost 주소를 proxy 적용 대상에서 제외해야함

- windows : 프록시 설정 > 로컬 호스트 제외
- linux : 환경 변수 설정(export no_proxy="localhost,...")

## 3. CephFS (Ceph File System) — 공유 파일 시스템

- 용도: 여러 대의 서버가 동시에 같은 디렉토리에 접근하여 파일을 공유하는 방식(NFS와 유사)입니다.
- 특징: 여러 클라이언트가 동시에 읽고 쓰기가 가능합니다.
- 활성화 (MicroCeph):

```bash
# 1. 메타데이터 서버(MDS) 활성화 (서버 프로세스 먼저 준비)
sudo microceph enable mds

# 2. 메타데이터용 풀 생성
sudo ceph osd pool create cephfs_metadata 32

# 3. 데이터 저장용 풀 생성
sudo ceph osd pool create cephfs_data 32

# 4. 두 풀을 묶어서 'myfs'라는 이름의 파일시스템 생성
sudo ceph fs new myfs cephfs_metadata cephfs_data

# 5. status 확인
sudo ceph fs ls
sudo ceph status
# status 결과에서 mds: myfs:1 {0=DESKTOP-8G3VRF3=up:active}와 같은 내용이 보이면 성공


# myfs를 실제 디렉토리에 마운트
## 마운트할 지점 생성
mkdir ~/my_shared_folder

## 커널 드라이버를 이용해 마운트 (비밀번호는 admin 키 사용)
sudo mount -t ceph :/ ~/my_shared_folder -o name=admin
```

## 4. 그 외 관리용 서비스

- Dashboard: 웹 UI를 통해 클러스터 상태를 그래프로 보고 관리할 수 있습니다.

```bash
sudo microceph enable dashboard
```

- Monitoring: Prometheus나 Grafana를 연동하여 상세한 성능 지표를 확인할 수 있습니다.
