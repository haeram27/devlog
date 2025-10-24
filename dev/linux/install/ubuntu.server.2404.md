# Ubuntu Server 24.04

## 설치 환경

ubuntu 24.04 LTS

## 설정

### sudoers 설정

```bash
$ sudo vi /etc/sudoers
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
myname  ALL=(ALL) NOPASSWD:ALL
```

### root 비밀번호 변경 (su 사용 필요 시)

```bash
sudo passwd root
```

### 고정 IP Manual 설정

- ubuntu server는 설치 과정에 네트워크 설정 과정을 포함함
- ubuntu는 rocky와 달리 network-manager(nmtui, nmcli)를 사용하지 않으며, 대신 netplan을 사용
- network if 설정 파일 이름은
  - normal image: 00-installer-config.yaml
  - cloud image: 50-cloud-init.yaml

```bash
$ sudo vi /etc/netplan/50-cloud-init.yaml
network:
  version: 2
  ethernets:
    ens33: # 실제 인터페이스 이름으로 변경
      dhcp4: false # DHCP를 사용하지 않음
      addresses:
        - 10.101.10.100/16 # 원하는 고정 IP 및 서브넷 마스크
      routes:
        - to: default
          via: 10.101.0.1 # 기본 게이트웨이 IP 주소
      nameservers:
        addresses: # DNS 서버 주소 (google)
        - 8.8.8.8
        - 4.4.4.4

$ sudo netplan apply
$ ip a
```

#### DNS server list

- https://public-dns.info/

### APT sources.list

```bash
sudo vi /etc/apt/sources.list.d/ubuntu.sources`
```

> /etc/apt/sources.list.d/ubuntu.sources (default)

```text
Types: deb
URIs: http://archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
 
Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

> /etc/apt/sources.list.d/ubuntu.sources (kaist)

```text
Types: deb
URIs: http://ftp.kaist.ac.kr/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
 
Types: deb
URIs: http://ftp.kaist.ac.kr/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

#### update packages

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get autoremove
```

#### Official Archive Mirros for Ubuntu

- https://launchpad.net/ubuntu/+archivemirrors
- Korea, Republic of
  - http://ftp.kaist.ac.kr/ubuntu/

### SSHD 로그인 설정

```bash
$ sudo vi /etc/ssh/sshd_config
PubkeyAuthentication yes
PasswordAuthentication yes

$ sudo systemctl restart ssh
```

#### ssh 접속 허가 공개 키 등록

```bash
vi ~/.ssh/authorized_keys
```

### locale 변경

#### systemd

```bash
sudo apt install -y language-pack-ko
localectl list-locales
sudo localectl set-locale ko_KR.utf8
# default = en_US.utf8
# re-login
```

#### non-systemd

```bash
sudo apt install -y language-pack-ko
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

## 커널 변수 수정

```bash
echo fs.inotify.max_user_watches=1048576 | sudo tee -a /etc/sysctl.d/99-fs.conf
echo vm.max_map_count=1048576 | sudo tee -a /etc/sysctl.d/99-vm.conf
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

### 커널 변수 값 확인

```bash
# 커널 변수 목록 보기
sysctl -a
sysctl -aN

# 특정 변수의 현재 값 보기
sysctl -n vm.max_map_count

# apply values from all system directories
sysctl --system
```

### [docker 설치](https://docs.docker.com/engine/install/ubuntu/)

#### remove existing dockerr binaries

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

#### add Docker's official GPG key

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

#### add the repository to apt sources

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

#### install docker packages

- latest

```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl start docker
```

- specific version

```bash
# List the available versions:
$ apt-cache madison docker-ce | awk '{ print $3 }'

5:28.5.1-1~ubuntu.24.04~noble
5:28.5.0-1~ubuntu.24.04~noble
...

$ VERSION_STRING=5:28.5.1-1~ubuntu.24.04~noble
$ sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
$ sudo systemctl status docker
$ sudo systemctl start docker
```

#### dockerd에서 사용할 registry 추가

```bash
$ vi /etc/docker/daemon.json
{
    "insecure-registries": ["registry.k8s.io", "quay.io", "registry-1.docker.io"]
}
```

#### docker group에 사용자 추가

```bash
sudo usermod -aG docker ${USER}
```

### cri-dockerd 설치

#### downlaod items

- https://github.com/Mirantis/cri-dockerd/releases

cri-dockerd-*.amd64.tgz
Source code ( cri-dockerd-0.3.16.tar.gz ) 

#### 바이너리 설치

```bash
tar xvfz cri-dockerd-0.3.16.amd64.tgz
cd cri-dockerd/
mkdir -p /usr/local/bin
install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd
```

#### 서비스/docker 파일 설치

```bash
tar xvfz  cri-dockerd-0.3.16.tar.gz
cd cri-dockerd-0.3.16/
install packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable --now cri-docker.socket
```
