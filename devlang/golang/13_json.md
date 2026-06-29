# [go][json] 핸들링

**<> 참고**

[The Go Blog: JSON and Go](https://go.dev/blog/json)

[A Complete Guide to JSON in Golang (With
Examples)](https://www.sohamkamani.com/golang/json/)

golang에서 json은 struct, map, slice 등을 통해 다루어질 수 있음

struct와 map의 data를 json으로 변환하는 것을 Marshal이라고 함

Marshal이란 데이터를 네트워크로 전송하기 유리한 형식(메모리 최적화 =
공백, 줄바꿈 등 제거)으로 변환하는 것을 의미

반대로 raw json을 struct나 map으로 변환하는 것을 Unmarshal 이라고 함

Unmarshal이란 raw json을 코드에서 다루기 유리한 형식(프로그래밍에 편리한
형식 = struct or map)으로 변환하는 것을 의미

**["encoding/json"] package**

func Unmarshal(data []byte, v any) error

## 프로그램에서 처리가 편리하도록 구조화된 데이터(구조체, 맵)으로 변환

func Marshal(v any) ([]byte, error)

## 네트워크에 전송에 효율적이도록 공백없는 바이트 스트림으로 변환

Marshall()과 Unmarshall()은 string 또는 byte slice 로 명시된 json raw
데이터와 구조화된 데이터(structure, map) 사이에 변환

DOM 방식. 한번에 데이터를 모두 메모리에 올려 변환 하므로 빠르지만 대용량
데이터일 수록 메모리 사용량이 높음

func NewEncoder(w io.Writer) *json.Encoder ## marshal

func NewDecoder(r io.Reader) *Decoder ## unmarshal

Stream 형태의 json data를 marshal/unmarshal하기 위한 함수

SAX 방식으로 이용 가능.

**[struct와 json] feat "encoding/json" package**

golang은 기본적으로 json 데이터를 struct 형식으로 다룬다.

json은 네트워크 전송 포맷, struct는 json 형식의 데이터를 프로그래밍
자료형으로 다루는 개념이다. 이 개념을 구현하는 package가
"encoding/json" 패키지 이며 여기서 Marshal/UnMarshl의 용어가 사용된다.

Marshal : Struct -> Json

Marshal은 프로그래밍 언어에서 데이터를 네트워크로 전송할 때 코스트를
최소화하기 위해 데이터를 메모리 최소 사용 형태로 packaging하는것을
말한다.

golang의 json 핸들링 개념에서 이 용어는 struct 형식의 데이터를 "공백과
줄바꿈 문자를 제거한 json 포맷의 byte array"으로 변환 하는 것을
의미한다.

"encoding/json" 패키지를 이용해 Marshal(struct -> Json) 하기

```go
import "encoding/json"

...
jsonData, err := json.MarshalIndent(structData, "", " ")

if err != nil {

fmt.Println("Marshaling is failed")

fmt.Println(err)

return

}

fmt.Println(string(jsonData))
```

# byte slice인 json 데이트를 출력하는 방법

jsonData byte[]

fmt.Println(string(jsonData))

UnMarshal : Json -> Struct or Map

UnMarshal은 Marshal과 반대로 네트워크를 통해 들어온 데이터를 json 포맷의
byte array 데이터를 struct에 포맷으로 변환하는 것을 의미한다

"encoding/json" 패키지를 이용해 Marshal(struct -> Json) 하기

```go
import "encoding/json"

...
structData, err := json.UnMarshal(jsonData)

if err != nil {

fmt.Println("UnMarshaling is failed")

fmt.Println(err)

return

}

fmt.Println(structData)
```

# struct 포맷을 출력하는 방법

structData struct

fmt.Println(structData)

Unmarshal: Json to Pre-defined Struct

```go
import "encoding/json"

type Person struct {

Name struct {

First string

Last string

}

Job string

}

test := `{

"name": {

"first": "pink",

"last": "panther"

},

"job": "developer"

}`

data := Person{}

// Unmarshal 의 첫번째 파라미터 타입은 []byte 이므로

// raw json을 string 타입이 아닌 []byte 타입으로 형변환하여
전달합니다.

err := json.Unmarshal([]byte(test), &data)

if err != nil {

fmt.Println(err)

}

fmt.Println(data)

fmt.Println("full name: " + data.Name.First, data.Name.Last)

fmt.Println("job: " + data.Job)
```

{{pink panther} developer}

full name: pink panther

job: developer

Unmarshal: Json to Map

```go
test := `{

"name": {

"first": "pink",

"last": "panther"

},

"job": "developer"

}`

// 깊이를 알수 없는 json 이므로 interface 에 담길 수 있도록 처리

data := make(map[string]interface{})

err := json.Unmarshal([]byte(test), &data)

if err != nil {

fmt.Println(err)

}

fmt.Println(data)

// fmt.Println(data["name"]["last"]) // 오류 발생. 이런식의
접근을 원하면 JSON to Nested Map 참조

// 하위 요소 index 에 접근하기 위해 캐스팅

name := data["name"].(map[string]interface{})

// interface{} 를 string 연산해야하므로 캐스팅

// Nested가 아닌 Map 으로 변환할 경우 하위 요소에 접근할 때 마다
캐스팅을 해주어야 합니다

fmt.Println("full name: " + name["first"].(string) +
name["last"].(string))

fmt.Println("job: " + data["job"].(string))
```

map[job:developer name:map[first:pink last:panther]]

full name: pink panther

job: developer

Unmarshal: JSON to Nested Map

```go
test := `{

"name": {

"first": "pink",

"last": "panther"

},

"job": {

"major": "developer",

"minor": "animal"

}

}`

// map 형식을 json 의 깊이와 동일하게 구성

data := make(map[string]map[string]string)

err := json.Unmarshal([]byte(test), &data)

if err != nil {

fmt.Println(err)

}

fmt.Println(data)

fmt.Println("full name: " + data["name"]["first"] +
data["name"]["last"])

fmt.Println("job: " + data["job"]["major"] + ", " +
data["job"]["minor"])
```

map[job:map[major:developer minor:animal] name:map[first:pink
last:panther]]

full name: pink panther

job: developer, animal

Marshal: pre-defined struct to json

```go
type Person struct {

Name struct {

First string

Last string

}

Job string

}

p := Person{}

p.Name.First = "pink"

p.Name.Last = "phanther"

p.Job = "developer"

mashalledJson, err := json.Marshal(p)

if err != nil {

fmt.Println(err)

}

fmt.Println(string(mashalledJson))
```

{"Na
me":{"First":"pink","Last":"panther"},"Job":"developer"}

-------------------------------------------------------------------

## golang에서 json parsing하기

with structure - json 문서의 구조를 알고 있을 경우 사용

with map - json 문서의 구조를 모르고 있을 경우 사용

with jsonpath package - 특정 데이터만 확인하고자 하는 경우 사용

**example json:**

{

"birds": {

"pigeon":"likes to perch on rocks",

"eagle":"bird of prey"

},

"animals": "none"

}

**parsing with map**

```go
var mytoken string

var unmarshaledJson map[string]interface{}

json.Unmarshal([]byte(rawJson), &unmarshaledJson)

birds := unmarshaledJson["birds"].(map[string]interface{})

for key, value := range birds {

// Each value is an interface{} type, that is type asserted as a
string

fmt.Println(key, value.(string))

if key == "eagle" {

myeagle = value.(string)

}

}
```

**parsing with jsonpath pkg)**

```go
var myeagle string

var unmarshaledJson interface{}

json.Unmarshal([]byte(rawJson), &unmarshaledJson)

value, err := jsonpath.JsonPathLookup(unmarshaledJson,
"$.birds.eagle")

if err != nil {

fmt.Printf("err.Error(): %v\n", err.Error())

}

myeagle = value.(string)
```

**parsing with jsonpath pkg) one value**

```go
var json_unmar interface{}

json.Unmarshal([]byte(json_origin_str), &json_unmar)

v, err := jsonpath.JsonPathLookup(json_unmar, "$.value")

if err != nil {

fmt.Printf("err.Error(): %v\n", err.Error())

}
```

**parsing with jsonpath pkg) array**

