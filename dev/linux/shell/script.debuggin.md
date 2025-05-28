# bash script debugging

## script debugging

Debugging 대상 스크립트의 최상에 다음과 같이 정의

```bash
# set shell debugging options
set -euvxo pipefail

# store last executed command
prev_cmd="none"
current_cmd="none"
trap 'prev_cmd=$current_cmd; current_cmd=$BASH_COMMAND' DEBUG

# print last executed command just before terminating script
destructor() {
    echo "### Print last executed commands"
    echo "prev_cmd: ${prev_cmd}"
    echo "current_cmd: ${current_cmd}"
}
trap destructor EXIT
```

- `-e` : "set -o errexit"와 같음, 명령의 exitcode나 함수의 return 값이 비정상 종료코드(0이 아닌 정수)를 반환하는 경우 스크립트 종료, 단 if 명령의 인자로 사용된 명령과 함수에는 적용 안됨
- `-u` : `nounset`, 정의되지 않은(미선언) 변수를 사용하면 에러 출력
- `-v` : `verbose`, 명령라인 실행 전 변수확장(Extention) 직전 명령 라인 출력
- `-x` : `xtrace`, 명령라인 실행 전 변수확장(Extention) 직후 명령 라인 출력
- `-o pipefail` : 파이프라인 내 어느 하나라도 실패하면 전체 실패로 간주(스크립트 종료)
- trap DEBUG : 각 명령 실행 직전에 발생되어 실행 대상 명령을 변수에 저장
- trap EXIT : 스크립트 종료 직전에 발생되어 저장된 명령어 출력, 스크립트 종료 직전에 실행된 명령어 확인 가능
