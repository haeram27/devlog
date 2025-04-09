## 8/16진수 byte 표현 문자 이스케이핑
ANSI-C quoting은 `$'...'` 형식을 사용하며 지정된 8/16진수 ascii 코드 표현을 문자 이스케이핑 해준다.

bash에서 변수에 8/16진수 코드 값으로 문자를 할당하고 싶다면 ANSI-C quoting 또는 printf를 사용해야 한다.
zsh에서는 ANSI-C quoting이 없이 일반 double-quote("")로도 8/16진수 코드값을 문자 이스케이핑 해준다.

### ansi-c quoting을 사용한 16진수 코드 character escaping
bash
```bash
$ temp=$'\x68\x65\x6C\x6C\x6F'
$ echo $temp
hello
```

### printf를 사용한 16진수 코드 character escaping
bash
```bash
$ temp="$(printf '\x68\x65\x6C\x6C\x6F')"
$ echo $temp
hello
```

### zsh의 oct/hex 코드 문자 이스케이핑
zsh는 이뿐만 아니라 ansi-c quoting 이나 printf 방식도 모두 사용 가능하다.
```bash 
$ temp="\x68\x65\x6C\x6C\x6F"
$ echo $temp
hello
```