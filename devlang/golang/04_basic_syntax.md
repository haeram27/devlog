# [go] 기초 문법

## 키워드

| package | package <package name> | 현재 파일의 package name 지정 |
| --- | --- | --- |
| import | import **"**<package name>**"\ \ **import **(**\ **"**<package name1>**"\ "**<package name2>**"\ )** | import package.\ 괄호를 사용하면 import 키워드를 한번만 사용할 수 있다. |
| func | func\ func func_name(arg arg_type) return_var return_type **{** ...\ **}** | 함수 선언 |
| var | var <name>\ var <name> = <value>\ var **(**\ <name1> = <value> <name2> = <value>\ **)**\ \ (함수내 혹은 if/for의 statement) <name> **:=** <value>\ | 변수 선언\ =대신 :=를 쓰면 **함수 안에서** var를 명시하지 않고도 변수를 선언(단, 함수 밖에서는 안됨)\ 괄호를 사용하면 키워드를 한번만 사용할 수 있다. |
| const | const | 상수 선언 상수가 될 수 있는 타입으로는 character, string, boolean, numeric 타입 있다. 상수 선언은 const를 명시해야 하기 때문에 :=로는 선언할 수 없다. |
| for |  |  |
| if |  |  |
| switch |  |  |
| case |  |  |
| fall through |  |  |
| break |  |  |
| default |  |  |
| in terface |  |  |
| select |  |  |
| defer |  |  |
| go |  |  |
| map |  |  |
| struct |  |  |
| chan |  | channel 정의 키워드 |
| else |  |  |
| goto |  |  |
| range |  |  |
| type |  |  |
| c ontinue |  |  |
| return |  |  |

함수 밖의 모든 statement들은 키워드(func, var 등)으로 시작해야한다.

변수간 대입시 묵시적 형변환 없음. 반드시 명시적으로 형변환 해줘야 한다.

int, uint, uintptr은 32비트 시스템에서 32비트길이고 64비트 시스템에서
64비트 길이이다.

선언만 하고 초기화 하지 않은 변수는 zero value(0, false, "")로
초기화 - 함수 내부/외부 모두 적용된다.

string + 연산자 사용가능

{}를 사용하는 경우는 함수 func, struct, for, if, switch, 배열 초기화
이다.

## 함수 선언

argument와 return 모두 type을 뒤에 명시 한다.

```go
func func_name(arg arg_type) [return_var] return_type {

...
}
```

함수는 코드의 6번째 줄의 add1과 같이 선언합니다. 괄호 안에는
매개변수를 표시하는 데, 이 때 타입을 뒤에 표시합니다.

코드 6번째 줄 가장 뒤에 나오는 int는 리턴 타입입니다. 매개변수 타입과
마찬가지로 리턴 타입도 가장 뒤에 적어줍니다.

코드 11번째 줄 add2의 매개변수처럼 같은 타입의 변수를 여러 개 선언할
때에는 타입을 한 번만 적어줄 수 있습니다. 즉, add1과 add2는 동일하게
동작합니다.

```go
package main

import "fmt"

//① 매개변수 타입, 리턴 타입은 이름 뒤에 지정해줍니다

func add1(x int, y int) int {

return x + y

}

//② 매개변수 x, y가 같은 타입일 때에는 타입을 한 번만 명시해 줄 수
있습니다.

func add2(x, y int) int {

return x + y

}

func main() {

fmt.Println("add1(x int, y int)의 결과: ", add1(42, 13))

fmt.Println("add2(x, y int)의 결과: ", add2(42, 13))

}
```

## 함수 리턴

```
divide1과 divide2는 피제수(dividend)를 제수(divisor)로 나눠서
몫(quotient)과 나머지(remainder)를 return 하는 함수입니다.

함수는 여러 값을 한 번에 return 할 수 있는데요. 두 가지 방법이
있습니다.

divide1과 같이 return뒤에 리턴 타입을 적어주는 방법

divide2와 같이 return뒤에 리턴 할 변수를 선언하는 방법
```

```go
package main

import "fmt"

//① return 뒤에 리턴 타입을 적어주는 방법

func divide1(dividend, divisor int) (int, int) {

var quotient = (int)(dividend/divisor)

var remainder = dividend%divisor

return quotient, remainder

}

//② return뒤에 리턴할 변수를 선언하는 방법. ①과는 달리 함수 내부에서
`quotient`를 `var`로 선언하지 않고 바로 씁니다.

func divide2(dividend, divisor int) (quotient, remainder int) {

quotient = (int)(dividend/divisor)

remainder = dividend%divisor

return //return이라고만 적으면 미리 return값으로 정해 놓은
quotient와 remainder를 return합니다.

}

func main() {

//①로 한 번에 여러개의 결과를 return받는 부분

**var quotient, remainder int //①함수나 ②함수 모두 return 받을 변수를
선언해 주어야 한다.**

quotient, remainder = divide1(10, 3) //①함수나 ②함수나 return을
받는 방법은 같다

fmt.Println("①의 결과:", quotient, remainder)

//②로 한 번에 여러개의 결과를 return받는 부분

quotient, remainder = divide2(10, 3) //①함수나 ②함수나 return을
받는 방법은 같다

fmt.Println("②의 결과:", quotient, remainder)

}
```

## 변수 선언

**변수 선언시 var 키워드 무시 가능한 경우**

1. (함수 안) ':=' 사용할 경우

**변수 선언시 type 명시 무시 가능한 경우**

선언과 동시에 초기화 하는 경우

1. (함수 안/밖) 선언과 동시에 초기화('=', ':=' 사용)

2. (함수 밖) 선언과 동시에 초기화('=' 사용)

var 키워드와 type을 모두 무시 가능한 경우는 함수안에 ':='를 쓸 때
뿐이다.

SW 권장:

**함수 내, if/for statement) <name> := <value>**

**함수 외) var <name> = value**

```go
변수를 선언하는 방법은 여러 가지가 있습니다.

① 일반적으로 변수는 var로 선언하며, 특이하게 타입을 마지막에
적어줍니다.

var 변수명 타입

② 앞서 봤듯 같은 타입의 변수를 여러 개 선언할 때에는 타입을 한 번만
적어줍니다

③ 변수를 선언과 동시에 초기화할 때에는 타입을 지정하지 않아도 됩니다.

④ =대신 :=를 쓰면 함수 안에서 var를 명시하지 않고도 변수를 선언할 수
있습니다.

⑤ var 키워드 뒷부분을 괄호로 묶어, var를 한 번만 쓸 수도 있습니다

코드를 실행하면 15번째 줄에서 에러가 발생합니다.
함수 밖의 모든 statement들은 키워드(var, func 등)로 시작해야 하기
때문에 함수 밖에서는 :=를 쓸 수 없기 때문입니다.
이 줄을 지우고 다시 실행해보세요.
```

```go
package main

import "fmt"

//① 변수를 하나 선언

var num1 int

//② 같은 타입을 가지는 변수를 여러 개 선언

var num2, num3 int

//③ **여러 변수에 한 번에 값을 초기화 : 선언과 동시에 값을 초기화하면
타입을 명시할 필요가 없습니다.**

var num4, num5, str1 = 4, 5, "example"

//④ 함수 밖에서는 :=를 쓸 수 없습니다.

//errorvar := str1

//⑤ 다른 타입을 가지는 변수를 여러 개 선언

var (

i int

b bool

s string

)

func main(){

fmt.Println("①", num1)

fmt.Println("②", num2, num3)

fmt.Println("③", num4, num5, str1)

//④ **함수 안에서는 :=를 쓰면 var과 타입을 지정하지 않고 변수를
선언과 동시에 초기화할 수 있습니다.**

num6 := 6

fmt.Println("④", num6)

fmt.Println("⑤", i, b, s)

}
```

**[blank identifier] (_)**

<https://go.dev/doc/effective_go#blank>

사용되지 않을(다시 참조되지 않을) 임의의 값을 할당받는데 사용되는 식별자

일반적으로 사용될 값은 변수에 할당됨

