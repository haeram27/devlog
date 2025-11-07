# vi에서 `%` 의미

vim에서 `%`는 다음 두가지 의미로 사용된다.

1. range: 전체 줄 범위 (`1,$`와 동일), Ex 모드에서 명령줄의 시작 위치(range expr position)
2. file name: 현재 파일의 이름, Ex 모드에서 명령줄의 시작이 아닌 위치(명령의 뒤 - 명령의 인자)

## `%`의 두 가지 다른 의미

### 1. Ex 명령의 **범위(range)**로 사용될 때

EX 모드에서 명령줄의 시작(range expr)에 위치하면 `1,$` 범위 의미

```vim
:%s/old/new/g    " %는 '1,$' (전체 줄 범위)를 의미
:%d              " %는 '1,$' (전체 줄 범위)를 의미
```

### 2. `:!` (외부 명령) 또는 파일 경로에서 사용될 때

EX 모드에서 명령줄의 시작에 위치 하지 않으면(명령의 뒤-명령의 인자위치-에 사용되면) 파일 이름 의미

```vim
:!%              " %는 현재 파일명(test.sh)으로 치환
:!python %       " %는 현재 파일명으로 치환
:w %.bak         " %는 현재 파일명으로 치환
```

## `%`가 파일명으로 치환되는 다른 예시들

```vim
# 현재 파일을 Python으로 실행
:!python %

# 현재 파일을 다른 이름으로 복사
:!cp % %.backup

# 현재 파일의 줄 수 세기
:!wc -l %

# 현재 파일을 cat으로 출력
:!cat %

# 현재 파일명 확인
:echo @%
:echo expand('%')

# 현재 파일의 전체 경로
:echo expand('%:p')

# 파일명만 (확장자 제외)
:echo expand('%:r')

# 확장자만
:echo expand('%:e')
```

## 혼동되는 부분 정리

```vim
" ===== 범위로 사용 (전체 줄) =====
:%s/old/new/g        " 1번 줄부터 마지막 줄까지 치환
:%d                  " 1번 줄부터 마지막 줄까지 삭제
:%!sort              " 1번 줄부터 마지막 줄을 sort 명령으로 필터링

" ===== 파일명으로 사용 =====
:!%                  " 현재 파일 (test.sh) 실행 시도
:!python %           " python test.sh 실행
:w %.bak             " test.sh.bak으로 저장
:e %:r.txt           " test.txt 열기
```

## 실전 예시

```bash
# test.sh 내용
#!/bin/bash
echo "Hello World"

# vim에서
:!chmod +x %         # chmod +x test.sh
:!./%                # ./test.sh 실행
# 출력: Hello World

:!bash %             # bash test.sh 실행
# 출력: Hello World

:!cat %              # cat test.sh 실행
# 출력: 파일 내용 표시
```