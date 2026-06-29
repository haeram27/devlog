# [go] main / init 함수

**<> init() 함수**

main 보다 먼저 실행되는 함수

application scope에서 사용할 global 변수들을 선언하거나 초기화 하는데
사용한다.

예를 들어 logger 설정, 서버의 provision 설정

**<> main() 함수**

프로그램의 시작점 함수

main() 함수가 꼭 main.go 파일이나 main package에 들어있어야 하는 것은
아니다.

같은 디렉토리에 main() 함수를 갖는 여러개의 .go 파일이 존재할 수 있다.

단, 이 경우 go run/build 명령 사용시 반드시 main()함수가 존재하는 각
파일을 명시해 주어야 한다.

예를들어 prog.go 파일에 main()이 있는 경우

$ go run prog.go

---

# [go] main

1.  syscall (<https://pkg.go.dev/syscall>)

2.   

---

# [go] function과 method

<https://hyunmin1906.tistory.com/256>

**<> Go 언어 함수**

함수(function)은 파라미터와 반환 값을 갖는 다음 형식을 갖는다.

```go
func name(parameter) <return-type> {

return values...
```

Go 언어에서는 특별하게 다수의 리턴값을 가질 수 있으며, 또한 리턴할
변수를 미리 선정하고 리턴시킬 수 있다.

1) 한개의 값을 리턴하는 함수

예시) 자료형이 string인 한개의 값을 리턴하는 함수

func SingleReturn(name string) string {

return name

}

2) 다수의 값을 리턴하는 함수

예시) 자료형이 int, string인 다수의 값을 리턴하는 함수

func ManyReturn(name string) (int, string) {

return len(name), name

}

3) 변수를 지정해서 리턴하는 함수

예시) 자료형이 int, string인 변수를 지정하여 값을 리턴하는 함수

func FixReturn(name string) (lenName int, upperName string) {

lenName = len(name)

upperName = strings.ToUpper(name)

return

}

4) 다수의 값을 인자로 받는 함수

예시) 자료형이 string인 다수의 값을 인자로 받는 함수

func ManyArgument(names ...string) {

fmt.Println(names)

}

### Go 언어 메소드

메소드는 리시버를 갖는 함수를 말한다.

```
Func (<receiver-name> <type>) name(parameter) <return-type> {

return values...
```

메소드는 내부에서 리시버(receiver)에 접근하여 작업이 가능하다.

또한 메소드는 같은 패키지에 있는 type만을 리시버로 선언할 수 있으며,
주로 리시버 타입은 구조체(Struct) 이다.

1) 벨류 리시버(Value Receiver)로 필드 접근

* Value Receiver는 리시버에 해당하는 새로운 객체를 복제하기 때문에 실제
객체에는 영향이 없다.

예시) person 구조체의 모든 필드에 값을 삽입하고, 출력하시오.

```go
type Person struct {

name string

age int

address string

}

func (p Person) searchPerson() {

fmt.Println(p.name, p.age, p.address)

}

func main() {

myInfo := Person{"hyunmin", 28, "서울시 용산구"}

myInfo.searchPerson()

}

// 출력결과 : hyunmin 28 서울시 용산구
```

2) 포인터 리시버(Pointer Receiver)로 필드 접근

* Pointer Receiver는 실제 객체에 접근하여 작업이 가능하다.(수정, 검색
등)

포인터 리시버는 밸류형 변수나 포인터형 변수 모두에서 호출 가능하다.

예시) person 구조체의 모든 필드에 값을 수정하고 출력하시오.

type Person struct {

name string
age int
address string

}

func (p *Person) searchPerson() {

fmt.Println(p.name, p.age, p.address)

}

func (p *Person) changePerson() {

p.name = "donghun"
p.age = 27
p.address = "서울시 강서구"

}

func main() {

myInfo := Person{"hyunmin", 28, "서울시 용산구"}
// 수정 전
myInfo.searchPerson()
// 수정 작업
myInfo.changePerson()
// 수정 후
myInfo.searchPerson()

}

// 출력 결과

// hyunmin 28 서울시 용산구

// donghun 27 서울시 강서구

포인터 리시버에는 밸류형 변수나 포인터형 변수 모두를 전달 가능하다.