var value = 1

다시 참조되지 않을 값은 blank identifier에 할당 가능

var _ = 1

공백 식별자는 모든 유형의 값으로 할당하거나 선언할 수 있으며 해당 값은
무해하게 삭제됩니다.

이는 Unix /dev/null 파일에 쓰는 것과 약간 비슷합니다.

```go
if _, err := os.Stat(path); os.IsNotExist(err) {
fmt.Printf("%s does not exist\n", path)
}

From <<https://go.dev/doc/effective_go#blank>>
```

```go
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
fmt.Printf("%s is a directory\n", path)
}

From <<https://go.dev/doc/effective_go#blank>>
```

```go
package main

import (
"fmt"
"io"
"log"
"os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader // For debugging; delete when done.

func main() {
fd, err := os.Open("test.go")
if err != nil {
log.Fatal(err)
}
// TODO: use fd.
_ = fd
}

From <<https://go.dev/doc/effective_go#blank>>
```

## 기본자료형

Go 언어 기본 자료형은 다음과 같습니다

bool : true, false를 저장합니다

string : 문자 / 문자열을 저장합니다

int / int8 / int16 / int32 / int64 / uint / uint8 / uint16 / uint32 /
uint64 / uintptr

byte : uint8과 같습니다

rune : int32와 같습니다. 유니코드 포인트를 나타냅니다.

float32 / float 64

complex64 / complex128

**int, uint, uintptr 타입은 32-비트 시스템에서는 32비트 길이고,
64-비트 시스템에서는 64비트 길이**입니다.\
특별히 정수의 크기나 부호(unsigned)를 지정할 이유가 없다면 int를 쓰면
됩니다.

Go는 **묵시적 형 변환을 하지 않아, 타입이 맞지 않으면 값을 할당할 수
없습니다.\
이럴 때는 변환할 타입(변환할값)과 같이 형 변환을 꼭 해줘야 합니다.**

코드를 실행하면 21번째 줄에서 에러가 발생합니다.\
int타입 값을 float64 타입을 저장하는 변수에 할당하기 때문입니다. 이
부분을 지우고 실행해보세요.

```go
package main

import (

"fmt"

"math/cmplx"

)

var (

i int

f float64

MaxInt uint64 = 1<<64 - 1

z complex128 = cmplx.Sqrt(-5 + 12i)

)

func main() {

const format = "%T(%v)\n" //%T는 변수의 타입 %v는 변수의
값

fmt.Printf(format, MaxInt, MaxInt)

fmt.Printf(format, z, z)

// int에서 float64로 묵시적 타입 변환을 할 수 없습니다

//f = i

// 다른 타입을 저장하려면 변환할타입(변수) 와 같이, 형 변환을
해줘야 합니다.

f = float64(i)

}
```

## 변수 초기화

선언만하고 초기화하지 않은 변수는 zero value로 초기화됩니다.\
zero value는 변수의 타입에 따라 다음과 같이 나뉩니다.

숫자 타입 : 0

boolean 타입 : false

string 타입 : ""

```go
package main

import (

"fmt"

)

var (

i int // zero value = 0

f float64 // zero value = 0

b bool // zero value = false

s string // zero value = ""

)

func main() {

fmt.Printf("int의 zero value: %v\n", i)

fmt.Printf("float64의 zero value: %v\n", f)

fmt.Printf("boolean의 zero value: %v\n", b)

fmt.Printf("string의 zero value: %q\n", s)

}
```

## 상수 선언

```
상수 선언은 변수 선언의 var 키워드 대신 const 키워드를 쓰면 됩니다.
상수가 될 수 있는 타입으로는 character, string, boolean, numeric
타입이 있습니다.

상수와 일반 변수는 크게 다음 3가지 차이점이 있습니다.

1. 상수는 값을 변경할 수 없습니다

2. 숫자형 상수는 var로는 표현할 수 없는 범위를 저장하는 등 수를
정밀하게 표현할 수 있습니다.

3. 타입을 지정하지 않은 상수는 맥락에 따라 타입이 변합니다.

코드를 실행시키면 16번째 줄에서 에러가 발생합니다.
Big_var에 1<<100를 대입하면서 오버플로우가 발생하기 때문입니다.

반면 같은 값 1 << 100을 대입해도, Big_const에서는 에러가 발생하지
않습니다.

또, 24\~25번째 줄을 보면,
타입을 명시하지 않은 상수 Small_const가 함수에 따라서 int타입이
되기도 하고,
float64 타입이 되기도 합니다.

주의 상수 선언은 const를 명시해야 하기 때문에 :=로는 선언할 수
없습니다.
```

```go
package main

import "fmt"

// 상수도 선언과 동시에 초기화하면 타입을 지정하지 않아도 됩니다.

const Pi1 float32 = 3.14

const Pi2 = 3.14

// 괄호로 묶으면 상수 키워드를 한 번만 명시합니다.

const (

Big_const = 1 << 100

Small_const = Big_const >> 99

)

// 오버플로우가 발생합니다.

//var Big_var = 1 << 100

func needInt(x int) int { return x*10 + 1 }

func needFloat(x float64) float64 {

return x * 0.1

}

func main() {

fmt.Println("needInt(Small_const):", needInt(Small_const)) //
Small_const 상수 선언시 타입을 지정하지 않았기 때문에 int형으로
자동으로 변환됩니다

fmt.Println("needFloat(Small_const):", needFloat(Small_const)) //
Small_const 상수 선언시 타입을 지정하지 않았기 때문에 float64형으로
자동으로 변환됩니다

fmt.Println("needFloat(Big_const):", needFloat(Big_const))

}
```

## for 반복문

```bash
for문은 세미콜론을 기준으로 세 부분으로 나뉘며, 각각 비워두는 것도
가능합니다.

타 언어와는 달리 괄호()가 없고, 중괄호{}가 필수입니다.

init statement; condition expression; post statement {}

Go언어에 반복문은 for문이 유일합니다. while문을 써야할 경우에는
다음과 같이 for문을 써서 흉내냅니다.

for 조건 {}

마지막 줄에 for {} 를 추가해 무한루프를 실행해 보세요.
```

```go
package main

import "fmt"

func main() {

sum := 0

for i := 0; i < 10; i++ { **//for문의 init statement에서는 :=
사용**

sum += i

}

fmt.Println(sum)

// 세미콜론 없이, C의 while과 비슷하게 쓸 수 있습니다.

sum = 1

for sum < 1000 {

sum += sum

}

fmt.Println(sum)

// 0부터 99까지 출력

// Println() 에서는 format 적용 안됨. 단순 String으로 처리됨.

// ++i와 같이 연산자가 앞에 올 수 없음. i++ 사용

const format = "count: %v\n"

for i := 0; i < 100; i++ {

fmt.Printf(format, i)

}

// for{} 로 무한 루프를 만들어보세요!

// 무한루프로 실행시간이 5초를 초과하면 오류와 함께 실행중단

// for {}

}
```

## if 조건문

```bash
if문도 for문과 마찬가지로 괄호()는 필요없고, 중괄호{}는 필수입니다.

if문을 쓸 때는

①. if 조건문 {} : 바로 조건문을 검사

②. if statement; 조건문 {} : 조건을 검사하기 전에 간단한 statement를
실행

②에 if 뒤 statement에서 선언한 변수는 if문 내에서만 쓸 수 있습니다.

코드를 실행하면 12번째 줄에서 에러가 발생합니다.

Go언어 if-else문을 쓸 때, if문이 끝나는 줄에 else문을 선언해줘야
합니다.

정상)

if 조건문 {

...

} else if 조건문 {

...

} else {

...

}

오류)

if 조건문 {

...

**}
else {**

...

}
```

