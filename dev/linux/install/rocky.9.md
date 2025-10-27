# Rocky 9

## 설치 환경

Rocky 9.x

## 설정

### sudoers 설정 (non root 사용자인 경우)

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

- rocky는 network-manager를 렌더러로 사용하므로 nmtui, nmcli 명령어 사용

```bash
nmtui
```

#### DNS server list

- https://public-dns.info/

### yum repo url

- rocky default repo url
  - http://dl.rockylinux.org

- mirror list
  - https://mirrors.rockylinux.org/mirrormanager/mirrors/Rocky
    - KR
      - http://ftp.kaist.ac.kr/pub/rocky

```text
[AppStream]
name=Rocky Linux $releasever - AppStream
baseurl=http://ftp.kaist.ac.kr/$contentdir/$releasever/AppStream/$basearch/os/
enabled=1
gpgcheck=0
gpgkey=https://ftp.kaist.ac.kr/$contentdir/RPM-GPG-KEY-Rocky-$releasever

[BaseOS]
name=Rocky Linux $releasever - BaseOS
baseurl=http://ftp.kaist.ac.kr/$contentdir/$releasever/BaseOS/$basearch/os/
enabled=1
gpgcheck=0

[extras]
name=Rocky Linux $releasever - extras
baseurl=http://ftp.kaist.ac.kr/$contentdir/$releasever/extras/$basearch/os/
enabled=1
gpgcheck=0

[devel]
name=Rocky Linux $releasever - devel
baseurl=http://ftp.kaist.ac.kr/$contentdir/$releasever/devel/$basearch/os/
enabled=1
gpgcheck=0

[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/rhel/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/rhel/gpg
```

- gpgcheck가 반드시 필요한 경우가 아니면 `gpgcheck=0` 설정

#### update packages

```bash
dnf update && dnf upgrade -y
```

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

## 커널 변수 수정

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

### [docker 설치](https://docs.docker.com/engine/install/rhel/)

TBD