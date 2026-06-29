- [special parameters](#special-parameters)
  - [$$](#)
  - [$?](#-1)
  - [$!](#-2)
  - [$-](#-)
  - [$\_](#_)
  - [$#](#-3)


# special parameters

## \$$

- 현재 스크립트의 PID (Process ID)
- \$$ PID of current process/shell
- \${PPID} PID of current process's paren

> Expands to the process ID of the shell.  
> In a () subshell, it expands to the process ID of the current shell, not the subshell.

*사용예:*

- \$$의 경우 쉘프로그램 내에서 고유성을 가진 ID 스트링을 만드는데 주로 사용된다.
- \$$는 PID이므로 고유한 값은 아니며(보통 두자리 정수값이 된다) PID는 다른 프로세스에서 언제나 재사용되므로 이 값만으로 고유성을 가진 ID를 만들수는 없다.
- 다만 $$$(date +%s%N)과 같이 사용하면 PID+SECOND+NANOSECOND 값으로 구성되는 고유성을 가진 ID를 만들수 있다.

```bash
echo "$$$(date +%s%N)"

# 실행결과
# 1532014202392623100
```

## $?

- 바로 직전에 실행된 명령어, 함수, 스크립트 등 자식 프로세스의 종료 상태(exit status-code) 값
- true = success = 0
- false = failed = 1 [or not 0]

> Expands to the exit status(code) of the most recently executed foreground pipeline.

*사용예:*

check last exitcode using &&,||

```bash
true && echo success || echo failed     // success
false && echo success || echo failed    // failed
```

check last exitcode using [[]]

```bash
true
[[ $? -eq 0 ]] && echo success || echo failed
```

참고로 if 문을 사용하면 명령의 exitcode를 참조 하여 분기 할 수 있다.

```bash
if true; then echo y; else echo n; fi    # y
if false; then echo y; else echo n; fi    # n
```

## $!

- & 메타문자를 이용하여 가장 최근에 백그라운드로 실행된 process id를 나타냅니다.

*사용예:*

```bash
sleep 20 &; echo background-pid: $!; ps -ef | grep $!
```

- | 파이프로 연결된 명령 그룹일 경우 마지막 process id가 표시됩니다.

```bash
(echo hello | sleep 20) &; echo background-pid: $!; ps -ef | grep $!
```

> Expands to the process ID of the job most recently placed into the background, whether executed  as  an asynchronous command or using the bg builtin (see JOB CONTROL below)

## $-

- 현재 쉘에 적용된 옵션 플래그

> Expands to the current option flags as specified upon invocation, by the set builtin command, or  those set by the shell itself (such as the -i option).

*사용예:*


```bash
# 현재 set 명령에 설정된 옵션 표시
$ echo $-
569NXZilmsy

# -f(noglob) 옵션 설정 여부 테스트
$ [[ $- == *f* ]] && echo set || echo unset
```

## $_

- 이전 명령에서 사용된 마지막 인수를 값으로 가집니다.
- 사용된 인수가 없으면 명령을 리턴합니다.

> At shell startup, set to the absolute pathname used to invoke the shell or shell script being executed as  passed in the environment or argument list.  
> Subsequently, expands to the last argument to the previous command, after expansion.  
> Also set to the full pathname used to invoke each command executed and
> placed  in the environment exported to that command.  
> When checking mail, this parameter holds the name of the mail file currently being checked.

*사용예:*


```bash
mkdir -p foo/bar/zoo && cp myfile $_
```

## $#

number of positional parameters