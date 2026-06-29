# update-alternatives

- 동일한 명령 실행 파일이 여러 개(다른 버전 등)이 설치 되어 있을때 어떤 실행파일을 default로 실행할지 지정하는 명령어
- Ubuntu에서 update-alternatives로 지정된 명령은 `PATH`로 지정된 경로 보다 우선하여 실행됨
- `PATH`로 지정된 경로의 명령어가 실행되지 않는다면 `update-alternatives` 명령을 우선 확인 필요

## 시스템이 참조 하는 명령어 위치

`update-alternatives`를 사용하는 OS 명령어는 다음의 순서로 symlink를 구성되어 있다.

**java 예:**

사용자가 java 명령을 실행하면 update-alternatives를 통해 link > update-alternatives > path 순서로 등록된 symbolic link를 통해서 최종 path의 실제 명령어가 실행된다.

|link|/etc/alternatives|path|
|---|---|---|
|/usr/bin/java|/etc/alternatives/java|/opt/jdk/25.0.2/bin/java|


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
# sudo update-alternatives --install /usr/bin/java java /opt/jdk/25.0.2/bin/java 25
# sudo update-alternatives --install /usr/bin/gradle gradle /opt/gradle/9.3.0/bin/gradle 9
```

|인자|설명|예|
|---|---|---|
|`<link>`|사용자가 실행할 명령어 경로|/usr/bin/java|
|`<name>`|alternatives 시스템에서 관리하는 그룹 이름|java|
|`<path>`|실제 설치되어 있는 실행 파일의 위치|/usr/lib/jvm/java-25-openjdk/bin/java|
|`<priority>`|자동 모드일 때 우선순위 (숫자가 높을수록 우선)<br>값은 지정 범위가 없으므로 주로 package의 버전 값 등을 이용하여 사용함|25|

### 등록하려는 명령의`<link>`를 찾는 방법

이미 시스템에 등록되어 사용 중인 명령어의 버전을 추가하려는 것이라면, 현재 그 명령어가 어디에 있는지 확인하면 됩니다. 

- which <명령어>: 현재 실행 파일의 위치를 알려줍니다. (PATH 경로를 검색하여 발견하는 첫번째 실행파일)
  - 예: which java 입력 시 /usr/bin/java가 나오면 이 경로가 <link> 값이 됩니다.
- ls -l <경로>: 해당 경로가 심볼릭 링크인지 확인합니다. 보통 /usr/bin/ 아래의 파일들은 /etc/alternatives/를 가리키고 있습니다

## remove a new alternative path

aliternative 항목 삭제

```bash
sudo update-alternatives --remove <name> <path>
# Example:
# sudo update-alternatives --remove gcc /usr/bin/gcc-10
# sudo update-alternatives --remove java /opt/jdk/25.0.2/bin/java
# sudo update-alternatives --remove gradle /opt/gradle/9.3.0/bin/gradle
```

- `<priority>` 값은 지정 범위가 없으므로 주로 package의 버전 값 등을 이용하여 사용함