```go
var json_unmar interface{}

var varray []string

json.Unmarshal([]byte(json_origin_str), &json_unmar)

jarray, err := jsonpath.JsonPathLookup(json_unmar,
"$.arrays[:]")

if err == nil {

temp := jarray.([]interface{})

for _, v := range temp {

varray = append(varray, v.(string))

}

} else {

fmt.Printf("err.Error(): %v\n", err.Error())

}
```

---

# [go][json] [How to use json in
go](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go)

**<> Using a Map to Generate JSON**

main.go:
-----------------------------------------------------------

package main

import (

"encoding/json"
"fmt"

)

func main() {

data := map[string]interface{}{
"intValue": 1234,
"boolValue": true,
"stringValue": "hello!",
"objectValue": map[string]interface{}{
"arrayValue": []int{1, 2, 3, 4},
},
}

jsonData, err := json.Marshal(data)
if err != nil {
fmt.Printf("could not marshal json: %s\n", err)
return
}

fmt.Printf("json data: %s\n", jsonData)

}

Output:
-----------------------------------------------------------

json data:
{"boolValue":true,"intValue":1234,"objectValue":{"arrayValue":[1,2,3,4]},"stringValue":"hello!"}

**<> Encoding Times in JSON**

main.go:
-----------------------------------------------------------

