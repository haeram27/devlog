- [$@과 $\* 차이](#과--차이)
  - [주요 차이점](#주요-차이점)
  - [핵심 차이](#핵심-차이)
    - [$\*과 $@ 명확한 비교](#과--명확한-비교)
    - [`"$@"` - 각 인자를 개별적으로 유지 (권장)](#---각-인자를-개별적으로-유지-권장)
    - [`"$*"` - 모든 인자를 IFS 첫문자를 구분자로하는 하나의 문자열로 결합](#---모든-인자를-ifs-첫문자를-구분자로하는-하나의-문자열로-결합)
  - [IFS의 영향](#ifs의-영향)
  - [실무 권장사항](#실무-권장사항)
  - [예시 1](#예시-1)
  - [예시 2](#예시-2)
    - [실행 결과](#실행-결과)
  - [예시 3](#예시-3)
  - [핵심 차이: `"$@"` vs `"$*"`](#핵심-차이--vs-)
    - [`"$@"` - 각 인자를 개별적으로 유지 (권장)](#---각-인자를-개별적으로-유지-권장-1)
    - [`"$*"` - 모든 인자를 IFS 첫문자를 구분자로하는 하나의 문자열로 결합](#---모든-인자를-ifs-첫문자를-구분자로하는-하나의-문자열로-결합-1)
  - [IFS의 영향](#ifs의-영향-1)
  - [실무 권장사항](#실무-권장사항-1)


# $@과 $* 차이

- Bash에서 `$@`와 `$*`는 모두 **스크립트나 함수에 전달된 모든 위치 매개변수**를 참조하지만, **인용부호와 함께 사용할 때 중요한 차이**가 발생
- 거의 모든 경우에 `"$@"`를 사용하는 것이 안전
  - 스크립트와 함수에서 입력 인자 단위의 문자열을 그대로 유지하려면 항상 `"$@"` 표현을 사용
  - `"$@"` 는 공백이 포함된 문자열 입력도 정상적으로 매개변수로 전달함

## 주요 차이점

| 표현식 | 동작 | 결과 |
|--------|------|------|
| `$*` | 모든 인자를 **`공백` 기준으로 분리하여 단어 단위로 확장** | `$1 $2 $3` (단어로 분리됨) |
| `$@` | 모든 인자를 **`공백` 기준으로 분리하여 단어 단위로 확장** | `$1 $2 $3` (단어로 분리됨) |
| `"$*"` | 모든 인자를 **`IFS`의 첫 문자를 구분자로하여 하나의 문자열로 결합** | `"$1c$2c$3"` (c는 IFS) |
| `"$@"` | 각 인자를 입력 그대로 **개별 문자열로 유지** | `"$1" "$2" "$3"` (입력 인자 단위로 유지) |

- shell의 default IFS 값은 공백(space)

## 핵심 차이

- 인용부호 없이는 차이가 없고, 인용부호와 함께 사용할 때만 차이가 발생
- 인용부호 없이 사용하면 `$*`와 `$@`는 동일하게 동작 (둘 다 입력을 단어로 분리)
- 인용부호와 함께 사용할 때만 차이가 발생
- 입력된 인자 그대로 사용하려면 `"$@"` 사용

### $*과 $@ 명확한 비교

```bash
./test.sh "arg1" "arg2" "arg3"

# 인용부호 없음
$*  → arg1 arg2 arg3 (3개 단어)
$@  → arg1 arg2 arg3 (3개 단어)  ← 동일!

# 인용부호 있음
"$*" → "arg1 arg2 arg3" (1개 문자열)
"$@" → "arg1" "arg2" "arg3" (3개 문자열)  ← 다름!
```

### `"$@"` - 각 인자를 개별적으로 유지 (권장)

```bash
#!/bin/bash
function process_files() {
    for file in "$@"; do  # 각 인자를 독립적으로 처리
        echo "Processing: $file"
    done
}

process_files "file 1.txt" "file 2.txt"
# 출력:
# Processing: file 1.txt
# Processing: file 2.txt
```

### `"$*"` - 모든 인자를 IFS 첫문자를 구분자로하는 하나의 문자열로 결합

결합시 IFS를 단어 사이에 구분자로 삽입

```bash
#!/bin/bash
function log_message() {
    echo "[LOG] $*"  # 모든 인자를 하나의 메시지로
}

log_message User logged in successfully
# 출력: [LOG] User logged in successfully
```

## IFS의 영향

```bash
#!/bin/bash
IFS=','

echo "$*"  # 쉼표로 구분된 하나의 문자열
echo "$@"  # 여전히 개별 인자들

# 예시:
set -- "apple" "banana" "cherry"
echo "$*"  # 출력: apple,banana,cherry
echo "$@"  # 출력: apple banana cherry
```

## 실무 권장사항

```bash
# ✅ 올바른 사용 - 공백이 포함된 인자를 안전하게 전달
my_command "$@"

# ❌ 잘못된 사용 - 공백이 포함된 인자가 분리됨
my_command $@

# 예시: 파일 이름에 공백이 있을 때
cp "$@" /destination/    # ✅ 안전
cp $@ /destination/      # ❌ 파일명이 깨짐
```

## 예시 1

```bash
#!/bin/bash

echo "=== 인용부호 없음 (동일한 동작) ==="
set -- "hello world" "foo bar" "test"

for arg in $*; do
    echo "[$arg]"
done
# 출력:
# [hello]
# [world]
# [foo]
# [bar]
# [test]

for arg in $@; do
    echo "[$arg]"
done
# 출력:
# [hello]
# [world]
# [foo]
# [bar]
# [test]

echo "=== 인용부호 있음 (다른 동작) ==="

for arg in "$*"; do
    echo "[$arg]"
done
# 출력:
# [hello world foo bar test]  ← 하나의 문자열

for arg in "$@"; do
    echo "[$arg]"
done
# 출력:
# [hello world]  ← 각각 개별 문자열로 유지
# [foo bar]
# [test]
```

## 예시 2

```bash
#!/bin/bash

# 테스트 스크립트
echo "=== without quoting ==="
echo "Count with \$*: " $(printf '%s\n' $* | wc -l)
printf 'Arg: [%s]\n' $*
echo
echo "Count with \$@: " $(printf '%s\n' $@ | wc -l)
printf 'Arg: [%s]\n' $@

echo -e "\n=== with quoting ==="
echo "Count with \"\$*\": " $(printf '%s\n' "$*" | wc -l)
printf 'Arg: [%s]\n' "$*"
echo
echo "Count with \"\$@\": " $(printf '%s\n' "$@" | wc -l)
printf 'Arg: [%s]\n' "$@"
```

### 실행 결과

```bash
./test.sh "hello world" "foo bar" "test"

# 출력:
# === without quoting ===
# Count with $*:  5    # 공백으로 분리되어 5개 단어
# Arg: [hello]
# Arg: [world]
# Arg: [foo]
# Arg: [bar]
# Arg: [test]

# Count with $@:  5    # 공백으로 분리되어 5개 단어
# Arg: [hello]
# Arg: [world]
# Arg: [foo]
# Arg: [bar]
# Arg: [test]

# === with quoting ===
# Count with "$*":  1    # 하나의 문자열
# Arg: [hello world foo bar test]

# Count with "$@":  3    # 3개의 독립된 인자
# Arg: [hello world]
# Arg: [foo bar]
# Arg: [test]
```

## 예시 3

```bash
#!/usr/bin/env bash

set -- "a,b" "c d" "e"

# default IFS=$' \t\n'
echo "IFS :"; echo -n "$IFS" | hexdump -C

printf "%*s" ${COLUMNS:-$(tput cols)} '' | tr ' ' '-'
echo $#
echo $1
echo $2
echo $3

printf "%*s" ${COLUMNS:-$(tput cols)} '' | tr ' ' '-'
echo -e "\n####### DEFAULT IFS"
# default IFS=$' \t\n'
echo "IFS :"; echo -n "$IFS" | hexdump -C

echo -e "\n###### non-quoting test"
echo
echo '### echo $* :'
echo $*
echo '### for-in $* :'
for p in $*; do
    echo [$p]
done

echo
echo '### echo $@ :'
echo $@
echo '### for-in $@ :'
for p in $@; do
  echo [$p]
done

echo -e "\n###### quoting test"
## quoting test
echo
echo '### echo "$*" :'
echo "$*"
echo '### efor-incho "$*" :'
for p in "$*"; do
    echo [$p]
done

echo
echo '### echo "$@" :'
echo "$@"
echo '### for-in "$@" :'
for p in "$@"; do
    echo [$p]
done

printf "%*s" ${COLUMNS:-$(tput cols)} '' | tr ' ' '-'
echo -e "\n####### IFS CHANGED with comma"
IFS=,
echo "IFS :"; echo -n "$IFS" | hexdump -C

echo -e "\n###### non-quoting test"
echo
echo '### echo $* :'
echo $*
echo '### for-in $* :'
for p in $*; do
    echo [$p]
done

echo
echo '### echo $@ :'
echo $@
echo '### for-in $@ :'
for p in $@; do
  echo [$p]
done

echo -e "\n###### quoting test"
## quoting test
echo
echo '### echo "$*" :'
echo "$*"
echo '### efor-incho "$*" :'
for p in "$*"; do
    echo [$p]
done

echo
echo '### echo "$@" :'
echo "$@"
echo '### for-in "$@" :'
for p in "$@"; do
    echo [$p]
done


printf "%*s" ${COLUMNS:-$(tput cols)} '' | tr ' ' '-'
echo -e "\n###### function context test"
func-context() {
    echo "$@"
    for p in "$@"; do
        echo [$p]
    done
}
func-context "$@"
func-context "1" "2 and 3" "4"
```

## 핵심 차이: `"$@"` vs `"$*"`

### `"$@"` - 각 인자를 개별적으로 유지 (권장)

```bash
#!/bin/bash
function process_files() {
    for file in "$@"; do  # 각 인자를 독립적으로 처리
        echo "Processing: $file"
    done
}

process_files "file 1.txt" "file 2.txt"
# 출력:
# Processing: file 1.txt
# Processing: file 2.txt
```

### `"$*"` - 모든 인자를 IFS 첫문자를 구분자로하는 하나의 문자열로 결합

결합시 IFS를 단어 사이에 구분자로 삽입

```bash
#!/bin/bash
function log_message() {
    echo "[LOG] $*"  # 모든 인자를 하나의 메시지로
}

log_message User logged in successfully
# 출력: [LOG] User logged in successfully
```

## IFS의 영향

```bash
#!/bin/bash
IFS=','

echo "$*"  # 쉼표로 구분된 하나의 문자열
echo "$@"  # 여전히 개별 인자들

# 예시:
set -- "apple" "banana" "cherry"
echo "$*"  # 출력: apple,banana,cherry
echo "$@"  # 출력: apple banana cherry
```

## 실무 권장사항

```bash
# ✅ 올바른 사용 - 공백이 포함된 인자를 안전하게 전달
my_command "$@"

# ❌ 잘못된 사용 - 공백이 포함된 인자가 분리됨
my_command $@

# 예시: 파일 이름에 공백이 있을 때
cp "$@" /destination/    # ✅ 안전
cp $@ /destination/      # ❌ 파일명이 깨짐
```
