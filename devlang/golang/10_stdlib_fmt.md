## [go][pkg][builtin] os

```bash
go run main.go
```

main.go

```go
package main

import (
    "fmt"
    "os"
    "strings"
)

func main() {
    // Set Environment Variables
    os.Setenv("MY_ID", "admin")

    // Get the value of an Environment Variable
    id := os.Getenv("MY_ID")
    fmt.Printf("MY_ID: %s\n", id)

    // Unset an Environment Variable
    os.Unsetenv("MY_ID")
    fmt.Printf("After unset, MY_ID: %s\n", os.Getenv("MY_ID"))

    // Expand a string containing environment variables in the form of $var or ${var}
    os.Setenv("MY_ID", "admin2")
    str := os.ExpandEnv("MY_ID is ${MY_ID}.")
    fmt.Println("Content: ", str)

    // Checking that an environment variable is present or not.
    redisHost, ok := os.LookupEnv("REDIS_HOST")

    if !ok {
        fmt.Println("REDIS_HOST is not present")
    } else {
        fmt.Printf("Redis Host: %s\n", redisHost)
    }

    // Environ() returns a slice of string containing all the environment variables in the form of key=value.
    for _, env := range os.Environ() {
        // env is
        envPair := strings.SplitN(env, "=", 2)
        key := envPair[0]
        value := envPair[1]
        fmt.Printf("%s : %s\n", key, value)
    }

    // Delete all environment variables

    // os.Clearenv()
    fmt.Println("Number of environment variables: ", len(os.Environ()))
}
```

---

## [go][cheat] fmt

### 참고

- [Go Standard library: fmt](https://pkg.go.dev/fmt)
- [fmt.Printf formatting tutorial and cheat sheet - YourBasic Go](https://yourbasic.org/golang/fmt-printf-reference-cheat-sheet/)

### %v format 사용시 각 type별 default format

```go
The default format for %v is:

bool: %t
int, int8 etc.: %d
uint, uint8 etc.: %d, %#x if printed with %#v
float32, complex64, etc: %g
string: %s
chan: %p
pointer: %p

For compound objects, the elements are printed using these rules,
recursively, laid out like this:

struct: {field0 field1 ...}
array, slice: [elem0 elem1 ...]
maps: map[key1:value1 key2:value2 ...]
pointer to above: &{}, &[], &map[]
```

주의) 구조체(struct)의 포인터를 출력하려면? %p format 사용

```go
...
type mystruct struct{}
var r mystruct

...

fmt.Printf("%p\n", &r)
```

> 0x414020