package main

import (

"encoding/json"
"fmt"
"time"

)

func main() {

data := map[string]interface{}{
"intValue": 1234,
"boolValue": true,
"stringValue": "hello!",
"dateValue": time.Date(2022, 3, 2, 9, 10, 0, 0, time.UTC),
"objectValue": map[string]interface{}{
"arrayValue": []int{1, 2, 3, 4},
},
}

...

}

Output:
-----------------------------------------------------------

json data:
{"boolValue":true,"dateValue":"2022-03-02T09:10:00Z","intValue":1234,"objectValue":{"arrayValue":[1,2,3,4]},"stringValue":"hello!"}

**<> Encoding null Values in JSON**

main.go:
-----------------------------------------------------------

func main() {

data := map[string]interface{}{
"intValue": 1234,
"boolValue": true,
"stringValue": "hello!",

"dateValue": time.Date(2022, 3, 2, 9, 10, 0, 0, time.UTC),

"objectValue": map[string]interface{}{
"arrayValue": []int{1, 2, 3, 4},
},
"nullStringValue": nil,
"nullIntValue": nil,
}

...

}

Output:
-----------------------------------------------------------

json data:
{"boolValue":true,"dateValue":"2022-03-02T09:10:00Z","intValue":1234,"nullIntValue":null,"nullStringValue":null,"objectValue":{"arrayValue":[1,2,3,4]},"stringValue":"hello!"}

**<> Using a Struct to Generate JSON**

main.go:
-----------------------------------------------------------

...

type myJSON struct {

IntValue int `json:"intValue"`
BoolValue bool `json:"boolValue"`
StringValue string `json:"stringValue"`
DateValue time.Time `json:"dateValue"`
ObjectValue *myObject `json:"objectValue"`
NullStringValue *string `json:"nullStringValue"`
NullIntValue *int `json:"nullIntValue"`

}

type myObject struct {

ArrayValue []int `json:"arrayValue"`

}

func main() {

otherInt := 4321
data := &myJSON{
IntValue: 1234,
BoolValue: true,
StringValue: "hello!",
DateValue: time.Date(2022, 3, 2, 9, 10, 0, 0, time.UTC),
ObjectValue: &myObject{
ArrayValue: []int{1, 2, 3, 4},
},
NullStringValue: nil,
NullIntValue: &otherInt,
}

...

}

Output:
-----------------------------------------------------------

json data:
{"intValue":1234,"boolValue":true,"stringValue":"hello!","dateValue":"2022-03-02T09:10:00Z","objectValue":{"arrayValue":[1,2,3,4]},"nullStringValue":null,"nullIntValue":4321}

main.go:
-----------------------------------------------------------

