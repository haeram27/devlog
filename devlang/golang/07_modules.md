# [go] module / package / import

## Package 관련 참고 페이지

- [Go. Discover Package](https://pkg.go.dev/)
- [Go. Standard Library Documentation](https://pkg.go.dev/std)

## go project 및 개발 순서

```bash
go mod init <module-name>
```

implements sources

```bash
go mod tidy
```

소스 코드에 명시된 import package들을

```bash
go build
```

## module vs package

### module

module = 릴리즈, 버전, 배포된 package 집합

**같은 모듈의 소스는 빌드시 하나의 바이너리에 묶인다.**

module은 `go.mod` 파일이 선언된 위치인 `module path`에 의해 식별된다.

`go.mod` 파일에는 module 이름, 버전, 디펜던시 정보가 포함된다.

module의 이름이 go build 결과물의 이름이 된다.

module은 빌드의 결과인 실행파일이 되는 구분단위?

**module 이름 예:**

- "golang.org/x/net"
- "github.com/a/b"
- "hello"
- "my-project

**go.mod 파일 생성하기**

```bash
go mod init
```

### package

**같은 디렉토리의 소스들은 모두 같은 package가 되어야한다.**

- 함께 컴파일 되는 같은 디렉토리 내 source 파일의 집합
- `.go` 파일은 package 이름을 선언해야만 한다.
- package 이름은 directory 이름과 같거나 다를수 있다.
- 하지만 **같은 directory에 속하는 .go 파일의 package 이름은 모두 같아야만
한다.**
- 일반적으로 `.go` 파일이 위치하는 경로의 최종 direcroty 명을 사용해야한다.
- `.go` 파일이 위치하는 경로의 최종 direcroty 명과 package이 이름이 같을 경우:
  - import 측에서 중복된 이름의 package들을 import하는 경우에도 중복된
  - package 이름을 변경할 수 있어서 중복 문제를 해결할 수 있다.
- 특별한 경우에만 최종 directory 명과 package의 이름을 다르게 설정한다.
- 이 경우, import하는 측에서 그 package 이름을 변경할 수 없게 된다.

### import

syntax:

```go
import [<package>] "<module>/path/to/directory"
```

- `<package>` : import 대상 .go 파일에 선언된 package 사용, 선언된 `<package>`와 .go파일의 경로상 최종 dir의 이름이 같다면 임의의 이름 사용가능
- `<module>` : go.mod 파일에 정의된 module 이름. module 이름은 단일 문자열이거나 /로 구분된 경로 표현(ex> encoding/json,github/myrepo/project)일 수 있다.
- `path/to/directory` : import 대상 `.go` 파일이 위치한 경로. 경로의 시작은 `<module>` 이름으로 부터 시작하고 go.mod 파일이 있는 경로의 이하 경로를 명시해 주면 된다.
- `import`는 기본적으로 `import` 하고자하는 `.go` 파일이 위치한 경로(directory path)를 명시해 주는 것이다. 이 때 `.go` file에 선언된 `<packag>`와 그 `.go` 파일의 최종 directory 이름이 같은 경우와 다른 경우에 따라 제약사항이 달라질 수 있다.
- `.go` file에 선언된 `<package>`과 그 `.go` 파일의 최종 directory 이름이 같은 경우
  - import 문에서 `<package>`는 생략이 가능하다.
  - 생략하지 않을 경우 import 대상 `.go` 파일에 선언된 `<package>`를 사용하거나 import 문에서 `<package>`로 사용할 임의의 이름을 지정할 수 있다.(중복된 이름을 가진 package도 이름 지정을 통해 import가능)

- `.go` file에 선언된 `<packag>`와 그 `.go` 파일의 최종 directory 이름이 다른 경우:
  - import 문에서 `<package>`는 생략이 불가능하게 되며 import 대상 `.go` 파일에 선언된 `<package>` 이름을 그대로 사용하여야만 한다.(import측에서 package 이름 지정 불가능)
