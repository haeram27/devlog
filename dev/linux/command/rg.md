# ripgrep (rg)

- 지정한 파일/디렉토리내에서 `regex` 패턴 매칭 방식의 contents search
- rust로 개발되었으므로 regex pattern도 rust의 문서를 참고

## doc links

- [github](https://github.com/BurntSushi/ripgrep?tab=readme-ov-file)
- [userguide](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md)
- [regex syntax](https://docs.rs/regex/latest/regex/index.html#syntax)
- [daleseo.ripgrep(ko)](https://daleseo.com/ripgrep/)

## terms

### type

- 검색 대상 파일 형식 모음
- 검색 대상 파일 또는 디렉토리 이름의 포함/제외 옵션으로 단일 glob 형식은 `-g, --glob` 옵션으로 지정
- 검색 대상의 확장자 묶음을 지정할 때에는 `type(-t, --type)` 사용
- `type`은 의미있는 확장자의 묶음(bunch)이며, ripgrep 설정 파일(`RIPGREP_CONFIG_PATH` 환경변수로 지정)에 사전 정의 가능함

## [Configuration file](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md#configuration-file)

- ripgrep 기본 설정 파일 위치가 없으며, 설정 파일을 사용하려면 반드시 `RIPGREP_CONFIG_PATH` 환경 변수를 설정 필요

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

## synopsis

```text
       rg [OPTIONS] PATTERN [PATH...]

       rg [OPTIONS] -e PATTERN... [PATH...]

       rg [OPTIONS] -f PATTERNFILE... [PATH...]

       rg [OPTIONS] --files [PATH...]

       rg [OPTIONS] --type-list

       command | rg [OPTIONS] PATTERN

       rg [OPTIONS] --help

       rg [OPTIONS] --version
```

- PATTERN은 검색 word에 대한 pattern을 의미하며, regex pattern 사용
- ripgrep은 기본적으로 지정된 PATH 하위로 recursive로 동작한다.

## type 관리

- type(검색 대상 묶음(bunch))을 추가

### argument

```text
type:glob ex. html:*.html
```

### 현재 정의된 type 리스트 확인 (`--type-list`)

```bash
rg --type-add 'web:*.{html,css,js}' --type-list | grep web
```

### command-line 에서 type 추가 하기

- pattern에 quoting(`''`) 사용 필요
  - quoting 미사용시 wildcard(`*`)문자가 shell에서 확장되므로 syntax 오류 발생

```bash
rg --type-add 'web:*.{html,css,js}' -tweb title
rg --type-add='web:*.{html,css,js}' -tweb title
```

### type 추가 여부 확인

```bash
rg --type-add 'web:*.{html,css,js}' --type-list | grep web
```

### config-file

- `option=value`와 같이 equal(`=`) 사용 필요
- argument에 quoting(`''`) 사용 금지

```text
--type-add=web:*.{html,css,js}
```

## options

### type 관련 옵션

- `--type-add`: type 추가
- `--type-list`: 현재 정의된 type list 출력 (default, 사용자 정의 모두 포함)
- `-t, --type`: 검색시 include type
- `-T, --type-not`: 검색시 exclude type

- -t와 -T 옵션을 동시 사용할 경우, 조건의 AND 연산으로 최종 대상 지정됨, -t 옵션으로 지정된 type 목록에서 -T 옵션에 명시된 항목을 제외한 목록이 최종 검색 대상으로 선택됨

```bash
rg "search_term" -t py -T py
```

This command will yield zero results because you are explicitly including Python files and then immediately excluding them.

### 확장자 지정

- `-g, --glob`
  - glob 패턴에 매칭된 파일 또는 디렉토리 이름만 검색 대상으로 지정함

```bash
# 확장자 지정(include)
rg "검색어" -g "*.js"
rg "검색어" -g "*.js" -g "*.ts"
rg "검색어" -g "*.{js,ts,html}"

# 확장자 제외(exclude)
rg "검색어" -g "!*.md"
rg "검색어" -g "!*.{mp4,avi,mkv}"
```