```go
package main

import (

"fmt"

"math"

)

//else문은 if문이 닫히는(}) 줄과 함께 쓰여야 합니다.

func sqrt(x float64) string {

if x < 0 {

return sqrt(-x) + "i"

} else if x >= 0 && x < 5 {

return fmt.Sprint(math.Sqrt(x)) + "m"

} else {

return fmt.Sprint(math.Sqrt(x))

}

}

func pow(x, n, lim float64) float64 {

if v := math.Pow(x, n); v < lim {

return v

}

// v는 if문 내부에서만 쓸 수 있고, 여기부터는 쓸 수 없습니다.

return lim

}

func main() {

fmt.Println(sqrt(10), sqrt(2), sqrt(-4))

fmt.Println(

pow(3, 2, 10),

pow(3, 3, 20),

)

}
```

## switch 문

```
switch는 위에서부터 아래로 case를 검사하며, break를 하지 않아도
자동으로 case를 종료합니다.

예를 들어 아래 코드에서 i==0이면 f()는 실행되지 않습니다.

switch i {

case 0:

case f():

}

case가 매칭 되고 실행 후 종료하고 싶지 않으면(다음 case나 default를
이어서 진행하고 싶으면) 끝에 fallthrough를 추가하면 됩니다. 단,
**fallthrough 사용시 반드시 다음 case문이나 default문이 있어야
합니다.** 그렇지 않으면 syntax 오류가 발생합니다.
switch i {

case 0:

fallthrough

case 1:

~~fallthrough~~

}
```

```go
package main

import (

"fmt"

)

func no_fallthrough(score int) {

var grade string

switch {

case score > 90:

grade = "A"

case score > 70:

grade = "B"

case score > 50:

grade = "C"

default:

grade = "F"

}

fmt.Println("fallthrough를 쓰지 않으면", grade, "란다")

}

func yes_fallthrough(score int) {

var grade string

switch {

case score > 90:

grade = "A"

fallthrough

case score > 70:

grade = "B"

fallthrough

case score > 50:

grade = "C"

fallthrough

default:

grade = "F"

}

fmt.Println("fallthrough를 쓰면", grade, "란다")

}

func main() {

fmt.Println("교수님 제 성적이 어떻게 되나요?")

score := 100

yes_fallthrough(score)

no_fallthrough(score)

}
```

실행결과)

교수님 제 성적이 어떻게 되나요?

fallthrough를 쓰면 B 란다

fallthrough를 쓰지 않으면 A 란다

## 포인터 사용하기

포인터는 변수의 메모리 주소를 저장합니다.

***T** 타입은 **T value의 포인터**이며, **zero value**는 **nil**로
설정되어 있습니다.

**&**연산자는 **피연산자의 포인터를 생성**하며, *****연산자는
**포인터가 가리키는 값을 참조**합니다

또한 C와 달리 **포인터 연산은 불가능**합니다.

```go
package main

import "fmt"

func main() {

i, j := 42, 2701

p := &i // i를 가리키는 포인터

fmt.Println(*p) // 포인터를 통해 i 값을 읽습니다

*p = 21 // 포인터를 통해 i값을 설정합니다

fmt.Println(i)

p = &j // j를 가리킵니다

p = p / 37 // 포인터를 통해 j를 나눕니다

fmt.Println(j)

}
```

## 구조체 사용하기

구조체는 필드(들)의 집합으로, 다음과 같이 선언할 수 있습니다.

**type <name> struct {}**

위 선언에서, 구조체 type 이름은 name으로 쓸 수 있으며, 구조체의
필드는 **.**로 접근할 수 있습니다

구조체 인스턴스를 생성할 때, 특정 필드만 초기화 해주고 싶으면
**<name>{초기화할필드: 초기화할값}**을 지정합니다.

```go
package main

import "fmt"

type Vertex struct {

X int

Y int

}

// 구조체 인스턴스 선언 방법

var (

//① 일반적인 선언방식입니다. X가1, Y가 2로 초기화됩니다

v1 = Vertex{1, 2}

//② X만 값을 지정해주고, Y는 int에 zero value로 설정됩니다.

v2 = Vertex{X: 1}

//③ X, Y모두 int에 zero value로 설정됩니다.

v3 = Vertex{}

)

func main() {

fmt.Println("v1.X값:", v1.X)

v1.X = 4

fmt.Println("v1.X = 4로 바꾼 v1.X값:", v1.X)

//④ 구조체 포인터로도 구조체의 값을 바꿀 수 있습니다.

var p = &v1

p.X = 10

fmt.Println("포인터로 바꾼 v1.X값:", v1.X)

}
```

## 배열 사용하기

```go
타입 [n]T는 타입 T 값을 n개 저장하는 배열이며, **배열 크기는 한 번
설정하면 바꿀 수 없습니다.**

//배열 선언

var a [10]int

//배열의 초기화

var a1 = [3]int{1, 2, 3}

var a2 = [...]int{1, 2, 3} //배열 크기 자동으로

//다차원배열

var multiArray [3][4][5]int //정의

multiArray[0][1][2] = 10 //사용

//다차원 배열의 초기화

func main() {

var a= [2][3]int{

{1, 2, 3},

{4, 5, 6},

}

println(a[1][2])

}

위 코드는 a를 정수 10개를 저장하는 배열로 선언합니다.
```

```go
package main

import "fmt"

func main() {

// ① 배열선언과 원소 초기화를 따로

var a [2]string

a[0] = "Hello"

a[1] = "World"

fmt.Println("a[0], a[1]:", a[0], a[1])

fmt.Println("a:", a)

// ② 배열선언과 초기화를 동시에

primes := [6]int{2, 3, 5, 7, 11, 13}

fmt.Println("primes:", primes)

}
```

## 슬라이스 사용하기1

```
slice (슬라이스)? 배열의 레퍼런스라고 생각할 수 있다.
단, 슬라이스는 배열과 연동하여 생성 되므로 최대 크기가 배열
배열은 고정 길이인 반면 슬라이스는 가변 길이로, 슬라이스를 쓰면
배열을 동적인것 처럼 쓸 수 있습니다.

[]T는 타입 T원소들에 대한 slice이며, 다음 코드는 배열 a의
첫번째(index 0) 원소부터 다섯번째(index 4) 원소까지의 슬라이스를
생성합니다.

a[0:5]

슬라이스는 배열의 참조와 비슷합니다. 값을 저장하지는 않지만,
슬라이스를 통해 배열의 값에 접근하거나 값을 수정할 수 있습니다. 또
슬라이스의 슬라이스를 만드는것 또한 가능해 Go언어에서는 배열보다
슬라이스를 더 많이 씁니다.
참고) 슬라이스에서 값을 바꾸면 원본 배열의 값도 바뀐다.
```

```
슬라이스를 생성하는 방법
1) make 함수 사용

x := make([]float64, 5) //make(slice_type, slice_length)

x := make([]float64, 5, 10) //make(slice_type, slice_length,
slice_inner_capacity)

2) [low:high] syntax 사용

참고) <arrayname>[low:high] syntax를 통해서 슬라이스를 생성
```

```go
package main

import "fmt"

func main() {

names := [4]string{

"John",

"Paul",

"George",

"Ringo",

}

fmt.Println("배열 names:", names)

fmt.Println("①슬라이스 선언")

// 슬라이스 선언방법

// ① 일반적인 선언방법 : 변수 선언과 비슷합니다. 슬라이스타입은
[]type입니다.

var s1 []string = names[0:3]

// ② 슬라이스도 var키워드와 타입 명시를 생략할 수 있습니다.

s2 := names[0:2]

fmt.Println("names[0:3]:", s1)

fmt.Println("names[0:2]:", s2)

//s1에서 값을 바꾸면 names, s1에서도 바뀐 값을 볼 수 있습니다.

fmt.Println("②슬라이스로 값 변경")

fmt.Println("s1[0]", s1[0])

s1[0] = "XXX"

fmt.Println("s1[0] = XXX 실행 후 s1:", s1)

fmt.Println("s1[0] = XXX 실행 후 s2:", s2)

fmt.Println("s1[0] = XXX 실행 후 names:", names)

s2 = s1[0:2]

fmt.Println("s2 = s1[0:2] 실행 후 s2:", s2)

}
```

## 슬라이스 사용하기2

