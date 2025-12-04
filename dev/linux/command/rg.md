# ripgrep (rg)

- 지정한 파일/디렉토리에서 regex 패턴 매칭 방식의 contents search
- rust로 개발되었으므로 regex pattern도 rust의 문서를 참고

## doc links

- [github](https://github.com/BurntSushi/ripgrep?tab=readme-ov-file)
- [userguide](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md)
- [regex syntax](https://docs.rs/regex/latest/regex/index.html#syntax)

## terms

### type

- 검색 대상 파일 형식 모음
- 검색 대상 포함/제외 옶션으로 단일 glob 형식은 `--glob` 옵션으로 지정
- 검색 대상의 묶음을 지정할 때에는 type 사용
- ripgrep 설정 파일에 미리 명시하여 사전 정의 가능함
- 관련 옵션
  - `--type-list`: 현재 정의된 type list 출력 (default, 사용자 정의 모두 포함)
  - `-t, --type`: 호출에 사용할 type 지정
  - `--type-add`: type 추가

## [Configuration file](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md#configuration-file)

- ripgrep은 고정된 설정 파일 위치가 없으며, 설정 파일을 사용하려면 반드시 `RIPGREP_CONFIG_PATH` 환경 변수를 설정 필요

```bash
export RIPGREP_CONFIG_PATH=$HOME/.ripgreprc
```

### 작성시 주의 사항

- value 지정시 반드시 `=` 사용

```txt
--type-add=bin:*.{exe,dll,bin,so,o,a,jar,class}
-T=bin
```

- 옵션 지정 라인에 comment 함께 사용 금지

```txt
--column # show column number    ❌ 에러

# show column number
--column                         ✅ 정상
```

## options

### type-add

- type(검색 대상 묶음(bunch))을 추가

argument

```text
type:glob ex. html:*.html
```

command-line

- pattern에 quoting(`''`) 사용 필요
  - quoting 미사용시 wildcard(`*`)문자가 shell에서 확장되므로 syntax 오류 발생

```bash
rg --type-add 'web:*.{html,css,js}' -tweb title
rg --type-add='web:*.{html,css,js}' -tweb title
```

type 추가 여부 확인

```bash
rg --type-add 'web:*.{html,css,js}' --type-list | grep web
```

config-file

- `option=value`와 같이 equal(`=`) 사용 필요
- argument에 quoting(`''`) 사용 금지

```text
--type-add=web:*.{html,css,js}
```
