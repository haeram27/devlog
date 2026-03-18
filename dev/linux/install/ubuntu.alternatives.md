# update-alternatives

- 동일한 명령 실행 파일이 여러 개(다른 버전 등)이 설치 되어 있을때 어떤 실행파일을 default로 실행할지 지정하는 명령어
- Ubuntu에서 update-alternatives로 지정된 명령은 `PATH`로 지정된 경로 보다 우선하여 실행됨
- `PATH`로 지정된 경로의 명령어가 실행되지 않는다면 `update-alternatives` 명령을 우선 확인 필요

## List all master alternative names

현재 선택되어 있는 대표 실행파일과 경로 리스트

```bash
update-alternatives --get-selections
```

## To see all symbolic links managed in /etc/alternatives

`--get-selections`이 선택 되어 있는 아이템만 보여 준다면, `/etc/alternatives` 경로는 `alternatives`에서 관리되고 있는 모든 link의 리스트를 가지고 있음

```bash
ls -l /etc/alternatives
```

## List all choices for a specific command

특정 command의 aliternatives 리스트를 출력

```bash
update-alternatives --list <command_name>
# Example: update-alternatives --list java
```

## Configure/Switch default version interactively

특정 command의 대표 경로를 선택

```bash
sudo update-alternatives --config <command_name>
# Example: sudo update-alternatives --config gcc
```

## Display status and available paths for a command

지정한 command의 상세 내용 출력

```bash
update-alternatives --display <command_name>
```

## Install a new alternative path

새로운 aliternative 항목 추가

```bash
sudo update-alternatives --install <link> <name> <path> <priority>
# Example:
# sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 10
# sudo update-alternatives --install /usr/bin/java java /opt/jdk/21.0.2/bin/java 21
# sudo update-alternatives --install /usr/bin/gradle gradle /opt/gradle/9.3.0/bin/gradle 9
```

- `<priority>` 값은 지정 범위가 없으므로 주로 package의 버전 값 등을 이용하여 사용함

## remove a new alternative path

aliternative 항목 삭제

```bash
sudo update-alternatives --remove <name> <path>
# Example:
# sudo update-alternatives --remove gcc /usr/bin/gcc-10
# sudo update-alternatives --remove java /opt/jdk/21.0.2/bin/java
# sudo update-alternatives --remove gradle /opt/gradle/9.3.0/bin/gradle
```

- `<priority>` 값은 지정 범위가 없으므로 주로 package의 버전 값 등을 이용하여 사용함