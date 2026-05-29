# [go] built-in type and func

type bool bool

const (

true = 0 == 0 // Untyped bool.
false = 0 != 0 // Untyped bool.

)

type uint8 uint8

type uint16 uint16

type uint32 uint32

type uint64 uint64

type int8 int8

type int16 int16

type int32 int32

type int64 int64

type float32 float32

type float64 float64

type complex64 complex64

type complex128 complex128

type string string // may be empty but not nil

type int int

type uint uint

type uintptr uintptr

type byte = uint8

type rune = int32

type any = interface{}

type comparable interface{ comparable }

const iota = 0 // Untyped int.

var nil Type **// Type must be a pointer, channel, func, interface, map,
or slice**

type Type int

type Type1 int

type IntegerType int

type FloatType float32

type ComplexType complex64

func append(slice []Type, elems ...Type) []Type

func copy(dst, src []Type) int

func delete(m map[Type]Type1, key Type)

func len(v Type) int **// Type must be a pointer, array, slice, string,
channel**

func cap(v Type) int **// Type must be a pointer, array, slice,
channel**

func make(t Type, size ...IntegerType) Type

func new(Type) *Type

func complex(r, i FloatType) ComplexType

func real(c ComplexType) FloatType

func imag(c ComplexType) FloatType

func close(c chan<- Type)

func panic(v any)

func recover() any

func print(args ...Type)

func println(args ...Type)

type error interface {

Error() string

}

**[builtin.go]======================================================================**

// Copyright 2011 The Go Authors. All rights reserved.

// Use of this source code is governed by a BSD-style

// license that can be found in the LICENSE file.

/*

Package builtin provides documentation for Go's predeclared
identifiers.
The items documented here are not actually in package builtin
but their descriptions here allow godoc to present documentation
for the language's special identifiers.

*/

package builtin

// bool is the set of boolean values, true and false.

type bool bool

// true and false are the two untyped boolean values.

const (

true = 0 == 0 // Untyped bool.
false = 0 != 0 // Untyped bool.

)

// uint8 is the set of all unsigned 8-bit integers.

// Range: 0 through 255.

type uint8 uint8

// uint16 is the set of all unsigned 16-bit integers.

// Range: 0 through 65535.

type uint16 uint16

// uint32 is the set of all unsigned 32-bit integers.

// Range: 0 through 4294967295.

type uint32 uint32

// uint64 is the set of all unsigned 64-bit integers.

// Range: 0 through 18446744073709551615.

type uint64 uint64

// int8 is the set of all signed 8-bit integers.

// Range: -128 through 127.

type int8 int8

// int16 is the set of all signed 16-bit integers.

// Range: -32768 through 32767.

type int16 int16

// int32 is the set of all signed 32-bit integers.

// Range: -2147483648 through 2147483647.

type int32 int32

// int64 is the set of all signed 64-bit integers.

// Range: -9223372036854775808 through 9223372036854775807.

type int64 int64

// float32 is the set of all IEEE-754 32-bit floating-point numbers.

type float32 float32

// float64 is the set of all IEEE-754 64-bit floating-point numbers.

type float64 float64

// complex64 is the set of all complex numbers with float32 real and

// imaginary parts.

type complex64 complex64

// complex128 is the set of all complex numbers with float64 real and

// imaginary parts.

type complex128 complex128

// string is the set of all strings of 8-bit bytes, conventionally but
not

// necessarily representing UTF-8-encoded text. A string may be empty,
but

// not nil. Values of string type are immutable.

type string string

// int is a signed integer type that is at least 32 bits in size. It is
a

// distinct type, however, and not an alias for, say, int32.

type int int

// uint is an unsigned integer type that is at least 32 bits in size. It
is a

// distinct type, however, and not an alias for, say, uint32.

type uint uint

// uintptr is an integer type that is large enough to hold the bit
pattern of

// any pointer.

type uintptr uintptr

// byte is an alias for uint8 and is equivalent to uint8 in all ways. It
is

// used, by convention, to distinguish byte values from 8-bit unsigned

// integer values.

type byte = uint8

// rune is an alias for int32 and is equivalent to int32 in all ways. It
is

// used, by convention, to distinguish character values from integer
values.

type rune = int32

// any is an alias for interface{} and is equivalent to interface{} in
all ways.

type any = interface{}

// comparable is an interface that is implemented by all comparable
types

// (booleans, numbers, strings, pointers, channels, arrays of comparable
types,

// structs whose fields are all comparable types).

// The comparable interface may only be used as a type parameter
constraint,

// not as the type of a variable.

type comparable interface{ comparable }

// iota is a predeclared identifier representing the untyped integer
ordinal

// number of the current const specification in a (usually
parenthesized)

// const declaration. It is zero-indexed.

const iota = 0 // Untyped int.

// nil is a predeclared identifier representing the zero value for a

// pointer, channel, func, interface, map, or slice type.

var nil Type // Type must be a pointer, channel, func, interface, map,
or slice type

// Type is here for the purposes of documentation only. It is a stand-in

// for any Go type, but represents the same type for any given function

// invocation.

type Type int

// Type1 is here for the purposes of documentation only. It is a
stand-in

// for any Go type, but represents the same type for any given function

// invocation.

type Type1 int

// IntegerType is here for the purposes of documentation only. It is a
stand-in

// for any integer type: int, uint, int8 etc.

type IntegerType int

// FloatType is here for the purposes of documentation only. It is a
stand-in

// for either float type: float32 or float64.

type FloatType float32

// ComplexType is here for the purposes of documentation only. It is a

// stand-in for either complex type: complex64 or complex128.

type ComplexType complex64

// The append built-in function appends elements to the end of a slice.
If

// it has sufficient capacity, the destination is resliced to
accommodate the

// new elements. If it does not, a new underlying array will be
allocated.

// Append returns the updated slice. It is therefore necessary to store
the

// result of append, often in the variable holding the slice itself:

//        slice = append(slice, elem1, elem2)

//        slice = append(slice, anotherSlice...)

// As a special case, it is legal to append a string to a byte slice,
like this:

//        slice = append([]byte("hello "), "world"...)

func append(slice []Type, elems ...Type) []Type

// The copy built-in function copies elements from a source slice into a

// destination slice. (As a special case, it also will copy bytes from a

// string to a slice of bytes.) The source and destination may overlap.
Copy

// returns the number of elements copied, which will be the minimum of

// len(src) and len(dst).

func copy(dst, src []Type) int

// The delete built-in function deletes the element with the specified
key

// (m[key]) from the map. If m is nil or there is no such element,
delete

// is a no-op.

func delete(m map[Type]Type1, key Type)

// The len built-in function returns the length of v, according to its
type:

//        Array: the number of elements in v.

//        Pointer to array: the number of elements in *v (even if v is
nil).

//        Slice, or map: the number of elements in v; if v is nil,
len(v) is zero.

//        String: the number of bytes in v.

//        Channel: the number of elements queued (unread) in the channel
buffer;

//         if v is nil, len(v) is zero.

// For some arguments, such as a string literal or a simple array
expression, the

// result can be a constant. See the Go language specification's
"Length and

// capacity" section for details.

func len(v Type) int

// The cap built-in function returns the capacity of v, according to its
type:

//        Array: the number of elements in v (same as len(v)).

//        Pointer to array: the number of elements in *v (same as
len(v)).

//        Slice: the maximum length the slice can reach when resliced;

//        if v is nil, cap(v) is zero.

//        Channel: the channel buffer capacity, in units of elements;

