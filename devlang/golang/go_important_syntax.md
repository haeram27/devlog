# [go] 주의 문법

## 함수(function)과 메소드(method)

Go 언어에서 일반적으로 함수는 특정 type에 종속되지 않는다.

리시버를 가지고 특정 type에 종속되는 멤버 함수를 메소드라고 한다.

리시버는 밸류 리시버와 포인터 리시버가 있으며 포인터 리시버에는 밸류형 변수나 포인터형 변수 모두를 전달 가능하다.

## empty와 nil 차이

empty는 인스턴스의 주소가 있으나 데이터는 없는 상태 이고,

nil은 인스턴스의 주소가 없는 상태이다.

## 모듈 멤버의 첫문자의 대소문자에 따른 package간 access scope 제한

.go 파일에 정의되는 global variable과 함수에 대해서

그 이름의 첫문자가 대문자이면 다른 외부 package에서 접근 가능

그 이름의 첫문자가 소문자이면 다른 외부 package에서 접근 불가능

`var PackageExternalGlobalVariable int`

`var packageInternalGlobalVariable int`

`func PackageExternalFunction(){}`

`func packageInternalFunction(){}`

## var

변수 선언시 type deduction 표현

```go
var v int = 1;
```

global variable에는 항상 var를 사용

local variable에서는 := 사용하여 초기화시 var 생략 가능

## pass type call by value

golang은 함수에 인자를 전달할 때 기본적으로 call by value 또는 call by
pointer를 사용한다.

**call by value를 사용하는 경우 copy action이 발생**한다.

다만 struct를 제외한 **string, slice, map, queue, interface 등 golang에
사전 정의된 complex container들은** 내부에 데이터에 대한 포인터를 가지는
구조이므로 **call by value를 사용하여도 call by pointer를 사용한 것과
유사한 퍼포먼스를 가진다.**

그러므로 struct 인스턴스를 reference로 함수에 전달하는 경우에만 call by
pointer 사용을 고려하고 그외에는 기본적으로 call by value를 사용하면
된다.

call by pointer 사용 목적:

- 전달 되는 원본 인스턴스의 값에 대한 변경이 필요할 때
- 복사가 발생하는 비용이 매우 높아 복사 비용을 없애야 할 때
  - primitive type :: call by value or call by pointer(reference)
  - string :: call by value
  - slice :: call by value
  - map :: call by value
  - queue :: call by value
  - interface :: call by value
  - struct :: call by pointer

## receiver의 pass type

receiver는 struct의 멤버 메소드(struct member function)을 정의할 때 사용되는 struct instance 전달자 이다.

메소드에 리시버를 지정할 때의 type 표현에 따라(포인터이냐 아니냐)서 멤버 함수 내부에서 struct 원본 인스턴스의 사용방식이 달라진다.

값 타입 리시버(Value Receiver)와 포인터 타입 리시버(Pointer Receiver)의 가장 핵심적인 차이는 "메서드 내부에서 원본 데이터를 수정할 수 있는가"와 "호출 시 데이터가 복사되는가"입니다.

### 한눈에 보는 핵심 차이점

| 구분 | 값 타입 리시버 ((t T)) | 포인터 타입 리시버 ((t *T)) |
|---|---|---|
| 데이터 전달 방식 | 원본 값의 복사본을 전달 | 원본 값의 메모리 주소를 전달 |
| 원본 수정 가능 여부 | 불가능 (메서드 내 변경은 로컬에서만 유효) | 가능 (메서드 내 변경이 원본에 반영됨) |
| 성능 (메모리/복사) | 구조체가 크면 복사 비용이 커짐 | 주소값만 전달하므로 복사 비용이 매우 작음 |
| 호출 가능 대상 | 값 변수, 포인터 변수 모두 가능 | 값 변수, 포인터 변수 모두 가능 (자동 변환) |

### 코드 예시

```go
package main
import "fmt"

type User struct {
    Name string
}
// 1. 값 타입 리시버: 복사본을 받음
func (u User) ChangeNameValue(newName string) {
    u.Name = newName // 복사본의 값을 바꾸므로 원본은 변하지 않음
}
// 2. 포인터 타입 리시버: 원본 주소를 받음
func (u *User) ChangeNamePointer(newName string) {
    u.Name = newName // 원본 변수의 값이 직접 변경됨
}
func main() {
    user := User{Name: "Kim"}

    user.ChangeNameValue("Lee")
    fmt.Println(user.Name) // 출력: Kim (변경되지 않음)

    user.ChangeNamePointer("Lee")
    fmt.Println(user.Name) // 출력: Lee (변경됨)
}
```

