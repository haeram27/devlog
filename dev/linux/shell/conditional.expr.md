# shell에서 conditional expr 사용(TODO)

## if 와 test의 차이

`if` 는 command의 exitcode를 기준으로 참/거짓을 판단한다.  
exitcode가 0이면 참 그외 값이면 거짓으로 판단한다.  
`if`의 인자로는 command을 사용해야한다. 그래서 `test` 명령이 if의 인자로 사용될 수 있다.  
command가 여러개일 경우(`;`으로 구분된) 마지막 command의 결과를 기준으로 판단한다.

```bash
if [!] <command>; ... then
    <command;>...;
elif [!] <command>; ... then
    <command;>...
else
    <command;>...
fi
```

`test`(`[]` or `[[ ]]`) 명령어는 논리연산 결과를 기준으로 참/거짓을 판단한다.  
`test`의 인자로는 `논리연산 expr`이 사용된다.  

`if`와 `test`에 사용되는 부정연산자 `!`는 직후 오는 단일 연산의 결과에 대해서만 적용된다.
