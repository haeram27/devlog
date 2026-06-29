## [go] unittest 작성법

### **test 파일 이름 ::** XXX**_test**.go

형태로 **_test** suffix를 가져야만 한다.

'_' 또는 '.'으로 이름이 시작하는 테스트 파일은 무시된다.

### **test 함수 이름 ::func **Test**Fn (**t *testing.T**) {}

### test 패키지: **import "testing"**

"testing" package를 import하여 그 포인터를 입력으로 한다.

###  test 함수명 prefix: func **Test**Fn (t *testing.T) {}

test 함수는 "Test" prefix를 사용 해야만한다.

### test 함수명 parameter: func TestFn (**t *testing.T**) {}

`(t *testing.T)` 입력 파라미터를 가져야 한다.

testing.T 포인터를 이용하여 중요한 로그 함수 `t.Log()`/`t.Error()` 를 사용할 수 있다.

## **test 로그**

"testing" package를 import하여 그 테스트 로그 함수를 이용한다.

t.Log() :: fmt.Println() 과 같다.

t.Logf() :: fmt.Printf() 와 같다.

t.Error() :: fmt.Println() 과 같다. 호출 되었을 때 해당 Test 함수는 실패(FAIL)로 간주 된다.

t.Errorf() :: fmt.Printf() 와 같다. 호출 되었을 때 해당 Test 함수는 실패(FAIL)로 간주 된다.

## test 오류 출력:

test 중에 오류가 발생하면 `t.Error(<error-string>)`이나 `t.Fatal(<error-string>)`을 호출해 주어야 한다.

## **go test 실행 명령어**

### 자주 쓰는 go test 명령어

go test 명령어는 목적 테스트 함수가 정의된 테스트 파일(*_test.go)이
위치하는 directory 나 그 상위 디렉토리에서 실행한다.

현재 디렉토리에 목적 테스트 파일이 있는 경우 path를 명시 하지 않거나
"."(dot: current directory)를 지정한다.

현재 디렉토리 이하에 모든 테스트 파일을 대상으로 경로를 지정할 경우
"./..."을 사용한다.

$ go test -v -timeout 30s [-run \^TestFunctionName$] <directory
path containing *_test.go from current directory>

$ go test -v -timeout 30s [-run \^TestFunctionName$] .

"." 표현은 ## this directory:: when only go.mod is found in current
and parent directory of given path

$ go test -v -timeout 30s [-run \^TestFunctionName$] ./...

"./..." 표현은 ## this and whole sub directories:: when only go.mod
is found in current and parent directory

### go test 명령

```bash
go test <options> [<directory path containing *_test.go]
```

go test 명령은 기본적으로 지정한 패키지(=디렉토리)의 `XXX_test.go` 파일의
전부 컴파일 하여 `<package>.test[.exe]`라는 실행파일을 생성하고
실행한다. `go test <package>` 명령을 사용하면 `GOROOT/src/<package>`
에서 test 명령을 실행하는 것과 같다.

#### `-v` 옵션: 각 테스트 함수 별 메타정보 출력

```bash
go test -v
```

각 테스트 함수 별 메타정보(함수명과 함수 실행 결과 및 수행 시간)를
표시해 준다.

미사용시 각 테스트 함수 별 메타정보 정보는 출력하지 않는다.

미사용시 t.Error()가 호출된 Test 함수에 대해서만 표준 출력(fmt.PrintX(),
t.LogX())이 허용된다.

#### ./... : test recursively

```bash
$ go test ./...
```

현재 디렉토리 뿐만 아니라 이하 모든 하위 디렉토리에서 테스트 실행

#### -run 옵션 : 특정 test 함수만 실행

```bash
go test [-v] -run <test function name> [./...]
```

---

## [go] benchmark

<https://medium.com/a-journey-with-Dgo/go-should-i-use-a-pointer-instead-of-a-copy-of-my-struct-44b43b104963>