### 리시버 타입을 선택하는 기준

- 포인터 리시버를 사용해야 하는 경우
- 메서드 내부에서 구조체의 필드 값을 변경해야 할 때
  - 구조체의 크기가 너무 커서 값 복사 시 성능 저하가 우려될 때
  - sync.Mutex처럼 복사하면 안 되는 필드를 구조체가 포함하고 있을 때
- 값 리시버를 사용해야 하는 경우
- 구조체가 단순한 읽기 전용 데이터를 담고 있을 때 (예: time.Time)
  - 구조체의 크기가 작고 변경이 필요 없을 때
  - 기본 타입(int, string 등)을 기반으로 정의한 커스텀 타입일 때

⚠️ Go의 관례: 하나의 타입에 정의된 메서드들 중 단 하나라도 포인터 리시버를 사용한다면, 일관성을 위해 모든 메서드를 포인터 리시버로 통일하는 것이 좋습니다.


## go 함수의 closure 표현에서 outter 변수 사용은 모두 call by reference 이다.

### Rule 1, Go 언어에는 함수는 일급 함수로써 하나의 객체로 다루어진다.

함수를 변수에 담거나 함수의 인자로 전달 하거나 리턴 값으로 사용할 수
있다.

### Rule 2, `Anonymous function` = `function literal` = `lambda function`

`closure`? 외부 함수의 로컬 변수에 접근할 수 있는 내부 함수 또는 그러한
문법

Go language에서 closure 문법은 Anonymous function, lambda, go routine
등에서 모두 사용될 수 있다.

*내부 함수가 외부 함수의 로컬 변수를 접근할 때 이는 참조로써 접근을
수행하며 외부 함수의 값을 내부 함수에서 변형할 수 있다. 그래서 read
동작만 할 때는 문제가 없지만 내부 함수에서 외부함수 변수에 대한 변경을
수행할 때는 적절한 동기화 방식을 적용(mutex or "atomic variable")해야
한다.*

내부 함수가 외부 함수의 로컬 변수를 call by value로 접근하고자 한다면
변수를 내부 함수에 parameter로써 전달하는 방식을 사용해야 한다.

### Rule 3, closure 상황에서 외부함수의 로컬 변수와 같은 이름의 변수를 내부함수에서 다른 인스턴스로 선언 가능하다.

다음의 예에서 외부 함수의 redeclare 변수와 go routine에 사용된 내부
함수의

```go
func TestClosureCaptureReDecl(t *testing.T) {

    redeclare := 1 // outter

    fmt.Printf("outter: %v, %v\n", redeclare, &redeclare) //
    outter: 1, 0xc0000ba288

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

## 변수 선언시과 default 값
golang 에서 모든 변수는 따로 초기값을 부여 코드를 작성하지 않는다면
default 값(0, nil, false)으로 초기화 된다.

primitive type, bool, struct, pointer 등 모든 type은 default 값으로
초기화 되므로 C/C++과 달리 로컬 변수 선언시 쓰레기 값을 염려해 zero
initialize를 명시할 필요가 없다.

모든 메모리가 할당이 발생하는 변수 표현은 기본적으로 default 값으로
초기화 된다.

```go
var myint int // 0
var mybool bool // false

myType := MyType{} // 모든 멤버도 default값(0, nil, fase) 가짐
mypointer := new(MyType) // 모든 멤버도 default값(0, nil, fase) 가짐
mypointer := &MyType{} // 모든 멤버도 default값(0, nil, fase) 가짐
```

## 구조체 초기화: new(), {}

### `{}` 초기화

struct 초기화에는 {}를 사용한다.


```text
type <struct-name> struct {}

v := <struct-name>{초기화할 필드: 초기화할 값}
```

### `new()` 초기화

struct 타입에 대해 메모리를 할당하고 zero initialization 한 후 포인터를
타입의 포인터를 반환.

struct 타입에 대해 {} instanciation하는 것은 new()로 초기화 하는 것과
같다.

### struct 초기화 요약

아래 두 표현식 같은 의미이다

```go
m := new(MyType)

fmt.Print("%T %#v", m, m) // *main.MyType &main.MyType{}
```

```go
m := &MyType{}

