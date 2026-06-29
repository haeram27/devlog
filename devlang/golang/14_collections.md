# [go][slice] empty vs nil

*From <<https://gosamples.dev/empty-vs-nil-slice/>>*

empty는 인스턴스의 주소가 있으나 데이터는 없는 상태 이고,

nil은 인스턴스의 주소가 없는 상태이다.

* *

See the table to compare the properties of the nil and empty slice

  ------------------------------------------------------------------------
  **초기화 표현**                    **ptr**       **len**       **cap**
  ---------------------------------- ------------- ------------- ---------
  nil: []string                    0             0             0

  empty: []string{}                <addr>      0             0

  empty: make([]string, 0)         <addr>      0             0
  ------------------------------------------------------------------------

The <addr> is a non-zero address to an empty, non-nil array.

Slices in Go can be represented by 3 elements:

-   ptr - a pointer to the underlying array that contains data of the
    slice

-   len - length, number of elements in the slice

-   cap - capacity, number of elements in the underlying data array,
    starting from the element pointed by the ptr

A nil slice declared as var s1 []string has no underlying data array -
it points to nothing. An empty slice, declared as s2 := []string{} or
s3 := make([]string, 0) points to an empty, non-nil array.

See also in detail [what is the difference between length and capacity
of slices](https://gosamples.dev/capacity-and-length/)

```go
package main

import "fmt"

func main() {

var s1 []string // nil

s2 := []string{} // empty, following {} means instanciation(hold a
memory)

s3 := make([]string, 0) // empty, equivalent to s2 := []string{}

fmt.Printf("s1 is nil: %t, len: %d, cap: %d\n", s1 == nil,
len(s1), cap(s1))

fmt.Printf("s2 is nil: %t, len: %d, cap: %d\n", s2 == nil,
len(s2), cap(s2))

fmt.Printf("s3 is nil: %t, len: %d, cap: %d\n", s3 == nil,
len(s3), cap(s3))

}
```

Output:

s1 is nil: true, len: 0, cap: 0

s2 is nil: false, len: 0, cap: 0

s3 is nil: false, len: 0, cap: 0

Empty and nil slices behave in the same way that is the built-in
functions like len(), cap(), append(), and for .. range loop return the
same results. So, since the nil slice declaration is simpler, you should
prefer it to creating an empty slice. However, there are some cases when
you may need the empty, non-nil slice. For instance, when you want to
return an empty JSON array [] as an HTTP response, you should create
the empty slice ([]string{} or make([]string, 0)) because if you use
a nil slice, you will get a null JSON array after encoding:

```go
package main

import (

"encoding/json"

"fmt"

"log"

)

func main() {

var s1 []string // nil

s2 := []string{} // empty

s3 := make([]string, 0) // empty, equivalent to s2 :=
[]string{}

s1JSON, err := json.Marshal(s1)

if err != nil {

log.Fatal(err)

}

fmt.Printf("s1 JSON: %s\n", string(s1JSON))

s2JSON, err := json.Marshal(s2)

if err != nil {

log.Fatal(err)

}

fmt.Printf("s2 JSON: %s\n", string(s2JSON))

s3JSON, err := json.Marshal(s3)

if err != nil {

log.Fatal(err)

}

fmt.Printf("s3 JSON: %s\n", string(s3JSON))

}
```

Output:

s1 JSON: null

s2 JSON: []

s3 JSON: []

---

# [go][keyword] range

**returns by range from collection:**

  ------------------------------------------------------------------------
  **DataStructure**                **1st**              **2nd**
  -------------------------------- -------------------- ------------------
  **String**                       **index**            **rune int**

  **Array or slice**               **index**            **element**

  **Map**                          **key**              **value**

  **Channel**                      **element**          **none**
  ------------------------------------------------------------------------

In Golang Range keyword is used in different kinds of data structures in
order to iterates over elements. The range keyword is mainly used in for
loops in order to iterate over all the elements of a map, slice,
channel, or an array. When it iterates over the elements of an array and
slices then it returns the index of the element in an integer form. And
when it iterates over the elements of a map then it returns the key of
the subsequent key-value pair. Moreover, range can either returns one
value or two values. Lets see what range returns while iterating over
different kind of collections in Golang.

-   Array or slice: The first value returned in case of array or slice
    is index and the second value is element.

-   String: The first value returned in string is index and the second
    value is rune int.

-   Map: The first value returned in map is key and the second value is
    the value of the key-value pair in map.

-   Channel: The first value returned in channel is element and the
    second value is none.

Now, let's see some examples to illustrate the usage of range keyword in
Golang.

**Example 1: array**

```go
// Golang Program to illustrate the usage

// of range keyword over items of an

// array in Golang

package main

import "fmt"

// main function

func main() {

// Array of odd numbers

odd := [7]int{1, 3, 5, 7, 9, 11, 13}

// using range keyword with for loop to

// iterate over the array elements

for idx, value := range odd {

// Prints index and the elements

fmt.Printf("odd[%d] = %d \n", idx, value)

}

}
```

Output:

odd[0] = 1

odd[1] = 3

odd[2] = 5

odd[3] = 7

odd[4] = 9

odd[5] = 11

odd[6] = 13

**Example 2: string**

```go
// Golang Program to illustrate the usage of

// range keyword over string in Golang

package main

import "fmt"

// Constructing main function

func main() {

// taking a string

var string = "GeeksforGeeks"

// using range keyword with for loop to

// iterate over the string

for idx, value := range string {

// Prints index of all the

// characters in the string

fmt.Printf("string[%d] = %d \n", idx, value)

}

}
```

Output:

string[0] = 71

string[1] = 101

string[2] = 101

string[3] = 107

string[4] = 115

string[5] = 102

string[6] = 111

string[7] = 114

string[8] = 71

string[9] = 101

string[10] = 101

string[11] = 107

string[12] = 115

**Example 3: map**

```go
// Golang Program to illustrate the usage of

// range keyword over maps in Golang

package main

import "fmt"

// main function

func main() {

// Creating map of student ranks

student_rank_map := map[string]int{"Nidhi": 3,

"Nisha": 2, "Rohit": 1}

// Printing map using keys only

for student := range student_rank_map {

fmt.Println("Rank of", student, "is: ",

student_rank_map[student])

}

// Printing maps using key-value pair

for student, rank := range student_rank_map {

fmt.Println("Rank of", student, "is: ", rank)

}

}
```

Output:

Rank of Nidhi is: 3

Rank of Nisha is: 2

Rank of Rohit is: 1

Rank of Nidhi is: 3

Rank of Nisha is: 2

Rank of Rohit is: 1

**Example 4: channel**

goroutine context change 없이 channel로 부터 buffer 사이즈 만큼
element를 반복하여 pop할 때 사용한다.

channel의 버퍼는 버퍼 없는 channel에 push/pop할 때 마다 발생하는
goroutine context change의 횟수를 줄임으로써 performance를 향상 시키기
위해서 사용한다.

보통 channel의 buffer size가 1 이상인 channel에 대해 range를 사용할 수
있다.

channel의 buffer가 1 이상인 go는 경우 최선으로 한번에 최대 buffer
사이즈만큼의 횟수로 push 또는 pop action을 goroutine context change 없이
연속 실행해 준다.

  -----------------------------------------------------------------------
  for v := range ch {}
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

for-range를 사용하는 경우 송신측(channel에 push 하는 goroutine 측)에서
모든 데이터를 push 한 후에 반드시 channel을 close해 주어야만 한다.

range 문을 사용해 channel을 pop하는 경우, channel의 상태가 "non-closed
and non-empty" 상태인 경우 channel로 부터 pop을 수행하고 "non-closed
and empty" 상태인 경우 pop action이 blocking(deadlock)되며, "closed
and empty"가 되면 for loop를 escaping한다.

```go
ch := make(chan int, 3)

go func() {

t.Log("start go")

for v := range ch {

t.Logf("print: %v", v)

}

}()

ch <- 1

ch <- 1

ch <- 1

close(ch)
```

```go
func TestChannelForRange(t *testing.T) {

ch1 := make(chan int)

go func() {

for i := 0; i < 1000; i++ {

ch1 <- i

t.Log("send: ", i)

}

close(ch1)

}()

for v := range ch1 {

t.Log("recv: ", v)

}

t.Log("finish")

}
```

---

# [go] print struct

<https://www.delftstack.com/ko/howto/go/how-to-print-struct-variables-in-console/>

[Go 언어에서 Tag 사용](https://www.joinc.co.kr/w/man/12/golang/tag)

<https://programmers.co.kr/learn/courses/13/lessons/631>

<https://stackoverflow.com/questions/24216510/empty-or-not-required-struct-fields>

<https://www.joinc.co.kr/w/man/12/golang/tag>

<http://pyrasis.com/book/GoForTheReallyImpatient/Unit41>
