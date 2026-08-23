# fd

`fd`는 파일과 디렉토리 이름을 빠르게 검색하는 명령어이다. 기존 `find` 명령어보다 간결한 옵션과 사람이 읽기 쉬운 기본 출력 형식을 제공한다.

## 설치

```bash
# Debian/Ubuntu
sudo apt install fd-find

# Fedora
sudo dnf install fd-find

# Arch Linux
sudo pacman -S fd
```

Debian/Ubuntu에서는 패키지 이름이 `fd-find`이고 실행 명령어가 `fdfind`일 수 있다. `fd`로 사용하려면 alias를 설정한다.

```bash
alias fd=fdfind
```

## synopsis

```text
fd [OPTIONS] [PATTERN] [PATH...]
fd [OPTIONS] --exec COMMAND {} ...
fd [OPTIONS] --exec-batch COMMAND {}
```

- `PATTERN`은 기본적으로 정규식(regex) 패턴으로 해석된다.
- `PATH`를 생략하면 현재 디렉토리(`.`)부터 하위 디렉토리를 재귀적으로 검색한다.
- 기본적으로 숨김 파일과 `.gitignore`에 의해 무시되는 파일은 검색하지 않는다.
- 검색 결과는 기본적으로 색상과 파일 타입에 따른 표시를 포함할 수 있다.

## 기본 검색

```bash
# 현재 디렉토리 하위에서 이름에 'config'가 포함된 파일과 디렉토리 검색
fd config

# 특정 디렉토리 하위에서 검색
fd config /etc

# 이름이 정확히 app.conf인 항목 검색
fd '^app\\.conf$'

# 파일만 검색
fd --type f config

# 디렉토리만 검색
fd --type d config
```

`fd`는 기본적으로 파일명과 디렉토리명만 검색한다. 경로 전체를 패턴과 비교하려면 `-p, --full-path`를 사용한다.

```bash
# 파일명만 비교: src/api/user.go의 'api'와는 매칭되지 않을 수 있음
fd api

# 경로 전체를 비교
fd -p 'src/.*/user\\.go$'
```

## 대소문자와 패턴

패턴에 대문자가 없으면 대소문자를 구분하지 않는 smart case가 기본 적용된다. 패턴에 대문자가 포함되면 대소문자를 구분한다.

```bash
# Hello와 hello를 모두 검색
fd hello

# 대소문자를 구분하지 않고 검색
fd -i hello

# 정규식 대신 glob 패턴 사용
fd -g '*.log'
fd -g 'test_*.py'
```

- `-i, --ignore-case`: 대소문자를 구분하지 않음
- `-s, --case-sensitive`: 대소문자를 구분함
- `-g, --glob`: PATTERN을 glob 패턴으로 해석

## 숨김 파일과 ignore 대상

```bash
# 숨김 파일도 포함
fd -H config

# .gitignore 대상도 포함
fd -I config

# 숨김 파일과 ignore 대상 모두 포함
fd -u config
```

- `-H, --hidden`: 숨김 파일과 디렉토리를 검색 대상에 포함
- `-I, --no-ignore`: ignore 파일의 규칙을 무시
- `-u, --unrestricted`: `-H`와 `-I`를 함께 적용

## 확장자와 파일 타입

```bash
# 확장자가 rs인 파일 검색
fd -e rs

# 여러 확장자 검색
fd -e js -e ts

# 이름 패턴과 확장자를 함께 사용
fd 'config' -e yaml

# 실행 가능한 파일 검색
fd --type x

# 빈 파일 또는 빈 디렉토리 검색
fd --type e
```

주요 `--type` 값은 다음과 같다.

| 값 | 검색 대상 |
|---|---|
| `f` | 일반 파일(file) |
| `d` | 디렉토리(directory) |
| `l` | 심볼릭 링크(symbolic link) |
| `x` | 실행 가능한 파일(executable) |
| `e` | 빈 파일 또는 빈 디렉토리(empty) |

## 검색 결과 제어

```bash
# 결과를 최대 10개로 제한
fd -u -l 10 config

# 검색 깊이를 현재 디렉토리부터 2단계로 제한
fd --max-depth 2 config

# 지정한 디렉토리 바로 아래만 검색
fd --max-depth 1 config ./src

# 파일 타입과 경로를 함께 출력하지 않고 절대 경로 출력
fd -a config
```

- `-l, --max-results`: 출력할 최대 결과 수
- `-d, --max-depth`: 검색할 최대 깊이
- `-a, --absolute-path`: 결과를 절대 경로로 출력
- `-p, --full-path`: 파일명뿐 아니라 전체 경로를 패턴과 비교
- `-0, --print0`: 결과를 NULL 문자(`\\0`)로 구분하여 출력

## 다른 명령어와 연동

파일명에 공백이나 특수 문자가 포함될 수 있으면 `-0`과 `xargs -0`을 함께 사용한다.

```bash
# 검색한 파일을 안전하게 wc에 전달
fd -e log -0 | xargs -0 wc -l

# 검색한 파일 목록을 tar에 전달
fd -e jpg -0 | xargs -0 tar -cvf images.tar
```

## 검색 결과에 명령어 실행

`-x, --exec`는 검색 결과마다 명령어를 실행한다. `{}`는 현재 검색 결과로 대체된다.

```bash
# 실행 전에 처리할 명령을 확인
fd -e tmp -x echo rm '{}'

# 실제로 .tmp 파일 삭제
fd -e tmp -x rm '{}'
```

파일을 변경하거나 삭제하는 명령은 먼저 `echo`로 결과를 확인한 후 실행한다.

`-X, --exec-batch`는 검색 결과 전체를 하나의 명령어 인자로 전달한다.

```bash
# 검색한 모든 Markdown 파일을 한 번에 wc에 전달
fd -e md -X wc -l
```

## 심볼릭 링크와 디렉토리 제외

```bash
# 심볼릭 링크가 가리키는 대상까지 따라가며 검색
fd -L config

# node_modules 디렉토리를 검색 대상에서 제외
fd config --exclude node_modules

# 여러 디렉토리 제외
fd config -E node_modules -E target
```

- `-L, --follow`: 심볼릭 링크를 따라감
- `-E, --exclude`: 지정한 glob 패턴을 검색 대상에서 제외

## find와 비교

```bash
# find
find . -type f -name '*.log'

# fd
fd -t f -e log
```

`fd`는 간단한 파일명 검색에 적합하고, 복잡한 조건식, 소유자/권한 조건, `-prune`과 같은 세밀한 탐색 조건이 필요하면 `find`가 더 적합하다.

## 도움말과 버전

```bash
fd --help
fd --version
```