fmt.Print("%T %#v", m, m) // *main.MyType &main.MyType{}
```

## channel, slice, map 초기화:  `make()`, `{}`

channel, slice, map 초기화에는 size 설정을 위하여 `make()`를 사용한다.

**channel, slice, map을 생성할 때는 `new()` 대신 반드시 `{}`나 `make()`를
사용해야 한다.**

channel, slice, map에 대해 `{}` 초기화를 하는 것은 `make(<type>, 0)`로
초기화하는 것과 같다.

`new()`는 메모리 할당 후 할당 영역을 `zero(0)`로 초기화하고 포인터
반환하는데 반해

`make()`는 메모리 할당 후 channel, slice, map struct 타입에 맞는 각
멤버들에 대해서 기본값으로 초기화하고 레퍼런스 반환

- slice를 초기화하는 표현과 ptr 여부

|**초기화 표현**|**ptr**|**len**|**cap**|
|---|---|---|---|
|nil: []string| 0 | 0 | 0 |
|empty: []string{}| `<addr>`| 0  | 0 |
|empty: make([]string, 0)| `<addr>`| 0 | 0 |


## [**Anonymous struct**](https://blog.boot.dev/golang/anonymous-structs-golang/)

composit literal

anonymous struct type:

```go
newCar := struct {
    make string
    model string
    mileage int
}{
    make: "Ford",
    model: "Taurus",
    mileage: 200000,
}
```

named struct type:

```go
// declare the 'car' struct type
type car struct {
    make string
    model string
    mileage int
}

// create an instance of a car
newCar := car{
    make: "Ford",
    model: "taurus",
    mileage: 200000
}
```

anonymous struct는 단 한번만 사용되는 struct가 필요할 때 사용한다.

예를 들면 http로 전달된 json object를 decode하는 경우와 같이 말이다.

```go
func createCarHandler(w http.ResponseWriter, req *http.Request) {

    defer req.Body.Close()

    decoder := json.NewDecoder(req.Body)

    newCar := struct {
        Make string `json:"make"`
        Model string `json:"model"`
        Mileage int `json:"mileage"`
    }{}

    err := decoder.Decode(&newCar)

    if err != nil {
        log.Println(err)
        return
    }

    makeCar(newCar.Make, newCar.Model, newCar.Mileage)

    return

}
```

**Don't use `map[string]`, `interface{}` for JSON data if you can avoid it.**

Instead of declaring a quick anonymous struct for JSON unmarshalling,
I've often seen map[string]interface{} used. This is terrible in most
scenarios for several reasons:

No type checking. If the client sends a key called "name" with a bool
value, but your code is expecting a string, then unmarshalling into a
map won't catch the error

Maps are vague. After unmarshalling the data, we are forced to use
runtime checks to make sure the data we care about exists. If those
checks aren't thorough, it can lead to a nil pointer dereference panic
being thrown.

map[string]interface{} is verbose. Digging into the map isn't as
simple as accessing a named field using a dot operator, for example,
newCar.model. Instead, it is something like:

```go
func createCarHandler(w http.ResponseWriter, req *http.Request) {

    myMap := map[string]interface{}{}
    decoder := json.NewDecoder(req.Body)
    err := decoder.Decode(&myMap)

    if err != nil {
        log.Println(err)
        return
    }

    model, ok := myMap["model"]
    if !ok {
        fmt.Println("field doesn't exist")
        return
    }

    modelString, ok := model.(string)
    if !ok {
        fmt.Println("model is not a string")
    }

    // do something with model field
}
```

## interface vs empty interface(interface{}, any)

**interface와 empty interface는 완전히 다른 역할을 하는 타입이다.**

interface는 메소드들의 집합이며 struct와 마찬가지로 struct 인스턴스의
타입으로 사용된다.

empty interface는 interface에 {}가 붙은 `interface{}` 표현식 또는 `any`
타입으로 표현이 가능하며 golang에서 `dynamic type`의 역할을 한다. `empty interface`는 모든 `primitive` 타입과 `pointer`, `slice` 타입의 인스턴스를 담을 수 있다.

empty interface와 유사하게 empty struct 타입(struct{})도 존재하나 이는
테스트의 경우 외에는 거의 사용처가 없다.

### interface

struct가 변수들의 집합이라면 interface는 메소드들의 집합이다.

struct가 타입으로 사용되는 것과 같이 interface 또한 타입으로 사용된다.

단, interface 타입 변수에 할당되는 인스턴스는 interface 타입으로 생성할
수 없고

interface에 정의된 모든 메소드를 구현하는 struct 인스턴스를 생성하여
interface 타입변수에 할당 할 수 있다.

만약 A interface에 정의된 모든 메소드를 구현하는 struct는 여러 개이며
이들을 B, C, D struct 라고 한다면 B, C, D struct 는 모두 A interface가
될 수 있다. 이러한 방식을 덕타이핑 이라고 한다.

interface를 파라미터로 하는 함수에 인스턴스를 넘기는 방법?

1, 인스턴스의 포인트를 넘긴다.

### 덕타이핑

struct가 타입으로 사용되는 것과 같이 interface 또한 타입으로 사용된다.

단, interface 타입 변수에 할당되는 인스턴스는 interface 타입으로 생성할
수 없다.

`interface에 정의된 모든 메소드를 구현하는 struct`의 인스턴스가 interface
타입의 값으로 할당 될 수 있다.

**만약 A interface에 정의된 모든 메소드를 구현하는 struct는 여러 개이며
이들을 B, C, D struct 라고 한다면 B, C, D struct 는 모두 A interface가
될 수 있다.** 이러한 방식을 `덕타이핑` 이라고 한다.

아래는 덕타이핑의 예시로써 bufio.NewWriter(buffer)에 사용되는
bytes.Buffer struct는 io.Writer interface의

Write(p []byte) (n int, err error) 메소드를 구현하고 있으므로
io.Writer를 파라메터로 하는 NewWriter(w io.Writer) 메소드에 io.Writer의
인스턴스로 사용될 수 있다.

```go
func JsonUnEscape(value string) string {
    buffer := &bytes.Buffer{}
    bufio.NewWriter(buffer)
    encoder := json.NewEncoder(buffer)
    encoder.SetEscapeHTML(false)
    encoder.Encode(value)
    return buffer.String()
}