슬라이스 리터럴은 길이가 없는 배열 리터럴과 같습니다.

배열 리터럴은 다음과 같이 표현할 수 있는데요,

**[3]**bool{true, true, false}

아래 코드는 위와 같은 배열을 만든 후, 그 배열을 참조하는 슬라이스를
만듭니다:

**[]**bool{true, true, false}

**\^ 슬라이스는 배열과 달리 길이 표기가 없다.**

슬라이스의 상한과 하한을 지정하지 않을 경우 기본 값으로 설정됩니다.\
배열 var a [10]int에 대해 다음 슬라이스는 모두 같은 의미입니다.

a[0:10]

a[:10]

a[0:]

a[:]\

```go
package main

import "fmt"

func main() {

fmt.Println("① 슬라이스 리터럴 선언")

//① 기본형 슬라이스 리터럴

q := []int{2, 3, 5, 7, 11, 13}

fmt.Println("기본형 슬라이스 리터럴:", q)

//②구조체 슬라이스 리터럴

s := []struct {

i int

b bool

}{

{2, true},

{3, false},

{5, true},

{7, true},

{11, false},

{13, true},

}

fmt.Println("구조체 슬라이스 리터럴:", s)

fmt.Println("② 슬라이스를 슬라이스")

q = q[:2]

fmt.Println("q[:2]:", q)

q = q[1:]

fmt.Println("q[1:]:", q)

}
```

## 슬라이스 사용하기3

슬라이스 s에 대해 슬라이스의 length와 capacity는 다음과 같이 표현할
수 있습니다.

length : **len**(s) - 슬라이스가 포함하고 있는 원소의 개수

capacity : **cap**(s) - 슬라이스가 가리키는 **배열**에서 슬라이스의
첫번째 원소부터 센 원소 개수

슬라이스의 zero value는 nil이고, 가리키는 **배열**이 없으며 length와
capacity 모두 0입니다.

다음 코드에서 capacity가 어떻게 증가하고 있는지 확인해보세요

make함수로 0으로 초기화된 배열을 생성하고, 이를 가리키는 슬라이스를
리턴받아 가변 길이 배열을 만들어낼 수 있습니다. 새 원소를 추가할 때는
**func append(s []T, vs ...T) []T**를 씁니다.\
append 함수의 첫번째 인자에 타입 T 슬라이스 s를 넣고,\
그 다음으로 추가할 T값(들)을 전해주면 값이 추가된 슬라이스를
리턴합니다.

```go
package main

import "fmt"

func main() {

// make()로 가변 길이 배열 만들기

a := make([]int, 5)

fmt.Printf("a := make([]int, 5)의\t")

printSlice(a)

b := make([]int, 0, 5)

fmt.Printf("b := make([]int, 0, 5)의\t")

printSlice(b)

c := b[:2]

fmt.Printf("c := b[:2]의\t\t")

printSlice(c)

d := c[2:5]

fmt.Printf("d := c[2:5]의\t\t")

printSlice(d)

// 한번에 여러개 원소를 추가할 수 있습니다.

d = append(d, 1,2,3)

fmt.Printf("d = append(d, 1,2,3)후\t")

printSlice(d)

}

func printSlice(s []int) {

fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)

}
```

```
실행결과>

a := make([]int, 5)의        len=5 cap=5 [0 0 0 0 0]

b := make([]int, 0, 5)의        len=0 cap=5 []

c := b[:2]의                len=2 cap=5 [0 0]

d := c[2:5]의                len=3 cap=3 [0 0 0]

d = append(d, 1,2,3)후        len=6 cap=6 [0 0 0 1 2 3]
```

## 반복문과 range

반복문에서 **range**문을 쓰면 슬라이스나 맵을 이터레이트할 수
있습니다.

슬라이스에서 range를 쓸 경우, 인덱스, 원소 값이 매 순회마다 리턴
됩니다.\
이 때 인덱스나 원소 값이 필요 없다면 변수를 지정하지 않고 **_**를 쓸
수 있습니다.

```go
package main

import "fmt"

var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {

//① 일반적인 range

fmt.Println("① 일반적인 range")

for i, v := range pow {

fmt.Printf("2**%d = %d\n", i, v)

}

//② 인덱스가 필요없는 경우 _로 비워둘 수 있습니다.

fmt.Println("\n② 인덱스가 필요없는 경우")

for _, v := range pow {

fmt.Println(v)

}

}
```

## Map 선언하기

맵은 key에 value를 지정하는 자료형입니다.\
맵을 만들 때에는 **make 함수**를 써야 하며, zero value는
[nil]{.mark}입니다.

key값에 접근할 때에는 [맵변수[키]]{.mark}와 같이 접근합니다.\
\
맵 리터럴은 구조체 리터럴에 key를 추가한 것과 같고, 맵 리터럴은 make
함수가 필요 없습니다.

```go
package main

import "fmt"

type Vertex struct {

Lat, Long float64

}

func main() {

//① map 사용

//map[string] 타입 변수 선언

var mymap map[string]Vertex
//make()로 맵 생성

mymap = make(map[string]Vertex)

mymap["Bell Labs"] = Vertex{

40.68433, -74.39967,

}

fmt.Println("① mymap["Bell Labs"]: ", mymap["Bell
Labs"])
//② map literal 사용

var mymap_literal = map[string]Vertex{

"Bell Labs": Vertex{

40.68433, -74.39967,

},

"Google": Vertex{

37.42202, -122.08408,

},

}

fmt.Println("② mymap_literal["Bell Labs"]",
mymap_literal["Bell Labs"])

}
```

① mymap["Bell Labs"]: {40.68433 -74.39967}

② mymap_literal["Bell Labs"] {40.68433 -74.39967}

## Map 사용하기

맵을 조작하는 방법에 대해 알아봅시다.

1. 맵에 원소를 추가하려면

[m[key] = elem]{.mark}

2. 맵에서 특정 키 값을 가져오려면

[elem = m[key]]{.mark}

3. 맵에 원소를 제거하려면

[**delete**(m, key)]{.mark}

4. 맵에 키가 존재하는지 확인하려면

[elem, ok = m[key]]{.mark}

[m]{.mark}에 [key]{.mark}가 있으면 [ok]{.mark}가 [true]{.mark}이고,
없으면 [false]{.mark}입니다.

```go
package main

import "fmt"

func main() {

m := make(map[string]int)

//① key-value 지정하기

m["Answer"] = 42

fmt.Println("m["Answer"]값은:", m["Answer"])

//② key-value 삭제하기

delete(m, "Answer")

fmt.Println("m["Answer"]값은", m["Answer"])

//③ key존재 확인하기

v, ok := m["Answer"]

fmt.Println("m["Answer"]값은", v, "존재하나요?", ok)

}
```

m["Answer"]값은: 42

m["Answer"]값은 0

m["Answer"]값은 0 존재하나요? false

## 함수 값(Function values)

함수도 값이기 때문에 다른 값과 똑같이 사용할 수 있습니다.\
함수를 변수에 대입하거나, 다른 함수에 인자로 넘기는 것도 가능합니다.

22번째 줄은 함수 hypot을 변수에 저장하고 있으며,

28번째 줄은 함수 hypot을 다른 함수에 인자로 쓰고 있습니다.

```go
package main

import (

"fmt"

"math"

)

func compute(fn func(float64, float64) float64) float64 {

return fn(3, 4)

}

func main() {

//① 함수를 변수에 할당해, 변수를 함수처럼 씁니다.

hypot := func(x, y float64) float64 {

return math.Sqrt(xx + yy)

}

fmt.Println("①변수를 통해 함수 호출", hypot(5, 12))

//② 함수를 compute함수에 인자로 전달합니다.

fmt.Println("②함수를 함수에 인자로 전달")

fmt.Println("compute(hypot):\t\t", compute(hypot))

fmt.Println("compute(math.Pow):\t", compute(math.Pow))

}
```

```
①변수를 통해 함수 호출 13

②함수를 함수에 인자로 전달

compute(hypot):                 5

compute(math.Pow):         81
```

