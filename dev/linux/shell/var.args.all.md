# $@과 $* 차이

Bash에서 `$@`와 `$*`는 모두 **스크립트나 함수에 전달된 모든 위치 매개변수**를 참조하지만, **인용부호와 함께 사용할 때 중요한 차이**가 있습니다.
거의 모든 경우에서 `"@"` 표현을 사용하는 것이 안전하고 올바른 결과를 출력할 수 있습니다.

## 주요 차이점

| 표현식 | 동작 | 결과 |
|--------|------|------|
| `$*` | 모든 인자를 **공백으로 분리하여 확장** | `$1 $2 $3` (단어 분리됨) |
| `$@` | 모든 인자를 **공백으로 분리하여 확장** | `$1 $2 $3` (단어 분리됨) |
| `"$*"` | 모든 인자를 **IFS의 첫 문자로 결합한 하나의 문자열** | `"$1c$2c$3"` (하나의 단어, c는 IFS) |
| `"$@"` | 각 인자를 **개별 문자열로 유지** | `"$1" "$2" "$3"` (여러 단어) |

## 핵심 차이

**인용부호 없이 사용하면 `$*`와 `$@`는 동일하게 동작합니다** (둘 다 단어 분리 발생)

**인용부호와 함께 사용할 때만 차이가 발생합니다:**

## 명확한 비교

```bash
./test.sh "arg1" "arg2" "arg3"

# 인용부호 없음
$*  → arg1 arg2 arg3 (3개 단어)
$@  → arg1 arg2 arg3 (3개 단어)  ← 동일!

# 인용부호 있음
"$*" → "arg1 arg2 arg3" (1개 문자열)
"$@" → "arg1" "arg2" "arg3" (3개 문자열)  ← 다름!
```

지적해주셔서 감사합니다. **인용부호 없이는 차이가 없고, 인용부호와 함께 사용할 때만 차이가 발생한다**는 점이 핵심입니다!


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

### `"$*"` - 모든 인자를 하나로 결합

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

## 요약

- **`"$@"`**: 각 인자를 독립적으로 유지 → **대부분의 경우 이것을 사용하세요**
- **`"$*"`**: 모든 인자를 하나의 문자열로 결합
- **인용부호 없이** 사용하면 둘 다 비슷하게 동작하지만, **공백 처리 문제**가 발생할 수 있습니다

**베스트 프랙티스**: 거의 모든 경우에 `"$@"`를 사용하는 것이 안전합니다!