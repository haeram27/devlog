# dnf - RHEL package manager

## package naming (dash'-'로 구분)

syntax:
```<package_name>[-<version>[-<release>.<os_dist>]]```

예: nginx-1.20.1-10.el9

## 현재 설치된 package 검색

```bash
dnf list --installed | grep <package>
```

## repository의 설치 가능한 package 검색

### 패키지 버전 정렬

```bash
dnf list --available --showduplicates <package>
```

### list-available과 동일 결과 패키지 이름 정렬

```bash
dnf repoquery --showduplicates <package>
```

### 주어진 term이 package 이름이나 descriptions에 포함되어 있는 모든 패키지 리스트 출력

```bash
dnf search --showduplicates <term>
```

options:

- `--showduplicates`
  - package에 여러 버전이 있을 때 모든 버전 표시

ex:

```bash
$ dnf list --available --showduplicates mongodb-org
#package-name        #version       #repository-name
mongodb-org.x86_64   7.0.18-1.el9   mongodb-org-7.0
mongodb-org.x86_64   8.0.0-1.el9    mongodb-org-8.0
```

#### repository-name

- `dnf`와 같은 패키지 관리자가 소프트웨어 패키지를 다운로드하는 서버의 단축된 이름 또는 식별자
- 쉽게 말해, **패키지들이 보관되어 있는 창고의 이름**
- 시스템에는 여러 개의 리포지토리(창고)가 등록 가능. 예를 들면:
  - `appstream`: RHEL/CentOS의 공식 애플리케이션 스트림 리포지토리
  - `baseos`: RHEL/CentOS의 기본 운영체제 패키지 리포지토리
  - `epel`: 추가적인 패키지들을 제공하는 외부 리포지토리
  - `mongodb-org-7.0`: MongoDB에서 직접 제공하는 MongoDB 7.0 버전용 리포지토리
- `dnf`는 이 창고들의 목록을 가지고 있다가, 사용자가 패키지 설치를 요청하면 각 창고를 확인하여 요청한 패키지가 있는지 찾고, 있다면 해당 창고에서 패키지를 가져와 설치

## package 설치

```bash
dnf install -y <package_name>[-<version>[-<release>.<os_dist>]]
```

options:

- `--allowerasing`
  - allow erasing of installed packages to resolve dependencies
  - 종속성 해결을 위해 이미 설치된 패키지를 삭제 가능 = 상위 버전 업그레이드 허용 옵션

- `-b`, `--best`
  - try the best available package versions in transactions.

- `--nobest`
  - do not limit the transaction to the best candidate

ex:

```bash
dnf install nginx-1.20.1-10.el9
```

- `nginx`
  - 패키지 이름
- `1.20.1`
  - 버전
- `10.el9`
  - \<릴리즈 번호>.\<OS 배포판 정보\>

```bash
dnf install httpd-2.4.57
```

- `httpd`
  - 패키지 이름
- `2.4.57`
  - 버전

## package [모듈](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/9/html/managing_software_with_the_dnf_tool/con_modules_assembly_distribution-of-content-in-rhel-9) 비활성화

```bash
dnf -qy module disable postgresql
dnf module list postgresql
```

- os repository에서 기본 제공되는 모듈(지정 패키지)가 설치되지 않도록 설정
  - 기본 제공 패키지가 오래된 버전일 때
  - 다른 repository(제조사 제공 repository)에서 설치하려고 할 때
  - 커스텀 빌드 또는 별도의 rpm으로 설치하려고 할 때

## [dnf command list](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_software_with_the_dnf_tool/assembly_yum-commands-list_managing-software-with-the-dnf-tool)

### Commands for listing content

