
# mount
1. [mount](#mount)
   1. [mount 방식의 종류](#mount-방식의-종류)
   2. [일반 마운트와 바인드 마운트의 원본 저장소의 차이](#일반-마운트와-바인드-마운트의-원본-저장소의-차이)
      1. [일반 마운트 대상](#일반-마운트-대상)
   3. [명령어 예제](#명령어-예제)
      1. [normal mount 예제](#normal-mount-예제)
      2. [bind mount 예제](#bind-mount-예제)
   4. [mount propagation(전파) 옵션](#mount-propagation전파-옵션)
      1. [옵션 설명](#옵션-설명)
      2. [예제](#예제)
---
mount 명령은 임의의 디렉토리로 파티션이나 다른 디렉토리를 연결하기 위해서 사용한다.

mount point(마운트 대상 디렉토리)는 사전에 반드시 존재 해야하며, 존재 하지 않으면 mount 명령은 실패한다. 만약 mount point에 데이터가 존재하고 있었다면 마운트가 해제 되었을 때 원본 데이터가 다시 보여진다. 이 특성은 일반 마운트 또는 바인드 마운트 모든 방식에서 동일하다.

mount 명령은 현재 시스템이 종료 되기 전까지 유지되며, 시스템 재부팅 시에도 영구 마운트 되어야 한다면 /etc/fstab 에 마운트 정보를 명시해야한다.

## mount 방식의 종류
* 일반 mount
    * 별도의 옵션이 없는 경우의 기본 동작
    * 파티션을 지정된 디렉토리로 마운트

* bind mount
    * --bind 옵션 사용
    * 디렉토리를 지정된 다른 디렉토리로 마운트
    * 서로 다른 파티션 간에 디렉토리도 bind mount 가능

* recursive bind mount
    * --rbind 옵션 사용
    * 바인드 마운트 할때 원본 디렉토리의 서브 디렉토리로 마운트 포인트가 있다면  

## 일반 마운트와 바인드 마운트의 원본 저장소의 차이
일반 마운트의 주로 파티션 단위를 원본으로 한다.  
바인드 마운트는 디렉토리를 원본으로 한다.

### 일반 마운트 대상
| **마운트 대상** | **마운트 가능 여부** | **설명** |
|---|---|---|
| **파티션 (`/dev/sda1`)** | ✅ 가능 | **일반적인 마운트 대상** |
| **전체 블록 디바이스 (`/dev/sdb`)** | ✅ 가능 | GPT/MBR 없이 파일 시스템이 직접 있는 경우 가능 |
| **LVM 볼륨 (`/dev/mapper/vg-lv`)** | ✅ 가능 | LVM 논리 볼륨도 마운트 가능 |
| **RAID 디바이스 (`/dev/md0`)**     | ✅ 가능 | RAID 볼륨도 블록 디바이스처럼 마운트 가능 |
| **ISO 이미지 (`image.iso`)**       | ✅ 가능 | `loop` 옵션을 사용하면 마운트 가능 |
| **스왑 파티션 (`/dev/sda2`)**       | ❌ 불가능 | `mount` 대신 `swapon` 사용 |


## 명령어 예제
### normal mount 예제
```bash
mkdir -p /mnt/data
sudo mount -o noexec,nosuid,nodev /dev/sdb1 /mnt/data
```
* noexec: 실행 파일 실행 불가.
* nosuid: setuid 비트 무시.
* nodev: 장치 파일(/dev/null 등) 사용 불가.

### bind mount 예제
```bash
# 마운트 포인트 생성
mkdir -p /mnt/bind_data

# 기존 디렉토리를 바인드 마운트
sudo mount --bind /data /mnt/bind_data
```


## mount propagation(전파) 옵션
propagation 옵션은 파티션 마운트와 바인드 마운트 모두 적용 가능

### 옵션 설명
|옵션                | 설명|
|:---|:---|
|--make-private      | 마운트가 다른 네임스페이스로 전파되지 않음.|
|--make-shared       | 마운트가 다른 네임스페이스로 전파됨.|
|--make-slave        | 부모 네임스페이스에서 변경된 사항을 수신하지만, 반대로 전달하지 않음.|
|--make-unbindable   | 마운트 해제가 불가능한 상태로 만듦.|

### 예제
```bash
# 마운트 포인트 생성
mkdir -p /mnt/bind_data

# 기존 디렉토리를 바인드 마운트
sudo mount --bind /data /mnt/bind_data

# 해당 바인드 마운트를 private으로 설정
sudo mount --make-private /mnt/bind_data
```