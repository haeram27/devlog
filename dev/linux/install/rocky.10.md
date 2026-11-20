# Rocky 10 설치

## 설치 환경

Rocky 10.x

## 설치

### 설치과정 지원언어

- 한국어

### keyboard

- 한국어(kr)

- `한국어`와 `한국어(101/104키 호환)` 차이:
  - 한국어 키보드는 106키, 한/영, 한자 키 지원
  - 한국어(101/104)는 미국표준키보드(한/영, 한자 키 없음)에서 Shift+Space로 한/영 변환 지원

### 시스템 언어

- main : 한국어, sub : 영어
- 시스템 언어가 한국어를 지원하면, git log등에서 한글 내용을 정상 출력함

### 파티션 (중요)

> "사용자 정의" > 완료 > 파티션 구성

구성: 일반 개발 환경의 경우 다음의 3개 파티션이면 충분

```text
/boot/efi   600MB
/boot       1GB
/           나머지 전부 (LVM + XFS)
```

- LVM 사용할 것 : VM에서 설치하는 경우 LVM을 사용하지 않으면 설치한 OS의 filesystem이 read-only로 고정 부팅되는 현상 발생 가능
- swap 파티션 없음
  - ssd에서는 swapfile 사용이 더욱 편리
  - 개발용도가 아닌 운영 서버일 경우 oom-killer 대비 메모리의 1/2 용량으로 설정 추천

부팅 과정에서 파티션 참조 순서 및 역할

```text
/boot/efi   -> EFI System Partition (ESP)
/boot       -> 커널, initramfs, GRUB 파일
/           -> 실제 운영체제 설치 파티션
```

- `/boot/efi`
  - UEFI 펌웨어가 직접 읽을 수 있는 부팅 영역
  - EFI 실행파일(.efi) 저장
  - 부팅 시작점
- `/boot`
  - Linux 커널 관련 파일을 저장
  - /boot/efi의 GRUB가 /boot를 읽어서 커널을 로드

### 네트워크 설정

- IPv4
  - Method: Manual
  - Address
    - IP: 1.2.3.4
    - NetMask: 255.255.0.0
    - Gateway: 1.2.3.1
  - DNS: 8.8.8.8, 9.9.9.9
  - Check - Require IPv4 addressing ...
- 연결
- 호스트이름

#### 공개 DNS 서버 목록

- https://public-dns.info/

### ntp (시간과 날짜)

- 필요시 ntp 서버 지정
- 네트워크가 연결 되어야 설정 가능

### root 사용여부 및 사용자 생성

- 개발용 서버라면 root 활성화가 편리
- 기본 사용자는 생성하는 것이 좋음

## 설치 후 설정

### sudoers 설정 (non root 사용자인 경우)

```text
$ sudo vi /etc/sudoers
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
myname  ALL=(ALL) NOPASSWD:ALL
```

### selinux disable

도커 이미지 생성시 local 환경의 rpm repository를 구성, 해당 구성시 selinux disabled 필요

```bash
ls -al /etc/sysconfig/selinux /etc/selinux/config
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

### firewalld disable

```bash
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld
systemctl mask firewalld
systemctl status firewalld
```

한줄 명령

```bash
systemctl status firewalld && systemctl stop firewalld && systemctl disable firewalld && systemctl mask firewalld && systemctl status firewalld
```

### SSHD 로그인 설정

#### 설정 확인

```bash
# openssh 현재 설정 확인
sshd -T | egrep 'pubkeyauthentication|permitrootlogin|passwordauthentication'

# 특정 설정 위치 찾기
grep -R "PubkeyAuthentication" /etc/ssh
```

#### 설정 변경

```bash
$ sudo vi /etc/ssh/sshd_config
PubkeyAuthentication no
PasswordAuthentication no