| **Command** | **Description** |
| --- | --- |
| dnf search \<term\>                                              | Search for a package by using term related to the package. |
| dnf repoquery \<package\>                                        | Search for enabled **DNF**repositories for a selected package and its version. |
| dnf list                                                         | List information about all installed and available packages. |
| dnf list --installed<br>dnf repoquery --installed                | List all packages installed on your system. |
| dnf list --available \<package\><br>dnf repoquery                | List all packages in all enabled repositories that are available to install.<br>$ dnf list --available --showduplicates \<package\> |
| dnf repolist                                                     | List all enabled repositories on your system. |
| dnf repolist --disabled                                          | List all disabled repositories on your system. |
| dnf repolist --all                                               | List both enabled and disabled repositories. |
| dnf repoinfo                                                     | List additional information about the repositories. |
| dnf info \<package_name\><br>dnf repoquery --info \<package_name\> | Display details of an available package. |
| dnf repoquery --info --installed \<package_name\>                | Display details of a package installed on your system. |
| dnf module list                                                  | List modules and their current status. |
| dnf module info \<module_name\>                                  | Display details of a module. |
| dnf module list \<module_name\>                                  | Display the current status of a module. |
| dnf module info --profile \<module_name\>                        | Display packages associated with available profiles of a selected module. |
| dnf module info --profile \<module_name\>:\<stream\>             | Display packages associated with available profiles of a module by using a specified stream. |
| dnf module provides \<package\>                                  | Determine which modules, streams, and profiles provide a package.<br>Note that if the package is available outside any modules, the output of this command is empty. |
| dnf group summary                                                | View the number of installed and available groups. |
| dnf group list                                                   | List all installed and available groups. |
| dnf group info \<group_name\>                                    | List mandatory and optional packages included in a particular group. |

### Commands for installing content

| **Command** | **Description** |
| --- | --- |
| dnf install *package_name*                                                                        | Install a package.<br>If the package is provided by a module stream, dnf resolves the required module stream and enables it automatically while installing this package. This also happens recursively for all package dependencies. If more module streams satisfy the requirement, the default ones are used. |
| dnf install *package_name_1* *package_name_2*                                                     | Install multiple packages and their dependencies simultaneously. |
| dnf install *package_name.arch*                                                                   | Specify the architecture of the package by appending it to the package name when installing packages on a *multilib*system (AMD64, Intel 64 machine). |
| dnf install */usr/sbin/binary_file*                                                               | Install a binary by using the path to the binary as an argument. |
| dnf install */path/*                                                                              | Install a previously downloaded package from a local directory. |
| dnf install *package_url*                                                                         | Install a remote package by using a package URL. |
| dnf module enable *module_name:stream*                                                            | Enable a module by using a specific stream.<br>Note that running this command does not install any RPM packages. |
| dnf module install *module_name:stream*<br>dnf install @*module_name:stream*                      | Install a default profile from a specific module stream.<br>Note that running this command also enables the specified stream. |
| dnf module install *module_name:stream/profile*<br>dnf install @*module_name:stream/profile* | Install a selected profile by using a specific stream. |
| dnf group install *group_name*                                                                    | Install a package group by a group name. |
| dnf group install *group_ID*                                                                      | Install a package group by the groupID. |

### Commands for installing content

| **Command** | **Description** |
| --- | --- |
| dnf remove *package_name*                      | Remove a particular package and all dependent packages. |
| dnf remove *package_name_1package_name_2*      | Remove multiple packages and their unused dependencies simultaneously. |
| dnf group remove group_name                    | Remove a package group by the group name. |
| dnf group remove *group_ID*                    | Remove a package group by the groupID. |
| dnf module remove --all *module_name:stream*   | Remove all packages from the specified stream.<br>Note that running this command can remove critical packages from your system. |
| dnf module remove *module_name:stream/profile* | Remove packages from an installed profile. |
| dnf module remove *module_name:stream*         | Remove packages from all installed profiles within the specified stream. |
| dnf module reset *module_name*                 | Reset a module to the initial state.<br>Note that running this command does not remove packages from the specified module. |
| dnf module disable *module_name*               | Disable a module and all its streams.<br>Note that running this command does not remove packages from the specified module. |
