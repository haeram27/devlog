# update-alternatives

### List all master alternative names

- 동일한 명령 실행파일이 여러 개(다른 버전 등)이 설치 되어 있을때 어떤 실행파일을 default로 실행할지 지정하는 명령어
- Ubuntu에서 update-alternatives로 지정된 명령은 `PATH`로 지정된 경로 보다 우선하여 실행됨
- `PATH`로 지정된 경로의 명령어가 실행되지 않는다면 `update-alternatives` 명령을 우선 확인 필요

```bash
update-alternatives --get-selections
```

### To see all symbolic links managed in /etc/alternatives

```bash
ls -l /etc/alternatives
```

### Install a new alternative path

```bash
sudo update-alternatives --install <link> <name> <path> <priority>
# Example:
# sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 10
# sudo update-alternatives --install /usr/bin/java java /opt/jdk/21.0.2/bin/java 21
# sudo update-alternatives --install /usr/bin/gradle gradle /opt/gradle/9.3.0/bin/gradle 9
```

- `<priority>` 값은 지정 범위가 없으므로 주로 package의 버전 값 등을 이용하여 사용함

### List all choices for a specific command

```bash
update-alternatives --list <command_name>
# Example: update-alternatives --list java
```

### Configure/Switch default version interactively

```bash
sudo update-alternatives --config <command_name>
# Example: sudo update-alternatives --config gcc
```

### Display status and available paths for a command

```bash
update-alternatives --display <command_name>
```
