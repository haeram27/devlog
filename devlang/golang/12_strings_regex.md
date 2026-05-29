# [go][cheat] string

[Go string handling overview \[cheat sheet\] - Your Basic Go](https://yourbasic.org/golang/string-functions-reference-cheat-sheet/)

### String literals (escape characters)

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
|  ""| | [Default zero value](https://yourbasic.org/golang/default-zero-value/) for type string |
|  "Japan 日本"           | Japan 日本 | Go code is [Unicode text encoded in UTF‑8](https://yourbasic.org/golang/rune/) |
| "\xe6\x97\xa5"          | 日      | \xNN specifies a byte |
| "\u65E5"                | 日      | \uNNNN specifies a Unicode value |
| "\\"                    | \       | Backslash |
| "\""                    | "       | Double quote |
| "\n"                    |         | Newline |
| "\t"                    |         | Tab |
| `\xe6`                  | \xe6    | Raw string literal* |
| html.EscapeString("<>") | &lt;&gt;| HTML escape for <, >, &, ' and " |
| url.PathEscape("A B")   | A%20B   | URL percent-encoding net/url |

- In `` string literals, text is interpreted literally and backslashes have no special meaning. See [Escapes and multiline strings](https://yourbasic.org/golang/multiline-string/) for more on raw strings, escape characters and string encodings.

### Concatenate

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
| "Ja" + "pan" | Japan | Concatenation |

**Performance tips**

See [3 tips for efficient string concatenation](https://yourbasic.org/golang/build-append-concatenate-strings-efficiently/) for
how to best use a string builder to concatenate strings without redundant copying.

### Equal and compare (ignore case)

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
| "Japan" == "Japan" | true | Equality |
| strings.EqualFold("Japan",   "JAPAN") | true | Unicode case folding |
| "Japan" < "japan"  | true | Lexicographic order |

### Length in bytes or runes

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
| len("日")  | 3 | Length in bytes |
| utf8.RuneCountInString("日") | 1 | in runes unicode/utf8  |
| utf8.ValidString("日") | true | unicode/utf8  |

### Index, substring, iterate

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
| "Japan"[2]   |'p' | Byte at position 2 |
| "Japan"[1:3] | ap | Byte indexing |
| "Japan"[:2]  | Ja |             |
| "Japan"[2:]  | pan |             |

A Go [range loop](https://yourbasic.org/golang/for-loop-range-array-slice-map-channel/) iterates over UTF-8 encoded characters ([runes](https://yourbasic.org/golang/rune/)):

```go
for i, ch := range "Japan 日本" {
    fmt.Printf("%d:%q ", i, ch)
}

// Output: 0:'J' 1:'a' 2:'p' 3:'a' 4:'n' 5:' ' 6:'日' 9:'本'
```

Iterating over bytes produces nonsense characters for non-ASCII text:

```go
s := "Japan 日本"

for i := 0; i < len(s); i++ {
    fmt.Printf("%q ", s[i])
}

// Output: 'J' 'a' 'p' 'a' 'n' ' ' 'æ' '\u0097' '¥' 'æ' '\u009c' '¬'
```

### Search (contains, prefix/suffix, index)

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
| strings.Contains("Japan",  "abc")    | false | Is abc in Japan? |
| strings.ContainsAny("Japan", "abc")  | true  | Is a, b or c in Japan? |
| strings.Count("Banana", "ana")       | 1     | Non-overlapping instances of ana |
| strings.HasPrefix("Japan", "Ja")     | true  | Does Japan start with Ja? |
| strings.HasSuffix("Japan", "pan")    | true  | Does Japan end with pan? |
| strings.Index("Japan", "abc")        | -1    | Index of first abc |
| strings.IndexAny("Japan", "abc")     | 1     | a, b or c |
| strings.LastIndex("Japan",  "abc")   | -1    | Index of last abc |
| strings.LastIndexAny("Japan", "abc") | 3     | a, b or c |

### Replace (uppercase/lowercase, trim)

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
| strings.Replace("foo", "o", ".", 2) | f.. | Replace first two "o" with "." Use -1 to replace all |
| f := func(r rune) rune { return r + 1 } strings.Map(f, "ab") | bc | Apply function to each character |
| st rings.ToUpper("Japan") | J APAN | Uppercase |
| st rings.ToLower("Japan") | j apan | Lowercase |
| s trings.Title("ja pan") | Ja Pan | Initial letters to uppercase |
| string s.TrimSpace(" foo\n") | foo | Strip leading and trailing white space |
| strings.Trim("foo", "fo") |  | Strip *leading and trailing* f:s and o:s |
| s trings.TrimLeft("foo", "f") | oo | *only leading* |
| st rings.TrimRight("foo", "o") | f | *only trailing* |
| str ings.TrimPrefix("foo", "fo") | o |  |
| str ings.TrimSuffix("foo", "o") | fo |  |

### Split by space or comma

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
| strings.Fields(" a\t b\n")     | ["a" "b"]  |  Remove white space |
| strings.Split("a,b", ",")      | ["a" "b"]  |  Remove separator |
| strings.SplitAfter("a,b", ",") | ["a," "b"] |  Keep separator |

### Join strings with separator

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
| strings.Join([]string{"a", "b"}, ":") | a:b   | Add separator |
| strings.Repeat("da", 2) | dada  | 2 copies of "da" |

### Format and convert

| **Expression** | **Result** | **Note** |
| --- | --- | --- |
| strconv.Itoa(-42)          | "-42" | Int to string |
| strconv.FormatInt(255, 16) | "ff"  | Base 16 |

### Sprintf

The [fmt.Sprintf](https://golang.org/pkg/fmt/#Sprintf) function is often your best friend when formatting data:

```go
s := fmt.Sprintf("%.4f", math.Pi) // s == "3.1416"
```

---

## [go] string concatenation

요약) golang string concatenation 가장 빠른 방법

- 1등 : <bytes.Buffer>.WriteString(s string)
- 2등 : strings.Join(ss []string, ...)
- 3등 : += 또는 + 연산자
- 4등 : fmt.Sprintf()

```go
package main

import (
    "bytes"
    "fmt"
    "strings"
)

func main() {
    // 1. + 연산자
    str1 := "Welcome "
    str2 := "Rain!"
    str := str1 + str2
    fmt.Println(str)

    str = str1 + "my " + str2
    fmt.Println(str)

    // 2. += 연산자
    str = ""
    str += str1
    str += str2
    fmt.Println(str)

    // 3. bytes.Buffer.WriteString()
    var b bytes.Buffer
    b.WriteString("R")
    b.WriteString("a")
    b.WriteString("i")
    b.WriteString("n")
    fmt.Println(b.String())

    // 4. fmt.Sprintf()
    str = fmt.Sprintf("%s%s", str1, str2)
    fmt.Println(str)

    // 5. strings.Join()
    mySlice := []string{"Welcome", "my", "Rain!"}
    str = strings.Join(mySlice, " * ")
    fmt.Println(str)
}
```

string 이어붙이기 성능을 알아보기 위해 위의 방법들에 대해 간단히 시간
측정을 해보았다.

`+` 와 `+=` 를 이용한 방법은 동일하다고 볼 수 있으므로 아래 4 가지 방법에 대한 측정 결과를 보자.

```go
package main

import (
    "bytes"
    "strings"
    "fmt"
    "time"
)

func main() {
    str := ""
    str1 := "AAAAAAAAAA"

    // 1. bytes.Buffer 의 WriteString() 함수를 이용하는 방법
    var b bytes.Buffer
    str = ""
    start = time.Now()

    for i := 0; i<100000; i++ {
        b.WriteString(str1)
    }

    str = b.String()
    elapsed = time.Since(start)
    fmt.Printf("strlen(%d) : %v\n", len(str), elapsed)

    // 2. strings.Join([]string)
    str = ""
    mySlice := []string{}

    for i := 0; i<100000; i++ {
        mySlice = append(mySlice, str1)
    }

    start = time.Now()
    str = strings.Join(mySlice, "")
    elapsed = time.Since(start)
    fmt.Printf("strlen(%d) : %v\n", len(str), elapsed)

    // 3. fmt.Sprintf() 함수를 이용하는 방법
    str = ""
    start = time.Now()
    for i := 0; i<100000; i++ {
        str = fmt.Sprintf("%s%s", str, str1)
    }

    elapsed = time.Since(start)

    fmt.Printf("strlen(%d) : %v\n", len(str), elapsed)

    // 4. += 연산자를 이용하는 방법
    start := time.Now()

    for i := 0; i<100000; i++ {
        str += str1
    }

    elapsed := time.Since(start)

    fmt.Printf("strlen(%d) : %v\n", len(str), elapsed)
}
```

```text
1. strlen(1000000) : 1.309558ms
2. strlen(1000000) : 1.644996ms
3. strlen(1000000) : 5.306749611s
4. strlen(1000000) : 11.813286425s
```

시험 결과를 빠른 순서대로 순위를 매겨보면 다음과 같다.

- 1등 : bytes.Buffer.WriteString(s string)
- 2등 : strings.Join(ss []string)
- 3등 : += 또는 + 연산자
- 4등 : fmt.Sprintf()

단지 string 을 이어붙이는 작업에서 왜 이런 큰 성능 차이가 발생할까?

이러한 부분을 간과하고 사용하면 성능에서 자칫 큰 손해를 볼 수 있으니 내부 구현을 좀 알고 사용하는 것이 좋겠다.

### bytes.Buffer 의 WriteString()을 이용한 이어붙이기

go standard library 에 구현된 WriteString() 의 코드를 보면 다음과 같다.

```go
type Buffer struct {
    buf []byte // contents are the bytes buf[off : len(buf)]
    off int // read at &buf[off], write at &buf[len(buf)]
    lastRead readOp // last read operation, so that Unread* can work correctly.
}

func (b *Buffer) tryGrowByReslice(n int) (int, bool) {
    if l := len(b.buf); n <= cap(b.buf)-l {
        b.buf = b.buf[:l+n]
        return l, true
    }
    return 0, false
}

func (b *Buffer) WriteString(s string) (n int, err error) {
b.lastRead = opInvalid
m, ok := b.tryGrowByReslice(len(s))

    if !ok {
        m = b.grow(len(s))
    }

    return copy(b.buf[m:], s), nil
}
```

WriteString() 함수는 결국 Buffer 구조체가 가지고 있는 byte slice(buf)에 인자로 받은 string 을 이어붙이기 하는 함수이다.

그런데, 처음에는 이미 할당된 slice 공간을 이용하려고 시도(tryGrowByReslice)하지만, 공간이 자라면 결국 slice 의 capacity 자체를 늘린 후(grow) string 을 복사하게 된다.

따라서, WriteString()함수는 slice 에 계속해서 데이터를 추가해나가는 작업의 속도라고 할 수 있다.

그러니까, 빠르다 !

Java 에서 빠른 이어붙이기를 할 때 StringBuilder 나 StringBuffer 를 사용하는 이유도 내부적으로 이와 유사하게 동작하기 때문이다.

### strings.Join() 을 이용한 이어붙이기

strings.Join() 의 경우도 slice 에 존재하는 요소들을 모두 이어붙이는데 WriteString()을 이용한 위의 방법과 거의 유사한 성능이 나왔다. (1.644996ms)

Join() 함수의 구현을 살펴보면 그 이유를 알 수 있는데, 그 핵심은 내부적으로 결국 slice 의 공간을 확보하고 이어붙이는 방법은 거의 WriteString()과 유사하기 때문이다.

Join() 함수의 경우 separator 의 공간 확보를 위한 계산을 제외하고는 내부에서 Builder.WriteString() 함수를 사용하는데, 해당 함수는 내부적으로 Builder 구조체 내의 byte slice 에 string 을 append 하는 작업이다.

```go
type Builder struct {
    addr *Builder // of receiver, to detect copies by value
    buf []byte
}

func Join(a []string, sep string) string {

    switch len(a) {
        case 0:
            return ""
        case 1:
            return a[0]
    }

    n := len(sep) * (len(a) - 1)
    for i := 0; i < len(a); i++ {
        n += len(a[i])
    }

    var b Builder
    b.Grow(n)
    b.WriteString(a[0])

    for _, s := range a[1:] {
        b.WriteString(sep)
        b.WriteString(s)
    }

        return b.String()
}

func (b *Builder) WriteString(s string) (int, error) {
    b.copyCheck()
    b.buf = append(b.buf, s...)
    return len(s), nil
}

func (b *Builder) String() string {
    return (string)(unsafe.Pointer(&b.buf))
}
```

###`+=` 또는 `+` 를 이용한 이어붙이기

+= 를 이용하여 이어붙이기하는 성능은 slice 를 이용한 위의 두 방법에 비해 상당히 좋지 않은 성능이 나왔다.(5.306749611s)

수백배 이상의 차이다.

직관적으로 사용하기에는 좋아서 작은 갯수의 string 을 이어붙일 때는 전혀 상관이 없지만(오히려 좋은 결과가 나올 수도 있다),

아주 빈번하고 많은 양의 이어붙이기를 하는 경우에는 `native plus operator`는 지양하는 것이 좋겠다.

`+=` 나 `+` 연산을 이용하는 것이 느린 이유는 string 을 이어붙일 때마다 새로운 공간을 할당하고 기존의 string 을 복사하는 작업을 수행하기 때문이다.

이러한 작업은 string 의 길이가 길어지면 길어질수록 기하급수적으로(O(N\^2)) 느려질 수 밖에 없다.

### fmt.Sprintf() 를 이용한 이어붙이기

fmt.Sprintf() 함수를 이용하는 방법이 압도적인 꼴찌를 기록했는데(11.813286425s), 그 이유는 formatting 을 위한 수많은 복잡한 연산을 수행하기 때문이다.

아래 코드 중 doPrintf() 함수가 이에 해당된다.

성능이 꼭 필요한 부분인데 format 을 굳이 사용하지 않아도 되는 경우라면 fmt.Sprintf() 를 이용하여 string 이어붙이기 작업을 할 필요는 없을 것이다.

```go
func Sprintf(format string, a ...interface{}) string {

    p := newPrinter()
    p.doPrintf(format, a)

    s := string(p.buf)
    p.free()

    return s
}
```

`+`, `+=` 연산자나 `Sprintf()` 의 성능이 좋지 않기 때문에 모든 경우에서 배제하는건 어리석은 일이다.

분명한 사용 편의성이 존재하고 코드를 명확하고 아름답게 해주는 장점 또한 존재하기 때문이다.

이게 맞네 저게 맞네하면서 편가르고 싸우는 것 보다, 다양한 방법들을 정확히 이해하는 상태에서 상황에 맞게 다양한 방법을 사용할 줄 아는 것 또한 개발자의 역량이라는 것을 잊지 말자.

---

## [go][cheat] regex

[Regexp tutorial and cheat sheet · YourBasicGo](https://yourbasic.org/golang/regexp-cheat-sheet/)

[https://gobyexample.com/regular-expressions](<https://mingrammer.com/gobyexample/regular-expressions/>)

<https://pkg.go.dev/regexp>

<https://github.com/google/re2/wiki/Syntax>

<https://cs.opensource.google/go/go/+/refs/tags/go1.19.1:src/regexp/regexp.go>

```go
package main

import (
    "bytes"
    "fmt"
    "regexp"
)

func main() {

    match, _ := regexp.MatchString("p([a-z]+)ch", "peach")
    fmt.Println(match)

    r, _ := regexp.Compile("p([a-z]+)ch")

    fmt.Println(r.MatchString("peach"))
    fmt.Println(r.FindString("peach punch"))
    fmt.Println("idx:", r.FindStringIndex("peach punch"))
    fmt.Println(r.FindStringSubmatch("peach punch"))
    fmt.Println(r.FindStringSubmatchIndex("peach punch"))
    fmt.Println(r.FindAllString("peach punch pinch", -1))
    fmt.Println("all:", r.FindAllStringSubmatchIndex(

    "peach punch pinch", -1))

    fmt.Println(r.FindAllString("peach punch pinch", 2))
    fmt.Println(r.Match([]byte("peach")))

    r = regexp.MustCompile("p([a-z]+)ch")

    fmt.Println("regexp:", r)
    fmt.Println(r.ReplaceAllString("a peach", "<fruit>"))

    in := []byte("a peach")
    out := r.ReplaceAllFunc(in, bytes.ToUpper)
    fmt.Println(string(out))
}
```

### Method Prefix

- Match? return type: bool(매칭 존재 여부)
- Find(String)? return type: byte array(매칭 부분 문자열)
- Replace?
- Expand?