```go
<https://go.dev/tour/methods/6>

package main

import "fmt"

type Vertex struct {

X, Y float64

}

func (v *Vertex) Scale(f float64) {

v.X = v.X * f

v.Y = v.Y * f

}

func ScaleFunc(v *Vertex, f float64) {

v.X = v.X * f

v.Y = v.Y * f

}

func main() {

v := Vertex{3, 4}

v.Scale(2) // 포인터형 리시버를 밸류형 변수로 호출 OK

ScaleFunc(&v, 10)

p := &Vertex{4, 3}

p.Scale(3) // 포인터형 리시버를 포인터형 변수로 호출 OK

ScaleFunc(p, 8)

fmt.Println(v, p) //{60 80} &{96 72}

}
```

---

# [go] function과 closure

**Go language의 closure(함수 안의 함수 = anonymous func, lambda)에서
외부 함수의 로컬 변수를 사용(캡쳐)은 기본적으로 레퍼런스 방식의 참조
이다.**

Go 언어는 함수형 프로그래밍 언어 이다.

함수(function)과 메소드(method)

Go 언어에서 일반적으로 함수는 특정 struct에 종속되지 않는다.

리시버를 가지고 특정 struct에 종속되는 멤버 함수를 메소드라고 한다.

**Rule 1, Go 언어에는 함수는 일급 함수로써 하나의 객체로 다루어 진다.**

함수를 변수에 담거나 함수의 인자로 전달 하거나 리턴 값으로 사용할 수
있다.

**Rule 2, Anonymous function == function literal == lambda function ==
closure**

**closure? 외부 함수의 로컬 변수에 접근할 수 있는 내부 함수( 또는 그러한
문법)**

Go language에서 closure 문법은 Anonymous function, lambda, go routine
등에서 모두 사용될 수 있다.

내부 함수가 외부 함수의 로컬 변수를 접근할 때 이는 참조로써 접근을
수행하며 외부 함수의 값을 내부 함수에서 변형할 수 있다. 그래서 read
동작만 할 때는 문제가 없지만 내부 함수에서 외부함수 변수에 대한 변경을
수행할 때는 적절한 동기화 방식을 적용(mutex or "atomic variable")해야
한다.

내부 함수가 외부 함수의 로컬 변수를 call by value로 접근하고자 한다면
외부 함수의 로컬 변수를 내부 함수에 parameter로써 전달하는 방식을
사용해야 한다.

**Rule 3, closure 상황에서 외부 함수의 로컬 변수와 같은 이름의 변수를
내부함수에서 다른 인스턴스로 선언 가능하다.**

다음의 예에서 외부 함수의 redeclare 변수와 go routine에 사용된 내부
함수의 redeclare 변수를 살펴보자

외부 함수의 redeclare

```go
func TestClosureCaptureReDecl(t *testing.T) {

redeclare := 1 // outter

fmt.Printf("outter: %v, %v\n", redeclare, &redeclare) // outter:
1, 0xc0000ba288

var wg sync.WaitGroup

wg.Add(1)

go func() {

redeclare := 10 // inner is re-declared

fmt.Printf("inner: %v, %v\n", redeclare, &redeclare) // inner:
10, 0xc0000ba2a0

wg.Done()

}()

wg.Wait()

}
```

**<> Go 언어에서 함수와 closure 외부 변수 참조에 대한 테스트 코드들**

```go
package test

import (

"fmt"

"sync"

"testing"

"time"

)

func TestGoCapture(t *testing.T) {

varOutter := "ref" // new instance

fmt.Printf("original: %v\n", &varOutter) // original:
0xc00006b030

print := func(name string) {

fmt.Printf("%s: %v\n", name, &varOutter) // accessed as
reference

}

go print("1") // 1: 0xc00006b030

go print("2") // 2: 0xc00006b030

}

func TestGoCapture2(t *testing.T) {

varOutter := 0

var wg sync.WaitGroup

wg.Add(1)

go func() {

for i := 0; i < 10; i++ {

varOutter++ // outter variable can be changed from closure

fmt.Printf("in %v\n", varOutter) // accessed as reference

time.Sleep(time.Second)

}

wg.Done()

}()

for i := 0; i < 10; i++ {

varOutter++

fmt.Printf("out %v\n", varOutter)

time.Sleep(time.Second)

}

wg.Wait()

}

func TestClosureCaptureReDecl(t *testing.T) {

redclare := 1 // outter

fmt.Printf("outter: %v, %v\n", redclare, &redclare) // outter:
1, 0xc0000ba288

var wg sync.WaitGroup

wg.Add(1)

go func() {

redclare := 10 // inner is re-declared

fmt.Printf("inner: %v, %v\n", redclare, &redclare) // inner: 10,
0xc0000ba2a0

wg.Done()

}()

wg.Wait()

}

func adder() func(int) {

varOutter := 0 // new instance

fmt.Printf("original: %v\n", &varOutter)

return func(x int) {

varOutter += x

fmt.Printf("%d: %v\n", varOutter, &varOutter) // accessed as
reference

}

}

func TestClosureCapture(t *testing.T) {

pos, neg := adder(), adder() // original: 0xc000128228, original:
0xc000128230

pos(1) // 1: 0xc000128228

neg(-2 * 1) // -2: 0xc000128230

}
```

