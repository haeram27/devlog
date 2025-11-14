# yum .repo 이해

- rocky linux는 [dnf package manager](https://docs.rockylinux.org/10/guides/package_management/dnf_package_manager/)를 통해서 package 관리를 한다.
- .repo 파일은 `dnf`의 패키지 저장소(repository)관련 설정 파일이다.
- `dnf`는 `/etc/yum.repos.d/*.repo` 파일을 참조하여 패키지 저장소 관련 정보를 획득한다.

## 참고

- [DNF Configuration Reference](https://dnf.readthedocs.io/en/latest/conf_ref.html)
- [DNF - Repo Options](https://dnf.readthedocs.io/en/latest/conf_ref.html#repo-options)
- [Rocky Linux Repositories - rockylinux.org](https://wiki.rockylinux.org/rocky/repo/)
- [kernel version up guide (ko) - ncloud-docs](https://docs.nhncloud.com/ko/Compute/Instance/ko/kernel-guide)
- [Check Linux OS repository settings - ncloud-docs](https://guide.ncloud-docs.com/docs/en/linux-os-repository-check)

- rocky default repo url
  - https://dl.rockylinux.org
- mirror list
  - https://mirrors.rockylinux.org/mirrormanager/mirrors/Rocky
    - KR
      - http://ftp.kaist.ac.kr/pub/rocky

## DNF 설정과 .repo 파일 관계

`dnf`는 계층적 구성 파일 참조 하여 최종 설정을 구성하는 형태이다.

- dnf는 다음의 설정을 참조 하여 최종 설정 값을 구한다.
- 우선 순위는 하위 항목이 높으며, 하위 위치의 설정 값이 상위 설정 값을 overwite 하여 최종 설정 값이 결정된다.
- `/etc/yum.repos.d`의 .repo 파일의 경우, repository 마다 적용될 dnf 설정 값을 별도로 지정할 수도 있다.

```text
- dnf 설정 default 값
- /etc/dnf/dnf.conf   # dnf 전역 옵션 설정 파일
- /etc/yum.repos.d/*.repo   # dnf repository별 옵션 설정 파일
```

- [현재 dnf 설정 값 확인 하기](https://docs.rockylinux.org/10/guides/package_management/dnf_package_manager/#dnf-config-manager)

```bash
dnf config-manager --dump
```

## [syntax](https://dnf.readthedocs.io/en/latest/conf_ref.html#repo-options)

.repo 파일 위치: /etc/yum.repos.d

```text
[baseos]
name=Rocky Linux $releasever - BaseOS
mirrorlist=https://mirrors.rockylinux.org/mirrorlist?arch=$basearch&repo=BaseOS-$releasever$rltype
#baseurl=http://dl.rockylinux.org/$contentdir/$releasever/BaseOS/$basearch/os/
gpgcheck=1
enabled=1
countme=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-Rocky-$releasever
```

- name
  - 저장소의 이름이며 변경하여도 무관합니다.
- mirrorlist
  - 패키지를 다운로드할 수 있는 미러 서버 목록을 제공하는 URL을 지정합니다.
  - 시스템의 아키텍처($basearch)와 Rocky Linux 버전($releasever)에 맞는 미러 서버 목록을 받아옵니다.
- baseurl
  - 패키지를 다운로드할 기본 URL을 지정합니다. 이 URL은 단일 서버를 가리키며, 해당 서버에서 직접 패키지를 다운로드합니다.
  - mirrorlist와 함께 지정되는 경우 mirrorlist가 우선하며, baseurl은 대체 옵션으로 동작합니다.
  - ⚠️ url은 대소문자를 구분. OS 배포 버전 마다 repository directory의 대소문자가 변경 되는 경우(예: Devel(8) -> devel(9))가 있으므로 주의 해야한다.
- gpgcheck
  - 다운로등 되는 rpm 패키지의 서명 인증 수행 여부를 지정합니다.
  - 값이 1인 경우 서명 인증 활성화, 0인 경우 비활성화입니다.
  - 활성화 하는 경우 유효한 gpgkey 항목도 지정하여야 합니다.
  - 보안이 엄격하게 요구되는 환경이 아닌 경우나 NAT 환경 등으로 네트워크를 통한 서명 검증이 어려운 경우에는 비활성화가 가능합니다..
- enabled
  - 이 저장소를 활성화 여부를 설정합니다.
  - 값이 1인 경우 활성화, 0인 경우 비활성화입니다.
- countme
  - Rocky Linux 사용 통계를 수집하는 기능입니다.
  - 값이 1인 경우 Rocky Linux 프로젝트 배포처에서 얼마나 많은 사용자가 있는지 파악할 수 있습니다.
  - 활성화 하는 경우 사용 통계 정보를 외부로 전송하게 되므로 원하지 않으면 비활성화 하는 것을 추천합니다.
- gpgkey
  - 패키지 서명을 검증할 GPG 키(공개키) 파일의 경로 또는 URL를 지정합니다. GPG(GNU Privacy Guard) 키는 rpm 패키지를 인증하는 데 사용하는 암호화 서명입니다.
  - GPG 키 파일은 remote repository에 공개된 파일을 사용하는 것이 좋으며 url 형식으로 지정합니다.
  - value example
    - `gpgkey=https://dl.rockylinux.org/$contentdir/RPM-GPG-KEY-Rocky-$releasever`  # remote official Rocky Linux repos
    - `gpgkey=https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-$releasever`  # for epel (Extra Packages for Enterprise Linux) of fedora project
    - `gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-$releasever`  # default gpg key installed with OS(Red Hat-based System) for rocky official repos

### variables in .repo file for rhel linux family

- [Rockylinux-Software Management-DNF](https://docs.rockylinux.org/10/books/admin_guide/13-softwares/#dnf-dandified-yum)
- [Variables in /etc/yum.repos.d/XXX.repo](https://unix.stackexchange.com/questions/19701/yum-how-can-i-view-variables-like-releasever-basearch-yum0)

#### RHEL/CentOS 8 and 9

```bash
/usr/libexec/platform-python -c '
import dnf, json
db = dnf.dnf.Base()
db.conf.substitutions.update_from_etc("/")
print(json.dumps(db.conf.substitutions, indent=2))'
```

#### Result Example

- rocky9

```bash
{
  "arch": "x86_64",
  "basearch": "x86_64",
  "releasever": "9",
  "contentdir": "pub/rocky",
  "rltype": "",
  "sigcontentdir": "pub/sig",
  "stream": "9-stream"
}
```

#### RHEL/CentOS 6 and 7

```bash
python -c 'import yum, json; yb = yum.YumBase(); print json.dumps(yb.conf.yumvar, indent=2)'
```

#### RHEL/CentOS 4 and 5

if you install python-simplejson

```bash
python -c 'import yum, simplejson as json; yb = yum.YumBase(); print json.dumps(yb.conf.yumvar, indent=2)'
```

#### otherwise

```bash
python -c 'import yum, pprint; yb = yum.YumBase(); pprint.pprint(yb.conf.yumvar, width=1)'
```

## rocky 9 사설 mirror respository 사용시 .repo 샘플 (/etc/yum.repos.d/kr.repos)

- 필수 repository
  - appstream
  - baseos
  - devel
  - extras
  - epel
  - docker

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

## repo 파일 변경후 시스템 반영

```bash
rm -rf /var/cache/dnf    # 기존 다운로드된 패키지의 메타데이터 캐시 삭제
dnf update -y            # 시스템 패키지를 최신으로 업데이트 및 의존성 정리. "dnf upgrade" 명령과 동일하며 내부적으로 upgrade로 처리됨
```