## 클로져

자기 바디 외부에 있는 변수를 참조하는 함수 값을 클로져라고 합니다.\
함수가 자신이 참조하는 변수에 접근하거나 값을 변경하는 경우, 함수가
변수에 bound되었다고 합니다

예를 들어 이 코드에서 adder는 클로져를 리턴 하며,

클로져 pos와 클로져 neg는 서로 다른 변수 sum을 가집니다.

```go
func adder() func(int) int {

sum := 0

return func(x int) int {

sum += x

return sum

}

}

func main() {

// pos, neg는 클로져이며 서로 다른 변수 sum을 가집니다.

pos, neg := adder(), adder()

for i := 0; i < 10; i++ {

fmt.Println(

i, ":",

pos(i),

neg(-2*i),

)

}

}
```

0 : 0 0

1 : 1 -2

2 : 3 -6

3 : 6 -12

4 : 10 -20

5 : 15 -30

6 : 21 -42

7 : 28 -56

8 : 36 -72

9 : 45 -90

## 메소드 선언하기

```go
Go에는 클래스가 없는 대신 리시버 인자를 갖는 함수로 struct의 멤버
함수를 정의할 수 있습니다.
리시버는 func 키워드와 함수 이름 사이에 인자로 들어갑니다.

Go언어도 다른 언어와 같이 타입 뒤에 점을 찍어 멤버 함수에 접근합니다.

---------------------------------
-----------------------------------

func (리시버 인자) 함수이름 리턴타입
---------------------------------
-----------------------------------

리시버는 c++이나 java에서 this 키워드와 같이 struct instance를
지정하는데 사용 됩니다.

다음과 같이 Vertext struct의 멤버 함수를 선언하면

func (v Vertex) Abs() float64 {

return math.Sqrt(v.Xv.X + v.Yv.Y)

}

v.X 와 v.Y는 java/c++에서

this.X, this.Y와 같은 표현이 됩니다.

일반 리시버와 포인터 리시버가 있는데

일반 리시버는 struct를 call by value 방식(값 복사)으로 참조하고,

포인터 리시버는 struct를 call by reference 방식(주소 참조)으로
참조합니다.

일반 리시버 선언

func (v Vertex) Abs() float64 {}

포인터 리시버 선언

func (v *Vertex) Abs() float64 {}

멤버 함수 내에서 struct 인스턴스의 field 값을 바꾸려면 포인터
리시버를 사용해야 합니다.
48, 51번째 줄에서 메소드 power10과 power100이 각각 어떻게 다른지
확인해보세요.

18, 43번째 줄은 Myfloat과 같은 기본형 타입에 Abs 메소드를 선언해
접근하고 있습니다.
```

```go
package main

import (

"fmt"

"math"

)

type Vertex struct {

X, Y float64

}

//① Abs 메소드는 리시버인자로 v Vertex를 받습니다.

func (v Vertex) Abs() float64 {

return math.Sqrt(v.Xv.X + v.Yv.Y)

}

//② 기본형 타입(여기는 float64)도 메소드를 만들수 있습니다.

type MyFloat float64

func (f MyFloat) Abs() float64 {

if f < 0 {

return float64(-f)

}

return float64(f)

}

//③ MyFloat이 포인터가 아닌 리시버 인자입니다

func (f MyFloat) power10() {

f = f * MyFloat(10)

}

//④ MyFloat이 포인터 리시버 인자입니다.

func (f *MyFloat) power100() {

f = f * MyFloat(100)

}

func main() {

v := Vertex{3, 4}

fmt.Println("① 점을 찍어 메소드에 접근합니다")

fmt.Println("v.Abs():", v.Abs())

f := MyFloat(-math.Sqrt2)

fmt.Println("②numeric type도 메소드 정의가 가능합니다")

fmt.Println("f.Abs():", f.Abs())

fmt.Println("③포인터 리시버를 쓰면 메소드 내부에서 값을 바꿀 수
있습니다")

fmt.Println("기존의 f\t\t\t\t", f)

f.power10()

fmt.Println("일반 리시버를 써서 10을 곱한 경우\t", f)

f.power100()

fmt.Println("포인터 리시버를 써서 100을 곱한 경우\t", f)

}
```

① 점을 찍어 메소드에 접근합니다

v.Abs(): 5

②numeric type도 메소드 정의가 가능합니다

f.Abs(): 1.4142135623730951

③포인터 리시버를 쓰면 메소드 내부에서 값을 바꿀 수 있습니다

기존의 f                                 -1.4142135623730951

일반 리시버를 써서 10을 곱한 경우         -1.4142135623730951

포인터 리시버를 써서 100을 곱한 경우         -141.4213562373095

## 리시버 타입

```
전 강의를 유심히 보신 분이라면, 포인터를 인자로 받는 함수는
값(value)을 전달받을 수 없고,
값(value)을 인자로 받는 함수는 포인터를 전달받을 수 없다는 걸
알아채셨을 겁니다.

-------
-----------------------------------
-----------------------------------

var v Vertex

ScaleFunc(v) // 컴파일 에러

ScaleFunc(&v) // 정상

--------
-----------------------------------
-----------------------------------

반면에 포인터 리시버는 값(value)과 포인터 모두 접근 가능합니다.

---------
-----------------------------------
-----------------------------------

var v Vertex

v.Scale(5) // 값으로도 접근할 수 있습니다

p := &v

p.Scale(10) // 포인터로도 접근할 수 있습니다

--------
-----------------------------------
-----------------------------------

포인터 리시버를 쓰는 이유는 다음과 같습니다.

함수 내부에서 리시버가 가리키는 값을 바꾸고 싶다.

함수가 호출될 때, 값이 복사(copy)되는 것을 피하고 싶다.

주의 value리시버나 포인터 리시버 모두 자유롭게 쓸 수 있지만, 보통
한 함수 안에서 둘을 한꺼번에 쓰지는 않습니다.
```

```go
package main

import "fmt"

type Vertex struct {

X, Y float64

}

//① Vertex 포인터 리시버가 있는 함수입니다. Vertex 혹은 Vertex
포인터로 접근할 수 있습니다.

func (v *Vertex) Scale(f float64) {

v.X = v.X * f

v.Y = v.Y * f

}

//② Vertex 포인터 인자가 있는 함수입니다.

// Vertex포인터만 인자로 들어올 수 있습니다.

func ScaleFunc(v *Vertex, f float64) {

v.X = v.X * f

v.Y = v.Y * f

}

func main() {

v := Vertex{3, 4}

v.Scale(2)

ScaleFunc(&v, 10)

fmt.Println("① Vertex{3, 4}로 접근했을 때:", v)

p := &Vertex{3, 4}

p.Scale(2)

ScaleFunc(p, 10)

fmt.Println("② &Vertex{3, 4}로 접근했을 때:", p)

}
```

① Vertex{3, 4}로 접근했을 때: {60 80}

② &Vertex{3, 4}로 접근했을 때: &{60 80}

## 인터페이스 사용하기

```go
인터페이스는 함수의 집합으로, 인터페이스 타입 값은 함수를 구현하는
값을 담을 수 있으며,

타입이 인터페이스에 함수를 구현하면 자동으로 그 인터페이스도 구현한
게 됩니다.

(다른 언어와는 달리 implements등의 키워드가 필요없습니다)

인터페이스 값은 value와 구체적인(concrete) 타입으로 구성된
tuple이라고 볼 수 있습니다.

---
-----------------------------------
-----------------------------------

type myinterface interface {

myfunction() int

}

type MyInt int

func (rcv MyInt) myfunction() int {

return 0

}

var a myinterface = MyInt(3)

----
-----------------------------------
-----------------------------------
```