$ sudo systemctl restart ssh
```

#### ssh 접속 허가 공개 키 등록

```bash
vi ~/.ssh/authorized_keys
chmod 400 ~/.ssh/authorized_keys
```

### yum repository

- rocky default repo url
  - http://dl.rockylinux.org

- mirror list
  - https://mirrors.rockylinux.org/mirrormanager/mirrors/Rocky
    - KR
      - http://ftp.kaist.ac.kr/pub/rocky

#### `/etc/yum.repos.d` 설정

특정 저장소를 사용해야하는 경우에만 수정

yum.repos 변경 방법

```bash
# repos 설정 백업
cp -r /etc/yum.repos.d /etc/yum.repos.d.origin

# 기존 설정 삭제
rm -f /etc/yum.repos.d/*

# 신규 설정 추가
vi /etc/yum.repos.d/new.repo

# 패키지 저장소 연결 확인 (메타테이터 다운로드)
dnf makecache -v
```

신규 repo 파일 예제

```text
[rocky-kaist-appstream]
name=Rocky Linux $releasever - AppStream
baseurl=https://ftp.kaist.ac.kr/$contentdir/$releasever/AppStream/$basearch/os/
enabled=1
gpgcheck=0
countme=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[rocky-kaist-baseos]
name=Rocky Linux $releasever - BaseOS
baseurl=https://ftp.kaist.ac.kr/$contentdir/$releasever/BaseOS/$basearch/os/
enabled=1
gpgcheck=0
countme=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[rocky-kaist-devel]
name=Rocky Linux $releasever - Devel
baseurl=https://ftp.kaist.ac.kr/$contentdir/$releasever/devel/$basearch/os/
enabled=1
gpgcheck=0
countme=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[rocky-kaist-extras]
name=Rocky Linux $releasever - Extras
baseurl=https://ftp.kaist.ac.kr/$contentdir/$releasever/extras/$basearch/os/
enabled=1
gpgcheck=0
countme=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[rocky-kaist-ha]
name=Rocky Linux $releasever - HighAvailability
baseurl=https://ftp.kaist.ac.kr/$contentdir/$releasever/HighAvailability/$basearch/os/
enabled=1
gpgcheck=0
countme=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[rocky-kaist-nfv]
name=Rocky Linux $releasever - NFV
baseurl=https://ftp.kaist.ac.kr/$contentdir/$releasever/NFV/$basearch/os/
enabled=1
gpgcheck=0
countme=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[rocky-kaist-plus]
name=Rocky Linux $releasever - Plus
baseurl=https://ftp.kaist.ac.kr/$contentdir/$releasever/plus/$basearch/os/
enabled=1
gpgcheck=0
countme=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[rocky-kaist-rt]
name=Rocky Linux $releasever - Realtime
baseurl=https://ftp.kaist.ac.kr/$contentdir/$releasever/RT/$basearch/os/
enabled=1
gpgcheck=0
countme=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[rocky-kaist-resilient-storage]
name=Rocky Linux $releasever - ResilientStorage
baseurl=https://ftp.kaist.ac.kr/$contentdir/$releasever/ResilientStorage/$basearch/os/
enabled=1
gpgcheck=0
countme=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[fedora-epel]
name=Extra Packages for Enterprise Linux for RedHat based system
baseurl=https://dl.fedoraproject.org/pub/epel/$releasever/Everything/$basearch/
enabled=1
gpgcheck=1
countme=0
gpgkey=https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-$releasever

[rhel-docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/rhel/$releasever/$basearch/stable
enabled=1
gpgcheck=1
countme=0
gpgkey=https://download.docker.com/linux/rhel/gpg
```

- gpgcheck가 반드시 필요한 경우가 아니면 `gpgcheck=0` 설정 가능
- baseurl은 mirrorlist와 함께 명시되면 대체 옵션 평가되어 동작하므로 주의

#### update and install packages

```bash
# dnf timeout 설정 변경
printf "timeout=600\nminrate=100\nretries=10\n" >> /etc/dnf/dnf.conf

# 패키지 저장소로 부터 메타데이터 다운로드 및 로컬 캐시 저장, 저장소 연결 확인용
dnf makecache -v

# 패키지 저장소로 부터 시스템에 설치된 패키지 모두 업그레이드, makecache 명령 기능을 포함함
dnf upgrade -y --nobest

# make 등 native 개발 관련 도구 설치
dnf groupinstall -y 'Development Tools'

# 기타 기본 유틸리티
dnf install -y --allowerasing open-vm-tools zsh git vim btop java-25-openjdk maven nodejs-npm
```

### LOCALE 변경

#### systemd

```bash
yum list installed | grep -i langpack
yum install -y langpacks-ko
localectl list-locales
sudo localectl set-locale ko_KR.utf8
# default = en_US.utf8
# re-login
```

#### non-systemd

```bash
yum list installed | grep -i langpack
yum install -y langpacks-ko
locale -a
sudo update-locale LANG=ko_KR.utf8
# default = en_US.utf8
# re-login
```

#### en_US.utf8과 en_US.UTF-8 차이

- 차이 없음, UTF-8 인코딩 표기의 별칭(alias) 차이임, 내부적으로는 같은 로케일 디렉터리를 가리킴
en_US.UTF-8과 en_US.utf8은 동일한 로케일
- 로케일 표준 형식은 언어_국가.인코딩[@수정자]이고, glibc에서는 en_US.UTF-8이 정규(canonical) 표기
- glibc에서는 인코딩 이름이 대소문자 구분 없이, UTF-8, utf8, UTF8 등을 동등하게 취급
- 무엇을 쓰면 좋은가
  - 어떤 것을 사용해도 상관 없음 혼용만 피해서 일관성을 유지할 것
  - 설정 파일(예: /etc/default/locale, ~/.pam_environment)에는 en_US.UTF-8로 쓰는 것을 권장
  - 예) LANG=en_US.UTF-8

### 커널 변수 수정

```bash
echo fs.inotify.max_user_watches=1048576 | sudo tee -a /etc/sysctl.d/99-sysctl.conf
echo vm.max_map_count=1048576 | sudo tee -a /etc/sysctl.d/99-sysctl.conf
sudo sysctl --system
```

- fs.inotify.max_user_watches
  - inotify는 리눅스에서 파일/디렉터리의 변경(생성, 수정, 삭제 등)을 커널이 이벤트로 알려주는 메커니즘이고, watch 1개 = 특정 경로(대개 디렉터리) 감시 1개라고 보면 됨
  - IDE(Visual Studio Code, JetBrains), 빌드/번들러(Webpack), 동기화(Syncthing/Nextcloud), 개발 서버(라이브 리로드) 등은 프로젝트의 디렉터리마다 watch를 붙임
  - 프로젝트가 크면 몇십만 개가 필요할 수 있어 상한에 걸리면 오류 증상 발생
    - “inotify watch limit reached” / “ENOSPC on inotify_add_watch”
    - 자동 새로고침/변경 감지 불가, 느려짐
  - 현재 max_user_watches 커널 변수 값 확인
    - cat /proc/sys/fs/inotify/max_user_watches
- vm.max_map_count
  - 프로세스당 mmap 가능한 최대 영역 수(자바/일부 DB에 중요)
  - “프로세스당 mmap”은 한 프로세스가 동시에 보유할 수 있는 매핑(=VMA, Virtual Memory Area) 개수
  - 대형 자바/엘라스틱서치/브라우저/IDE/DB/크롤러 등은 작은 조각의 매핑을 엄청 많이 쓰는데 한도가 낮으면 Out-of-Memory가 아닌데도 매핑 개수 때문에 실패
  - process의 mmap 정보 확인
    - cat /proc/self/maps
  - 현재 max_map_count 커널 변수 값 확인
    - cat /proc/sys/vm/max_map_count

#### 커널 변수 값 확인

```bash
# 커널 변수 목록 보기
sysctl -a
sysctl -aN

# 특정 변수의 현재 값 보기
sysctl -n vm.max_map_count

# apply values from all system directories
sysctl --system
```

### [docker 설치](https://docs.docker.com/engine/install/rhel/)

rhel 계열은 기본적으로 podman을 사용한다. 꼭 필요한 경우에만 docker 설치

#### dnf를 이용한 설치 방법 안내

- https://docs.docker.com/engine/install/fedora/

#### 기존 버전 삭제 (Optional)

```bash
sudo dnf remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine
```

#### 최신 버전 설치

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### docker 특정 버전 설치

docker-ce version 확인

```bash
$ dnf list docker-ce --showduplicates | sort -r

docker-ce.x86_64    3:28.5.1-1.fc41    docker-ce-stable
docker-ce.x86_64    3:28.5.0-1.fc41    docker-ce-stable
...
CE_VERSION_STRING=28.5.0
```

docker-compose-plugin version 확인

```bash
$ dnf list docker-compose-plugin --showduplicates | sort -r
docker-compose-plugin.x86_64    2.28.1-1.el9    abis-centos-docker-ce
docker-compose-plugin.x86_64    2.27.1-1.el9    abis-centos-docker-ce
...
COMPOSE_VERSION_STRING=2.27.1
```

docker 특정 버전 설치

```bash
$ sudo dnf install docker-ce-<CE_VERSION_STRING> docker-ce-cli-<CE_VERSION_STRING> containerd.io docker-buildx-plugin docker-compose-plugin-<COMPOSE_VERSION_STRING>
```

runc 다른 버전 다운로드 방법

- [runc release page](https://github.com/opencontainers/runc/releases) 에서 원하는 버전 다운로드(runc.amd64)
- 설치 경로: /usr/bin/runc
- 원본 runc 백업하고 symlink로 '/usr/bin/runc'를 신규 바이너리로 지정 권장

#### dockerd에서 사용할 registry 추가

```bash
$ vi /etc/docker/daemon.json
{
    "insecure-registries": ["registry.k8s.io", "quay.io", "registry-1.docker.io"]
}
```

#### 사용자 docker group에 추가

```bash
sudo usermod -aG docker ${USER}
```

#### docker daemon 활성화 및 테스트

```bash
sudo systemctl enable --now docker
systemctl status docker
sudo docker run hello-world
```

#### Private Registry 사용시 docker pull timeout 오류 수정

```bash
$ vi /etc/docker/daemon.json
{
  "max-concurrent-downloads": 1,
  "max-concurrent-uploads": 1
}

$ systemctl restart docker
```

- 사내 Registry(Artifactory, Harbor, Nexus 등)를 사용할 때 병렬 다운로드시 timeout 오류 발생하는 경우 많음
- 이는 Layered 이미지의 병렬다운로드 제한 등의 서버 환경이 원인
- max-concurrent-downloads 미설정 시 기본 값은 3


### root 비밀번호 변경 (su 사용 필요 시)

```bash
sudo passwd root
```

### 네트워크 및 NIC 설정

- rocky는 network-manager를 렌더러로 사용하므로 nmtui, nmcli 명령어 사용

```bash
nmtui
```

### ntp 서버 동기화

```bash
$ vi /etc/chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server ntp.any.server iburst

$ systemctl enable chronyd; systemctl start chronyd
$ chronyc sourcestats -v; chronyc sources -v; chronyc -a makestep
$ timedatectl set-timezone Asia/Seoul; timedatectl set-ntp y; timedatectl

# --- 풀어쓰면
$ systemctl enable chronyd
$ systemctl start chronyd
$ chronyc sourcestats -v  # 소스 서버 상태
$ chronyc sources -v      # 시간 동기화 상태
$ chronyc -a makestep     # 동기화 명령
$ chronyc tracking        # tracking
$ timedatectl set-timezone Asia/Seoul
$ timedatectl set-ntp y
$ timedatectl
```