...

type myJSON struct {

...
 
NullStringValue *string `json:"nullStringValue,omitempty"`
NullIntValue *int `json:"nullIntValue"`
EmptyString string `json:"emptyString,omitempty"`

}

...

Output:
-----------------------------------------------------------

json data:
{"intValue":1234,"boolValue":true,"stringValue":"hello!","dateValue":"2022-03-02T09:10:00Z","objectValue":{"arrayValue":[1,2,3,4]},"nullIntValue":4321}

**<> Parsing JSON Using a Map**

main.go:
-----------------------------------------------------------

...

func main() {

jsonData := `
{
"intValue":1234,
"boolValue":true,
"stringValue":"hello!",
"dateValue":"2022-03-02T09:10:00Z",
"objectValue":{
"arrayValue":[1,2,3,4]
},
"nullStringValue":null,
"nullIntValue":null
}
`

var data map[string]interface{}
err := json.Unmarshal([]byte(jsonData), &data)
if err != nil {
fmt.Printf("could not unmarshal json: %s\n", err)
return
}

fmt.Printf("json map: %v\n", data)

}

Output:
-----------------------------------------------------------

json map: map[boolValue:true dateValue:2022-03-02T09:10:00Z
intValue:1234 nullIntValue:<nil> nullStringValue:<nil>
objectValue:map[arrayValue:[1 2 3 4]] stringValue:hello!]

main.go:
-----------------------------------------------------------

...

func main() {

...
 
fmt.Printf("json map: %v\n", data)

rawDateValue, ok := data["dateValue"]
if !ok {
fmt.Printf("dateValue does not exist\n")
return
}
dateValue, ok := rawDateValue.(string)
if !ok {
fmt.Printf("dateValue is not a string\n")
return
}
fmt.Printf("date value: %s\n", dateValue)

}

Output:
-----------------------------------------------------------

json map: map[boolValue:true dateValue:2022-03-02T09:10:00Z
intValue:1234 nullIntValue:<nil> nullStringValue:<nil>
objectValue:map[arrayValue:[1 2 3 4]] stringValue:hello!]

date value: 2022-03-02T09:10:00Z

**<> Parsing JSON Using a Struct**

main.go:
-----------------------------------------------------------

...

func main() {

...
 
var data *myJSON
err := json.Unmarshal([]byte(jsonData), &data)
if err != nil {
fmt.Printf("could not unmarshal json: %s\n", err)
return
}

fmt.Printf("json struct: %#v\n", data)
fmt.Printf("dateValue: %#v\n", data.DateValue)
fmt.Printf("objectValue: %#v\n", data.ObjectValue)

}

Output:
-----------------------------------------------------------

json struct: &main.myJSON{IntValue:1234, BoolValue:true,
StringValue:"hello!", DateValue:time.Date(2022, time.March, 2, 9, 10,
0, 0, time.UTC), ObjectValue:(*main.myObject)(0x1400011c180),
NullStringValue:(*string)(nil), NullIntValue:(*int)(nil),
EmptyString:""}

dateValue: time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC)

objectValue: &main.myObject{ArrayValue:[]int{1, 2, 3, 4}}

main.go:
-----------------------------------------------------------

...

func main() {

jsonData := `
{
"intValue":1234,
"boolValue":true,
"stringValue":"hello!",
"dateValue":"2022-03-02T09:10:00Z",
"objectValue":{
"arrayValue":[1,2,3,4]
},
"nullStringValue":null,
"nullIntValue":null,
"extraValue":4321
}
`

...

}

Output:
-----------------------------------------------------------

json struct: &main.myJSON{IntValue:1234, BoolValue:true,
StringValue:"hello!", DateValue:time.Date(2022, time.March, 2, 9, 10,
0, 0, time.UTC), ObjectValue:(*main.myObject)(0x14000126180),
NullStringValue:(*string)(nil), NullIntValue:(*int)(nil),
EmptyString:""}

dateValue: time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC)

objectValue: &main.myObject{ArrayValue:[]int{1, 2, 3, 4}}

---