//        if v is nil, cap(v) is zero.

// For some arguments, such as a simple array expression, the result can
be a

// constant. See the Go language specification's "Length and
capacity" section for

// details.

func cap(v Type) int

// The make built-in function allocates and initializes an object of
type

// slice, map, or chan (only). Like new, the first argument is a type,
not a

// value. Unlike new, make's return type is the same as the type of its

// argument, not a pointer to it. The specification of the result
depends on

// the type:

//        Slice: The size specifies the length. The capacity of the
slice is

//        equal to its length. A second integer argument may be provided
to

//        specify a different capacity; it must be no smaller than the

//        length. For example, make([]int, 0, 10) allocates an
underlying array

//        of size 10 and returns a slice of length 0 and capacity 10
that is

//        backed by this underlying array.

//        Map: An empty map is allocated with enough space to hold the

//        specified number of elements. The size may be omitted, in
which case

//        a small starting size is allocated.

//        Channel: The channel's buffer is initialized with the
specified

//        buffer capacity. If zero, or the size is omitted, the channel
is

//        unbuffered.

func make(t Type, size ...IntegerType) Type

// The new built-in function allocates memory. The first argument is a
type,

// not a value, and the value returned is a pointer to a newly

// allocated zero value of that type.

func new(Type) *Type

// The complex built-in function constructs a complex value from two

// floating-point values. The real and imaginary parts must be of the
same

// size, either float32 or float64 (or assignable to them), and the
return

// value will be the corresponding complex type (complex64 for float32,

// complex128 for float64).

func complex(r, i FloatType) ComplexType

// The real built-in function returns the real part of the complex
number c.

// The return value will be floating point type corresponding to the
type of c.

func real(c ComplexType) FloatType

// The imag built-in function returns the imaginary part of the complex

// number c. The return value will be floating point type corresponding
to

// the type of c.

func imag(c ComplexType) FloatType

// The close built-in function closes a channel, which must be either

// bidirectional or send-only. It should be executed only by the sender,

// never the receiver, and has the effect of shutting down the channel
after

// the last sent value is received. After the last value has been
received

// from a closed channel c, any receive from c will succeed without

// blocking, returning the zero value for the channel element. The form

//        x, ok := <-c

// will also set ok to false for a closed channel.

func close(c chan<- Type)

// The panic built-in function stops normal execution of the current

// goroutine. When a function F calls panic, normal execution of F stops

// immediately. Any functions whose execution was deferred by F are run
in

// the usual way, and then F returns to its caller. To the caller G, the

// invocation of F then behaves like a call to panic, terminating G's

// execution and running any deferred functions. This continues until
all

// functions in the executing goroutine have stopped, in reverse order.
At

// that point, the program is terminated with a non-zero exit code. This

// termination sequence is called panicking and can be controlled by the

// built-in function recover.

func panic(v any)

// The recover built-in function allows a program to manage behavior of
a

// panicking goroutine. Executing a call to recover inside a deferred

// function (but not any function called by it) stops the panicking
sequence

// by restoring normal execution and retrieves the error value passed to
the

// call of panic. If recover is called outside the deferred function it
will

// not stop a panicking sequence. In this case, or when the goroutine is
not

// panicking, or if the argument supplied to panic was nil, recover
returns

// nil. Thus the return value from recover reports whether the goroutine
is

// panicking.

func recover() any

// The print built-in function formats its arguments in an

// implementation-specific way and writes the result to standard error.

// Print is useful for bootstrapping and debugging; it is not guaranteed

// to stay in the language.

func print(args ...Type)

// The println built-in function formats its arguments in an

// implementation-specific way and writes the result to standard error.

// Spaces are always added between arguments and a newline is appended.

// Println is useful for bootstrapping and debugging; it is not
guaranteed

// to stay in the language.

func println(args ...Type)

// The error built-in interface type is the conventional interface for

// representing an error condition, with the nil value representing no
error.

type error interface {

Error() string

}

---

# [go] call by value/pointer

**golang에서 pointer는 struct 인스턴스를 참조로 전달할 때를 제외하고는
사용할 일이 없다고 해도 무방하다.**

**paramter로 전달 할 때 use (call by value) syntax : primitive type**

**paramter로 전달 할 때 use (call by pointer) syntax : struct**

**paramter로 전달 할 때 use (call by value) syntax but acting as (call
by pointer) : string, array, slice, channel, map**

**string, array, slice, channel, map은 내부에 실제 데이터인 array에 대한
포인터를 가지는 구조의 자료형이기 때문에 call by value로 전달 하여도
call by pointer로 전달한 것과 같은 효과가 있다.**

golang에서 함수에 인자를 전달하는 방식은 기본적으로 call by value로
동작한다.

전달한 인자가 원본 인스턴스와는 다른 새 인스턴스로 복사되면서 전달
된다는 의미이다.

struct를 제외한 go의 다른 복합 자료형(string, array, slice, channel,
map)은 내부에 큰 데이터에 대해 포인터 형식을 사용함으로 함수에 인자로
전달할 때 call by value로 전달해도 된다.

인스턴스를 call by reference로 전달해야 하는 복합 자료형: struct

struct는 사용자 정의 자료형으로써 내부에 사용 메모리를 큰 자료형이 들어
있을 수 있다.

다시말하면 메모리 사용량이 큰 자료형에 대해 포인터로 처리하는 것 처럼
최소한의 메모리 사용 형태를 보장 할 수 없다.

그래서 struct의 인스턴스를 함수의 인자로 전달할 때는 되도록 포인터
형태로 전달(call by reference)한다.

인스턴스를 call by value로 전달해도 되는 복합 자료형: string, array,
slice, map

이들은 내부의 자료를 포인터 형태(uintptr 또는 unsafe.Pointer 등)로
가지고 있어 인자 전달 과정에서 복사 방식으로 전달 되어도 내부적으로
크기가 클 수 있는 데이터에 대해서는 포인터 형태로 전달(call by
reference) 함으로 전달 비용이 크지 않다.

slice struct)

<https://github.com/golang/go/blob/master/src/runtime/slice.go> line#33
#type slice struct

```
type slice struct {

array unsafe.Pointer

len int

cap int

}
```

string struct)

<https://github.com/golang/go/blob/master/src/runtime/string.go>
line#238 #type stringStruct struct

```
type stringStruct struct {

str unsafe.Pointer

len int

}
```

channel struct)

<https://github.com/golang/go/blob/master/src/runtime/chan.go> line#33
#type hchan struct

```
type hchan struct {

qcount uint // total data in the queue

dataqsiz uint // size of the circular queue

buf unsafe.Pointer // points to an array of dataqsiz elements

elemsize uint16

closed uint32

elemtype *_type // element type

sendx uint // send index

recvx uint // receive index

recvq waitq // list of recv waiters

sendq waitq // list of send waiters

// lock protects all fields in hchan, as well as several

// fields in sudogs blocked on this channel.

//

// Do not change another G's status while holding this lock

// (in particular, do not ready a G), as this can deadlock

// with stack shrinking.

lock mutex

}
```

map struct)

<https://github.com/golang/go/blob/master/src/runtime/map.go> line#115
#type hmap struct