```go
package main

import (

"fmt"

"math"

)

type I interface {

M()

}

type T struct {

S string

}

//① 별도의 키워드를 쓰지 않아도 T가 인터페이스 I를 구현하게 됩니다.

func (t *T) M() {

fmt.Println(t.S)

}

type F float64

//② 별도의 키워드를 쓰지 않아도 F가 인터페이스 I를 구현하게 됩니다.

func (f F) M() {

fmt.Println(f)

}

func main() {

var i I

fmt.Println("① i = &T{"Hello"}에 대해")

i = &T{"Hello"}

describe(i)

i.M()

fmt.Println("② i = F(math.Pi)에 대해")

i = F(math.Pi)

describe(i)

i.M()

}

func describe(i I) {

fmt.Printf("인터페이스의 (값, 타입) : (%v, %T)\n", i, i)

}
```

```
① i = &T{"Hello"}에 대해

인터페이스의 (값, 타입) : (&{Hello}, *main.T)

Hello

② i = F(math.Pi)에 대해

인터페이스의 (값, 타입) : (3.141592653589793, main.F)

3.141592653589793
```

## empty 인터페이스

```go
메소드가 하나도 없는 인터페이스를 empty 인터페이스라고 하며,

이 empty 인터페이스는 어떤 타입이던 저장할 수 있습니다.

-------------------------

interface{}

-------------------------

empty 인터페이스는 보통 타입을 알 수 없는 값을 처리할 때 사용합니다.

예를 들어 아래 코드에서 fmt.Print는 interface{} 타입의 변수들을
처리합니다.
```

```go
package main

import "fmt"

func main() {

fmt.Println("① empty interface에 대해")

var i interface{}

describe(i)

fmt.Println("① i = 42에 대해")

i = 42

describe(i)

fmt.Println("① i = "hello"에 대해")

i = "hello"

describe(i)

}

func describe(i interface{}) {

fmt.Printf("인터페이스 i의 (값, 타입) : (%v, %T)\n", i, i)

}
```

① empty interface에 대해

인터페이스 i의 (값, 타입) : (<nil>, <nil>)

① i = 42에 대해

인터페이스 i의 (값, 타입) : (42, int)

① i = "hello"에 대해

인터페이스 i의 (값, 타입) : (hello, string)

## 고루틴

```
goroutine은 Go runtime가 담당하는 경량(lightweight) 쓰레드 입니다.

-------------------------

go f(x, y, z)

-------------------------

는 다음의 새로운 goroutine을 실행하며,

f, x, y, z의 값을 구하는 것은 현재 goroutine에서 진행되고,

f를 실행하는 건 새로운 goroutine에서 진행됩니다.

--------------------------

f(x, y, z)

--------------------------

주의 goroutine은 같은 주소 공간을 쓰기 때문에, shared memory에
접근할 때는 동기화 해줘야 합니다.
```

```go
package main

import (

"fmt"

"time"

)

func say(s string) {

for i := 0; i < 5; i++ {

time.Sleep(100 * time.Millisecond)

fmt.Println(s)

}

}

func main() {

go say("② 다른 루틴")

say("① 이 루틴")

}
```

① 이 루틴

② 다른 루틴

② 다른 루틴

① 이 루틴

② 다른 루틴

① 이 루틴

② 다른 루틴

① 이 루틴

② 다른 루틴

① 이 루틴

## 고루틴

```
goroutine은 Go runtime가 담당하는 경량(lightweight) 쓰레드 입니다.

-------------------------

go f(x, y, z)

-------------------------

는 다음의 새로운 goroutine을 실행하며,

f, x, y, z의 값을 구하는 것은 현재 goroutine에서 진행되고,

f를 실행하는 건 새로운 goroutine에서 진행됩니다.

--------------------------

f(x, y, z)

--------------------------

주의 goroutine은 같은 주소 공간을 쓰기 때문에, shared memory에
접근할 때는 동기화 해줘야 합니다.
```

```go
package main

import (

"fmt"

"time"

)

func say(s string) {

for i := 0; i < 5; i++ {

time.Sleep(100 * time.Millisecond)

fmt.Println(s)

}

}

func main() {

go say("② 다른 루틴")

say("① 이 루틴")

}
```

① 이 루틴

② 다른 루틴

② 다른 루틴

① 이 루틴

② 다른 루틴

① 이 루틴

② 다른 루틴

① 이 루틴

② 다른 루틴

① 이 루틴

## 채널

```go
채널은 파이프로, 채널 오퍼레이터 <-를 통해 값을 주고받을 수
있습니다.

map이나 slice처럼 채널도 쓰기 전에 채널임을 선언해줘야
합니다.

------------
-----------------------------------
-----------------------------------

ch := make(chan int)

ch <- v // 채널 ch를 통해 v를 보냄.

v := <-ch // ch로부터 값을 전달받아, v에 할당.

------------
-----------------------------------
-----------------------------------

채널은 디폴트로 상대방이 준비 된 후 값을 주고받을 수 있기 때문에,
별도의 동기화 과정이나 condition variable 설정 없이 goroutine을 쓸 수
있습니다.
```

```go
package main

import "fmt"

// 배열 s에 모든 원소를 더해, 채널 c에게 전달하는 함수

func sum(s []int, c chan int) {

sum := 0

for _, v := range s {

sum += v

}

c <- sum // c에게 sum을 전달

}

func main() {

s := []int{7, 2, 8, -9, 4, 0}

c := make(chan int)

go sum(s[:len(s)/2], c)

go sum(s[len(s)/2:], c)

x, y := <-c, <-c // c로부터 값을 전달받음

fmt.Println("sum(s[:len(s)/2], c)의 결과:", x)

fmt.Println("sum(s[:len(s)/2:], c)의 결과:", y)

fmt.Println("둘의 합:", x+y)

}
```

```
sum(s[:len(s)/2], c)의 결과: -5

sum(s[:len(s)/2:], c)의 결과: 17

둘의 합: 12
```

## 버퍼채널

```go
버퍼 길이와 시간(초 단위)을 make에 인자로 전달하면 buffered 채널을
만들 수 있습니다.

ch := make(chan int, 100)

이렇게 하면 buffered 채널은 버퍼가 꽉 찰 때까지 블락되거나(값을
전송할 때),

버퍼가 다 빌 때까지 블락됩니다(값을 전달받을 때).

코드를 실행하면 데드락이 발생합니다.

7번째 줄에을 수정해, 버퍼 크기를 키워보세요.
```

```go
package main

import "fmt"

func main() {

//① 버퍼크기가 1인 채널을 만듭니다. **에러를 수정하려면 2 이상으로
크기를 키움니다.**

ch := make(chan int, 1)

//② 채널에 1을 전달합니다. 버퍼가 꽉 찼습니다.

ch <- 1

//③ 버퍼가 꽉 찬 상태에서 2를 또 전달합니다 - 버퍼를 비워줄 루틴이
없어 데드락이 발생합니다.

ch <- 2

fmt.Println(<-ch)

fmt.Println(<-ch)

}
```

fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:

main.main()

/tmp/go11887-1-1fr1iod/main.go:11 +0x97

## Range와 Close

```go
sender가 더 이상 보낼 값이 없어 채널을 닫으면 reciever가 이를 알아챌
수 있어야 합니다.

채널이 열려있는지 닫혀있는지 알아내는 방법은 다음과 같습니다.

---------------------------------

v, ok := <-ch

---------------------------------

채널로부터 더 이상 받을 값이 없고 채널이 닫혔다면,

두 번째 인자 ok가 false가 되며, 그렇지 않다면 ok는
true입니다.

반복문 안에서 채널로부터 값을 전달받을 때에는 굳이 ok로 확인할 필요
없이,

for i: range c를 써, 채널 c가 닫힐 때까지 값을 계속 전달받습니다.

코드를 실행하면 데드락이 발생합니다.

fibonacci 함수(sender)에서 채널을 닫지 않아, main의
for문(receiver)이 종료되지 않기 때문입니다.

15번째 줄 close(c)의 주석을 지워보세요.

주의1 : 오직 sender만 채널을 닫아야 합니다.

reciever가 채널을 닫아, 한쪽이 닫힌 채널에 데이터를 전송하면 패닉이
발생합니다.

주의2 : 채널은 파일과는 다릅니다.

보통은 닫을 필요 없고, 오직 sender가 더 이상 보낼 값이 없다는 뜻을
전달하는 의미에서 씁니다.
SW comment : 채널은 queue 다

쉽게 이해하자면 channel은 goroutine 쓰레드간의 데이터 전송을 위한
공유 자원이자 QUEUE라고 이해 할 수 있습니다.

이 channel이라는 QUEUE에 데이터를 put 혹은 pop 하려면 우선 아래 두
조건을 만족해야만 합니다.
조건 1. channel이 open 상태(close()가 호출 되지 않은 상태)이어야
한다.

조건 2. pop은 add된 데이터가 있는 상태에서만 가능하다.

위 조건이 만족 되지않으면 에러(패닉)이 발생합니다.

sender Thread가 close()를 호출하지 않고 더이상 데이터를 삽입 하지
않은 상태로 channel을 점유하고 있는데 receiver Thread가 channel에
대해 reading을 시도하게 되면 deadlock이 발생합니다.
```