# [go][json] [How To Use Struct Tags in
Go](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go)

**<> What Does a Struct Tag Look Like?**

---------------------------------------------------------

package main

import "fmt"

type User struct {

Name string `example:"name"`

}

func (u *User) String() string {

return fmt.Sprintf("Hi! My name is %s", u.Name)

}

func main() {

u := &User{
Name: "Sammy",
}

fmt.Println(u)

}

Output:
----------------------------------------------------------

Hi! My name is Sammy

**<> Encoding JSON**

----------------------------------------------------------

package main

import (

"encoding/json"
"fmt"
"log"
"os"
"time"

)

type User struct {

Name string
Password string
PreferredFish []string
CreatedAt time.Time

}

func main() {

u := &User{
Name: "Sammy the Shark",
Password: "fisharegreat",
CreatedAt: time.Now(),
}

out, err := json.MarshalIndent(u, "", " ")
if err != nil {
log.Println(err)
os.Exit(1)
}

fmt.Println(string(out))

}

Output:
----------------------------------------------------------

{

"Name": "Sammy the Shark",

"Password": "fisharegreat",

"CreatedAt": "2019-09-23T15:50:01.203059-04:00"

}

----------------------------------------------------------

package main

import (

"encoding/json"
"fmt"
"log"
"os"
"time"

)

type User struct {

name string
password string
preferredFish []string
createdAt time.Time

}

func main() {

u := &User{
name: "Sammy the Shark",
password: "fisharegreat",
createdAt: time.Now(),
}

out, err := json.MarshalIndent(u, "", " ")
if err != nil {
log.Println(err)
os.Exit(1)
}

fmt.Println(string(out))

}

Output:
----------------------------------------------------------

{}

**<> Using Struct Tags to Control Encoding**

----------------------------------------------------------

package main

import (

"encoding/json"
"fmt"
"log"
"os"
"time"

)

type User struct {

Name string `json:"name"`
Password string `json:"password"`
PreferredFish []string `json:"preferredFish"`
CreatedAt time.Time `json:"createdAt"`

}

func main() {

u := &User{
Name: "Sammy the Shark",
Password: "fisharegreat",
CreatedAt: time.Now(),
}

out, err := json.MarshalIndent(u, "", " ")
if err != nil {
log.Println(err)
os.Exit(1)
}

fmt.Println(string(out))

}

Output:
----------------------------------------------------------

{

"name": "Sammy the Shark",

"password": "fisharegreat",

"preferredFish": null,

"createdAt": "2019-09-23T18:16:17.57739-04:00"

}

**<> Removing Empty JSON Fields**

----------------------------------------------------------

package main

import (

"encoding/json"
"fmt"
"log"
"os"
"time"

)

type User struct {

Name string `json:"name"`
Password string `json:"password"`
PreferredFish []string `json:"preferredFish,omitempty"`
CreatedAt time.Time `json:"createdAt"`

}

func main() {

u := &User{
Name: "Sammy the Shark",
Password: "fisharegreat",
CreatedAt: time.Now(),
}

out, err := json.MarshalIndent(u, "", " ")
if err != nil {
log.Println(err)
os.Exit(1)
}

fmt.Println(string(out))

}

Output:
----------------------------------------------------------

{

"name": "Sammy the Shark",

"password": "fisharegreat",

"createdAt": "2019-09-23T18:21:53.863846-04:00"

}

**<> Ignoring Private Fields**

----------------------------------------------------------

package main

import (

"encoding/json"
"fmt"
"log"
"os"
"time"

)

type User struct {

Name string `json:"name"`
Password string `json:"-"`
CreatedAt time.Time `json:"createdAt"`

}

func main() {

u := &User{
Name: "Sammy the Shark",
Password: "fisharegreat",
CreatedAt: time.Now(),
}

out, err := json.MarshalIndent(u, "", " ")
if err != nil {
log.Println(err)
os.Exit(1)
}

fmt.Println(string(out))

}

Output:
----------------------------------------------------------

{

"name": "Sammy the Shark",

"createdAt": "2019-09-23T16:08:21.124481-04:00"

}