```
// A header for a Go map.

type hmap struct {

// Note: the format of the hmap is also encoded in
cmd/compile/internal/reflectdata/reflect.go.

// Make sure this stays in sync with the compiler's definition.

count int // # live cells == size of map. Must be first (used by
len() builtin)

flags uint8

B uint8 // log_2 of # of buckets (can hold up to loadFactor *
2\^B items)

noverflow uint16 // approximate number of overflow buckets; see
incrnoverflow for details

hash0 uint32 // hash seed

**buckets unsafe.Pointer // array of 2\^B Buckets. may be nil if
count==0.**

**oldbuckets unsafe.Pointer // previous bucket array of half the
size, non-nil only when growing**

**nevacuate uintptr // progress counter for evacuation (buckets
less than this have been evacuated)**

extra *mapextra // optional fields

}
```

---

# [go][pointer] asterisk ampersand

[포인터 타입 선언시 패키지명 혼합]

*<package>.MyStructType는 *(<package>.MyStructType)과 같다.

"*ecr.OutputData" 표현은 ecr 패키지에 선언된 OutputData Struct의
포인터 타입임을 의미한다.

```go
package ecr
...

type OutputData struct {

...

}
```

* *

[*golang: Asterisk and Ampersand
Cheatsheet*](https://gist.githubusercontent.com/josephspurrier/7686b139f29601c3b370/raw/ba7944fadd441557b2af7e8fa387b96c26b8e106/values_pointers.go)

/*\
********************************************************************************\
Golang - Asterisk and Ampersand Cheatsheet\
********************************************************************************

Also available at: <https://play.golang.org/p/lNpnS9j1ma>

Allowed:\
--------\
p := Person{"Steve", 28}         stores the value\
p := &Person{"Steve", 28}         stores the pointer address
(reference)\
PrintPerson(p)                         passes either the value or
pointer address (reference)\
PrintPerson(*p)                 passes the value\
PrintPerson(&p)                 passes the pointer address (reference)\
func PrintPerson(p Person)        ONLY receives the value\
func PrintPerson(p *Person)        ONLY receives the pointer address
(reference)

Not Allowed:\
--------\
p := *Person{"Steve", 28}         illegal\
func PrintPerson(p &Person)        illegal

*/

package main

import (\
        "fmt"\
)

type Person struct {\
        Name string\
        Age int\
}

// This only works with *Person, does not work with Person\
// Only works with Test 2 and Test 3\
func (p *Person) String() string {\
        return fmt.Sprintf("%s is %d", p.Name, p.Age)\
}

// This works with both *Person and Person, BUT you can't modiy the
value and\
// it takes up more space\
// Works with Test 1, Test 2, Test 3, and Test 4\
/*func (p Person) String() string {\
        return fmt.Sprintf("%s is %d", p.Name, p.Age)\
}*/

//
*****************************************************************************\
// Test 1 - Pass by Value\
//
*****************************************************************************

func test1() {\
        p := Person{"Steve", 28}\
        printPerson1(p)\
        updatePerson1(p)\
        printPerson1(p)\
}

func updatePerson1(p Person) {\
        p.Age = 32\
        printPerson1(p)\
}

func printPerson1(p Person) {\
        fmt.Printf("String: %v | Name: %v | Age: %d\n",\
p,\
p.Name,\
p.Age)\
}

//
*****************************************************************************\
// Test 2 - Pass by Reference\
//
*****************************************************************************

func test2() {\
        p := &Person{"Steve", 28}\
        printPerson2(p)\
        updatePerson2(p)\
        printPerson2(p)\
}

func updatePerson2(p *Person) {\
        p.Age = 32\
        printPerson2(p)\
}

func printPerson2(p *Person) {\
        fmt.Printf("String: %v | Name: %v | Age: %d\n",\
p,\
p.Name,\
p.Age)\
}

//
*****************************************************************************\
// Test 3 - Pass by Reference (requires more typing)\
//
*****************************************************************************

func test3() {\
        p := Person{"Steve", 28}\
        printPerson3(&p)\
        updatePerson3(&p)\
        printPerson3(&p)\
}

func updatePerson3(p *Person) {\
        p.Age = 32\
        printPerson3(p)\
}

func printPerson3(p *Person) {\
        fmt.Printf("String: %v | Name: %v | Age: %d\n",\
p,\
p.Name,\
p.Age)\
}

//
*****************************************************************************\
// Test 4 - Pass by Value (requires more typing)\
//
*****************************************************************************

func test4() {\
        p := &Person{"Steve", 28}\
        printPerson4(*p)\
        updatePerson4(*p)\
        printPerson4(*p)\
}

func updatePerson4(p Person) {\
        p.Age = 32\
        printPerson4(p)\
}

func printPerson4(p Person) {\
        fmt.Printf("String: %v | Name: %v | Age: %d\n",\
p,\
p.Name,\
p.Age)\
}

//
*****************************************************************************\
// Main\
//
*****************************************************************************

/*\
Outputs:\
String: {Steve 28} | Name: Steve | Age: 28\
String: {Steve 32} | Name: Steve | Age: 32\
String: {Steve 28} | Name: Steve | Age: 28\
String: Steve is 28 | Name: Steve | Age: 28\
String: Steve is 32 | Name: Steve | Age: 32\
String: Steve is 32 | Name: Steve | Age: 32\
String: Steve is 28 | Name: Steve | Age: 28\
String: Steve is 32 | Name: Steve | Age: 32\
String: Steve is 32 | Name: Steve | Age: 32\
String: {Steve 28} | Name: Steve | Age: 28\
String: {Steve 32} | Name: Steve | Age: 32\
String: {Steve 28} | Name: Steve | Age: 28\
*/\
func main() {        \
        test1()\
        test2()\
        test3()\
        test4()\
}

---

# [go] type 변환 strconv / unsafe package

**<> Array to Slice**

# using [:] syntax

```go
func TestArrayToSlice(t *testing.T) {

arr := [10]byte{1, 2, 3, 4, 5, 6}

slice := arr[:]

t.Log(slice) // [1 2 3 4 5 6 0 0 0 0]

}
```

# using unsafe.Slice

[func Slice(ptr *ArbitraryType, len IntegerType)
[]ArbitraryType](https://pkg.go.dev/unsafe#Slice)

```go
func TestArrayToSlice(t *testing.T) {

arr := [...]int{1, 2, 3, 4, 5, 6}

sss := unsafe.Slice(&arr[0], 20)

// equivalent to sss :=
(*[20]int)(unsafe.Pointer(&arr[0]))[:]

// 0xc000164030

t.Logf("%p", &arr)

// 0xc000164030

t.Lo
g(unsafe.Pointer((*reflect.SliceHeader)(unsafe.Pointer(&sss)).Data))

t.Log(reflect.TypeOf(arr)) // [3]int

t.Log(reflect.TypeOf(sss)) // []int

t.Log(cap(sss)) // 20

t.Log(len(sss)) // 20

}
```

# [func Itoa(i int) string](https://pkg.go.dev/strconv#Itoa)

Itoa is equivalent to FormatInt(int64(i), 10).

```go
func main() {

i := 10

s := strconv.Itoa(i)

fmt.Printf("%T, %v\n", s, s) // string, 10

}
```

# FormatXXX functions

func FormatBool(b bool) string

func FormatComplex(c complex128, fmt byte, prec, bitSize int) string

func FormatFloat(f float64, fmt byte, prec, bitSize int) string

func FormatInt(i int64, base int) string

func FormatUint(i uint64, base int) string

# uint to string

[func FormatUint(i uint64, base int)
string](https://pkg.go.dev/strconv#FormatUint)

FormatUint returns the string representation of i in the given base, for
2 <= base <= 36. The result uses the lower-case letters 'a' to 'z'
for digit values >= 10.

```go
func main() {

v := uint64(42)

s10 := strconv.FormatUint(v, 10)

fmt.Printf("%T, %v\n", s10, s10) // string, 42

s16 := strconv.FormatUint(v, 16)

fmt.Printf("%T, %v\n", s16, s16) // string, 2a

}
```

**<> string to integer**

[func Atoi(s string) (int, error)](https://pkg.go.dev/strconv#Atoi)

Atoi is equivalent to ParseInt(s, 10, 0), converted to type int.

```go
func main() {

v := "10"

if s, err := strconv.Atoi(v); err == nil {

fmt.Printf("%T, %v", s, s) // int, 10

}

}
```

# ParseXXX functions

func ParseBool(str string) (bool, error)

func ParseComplex(s string, bitSize int) (complex128, error)

func ParseFloat(s string, bitSize int) (float64, error)

func ParseInt(s string, base int, bitSize int) (i int64, err error)

func ParseUint(s string, base int, bitSize int) (uint64, error)

# string to int

[func ParseInt(s string, base int, bitSize int) (i int64, err
error)](https://pkg.go.dev/strconv#ParseInt)

ParseInt interprets a string s in the given base (0, 2 to 36) and bit
size (0 to 64) and returns the corresponding value i.

```go
func main() {

v32 := "1234"

// decimal(10)

if s, err := strconv.ParseInt(v32, 10, 32); err == nil {

fmt.Printf("%T, %v\n", s, s) // 1234

}

// hexa decimal(16)

if s, err := strconv.ParseInt(v32, 16, 32); err == nil {

fmt.Printf("%T, %v\n", s, s) // 4660

}

v64 := "1234"

// decimal(10)

if s, err := strconv.ParseInt(v64, 10, 64); err == nil {

fmt.Printf("%T, %v\n", s, s) // 1234

}

// hexa decimal(16)

if s, err := strconv.ParseInt(v64, 16, 64); err == nil {

fmt.Printf("%T, %v\n", s, s) // 4660

}

}
```

---

# [go] type casting vs type assert

**<> Type Casting:**

변수의 타입을 명시적으로 변환할 때 사용

Syntax of Type Conversions

  -----------------------------------------------------------------------
  newDataTypeVariable = T(oldDataTypeVariable)
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

```
var badboys int = 1921

// explicit type conversion

var badboys2 float64 = float64(badboys)

var badboys3 int64 = int64(badboys)

var badboys4 uint = uint(badboys)
```

**<> Type Assertion**

inteface{}(empty interface) type 변수의 value의 실제 Type을 지정하는 것

Syntax:

interface type 변수 x

type T

```
var x interface{}

var t T

x = t
value, ok := x.(T)
```

주의 - interface type이 가진 값의 실제 타입과 다른 타입으로 Assertion
하는 경우 panic Runtime ERROR 발생

**<> "interface{}" empty interface type**

empty interface = 인터페이스 타입 = nil을 제외한 golang의 모든 타입을
담을 수 있는 컨테이너, Dynamic Type(주: java의 object, c/c++의 void*)

int, string 등 primitive 타입과 포인터와 슬라이스 까지 모든 형식을
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

---

# [go] type switch 와 reflect

**<> golang 에서 변수의 타입 체크 방법들:**

1, fmt.Printf("var = %T\n", var)

2, reflect.TypeOf(var).Kind()

3, reflect.ValueOf(var).Kind()

4, swith v := var.(type) // var SHOULD be interface{}

```go
package main

// import the fmt and reflect package

import (

"fmt"

"reflect"

)

//main function

func main() {

varString := "varString"

fmt.Printf("%T\n", varString) // string

if reflect.TypeOf(varString).Kind() == reflect.String {

fmt.Println("Equal")                                   //
Equal

fmt.Println(reflect.TypeOf(varString).Name())          //
string

fmt.Println(reflect.TypeOf(varString).String())        //
string

fmt.Println(reflect.TypeOf(varString).Kind().String()) //
string

}

if reflect.ValueOf(varString).Kind() == reflect.String {

fmt.Println("Equal") // Equal

}

var itfs interface{}

itfs = varString

switch t := itfs.(type) {

case string:

fmt.Println("string") // string

default:

fmt.Printf("type unknown %T\n", t)

}

}
```

"type switch"는 interface{} (empty interface) 타입의 변수의 실제
타입을 체크하는데 사용된다.

특히 interface{} 타입의 변수가 array([]interface{})인지 체크할 때
유용하다.

".(type)" expression은 switch 구문에서만 사용할 수 있다.

interface{}는 nil을 제외하고 method를 갖지 않는 golang의 모든 타입에
대응된다.

심지어 []interface{} (array interface{}) 도 interface{}로 체크된다.

그래서 case문에 interface{} 를 명시하면 nil을 제외한 모든 타입이 걸리게
된다.

```go
package main

import (

"fmt"

)

func typeSwitchTest(i interface{}) {

switch v := i.(type) {

case int:

fmt.Println("x is", v)

case bool, string:

fmt.Println("x is bool or string")

case nil:

fmt.Println("x is nil")

case []interface{}:

fmt.Println("x is array of interface{}")

// case interface{}:

//  fmt.Println("x is a interface{}")

default:

fmt.Printf("type unknown %T\n", v)

}

}

func main() {

typeSwitchTest("value") //x is bool or string

typeSwitchTest(23) //x is 23

typeSwitchTest(true) //x is bool or string

typeSwitchTest(nil) //x is nil

typeSwitchTest([]int{}) //type unknown []int

typeSwitchTest([]interface{}{}) //x is array of interface{}

}
```

다른 타입 체크 방법들:

1, reflect.TypeOf(var)

2, reflect.ValueOf(var).Kind()

3, fmt.Printf("var = %T\n", var)

```go
// Golang program to show the different ways

// to find the Type of a Variable

package main

// import the fmt and reflect package

import (

"fmt"

"reflect"

)

//main function

func main() {

// string type

var1 := "hello world"

// integer

var2 := 10

// float

var3 := 1.55

// boolean

var4 := true

// shorthand string array declaration

var5 := []string{"foo", "bar", "baz"}

// map is reference datatype

var6 := map[int]string{100: "Ana", 101: "Lisa", 102:
"Rob"}

// complex64 and complex128

// is basic datatype

var7 := complex(9, 15)

// using %T format specifier to

// determine the datatype of the variables

fmt.Println("Using Percent T with Printf")

fmt.Println()

fmt.Printf("var1 = %T\n", var1) // var1 = string

fmt.Printf("var2 = %T\n", var2) // var2 = int

fmt.Printf("var3 = %T\n", var3) // var3 = float64

fmt.Printf("var4 = %T\n", var4) // var4 = bool8

fmt.Printf("var5 = %T\n", var5) // var5 = []string

fmt.Printf("var6 = %T\n", var6) // var6 = map[int]string

fmt.Printf("var7 = %T\n", var7) // var7 = complex128

// using TypeOf() method of reflect package

// to determine the datatype of the variables

fmt.Println()

fmt.Println("Using reflect.TypeOf Function")

fmt.Println()

fmt.Println("var1 = ", reflect.TypeOf(var1)) // var1 =  string

fmt.Println("var2 = ", reflect.TypeOf(var2)) // var2 =  int

fmt.Println("var3 = ", reflect.TypeOf(var3)) // var3 =  float64

fmt.Println("var4 = ", reflect.TypeOf(var4)) // var4 =  bool

fmt.Println("var5 = ", reflect.TypeOf(var5)) // var5 =
[]string

fmt.Println("var6 = ", reflect.TypeOf(var6)) // var6 =
map[int]string

fmt.Println("var7 = ", reflect.TypeOf(var7)) // var7 =
complex128

// using ValueOf() method of reflect package

// to determine the value of the variable

// Kind() method returns the datatype of the

// value fetched by the ValueOf() method

fmt.Println()

fmt.Println("Using reflect.ValueOf.Kind() Function")

fmt.Println()

fmt.Println("var1 = ", reflect.ValueOf(var1).Kind()) // var1 =
string

fmt.Println("var2 = ", reflect.ValueOf(var2).Kind()) // var2 =
int

fmt.Println("var3 = ", reflect.ValueOf(var3).Kind()) // var3 =
float64

fmt.Println("var4 = ", reflect.ValueOf(var4).Kind()) // var4 =
bool

fmt.Println("var5 = ", reflect.ValueOf(var5).Kind()) // var5 =
slice

fmt.Println("var6 = ", reflect.ValueOf(var6).Kind()) // var6 =
map

fmt.Println("var7 = ", reflect.ValueOf(var7).Kind()) // var7 =
complex128

varString := "varString"

if reflect.TypeOf(varString).Kind() == reflect.String {

fmt.Println("Equal")                                   //
Equal

fmt.Println(reflect.TypeOf(varString).Name())
//string

fmt.Println(reflect.TypeOf(varString).String())
//string

fmt.Println(reflect.TypeOf(varString).Kind().String())
//string

}

if reflect.ValueOf(varString).Kind() == reflect.String {

fmt.Println("Equal") // Equal

}

}
```

---

# [go] unsafe package와 강제 형변환

**<> unsafe package**

**강제 형변환과 포인터 연산을 위한 type converting을 지원하는 패키지**

uintptr을 이용해 pointer 연산을 할 때 unsafe 패키지 함수나 Pointer 타입
캐스팅으로 부터 반환된 uintptr 값이 아닌 상수 리터럴을 가지고 +/- 연산을
하면 서로 다른 시스템 architecture(ex. x86 <-> x86-64)간에 이식을
보장할 수 없게 된다.

reflect 패키지와 함께 사용하면 활용도가 좋다.

**unsafe package는 강제 형변환과 주소를 베이스로 하는 메모리 접근을 위한
함수와 타입을 지원하며 주요 함수는 아래와 같다.**

**<> unsafe package:: 강제 형변환을 위한 타입과 함수**

type unsafe.Pointer

: 어떤 타입 변수의 메모리 주소(*ArbitraryType)를 나타내는 타입

이 타입은 포인터 연산을 위한 uintptr 타입으로 형변환 가능한다.

다음 함수들은 unsafe.Pointer로 얻은 변수의 주소값에 대해 포인터 연산을
위한 uintptr 타입 데이터의 항(term)을 획득하기 위한 함수 들이다.

type uintptr

: 10진수로 pointer 연산을 하기 위한 부호없는 정수 타입 unsafe.Pointer
타입과만 상호 형변환 될 수 있다.

**func Slice(ptr *ArbitraryType, len IntegerType) []ArbitraryType**

: array를 slice로 변환하는데 사용

ptr - arrary의 포인터

len - 새로 생성될 slice의 길이

**<> unsafe package:: 포인터 연산을 위한 함수들**

다음 세 함수는 uintptr 타입으로 byte 단위의 수(number)를 반환하는 함수
들이다.

func unsafe.Offsetof(x) uintptr

: 어떤 구조체 필드의 메모리 상에서 offset을 반환

offset이란 구조체 시작 주소(==첫번째 field의 시작 주소)로 부터 지정된
필드의 시작 까지의 거리(byte 단위)를 나타낸다. alignment에 의한
padding도 모두 포함하여 계산된다.

func unsafe.Sizeof(x) uintptr

: 주어진 변수 x의 메모리 상에서 실제로 차지하는 size를 반환

x가 일반 타입인 경우, 해당 타입의 size를 반환됨

x가 구조체인 경우, alignment에 의한 padding을 모두 포함한 size가 반환됨

x가 배열인 경우, "element의 타입 size x element의 수" 값이 반환됨\

func unsafe.Alignof(x) uintptr

:

x가 일반 타입인 경우, 해당 타입의 size를 반환됨

x가 구조체인 경우, alignment를 위한 pack size가 반환됨

x가 배열인 경우, element의 타입의 size가 반환됨\

**<> unsafe package types**

*Type <-> unsafe.Pointer(== *ArbitraryType) <-> uintptr (포인터
연산용 부호없는 10진수 정수)

type ArbitraryType int :

임의의 타입, any(interface{})가 포인터를 포함하는 임의의 타입이라면
ArbitraryType은 포인터를 제외한 임의의 타입을 의미

type Pointer *ArbitraryType :

pointer of arbitrary type

c/c++에서 void* 와 같은 의미이며 Log()나 Print() 문으로 출력하면 16진수
memory address expr로 출력된다.

어떤 변수의 포인터를 uintptr로 convert하기 위해서는 Pointer로 먼저
변환해야 한다.

unsafe.Pointer를 이용해서 pointer 연산은 불가능 하다.

주 용도는 다른 타입의 포인터 변수에 인스턴스를 강제 할당하기 위한
*ArbitraryType 타입 캐스팅(unsafe)과 포인터 연산을 위해 포인터를
uintptr로 캐스팅 하는 것이다.

unsafe.Pointer :

16진수 형태의 memory address를 담는 type, 사칙 연산 불가, 임의 변수의
pointer를 uintptr로 변환하기 위해서 사용

type uintptr uintptr :

포인터 연산을 위한 부호없는 정수 타입.

기본적으로 uint와 같으며 어떠한 부호없는 정수 값도 담을 수 있다.

Log()나 Print() 문으로 출력하면 10진수 정수 expr로 출력된다.

uint와 달리 uintptr은 unsafe.Pointer와 상호 타입 캐스팅이 가능하다.

uintptr의 용도는 pointer 연산(ex. pointer+offset)이며 10진수 4칙 연산을
통해 임의의 주소값을 생성하고 이를 다시 unsafe.Pointer 타입으로 변환할
수 있다.

Golang은 type에 엄격하여 서로 다른 type간에 사칙연산이 불가능
하므로(uintptr과 uint도 상호 사칙연산 불가) Golang에서 Pointer 연산을
하려면 uintptr을 사용하는 방법 밖에 없다.

uintptr을 이용해 pointer 연산을 할 때 unsafe 패키지 함수나 Pointer 타입
캐스팅으로 부터 반환된 uintptr 값이 아닌 상수 리터럴을 가지고 +/- 연산을
하면 서로 다른 시스템 architecture(ex. x86 <-> x86-64)간에 이식을
보장할 수 없게 된다.

var x uintptr

x = 8

t.Log(x) // 8

t.Log(unsafe.Pointer(x)) // 0x8

**<> 인스턴스로 부터 uintptr 형식의 값을 구하는 방법**

================================

var a int`

var u uintptr

u = unsafe.Pointer(&a)

u = reflect.ValueOf(a).Pointer()

u = reflect.ValueOf(new(int)).Pointer()

**<> Unsafe package :: 형변환을 위한 자료형과 함수**

**<> unsafe.Pointer와 uintptr 형변환**

```go
func TestUnsafePointer(t *testing.T) {

var a int64

var strt STRUCT

var uintptrT1 uintptr

// uintptrT1 = uintptr(&a)

// INVALID: cannot convert &a (value of type *int) to uintptr

uintptrT1 = uintptr(unsafe.Pointer(&a)) // OK

t.Logf("%p", &a) // 0xc0000182f8

t.Log(unsafe.Pointer(&a)) // 0xc0000182f8

t.Log(uintptrT1) // 824633819896

t.Log(reflect.TypeOf(unsafe.Pointer(&a)).Kind())

// unsafe.Pointer, HEX expr of ponter address

t.Log(reflect.TypeOf(uintptr(unsafe.Pointer(&a))).Kind())

// uintptr, DEC(uint) expr of ponter address

t.Log(unsafe.Pointer(&a))

// 0xc0000182d8, HEX expr of ponter address of a

t.Log(uintptr(unsafe.Pointer(&a)))

// 824633819864, DEC(uint) expr of ponter address of a

t.Log(unsafe.Pointer(&strt))

// 0xc0000182e0, address(uintptr(HEX)) of strt, WARINING: &strt ==
&strt.i

t.Log(uintptr(unsafe.Pointer(&strt)))

// 824633819872, address(uintptr(DEC)) of strt

// !!! unsafe.Pointer(&strt.i) == unsafe.Pointer(&strt)

t.Log(unsafe.Pointer(&strt.i))

// 0xc0000182e0, address(unsafe.Pointer(HEX)) of strt.i

t.Log(uintptr(unsafe.Pointer(&strt)) + unsafe.Offsetof(strt.i))

// 824633819872, address(uintptr(DEC)) of strt.i

t.Log(unsafe.Offsetof(strt.i))

// 0

}
```

**<> unsafe.Pointer 타입을 이용한 강제 형변환**

```go
func TestUnsafePointerCasting(t *testing.T) {

var a int32

var b *uint32

b = (*uint32)(unsafe.Pointer(&a))

t.Log(reflect.TypeOf(&a)) // *int32

t.Log(reflect.TypeOf(b)) // *uint32

t.Logf("%p", &a) //0xc000018368

t.Logf("%p", b) //0xc000018368

}
```

**<> unsafe.pointer 강제 형변환을 이용한 구조체 필드 접근 제한
무시하기 (decapsulation)**

일반적으로 구조체의 필드 변수 중 소문자로 시작하는 필드에 대해서는
외부에서 접근이 불가능 하다.

하지만 unsafe.pointer를 이용하여 접근이 가능한 형태의 구조체 타입으로
강제 형변환 하면 기존에 접근이 불가능하던 구조체 인스턴스의 필드에
접근할 수 있다.

decapsulation 방법

step 1, 외부 접근이 가능하도록 대문자로 시작하는 field 네임을 갖지만
캡슐화된 원본 구조체와 그 필드의 순서와 타입이 모두 갖은 구조체 타입
선언

step 2, 캡슐화된 원본 구조체 타입의 인스턴스의 포인터를 unsafe.Pointer를
통해 decapsulation용 타입으로 형변환 한다.

golang에서 공식 지원하는 decapsulation용 타입이 relect.SliceHeader와
reflect.StringHeader이다.

예) reflect.SliceHeader 사용예

```go
func TestDecapsulationByForceCasting(t *testing.T) {

// convert slice pointer to reflect.SliceHeader pointer

s := make([]int, 10)

s[0] = 9

s[1] = 8

s[2] = 7

t.Logf("s.type: %v", reflect.TypeOf(s))

t.Logf("s.pointer: %p", &s)

t.Logf("s[0].pointer: %p", &s[0])

t.Logf("s.array: %v", s) // [9 8 7 0 0 0 0 0 0 0]

t.Logf("len(s): %v", len(s))

t.Logf("cap(s): %v", cap(s))

srefhead := (*reflect.SliceHeader)(unsafe.Pointer(&s))

t.Logf("srefhead.pointer: %p", srefhead) // same as s.pointer

t.Logf("srefhead.Data: %p", unsafe.Pointer(srefhead.Data)) //
same as s[0].pointer and s.data

arr := (*[10]int)(unsafe.Pointer(srefhead.Data))

t.Logf("srefhead.array: %v", *arr) // array contents in slice,
[9 8 7 0 0 0 0 0 0 0]

t.Logf("srefhead.Len: %v", srefhead.Len) // same as len(s)

t.Logf("srefhead.Cap: %v", srefhead.Cap) // same as cao(s)

}
```

**<> func Slice(ptr *ArbitraryType, len IntegerType)
[]ArbitraryType**

array를 slice로 변환하는데 사용한다.

unsafe.Slice를 새로운 slice 인스턴스를 생성하고 그 pointer를 반환한다.

새 slice instance의 data 필드에는 파라미터로 주어진 array 포인터를
기준으로 새로운 길이에 맞추어 길이를 연장하고 연장된 공간에 zero 초기화
한 후 할당해 준다.

그러므로 첫번째 파라미터인 ptr에 반드시 array의 pointer를 지정 하여야만
한다.

다른 타입의 포인터(예를 들면 array 가 아닌 int pointer)를 지정할 경우
새로 생성되는 slice의 data 필드에는 array가 아니며 array로써 동작하지
않는 메모리 영역이 단순히 지정될 뿐이다. 이렇게 잘못 생성된 slice에는
정상적으로 값을 쓸 수 가 없다.

```go
func TestUnsafeSlice(t *testing.T) {

arr := [...]int{9, 8}

ss := unsafe.Slice(&arr[0], 10) // equivalent to sss :=
(*[10]int)(unsafe.Pointer(&arr[0]))[:]

ss[2] = 1

ss[3] = 2

ss = append(ss, 3, 4, 5) // ss: [9 8 1 2 0 0 0 0 0 0 3 4 5]

t.Logf("arr.type: %v", (reflect.TypeOf(arr))) // [2]int

t.Logf("ss.type: %v", reflect.TypeOf(ss))

t.Logf("ss: %v", ss)

//        t.Logf("ss[1]: %v", ss[1])

t.Logf("ss.pointer: %p", &ss)

t.Logf("ss[0].pointer: %p", &ss[0])

t.Logf("len(ss): %v", len(ss))

t.Logf("cap(ss): %v", cap(ss))

ssrefhead := (*reflect.SliceHeader)(unsafe.Pointer(&ss))

t.Logf("ssrefhead.pointer: %p", ssrefhead) // same as ss.pointer

t.Logf("ssrefhead.Data: %p", unsafe.Pointer(ssrefhead.Data)) //
same as ss[0].pointer

t.Logf("ssrefhead.Len: %v", ssrefhead.Len)

t.Logf("ssrefhead.Cap: %v", ssrefhead.Cap)

}
```

**<> Unsafe package :: 형변환을 위한 함수**

**<> func Offsetof(x ArbitraryType) uintptr**

임의 타입의 구조체 멤버 변수를 paramter로 받아서 구조체의 시작 주소 부터
해당 멤버까지의 byte offset 값을 uinptr 형식으로 반환한다.

구조체 멤버 변수의 인스턴스만 인수로 사용이 가능하며 구조체 멤버가 아닌
인스턴스는 인수로 지정이 불가능하다.

```go
type STRUCT struct {

i int64

j rune

}

func TestUnsafeOffsetof(t *testing.T) {

var strt STRUCT

t.Log(unsafe.Pointer(&strt))

// 0xc0000182e0, address(uintptr(HEX)) of strt, WARINING: &strt ==
&strt.i

t.Log(uintptr(unsafe.Pointer(&strt)))

// 824633819872, address(uintptr(DEC)) of strt

// !!! unsafe.Pointer(&strt.i) == unsafe.Pointer(&strt)

t.Log(unsafe.Pointer(&strt.i))

// 0xc0000182e0, address(unsafe.Pointer(HEX)) of strt.i

t.Log(uintptr(unsafe.Pointer(&strt)) + unsafe.Offsetof(strt.i))

// 824633819872, address(uintptr(DEC)) of strt.i

t.Log(unsafe.Offsetof(strt.i))

// 0

t.Log(unsafe.Pointer(&strt.j))

// 0xc0000182e8, address(unsafe.Pointer(HEX)) of strt.j

t.Log(uintptr(unsafe.Pointer(&strt)) + unsafe.Offsetof(strt.j))

// 824633819880, address(uintptr(DEC)) of strt.j

t.Log(unsafe.Offsetof(strt.j))

// 8 == distance between start of strt and start of strt.j == size
of strt.i

u := uintptr(unsafe.Pointer(&strt))

offset := unsafe.Offsetof(strt.j)

t.Log(u + offset) // 0xc0000182e8

}
```

**<> func Sizeof(x ArbitraryType) uintptr**

임의 타입의 변수를 받아서 그 타입의 바이트 사이즈(uintptr)를 반환

```go
type STRUCT struct {

i int64

j rune

}

func TestUnsafeSizeof(t *testing.T) {

var a int64

var strt STRUCT

// func Sizeof(x ArbitraryType) uintptr

// return size in bytes of type of variable x

t.Log(unsafe.Sizeof(a)) // 8

t.Log(unsafe.Sizeof(strt)) // 16

t.Log(unsafe.Sizeof(strt.i)) // 8

t.Log(unsafe.Sizeof(strt.j)) // 4

}
```

**<> func Alignof(x ArbitraryType) uintptr**

임의 타입의 변수를 받아서 그 타입의 Alignment (byte 단위)를 반환

x가 일반 타입인 경우, 해당 타입의 size를 반환됨

x가 구조체인 경우, alignment를 위한 pack size가 반환됨

x가 배열인 경우, element의 타입의 size가 반환됨

Alignment?

어떤 타입의 인스턴스를 메모리 생성할 때 실제로 메모리에서 사용할 위치와
공간을 고려하여 배치하는 것

또는

Alignment 과정에 필요한 되는 특정 size

특히, 구조체 타입의 경우 메모리에서 차지하는 공간은 복잡한 연산이
필요하다.(구조체 할당 메모리 크기 != 멤버 타입 크기의 합)

구조체 alignment에서 각 필드의 순서는 메모리상에서도 지켜지지만,

각 필드를 cpu가 최선의 횟수로 access 할 수 있도록 하는 형태의 배치가
필요하고 이를 구조체 필드 Alignment라고 한다.

구조체의 각 필드는 구조체 메모리 구역 내에서 pack 이라는 boundary 안에
포함되게 되며, 이 size of pack x number of pack이 실제로 구조체가
메모리에서 사용하는 크기가 된다.

struct field alignment 과정에서 구조체의 모든 필드를 pack boundary 안에
위치 시키기 위하여 padding이라는 빈공간이 사용된다.

**<> func Add(ptr Pointer, len IntegerType) Pointer**

unsafe.Pointer와 integer type 이나 untyped constant를 합연산 하는 함수.

이 함수 사용시 시스템 architecture(ex. x86 <-> x86-64)간에 이식을
보장할 수 없게 된다.

---

# [go] struct memory alignment

[**[cpp][struct][alignment][pack] 구조체 멤버
정렬하기**](onenote:12_1_CPP.one#%5bcpp%5d%5bstruct%5d%5balignment%5d%5bpack%5d%20구조체%20멤버&section-id={41D0A903-D9A6-47BC-888F-5BE01BF8A0EB}&page-id={B115E468-3CD5-480B-9704-21C3601B6DA6}&object-id={FB82C889-6932-4D86-A05B-3B0252220B51}&59&base-path=https://d.docs.live.net/34ee6020a41b9bff/Documents/onte_study)

<https://go.dev/ref/spec#Size_and_alignment_guarantees>

**golang에서는 struct alignment pack size에 대한 강제 변경을 지원하지
않는다.**

c/c++에서는 컴파일러 지시지를 통해 개발자가 pack size를 강제로 1로
변경할 수 있지만 golang에서는 이를 지원하지 않는다.

**구조체 alignment에 대해 최적화가 필요하다면 코딩할 때 직접 순서를
정렬하는 것을 고려해야한다.**

a, 사이즈가 큰 멤버를 작은 멤버 사이에 끼우면 padding으로 인해 구조체
인스턴스의 메모리 할당량이 늘어난다.

b, 큰 순서에서 작은 순서로 정렬하는 방법이 대부분의 상황에서 최적의
용량이 될 것 같다.

**컴파일러가 계산하는 각 구조체의 pack size를 알고 싶다면 unsafe
패키지의 unsafe.Alignof(x <struct type>) 함수를 사용해본다.**

---

# [go] ellipsis(...) usage

<https://yourbasic.org/golang/three-dots-ellipsis/>

**Variadic function parameters**

If the last parameter of a function has type ...T, it can be called
with any number of trailing arguments of type T. The actual type of
...T inside the function is []T.

This example function can be called with, for instance, Sum(1, 2, 3) or
Sum().

```go
**// ellipsis(...)로 전달받은 parameter 변수 nums는 slice 타입이
된다.**
func Sum(nums ...int) int {

res := 0

for _, n := range nums {

res += n

}

return res

}
```

**Arguments to variadic functions (**Unpacking slice as arguments**)**

You can pass a slice s directly to a variadic function if you unpack it
with the s... notation. In this case no new slice is created.

In this example, we pass a slice to the Sum function.

```go
primes := []int{2, 3, 5, 7}

fmt.Println(Sum(primes...)) // 17
```

**Array literals**

In an array literal, the ... notation specifies a length equal to the
number of elements in the literal.

  -----------------------------------------------------------------------
  stooges := [**...**]string{"Moe", "Larry", "Curly"} //
  len(stooges) == 3
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

<https://go.dev/ref/spec#Composite_literals>

```
x := [...]int{ 1:1, 2:2 }

x := []int{ 1:1, 2:2 }
```

**The go command**

Three dots are used by the go command as a wildcard when describing
package lists.

This command tests all packages in the current directory and its
subdirectories.

  -----------------------------------------------------------------------
  $ go test ./**...**
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

---

# [go] *composit literal

**<> composit literal**

<https://go.dev/ref/spec#Composite_literals>

composite type(array, slice, map, struct)에 대해서 정의(선언)과 동시에
인스턴스화 하는것.

따로 type 키워드를 이용한 type정의를 하지 않고도 literal을 생성할 수
있다.

일반적인 struct 사용법 : type 키워드를 이용해 typename을 정의후
typename을 이용하여 객체를 생성

```go
type CAR struct{

Speed int

Weight float

}

func TestCAR (t *testing.T) {

car := CAR {Speed:6, Weight:7.1}

}
```

struct literal : typename 정의 없이 struct 타입 정의와 동시에 객체 생성
가능

```
car := struct{

Speed int

Weight float

}{6, 7.1}

or

car := struct{

Speed int

Weight float

}{Speed: 6, Weight: 7.1}
```

구조)

첫번째 {}는 struct의 정의 (definition)

두번째 {}는 struct의 객체화 (instanciation)

예) struct literal 사용 vs emptry struct 사용

```go
func TestXXX(t *testing.T) {

// map for struct literal(without struct typename)

s1 := make(map[int]struct {

x int

y int

})

// OK,

s1[0] = struct {

x int

y int

}{1, 2}

// Compiler Error(IncompatibleAssign): cannot use (struct{}
literal) (value of type struct{}) as struct{x int; y int} value in
assignment

// s1[1] = struct{}{}

// map for empty struct

s2 := make(map[int]struct{})

s2[0] = struct{}{}

}
```

**<> empty struct literal (struct{}{})**

<https://stackoverflow.com/questions/42469833/golang-struct-meaning>

<https://stackoverflow.com/questions/45122905/how-do-struct-and-struct-work-in-go>

struct{} ? empty struct "type" - 멤버가 없는 빈 struct 타입

struct{}{} ? empty struct "literal"(= instance = value of empty struct
type)

사이즈가 0인 empty struct의 인스턴스

empty struct type의 값으로만 사용될 수 있으며 다른 struct나 composite
타입의 값으로 사용될 수 없다.

인스턴스의 사이즈가 0이어서 단순 이벤트용 channel에 사용시 퍼포먼스
향상을 기대할 수 있다.

empty struct type 사용처)

1, 다른 composite 타입(slice, channel, map)의 멤버 또는 elements로
사용하여 테스트 용도로 사용한다.

2, 이벤트 수신 용으로 사용할 channel의 type으로 empty struct를 사용하여
이벤트 용도의 channel이라는 개발자 의도 명시

예)

```go
func TestXXX(t *testing.T) {

// map for empty struct

s := make(map[int]struct{})

s[0] = struct{}{}

s[1] = struct{}{}

// print empty struct instance

// ok, check if a value is in the map

v0, ok := s[0]

t.Log(v0, ok) // {} true

}
```

**<> empty interface literal (interface{}{})**

interface{} ? empty interface "type" - 멤버가 없는 빈 interface 타입

nil을 제외한 모든 golang의 모든 자료형을 대신할 수 있다.

any 자료형으로 재정의 되어 있으며 그러므로 interface{} 라는 풀 expr 대신
any 지시어 사용을 권장한다.

interface{}{} ? empty interface "literal"(= instance = value of empty
interface type)

사이즈가 0인 empty interface의 인스턴스

empty struct type의 value으로만 사용될 수 있으며 다른 struct나 composite
타입의 값으로 사용될 수 없다.

---

# [go] struct embedding과 nesting

**<> struct embedding**

```
type Embedded struct {

}

type Outter struct {

Embedded // struct embedding

}
```

struct embedding은 struct 멤버로써 다른 struct를 변수없이 struct의
tag만으로 선언하는 것을 말한다.

struct embedding은 상속과 같은 효과를 내며 embedded struct는 다른 언어의
상속 개념에서 super의 역할하게 된다.

embedded struct의 멤버중 derived struct에 의해 override 되지 않은 멤버는
derived struct instance에서 direct로 접근 가능하다. 이것이 struct
embedding의 핵심 기능 이다.

ex) <derived-struct-instance>.<non-override-embedded-struct-member>

embedded struct의 멤버중 derived struct에 의해 override 된 멤버는
derived struct instance에서 embedded struct tag를 통해 접근 가능하다.
derived struct instance에서 embedded struct tag를 통하면 override와 관계
없이 embedded struct의 모든 멤버에 접근 가능다.

ex)
<derived-struct-instance>.<embedded-struct-type>.<embedded-struct-member>

**<> struct nesting**

```
type Nested struct {

}

type Nested struct {

nest Nested // struct nesting

}
```

struct nesting은 struct 멤버로써 다른 struct를 변수명과 함께 선언하는
것을 말한다.

nested struct의 모든 멤버는 derived struct instance에서 nested struct
변수 명으로만 접근 가능하다.

ex)
<outter-struct-instance>.<nested-struct-variable>.<nested-struct-member>

```go
package test

import (

"testing"

)

type Inner struct {

name_over string

name_no_over string

}

// overrided func by outter

func (i Inner) func_override() string {

return i.name_over

}

// NOT overrided func by outter

func (i Inner) func_no_override() string {

return i.name_over

}

type Outter struct {

Inner // struct embedding

nest Inner // struct nesting

name_over string

}

func (o Outter) func_override() string {

return o.name_over

}

func TestStructEmbed(t *testing.T) {

o := Outter{

// initialize embedded struct

Inner: Inner{

name_over: "Embedded",

name_no_over: "Embedded",

},

// initialize nested struct

nest: Inner{

name_over: "Nested",

name_no_over: "Nested",

},

name_over: "Outter",

}

t.Log("\n// outter ")

t.Log("o.oname: ", o.name_over) // Outter

t.Log("o.iname: ", o.name_no_over) // Embedded

t.Log("o.func_override(): ", o.func_override()) // Outter

t.Log("o.func_no_override(): ", o.func_no_override()) // Embedded

t.Log("\n// embedded ")

t.Log("o.Inner.name_over: ", o.Inner.name_over) // Embedded

t.Log("o.Inner.name_no_over: ", o.Inner.name_no_over) // Embedded

t.Log("o.Inner.func_override(): ", o.Inner.func_override()) //
Embedded

t.Log("o.Inner.func_no_override(): ", o.Inner.func_no_override())
// Embedded

t.Log("\n// nested ")

t.Log("o.nest.name_over: ", o.nest.name_over) // Nested

t.Log("o.nest.name_no_over: ", o.nest.name_no_over) // Nested

t.Log("o.nest.func_override(): ", o.nest.func_override()) // Nested

t.Log("o.nest.func_no_override(): ", o.nest.func_no_override()) //
Nested

}
```

---

# [go] generics

<https://go.dev/doc/tutorial/generics>

특징)

1, func에서만 사용 가능

2, 정의는 [] 안에 명시

형식)

func <func-name>[<generic-type-expression>](<parameter>)
(<return-value>) {}

<generic-type-expression> = <generic-name> <available-type> |
<available-type> | ...

= V comparable | any | int64 | int

예) a generic function to handle single types

```go
func RemoveFromSlice[T comparable](l []T, v T) []T {

for i, e := range l {

if e == v {

l = append(l[:i], l[i+1:]...)

}

}

return l

}
```

예) a generic function to handle multiple types

```go
// SumIntsOrFloats sums the values of map m. It supports both int64
and float64

// as types for map values.

func SumIntsOrFloats[K comparable, V int64 
V {

var s V

for _, v := range m {

s += v

}

return s

}
```

[Go by Example: Generics](https://gobyexample.com/generics)

```go
package main

import "fmt"

func MapKeys[K comparable, V any](m map[K]V) []K {

r := make([]K, 0, len(m))

for k := range m {

r = append(r, k)

}

return r

}

type List[T any] struct {

head, tail *element[T]

}

type element[T any] struct {

next *element[T]

val T

}

func (lst *List[T]) Push(v T) {

if lst.tail == nil {

lst.head = &element[T]{val: v}

lst.tail = lst.head

} else {

lst.tail.next = &element[T]{val: v}

lst.tail = lst.tail.next

}

}

func (lst *List[T]) GetAll() []T {

var elems []T

for e := lst.head; e != nil; e = e.next {

elems = append(elems, e.val)

}

return elems

}

func main() {

var m = map[int]string{1: "2", 2: "4", 4: "8"}

fmt.Println("keys:", MapKeys(m))

_ = MapKeys[int, string](m)

lst := List[int]{}

lst.Push(10)

lst.Push(13)

lst.Push(23)

fmt.Println("list:", lst.GetAll())

}
```

keys: [4 1 2]

list: [10 13 23]