```go
package main

import (

"fmt"

)

// sender에 해당하는 코드입니다.

// 0번째부터 n번째까지의 피보나치수를 채널 c를 통해 전달한 후, 채널을
닫습니다.

func fibonacci(n int, c chan int) {

x, y := 0, 1

for i := 0; i < n; i++ {

c <- x

x, y = y, x+y

fmt.Println("write at c: ",i)

}

//close(c)

}

func main() {

c := make(chan int, 10)

go fibonacci(cap(c), c)

//receiver에 해당하는 코드입니다.fibonacci 함수로부터 값을
전달받으며,
//sender측에서 채널을 닫으면 반복문이 종료됩니다.

for i := range c {

fmt.Println("read from c: ", i)

}

}
```

```go
//close() 미적용 시 실행 결과
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:

main.main()

/tmp/go11887-1-k7zu9v/main.go:23 +0xb2
```

```
//close() 적용 시 실행 결과
write at c: 0

write at c: 1

write at c: 2

write at c: 3

write at c: 4

write at c: 5

write at c: 6

write at c: 7

write at c: 8

write at c: 9

read from c: 0

read from c: 1

read from c: 1

read from c: 2

read from c: 3

read from c: 5

read from c: 8

read from c: 13

read from c: 21

read from c: 34
```

## 셀렉트 문에서 channel 사용

```
select문을 쓰면 goroutine은 적어도 하나의 case가 실행될 수 있을
때까지 블락됩니다.

case가 준비되면 case를 실행하는데, 여러 case가 준비된 경우는 준비된
case 중 하나를 무작위로 선택해 실행합니다. 준비된 케이스가 없을
때에는 select내의 default 케이스가 실행되며, 블락 없이 값을 주거나
받을 때 사용합니다.

------------------
-----------------------------------

select {

case i := <-c:

// i를 사용

default:

// c로부터 받는 게 블락

}

------------------
-----------------------------------
```

```go
package main

import (

"fmt"

"time"

)

func main() {

tick := time.Tick(100 * time.Millisecond)

boom := time.After(500 * time.Millisecond)

for {

select {

case <-tick:

fmt.Println("100\t밀리초 지났어요.")

case <-boom:

fmt.Println("500\t밀리초 지났어요. 종료합니다!")

return

default:

fmt.Println("50\t밀리초 지났어요.")

time.Sleep(50 * time.Millisecond)

}

}

}
```

50        밀리초 지났어요.

50        밀리초 지났어요.

100        밀리초 지났어요.

50        밀리초 지났어요.

50        밀리초 지났어요.

100        밀리초 지났어요.

50        밀리초 지났어요.

50        밀리초 지났어요.

100        밀리초 지났어요.

50        밀리초 지났어요.

50        밀리초 지났어요.

100        밀리초 지났어요.

50        밀리초 지났어요.

50        밀리초 지났어요.

100        밀리초 지났어요.

500        밀리초 지났어요. 종료합니다!

---

# [go] 주의 문법

**## 함수(function)과 메소드(method)**

Go 언어에서 일반적으로 함수는 특정 type에 종속되지 않는다.

리시버를 가지고 특정 type에 종속되는 멤버 함수를 메소드라고 한다.

리시버는 밸류 리시버와 포인터 리시버가 있으며

포인터 리시버에는 밸류형 변수나 포인터형 변수 모두를 전달 가능하다.

**## empty와 nil 차이**

empty는 인스턴스의 주소가 있으나 데이터는 없는 상태 이고,

nil은 인스턴스의 주소가 없는 상태이다.

**## 모듈 멤버의 첫문자의 대소문자에 따른 package간 access scope 제한**

.go 파일에 정의되는 global variable과 함수에 대해서

그 이름의 첫문자가 대문자이면 다른 외부 package에서 접근 가능

그 이름의 첫문자가 소문자이면 다른 외부 package에서 접근 불가능

```go
var PackageExternalGlobalVariable int

var packageInternalGlobalVariable int

func PackageExternalFunction(){}

func packageInternalFunction(){}
```

**## var**

변수 선언시 type deduction 표현

var v int = 1;

global variable에는 항상 var를 사용

local variable에서는 := 사용하여 초기화시 var 생략 가능

**## pass type call by value**

golang은 함수에 인자를 전달할 때 기본적으로 call by value 또는 call by
pointer를 사용한다.

call by value를 사용하는 경우 copy action이 발생한다.

다만 struct를 제외한 string, slice, map, queue, interface 등 golang에
사전 정의된 complex container들은 내부에 데이터에 대한 포인터를 가지는
구조이므로 call by value를 사용하여도 call by pointer를 사용한 것과
유사한 퍼포먼스를 가진다.

그러므로 struct 인스턴스를 reference로 함수에 전달하는 경우에만 call by
pointer 사용을 고려하고 그외에는 기본적으로 call by value를 사용하면
된다.

call by pointer 사용 목적:

전달 되는 원본 인스턴스의 값에 대한 변경이 필요할 때

복사가 발생하는 비용이 매우 높아 복사 비용을 없애야 할 때

primitive type :: call by value or call by pointer(reference)

string :: call by value

slice :: call by value

map :: call by value

queue :: call by value

interface :: call by value

struct :: call by pointer

**## receiver의 pass type**

receiver는 struct의 메소드(struct member function)을 정의할 때 사용되는
struct instance 전달자 이다.

메소드에 리시버를 지정할 때의 type 표현에 따라(포인터이냐 아니냐)서 멤버
함수 내부로의 원본 인스턴스의 전달 type이 달라진다.

예) 밸류형 리시버 - call by value(copy) receiver

func (p ProcessState) Pid() int {

return p.pid

}

예) 포인터형 리시버 - call by pointer(referece) receiver

func (p *ProcessState) Pid() int {

return p.pid

}

**## go 함수의 closure 표현에서 outter 변수 사용은 모두 call by
reference 이다.**

**Rule 1, Go 언어에는 함수는 일급 함수로써 하나의 객체로 다루어 진다.**

함수를 변수에 담거나 함수의 인자로 전달 하거나 리턴 값으로 사용할 수
있다.

**Rule 2, Anonymous function == function literal == lambda function ==
closure**

closure? 외부 함수의 로컬 변수에 접근할 수 있는 내부 함수 또는 그러한
문법

Go language에서 closure 문법은 Anonymous function, lambda, go routine
등에서 모두 사용될 수 있다.

내부 함수가 외부 함수의 로컬 변수를 접근할 때 이는 참조로써 접근을
수행하며 외부 함수의 값을 내부 함수에서 변형할 수 있다. 그래서 read
동작만 할 때는 문제가 없지만 내부 함수에서 외부함수 변수에 대한 변경을
수행할 때는 적절한 동기화 방식을 적용(mutex or "atomic variable")해야
한다.

내부 함수가 외부 함수의 로컬 변수를 call by value로 접근하고자 한다면
변수를 내부 함수에 parameter로써 전달하는 방식을 사용해야 한다.

**Rule 3, closure 상황에서 외부함수의 로컬 변수와 같은 이름의 변수를
내부함수에서 다른 인스턴스로 선언 가능하다.**