[go execution
tracer](https://blog.gopheracademy.com/advent-2017/go-execution-tracer/)

---

## [go][debug][delve] dlv debugger

<https://github.com/go-delve/delve>

### run dlv debugger

#### go program에 dlv debugger 붙이기

```
dlv run -- -run \^<function-name w/o quote or double quote> [-args] <args...>
```

#### go test program에 dlv debugger 붙이기

```bash
dlv test -- -test.run \^<test-function-name:TestXXX>$ [-args] <args...>
```

### set break

```
(dlv) b <pkg path to function>

(dlv) b prj/pkg/path.TestFunction

(dlv) b TestMy<Tab>

(dlv) b main<Tab>
```

```bash
go help test
```

### 전체 명령어

The following commands are available:

Running the program:

call ------------------------ Resumes process,
injecting a function call (EXPERIMENTAL!!!)

**continue (alias: c)** --------- Run until breakpoint or program termination.

**next (alias: n)** ------------- Step over to next source line.

rebuild --------------------- Rebuild the target
executable and restarts it. It does not work if the executable was not built by delve.

restart (alias: r) ---------- Restart process.

**step (alias: s)** ------------- Single step through
program.

step-instruction (alias: si) Single step a single cpu instruction.

**stepout (alias: so)** --------- Step out of the current
function.

Manipulating breakpoints:

**break (alias: b)** ------- Sets a breakpoint.

**breakpoints (alias: bp)** Print out info for active breakpoints.

clear ------------------ Deletes breakpoint.

clearall --------------- Deletes multiple breakpoints.

condition (alias: cond) Set breakpoint condition.

on --------------------- Executes a command when a
breakpoint is hit.

toggle ----------------- Toggles on or off a breakpoint.

trace (alias: t) ------- Set tracepoint.

watch ------------------ Set watchpoint.

Viewing program variables and memory:

args ----------------- Print function arguments.

**display** -------------- Print value of an expression
every time the program stops.

examinemem (alias: x) Examine raw memory at the given address.

**locals** --------------- Print local variables.

**print (alias: p)** ----- Evaluate an expression.

regs ----------------- Print contents of CPU registers.

set ------------------ Changes the value of a variable.

**vars** ----------------- Print package variables.

whatis --------------- Prints type of an expression.

Listing and switching between threads and goroutines:

goroutine (alias: gr) -- Shows or changes current goroutine

goroutines (alias: grs) List program goroutines.

thread (alias: tr) ----- Switch to the specified thread.

threads ---------------- Print out info for every traced
thread.

Viewing the call stack and selecting frames:

deferred --------- Executes command in the context of a deferred
call.

down ------------- Move the current frame down.

frame ------------ Set the current frame, or execute command
on a different frame.

stack (alias: bt) Print stack trace.

up --------------- Move the current frame up.

Other commands:

config --------------------- Changes configuration parameters.

disassemble (alias: disass) Disassembler.

dump ----------------------- Creates a core dump from the current process state

edit (alias: ed) ----------- Open where you are in $DELVE_EDITOR or $EDITOR

exit (alias: quit | q) ----- Exit the debugger.

funcs ---------------------- Print list of functions.

help (alias: h) ------------ Prints the help message.

libraries ------------------ List loaded dynamic libraries

**list (alias: ls | l)** ------- Show source code.

source --------------------- Executes a file containing a list of delve commands

sources -------------------- Print list of source files.

transcript ----------------- Appends command output to a file.

types ---------------------- Print list of types

---

## [go][debug] trace dump

### **trace dump 생성**

main 함수에 defer를 사용해 profile을 설정한다.

```go
func main() {
    defer profile.Start(profile.TraceProfile, profile.ProfilePath(".")).Stop()
    ...
}
```

### **trace dump 분석**

```go
go tool trace trace.out
```

예) <https://fransoaardi.github.io/posts/goroutine_lifecycle/>

```go
package main

import (
    "fmt"
    "github.com/pkg/profile"
    "time"
)

func main() {
    // trace.out 을 떨어뜨리는 helper function

    defer profile.Start(profile.TraceProfile,
    profile.ProfilePath(".")).Stop()

    fmt.Println(a())
    time.Sleep(time.Second)
}

func a() string {
    done := make(chan bool)
    defer close(done)
    // defer 에 done close 시켜서 goroutine 의 종료를 강제하여 종료가 보장되는 환경 설정
    // 지면관계상 종료가 보장되지 않는 환경은, 따로 작성하지 않는다. done 전달을 없애고, 아래 infinite 의 select 구문을 없애면 된다.
    // Note: 3개 function 이 모두 infinite() 라면 deadlock 이 발생할 것이다.
    resp := make(chan string, 3)
    go func() { resp <- infinite(1, done) }()
    go func() { resp <- infinite(2, done) }()
    go func() { resp <- notinfi() }()
    return <-resp
}

func infinite(a int, done chan bool) string {
defer fmt.Println("I'm done", a)
fmt.Println("infinite occurred", a)
for {
select {
case <-done: // a() 호출이 끝나서 done 이 close 되면 이 func 는
종료될것이다.
// select 구문을 없애면 infinite 는 종료되지 않을 function 이 된다.
return ""
}
}
}

func notinfi() string {
return ""
}

From <<https://fransoaardi.github.io/posts/goroutine_lifecycle/>>
```
