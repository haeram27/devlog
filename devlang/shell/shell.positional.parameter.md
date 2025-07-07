# positional parameter

## all parameters

Alyways just use `"$@"` to specify all parameters.  
Only if IFS separated one line string of all parameters is required then use `"$*"`.

|expr|out|
| --- | --- |
| $*   | 모든 위치 인자를 `하나의 문자열`로 확장하고, 그걸 IFS 기준으로 분해 |
| $@   | `각 위치 인자`를 개별적으로 확장하고, 각각을 IFS 기준으로 분해 |
| "$*" | 모든 인자 결합하여 `하나의 문자열`로 확장 (연결 지점에 IFS를 삽입) |
| "$@" | `각 위치 인자`를 어떠한 결합이나 분해없이 개별 인자로 유지 (안전한 방식) |

test with parameters of "arg1", "arg2 with space", "arg3" is follows:

### all parameters test

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