<buffer.go>

type Buffer struct {
    buf []byte // contents are the bytes buf[off : len(buf)]
    off int // read at &buf[off], write at &buf[len(buf)]
    lastRead readOp // last read operation, so that Unread* can work correctly.
}

<buffer.go>

func (b *Buffer) Write(p []byte) (n int, err error) {

    b.lastRead = opInvalid
    m, ok := b.tryGrowByReslice(len(p))

    if !ok {
        m = b.grow(len(p))
    }

    return copy(b.buf[m:], p), nil
}

<bufio.go>

func NewWriter(w io.Writer) *Writer {
    return NewWriterSize(w, defaultBufSize)
}

<bufio.go>

func NewWriterSize(w io.Writer, size int) *Writer {
    // Is it already a Writer?
    b, ok := w.(*Writer)
    if ok && len(b.buf) >= size {
        return b
    }

    if size <= 0 {
        size = defaultBufSize
    }

    return &Writer{
        buf: make([]byte, size),
        wr: w,
    }
}

<io.go>

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

struct 인스턴스가 지정한 interface type인지 확인하는 방법

```
interface{}(<struct instance>).(<interface type>)
```

```go
type Duck struct {
}

func (d Duck) quack() {
    fmt.Println("quack")
}

func (d Duck) feathers() {
    fmt.Println("feathers")
}

type Quacker interface {
    quack()
    feathers()
}

var donald Duck
    if v, ok := interface{}(donald).(Quacker); ok {
    fmt.Println(v, ok)
}

# 결과
# {} true
```

## ***[empty struct:struct{}](https://dave.cheney.net/2014/03/25/the-empty-struct)***

- `struct{}` ? 말그대로 "빈 구조체 타입"
- `struct{}{}` ? empty struct의 인스턴스

**용법 1, 테스트 struct 타입 변수에 빈 instance 할당하기**

empty struct는 주소값이 없고 사이즈가 0이다.

```go
emptyStruct := struct{}{}
fmt.Println(&emptyStruct) // &{}
// unsafe.Sizeof() 메소드는 해당 변수의 크기값을 반환함
fmt.Println(unsafe.Sizeof(emptyStruct)) // 0

boolVariable := true
fmt.Println(&boolVariable) // 0xc42001a088
fmt.Println(unsafe.Sizeof(boolVariable)) // 1
```

**용법 2, channel에 단순 시그널 보내기**

```go
done <- struct{}{}
```

- bool 과 empty struct로 signal보낼때 퍼포먼스 비교?
  - empty struct가 bool 사용 대비 절반의 시간만 소비된다.

```go
func GetBool() bool {
    return true
}

func GetEmptyStruct() struct{} {
    return struct{}{}
}

func BenchmarkGetBool (b *testing.B) {
    for i:= 0; i< b.N; i++ {
        GetBool()
    }
}

func BenchmarkGetStruct (b *testing.B) {
    for i := 0; i < b.N; i++ {
        GetEmptyStruct()
    }
}

==========================================================

$ go test -bench=. -v
goos: darwin
goarch: amd64
BenchmarkGetBool-8         2000000000         0.63 ns/op
BenchmarkGetStruct-8         2000000000         0.31 ns/op
```