다음의 예에서 외부 함수의 redeclare 변수와 go routine에 사용된 내부
함수의

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

**## 변수 선언시과 default 값**

golang 에서 모든 변수는 따로 초기값을 부여 코드를 작성하지 않는다면
default 값(0, nil, false)으로 초기화 된다.

primitive type, bool, struct, pointer 등 모든 type은 default 값으로
초기화 되므로 C/C++과 달리 로컬 변수 선언시 쓰레기 값을 염려해 zero
initialize를 명시할 필요가 없다.

모든 메모리가 할당이 발생하는 변수 표현은 기본적으로 default 값으로
초기화 된다.

var myint int // 0

var mybool bool // false

myType := MyType{} // 모든 멤버도 default값(0, nil, fase) 가짐

mypointer := new(MyType) // 모든 멤버도 default값(0, nil, fase) 가짐

mypointer := &MyType{} // 모든 멤버도 default값(0, nil, fase) 가짐

**## 구조체 초기화: new(), {}**

type <struct-name> struct {}

v := <struct-name>{초기화할 필드: 초기화할 값}

struct 초기화에는 {}를 사용한다.

new() :

struct 타입에 대해 메모리를 할당하고 zero initialization 한 후 포인터를
타입의 포인터를 반환.

struct 타입에 대해 {} instanciation하는 것은 new()로 초기화 하는 것과
같다.

아래 두 표현식 같은 의미이다

m := new(MyType)

fmt.Print("%T %#v", m, m) // *main.MyType &main.MyType{}

m := &MyType{}

fmt.Print("%T %#v", m, m) // *main.MyType &main.MyType{}

**## channel, slice, map 초기화: make(), {}**

channel, slice, map 초기화에는 size설정을 위하여 make()를 사용한다.

** channel, slice, map을 생성할 때는 new() 대신 반드시 {}나 make()를
사용해야 한다.

channel, slice, map에 대해 {} instanciation하는 것은 make(<type>, 0)로
초기화하는 것과 같다.

new()는 메모리 할당 후 할당 영역을 zero(0)로 초기화하고 포인터
반환하는데 반해

make()는 메모리 할당 후 channel, slice, map struct 타입에 맞는 각
멤버들에 대해서 기본값으로 초기화하고 레퍼런스 반환

slice를 초기화하는 표현과 ptr 여부

  ------------------------------------------------------------------------
  **초기화 표현**                    **ptr**       **len**       **cap**
  ---------------------------------- ------------- ------------- ---------
  nil: []string                    0             0             0

  empty: []string{}                <addr>      0             0

  empty: make([]string, 0)         <addr>      0             0
  ------------------------------------------------------------------------

**## [Anonymous
struct](https://blog.boot.dev/golang/anonymous-structs-golang/),
composit literal**

anonymous struct type:

```
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

```
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

mileage: 200000,

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

**Don't use map[string]interface{} for JSON data if you can avoid
it.**

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

**## interface vs empty interface(interface{}, any)**

**interface와 empty interface는 완전히 다른 역할을 하는 타입이다.**

interface는 메소드들의 집합이며 struct와 마찬가지로 struct 인스턴스의
타입으로 사용된다.

empty interface는 interface에 {}가 붙은 "interface{}" 표현식 또는 any
타입으로 표현이 가능하며 golang에서 dynamic type의 역할을 한다. empty
interface는 모든 primitive 타입과 pointer, slice 타입의 인스턴스를 담을
수 있다.

empty interface와 유사하게 empty struct 타입(struct{})도 존재하나 이는
테스트의 경우 외에는 거의 사용처가 없다.

**## interface**

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

**## 덕타이핑**

struct가 타입으로 사용되는 것과 같이 interface 또한 타입으로 사용된다.

단, interface 타입 변수에 할당되는 인스턴스는 interface 타입으로 생성할
수 없다.

interface에 정의된 모든 메소드를 구현하는 struct 인스턴스가 interface
타입으로 할당 될 수 있다.

만약 A interface에 정의된 모든 메소드를 구현하는 struct는 여러 개이며
이들을 B, C, D struct 라고 한다면 B, C, D struct 는 모두 A interface가
될 수 있다. 이러한 방식을 덕타이핑 이라고 한다.

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

lastRead readOp // last read operation, so that Unread* can work
correctly.

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

interface{}(<struct instance>).(<interface type>)

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

**## [empty struct:
struct{}](https://dave.cheney.net/2014/03/25/the-empty-struct)**

struct{} ? 말그대로 "빈 구조체 타입"

struct{}{} ? empty struct의 인스턴스

주소값이 없고 사이즈가 0이다.

unsafe.Sizeof() 메소드는 해당 변수의 크기값을 반환함

```go
emptyStruct := struct{}{}

fmt.Println(&emptyStruct) // &{}

fmt.Println(unsafe.Sizeof(emptyStruct)) // 0

boolVariable := true

fmt.Println(&boolVariable) // 0xc42001a088

fmt.Println(unsafe.Sizeof(boolVariable)) // 1
```

용법 1, 테스트 struct 타입 변수에 빈 instance 할당하기

**용법 2, channel에 단순 시그널 보내기**

  -----------------------------------------------------------------------
  done <- struct{}{}
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

bool 과 empty struct로 signal보낼때 퍼포먼스 비교?

empty struct가 bool 사용 대비 절반의 시간만 소비된다.

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

KAKAOui-MacBook-Pro-5:benc2 aidan$ go test -bench=. -v

goos: darwin

goarch: amd64

BenchmarkGetBool-8         2000000000         0.63 ns/op

BenchmarkGetStruct-8         2000000000         0.31 ns/op
```

**## empty interface**

**<> "interface{}" empty interface type**

empty interface = 인터페이스 타입 = nil을 제외한 golang의 모든 타입을
담을 수 있는 컨테이너, **Dynamic Type**(주: java의 object, c/c++의
void*)

int, string 등 **primitive 타입과 포인터와 슬라이스** 까지 모든 형식을
대신할 수 있다.

"interface{}" type은 golang의 모든 타입을 대신할 수 있다.

심지어 []interface{} (interface{}의 slice)도 slice 이므로
"interface{}"로 그 타입을 표현 할 수 있다.

interface{}는 "any" 라는 alias로 명시 가능하다.

**interface{} type의 변수에 접근하려면 반드시 type assert를 명시해
주어야한다.**

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

**"type switch": interface{} (empty interface) type 변수의 실제 타입이
slice인지 확인 하는 방법**

```bash
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

**## type casting**

**1. type conversion (형변환)**

golang 에서는 자동 형변환이 없으며 형변환을 하려면 반드시 명시적으로
type casting을 해주어야 한다.

var n int = 15

var v1 int64 = int64(n)

**2. type assertion (interface type의 형변환)**

type assertion은 "interface{}" 타입 변수가 가지고 있는 실제
값(concrete value)에 접근할 수 있게 해준다.

(golang의 "interface{}"는 임의의 타입을 의미하며 임의의 값을 가질 수
있기 때문에 concrete라는 표현을 쓴 듯 하다.)

var n interface{} = 15

v := n.(int)

or

v, ok := n.(int)

**## 구조체 :: member method on struct**

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
```

Rectangle is: {4 3}

Rectangle area is: 12

Rectangle area is: 12

Rectangle area is: 12

Rectangle area is: 12

Rectangle instace는 call by value나 call by reference 모두로 전달될 수
있고 그 결과는 같다.

컴파일러가 자동으로 call by reference로 모두 최적화 하기 때문이다.

**[TODO] [## 구조체 :: anonymous
field](https://golangbyexample.com/anonymous-fields-struct-golang/)**

**## 구조체 :: member method on anonymous field**

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

h := House{Kitchen{4, 4}} //the kitchen has 4 forks and 4 knives

fmt.Println("Sum of forks and knives in house: ",
h.totalForksAndKnives()) //called on House even though the
method is associated with Kitchen

}
```

Sum of forks and knives in house: 8