---

# [go] defer panic receiver

defer? Java의 finally 키워드와 유사

defer <function call> - return 전 항상 호출

defer <declare func> - return 전 항상 실행되어야 하는 함수 선언

함수가 return 되기 직전에 항상 실행되어야 하는 로직에 명시

panic() 발생 후에도 실행된다.

그래서 panic() 발생시 이를 catch하기 위한 recover()를 호출하는데 자주
사용된다.

panic()? Java의 throw exception에 해당

함수 실행을 즉시 멈추고 defer 실행 후 함수를 종료하며, 이를 콜스택을
따라 반복 적용한다.

recover()가 호출 되거나 프로그램이 종료 될 때 까지 반복 적용된다.

panic()은 parameter로 err를 받을 수 있다.

recover()? Java 의 catch 문에 해당

panic() 을 중단 시킨다.

<http://golang.site/go/article/20-Go-defer%EC%99%80-panic>

1. 지연실행 defer

Go 언어의 defer 키워드는 특정 문장 혹은 함수를 나중에 (defer를 호출하는
함수가 리턴하기 직전에) 실행하게 한다.

일반적으로 defer는 C#, Java 같은 언어에서의 finally 블럭처럼 마지막에
Clean-up 작업을 위해 사용된다.

아래 예제는 파일을 Open 한 후 바로 파일을 Close하는 작업을 defer로 쓰고
있다. 이는 차후 문장에서 어떤 에러가 발생하더라도 항상 파일을 Close할 수
있도록 한다.

```go
package main

import "os"

func main() {

f, err := os.Open("1.txt")

if err != nil {

panic(err)

}

// main 마지막에 파일 close 실행

defer f.Close()

// 파일 읽기

bytes := make([]byte, 1024)

f.Read(bytes)

println(len(bytes))

}
```

2. panic 함수

Go 내장함수인 panic()함수는 현재 함수를 즉시 멈추고 현재 함수에 defer
함수들을 모두 실행한 후 즉시 리턴한다.

이러한 panic 모드 실행 방식은 다시 상위함수에도 똑같이 적용되고, 계속
콜스택을 타고 올라가며 적용된다.

그리고 마지막에는 프로그램이 에러를 내고 종료하게 된다.

```go
package main

import "os"

func main() {

// 잘못된 파일명을 넣음

openFile("Invalid.txt")

// openFile() 안에서 panic이 실행되면

// 아래 println 문장은 실행 안됨

println("Done")

}

func openFile(fn string) {

f, err := os.Open(fn)

if err != nil {

panic(err)

}

defer f.Close()

}
```

3. recover 함수

Go 내장함수인 recover()함수는 panic 함수에 의한 패닉상태를 다시
정상상태로 되돌리는 함수이다.

위의 panic 예제에서는 main 함수에서 println() 이 호출되지 못하고
프로그램이 crash 하지만,

아래와 예제와 같이 recover 함수를 사용하면 panic 상태를 제거하고
openFile()의 다음 문장인 println() 을 호출하게 된다.

```go
package main

import (

"fmt"

"os"

)

func main() {

// 잘못된 파일명을 넣음

openFile("Invalid.txt")

// recover에 의해

// 이 문장 실행됨

println("Done")

}

func openFile(fn string) {

// defer 함수. panic 호출시 실행됨

defer func() {

if r := recover(); r != nil {

fmt.Println("OPEN ERROR", r)

}

}()

f, err := os.Open(fn)

if err != nil {

panic(err)

}

defer f.Close()

}
```

종료하는 panic(), 복구하는 recover()

<http://golang.site/go/article/20-Go-defer%EC%99%80-panic>

```go
package main

import "fmt"

func main() {

// divideWithZero()

defer func() {

if r := recover(); r != nil {

fmt.Print("!!! recover:: ")

fmt.Println(r)

/*

fmt.Println("!!! recover runs divideWithZero() again...")

divideWithZero()

*/

}

}()

num1 := 1

num2 := 0

result := num1 / num2

fmt.Println(result)

}
```