## empty interface type: **`interface{}`**

empty interface = 인터페이스 타입 = nil을 제외한 golang의 모든 타입을 담을 수 있는 컨테이너, **Dynamic Type**(주: java의 object, c/c++의 void*)

int, string 등 **primitive 타입과 포인터와 슬라이스** 까지 모든 형식을 대신할 수 있다.

"interface{}" type은 golang의 모든 타입을 대신할 수 있다.

심지어 []interface{} (interface{}의 slice)도 slice 이므로 "interface{}"로 그 타입을 표현 할 수 있다.

interface{}는 "any" 라는 alias로 명시 가능하다.

interface{} type의 변수에 접근하려면 반드시 type assert를 명시해 주어야한다.

```go
func TestInterface(t *testing.T) {
    var i int = 1
    var a interface{} = i
    j := a
    k := a.(int)

    var b interface{} = &i
    x := b
    y := b.(*int)

    println(i) // 1
    println(a) // (0x7a5540,0xc000054710)
    println(&a) // 0xc000055750
    println(j) // (0x7a5540,0xc000054710)
    println(&j) // 0xc000055750
    println(k) // 0xc000054730
    println(&k) // 1
    println(&i) // 0xc000054700
    println(b) // (0x799a20,0xc000054700)
    println(&b) // 0xc000054740
    println(x) // (0x799a20,0xc000054700)
    println(&x) // 0xc000054720
    println(y) // 0xc000054700
    println(*y) // 1
    println(&y) // 0xc000054718
}
```

**"type switch": `interface{}`(empty interface) type 변수의 실제 타입이 slice인지 확인 하는 방법**

```go
names, err := returnEmptyInterface()

if err != nil {
    print(err)
    return nil
}

repos := []interface{}{}

switch names.(type) {
    case []interface{}:
        repos = names.([]interface{})
    case interface{}:
        repos = append(repos, names)
    default:
        return nil
}
```

## type casting

**1. type conversion (형변환)**

golang 에서는 자동 형변환이 없으며 형변환을 하려면 반드시 명시적으로
type casting을 해주어야 한다.

```go
var n int = 15
var v1 int64 = int64(n)
```

**2. type assertion (interface type의 형변환)**

type assertion은 "interface{}" 타입 변수가 가지고 있는 실제 값(concrete value)에 접근할 수 있게 해준다. (golang의 "interface{}"는 임의의 타입을 의미하며 임의의 값을 가질 수 있기 때문에 concrete라는 표현을 쓴 듯 하다.)

```go
var n interface{} = 15
v := n.(int)
// or
v, ok := n.(int)
```

## 구조체 :: member method on struct

```go
package main

import "fmt"

type Rectangle struct {
    length, width int
}

func (r Rectangle) Area_by_value() int {
    return r.length * r.width
}

func (r *Rectangle) Area_by_reference() int {
    return r.length * r.width
}

func main() {
    r1 := Rectangle{4, 3}

    fmt.Println("Rectangle is: ", r1)
    fmt.Println("Rectangle area is: ", r1.Area_by_value())
    fmt.Println("Rectangle area is: ", r1.Area_by_reference())
    fmt.Println("Rectangle area is: ", (&r1).Area_by_value())
    fmt.Println("Rectangle area is: ", (&r1).Area_by_reference())
}

/*
Rectangle is: {4 3}
Rectangle area is: 12
Rectangle area is: 12
Rectangle area is: 12
Rectangle area is: 12
*/
```

Rectangle instace는 call by value나 call by reference 모두로 전달될 수 있고 그 결과는 같다.

컴파일러가 자동으로 call by reference로 모두 최적화 하기 때문이다.

***[TODO] ## 구조체 :: [anonymous field](https://golangbyexample.com/anonymous-fields-struct-golang/)***

## 구조체 :: member method on anonymous field

```go
package main

import "fmt"

type Kitchen struct {
    numOfForks int
    numOfKnives int
}

func(k Kitchen) totalForksAndKnives() int {
    return k.numOfForks + k.numOfKnives
}

type House struct {
    Kitchen //anonymous field
}

func main() {
    * h := House{Kitchen{4, 4}} //the kitchen has 4 forks and 4
    knives*

    * fmt.Println("Sum of forks and knives in house: ",
    h.totalForksAndKnives()) //called on House even though the method is associated with Kitchen*
}

// Sum of forks and knives in house: 8
```
