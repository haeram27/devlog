# [go][pattern] Resolver Maker

Resolver Maker 패턴:

서로 다른 형태의 입력 값들로 부터 같은 타입의 데이터를 생성해야 할 때
사용

다른 프로그래밍 언어의 overloading에 해당하는 기능을 구현하는 방식이다.

return type으로 동일한 fuction type을 갖는 function들을 Maker로 사용하여
실제 필요한 값을 채우는 resolver 익명 함수를 반환하도록 한다.

```go
type Container struct {

iv int

sv string

}

type ResolverFuncType func(*Container) error

// ResolverFuncMakerX returns ResolverFunc

// ResolverFunc는

func ResolverFuncMakerA(i int) ResolverFuncType {

return func(c *Container) error {

c.iv = i

return nil

}

}

func ResolverFuncMakerB(s string) ResolverFuncType {

return func(c *Container) error {

c.sv = s

return nil

}

}

func Consumer(resolverFns ...func(*Container) error) Container {

func Consumer(resolverFns ...ResolverFuncType) Container {

var containerWithValue Container

for _, resolverFn := range resolverFns {

if err := resolverFn(&containerWithValue); err != nil {

return Container{}, err

}

}

return containerWithValue

}

TestResolver (t.esafasf) {

containerWithIntValue := Consumer(ResolverFuncMakerA(3))

containerWithStringValue := Consumer(ResolverFuncMakerB("three"))

}
```

---

# [go][pattern] singleton

sync.Once 객체를 사용한다.

sync.Once는 단 한번만 실행할 수 있는 Do() 메소드를 제공하는 객체이다.

```go
var once sync.Once

var instance type

func GetSingletonInstance() *type {

once.Do(func() {

instance = initialize()

})

return &instance

}
```

---

# [go][orm] ent

<https://entgo.io/docs/getting-started/>

<https://github.com/ent/ent>

<https://seongmin.dev/develop-backend-with-golang-2-choose-orm>

<https://www.vompressor.com/entgo1/>

<https://velog.io/@leeeeeoy/Go-Ent-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0>

**1. ORM (Object-Relational Mapping)이란**

프로그래밍 언어 상의 Object와 데이터베이스의 Relation(Table)을 매핑하고
데이터베이스에 대한 Action(CRUD 등)을 추상화 하여 코드상에서 편리하게
데이터베이스를 사용할 수 있도록 하는 구조화하는 것 또는 구현체(API 나
Component 등)

**2. ent. 란?**

An entity framework(ORM) for Go

Go 언어용 ORM을 생성하고 관리할 수 있는 framework으로 ORM용 소스를
자동으로 생성해주는 것이 큰 특징이다.

Facebook Connectivity 팀의 멤버에 의해서 개발 되고 있으며, 최근 Go
언어용 ORM 도구로 큰 호응을 얻고 있다.

**ent. 개발의 [Motivation
(Principal)](https://entgo.io/blog/2019/10/03/introducing-ent/)**

-   Schema As Code - 유형, 관계 및 제약 조건을 정의하는 것은 Go
    코드(구조 태그가 아닌)여야 하며 CLI 도구를 사용하여 검증해야 합니다.

-   Statically typed and explicit API using codegen - interface{}를
    사용하는 모든 API는 개발자의 능률에 부정적인 영향을 끼친다.

-   Queries, aggregations and graph traversals should be simple -
    개발자는 원시 SQL 쿼리나 SQL 용어를 다루지 않습니다.

-   Full support for context.Context - trace 및 log 시스템을 완벽하게
    파악하는 데 도움이 되며 cancellation와 같은 다른 기능에 중요합니다.

-   Storage agnostic - 템플릿을 이용하여 코드를 생성하는 방식으로
    스토리지 계층을 동적으로 유지함으로써, DATABASE Program이 바뀌어도
    쉽게 대응할 수 있습니다.

**# 고려할 점**

table schema를 sql로 직접 생성 했을 때, Type에 각 DB에 존재하는 상세
타입으로 명시할 수 있다.

agentdb=> \d node

Table "public.node"

Column \

-----------+-------------------
-----+-----------+----------+---------

node_name \

pods_info \

timestamp \

Indexes:

"node_pkey" PRIMARY KEY, btree (node_name)

ent를 사용하여 table schema를 생성 했을 때, 동작에는 문제가 없지만 상세
데이터 타입을 명시할 수 없다. ent의 코드에서 문자열은 모두 String()으로
지정되기 때문이다.

nodedb=# \d+ node

Table "public.node"

Column \

-----------+--------------
-----+-----------+----------+---------

node_name \

timestamp \

pod_info \

Indexes:

"node_pkey" PRIMARY KEY, btree (node_name)

**3, 사용 방법**

**# 작업 순서**

1, 사용할 Database 생성 (ent orm은 TABLE 부터 생성/관리가 가능하므로
database 는 미리 생성해 두어야 함)

2, ent orm 용 초기 디렉토리 구조 생성

3, ent orm 소스를 생성하기 위한 schema 생성

  (option) 3-1, schema go 소스를 직접 구현

  (option) 3-2, entimport 도구를 이용하여 기존의 database로 부터 schema
소스 자동 생성

4, ent generate 명령을 실행하여 orm 소스 생성

5, 생성된 orm 소스 테스트

**# 작업 예제**

nodedb.sql

CREATE ROLE nodedb WITH PASSWORD 'nodedb' LOGIN CREATEDB
REPLICATION;

DROP DATABASE IF EXISTS nodedb;

CREATE DATABASE nodedb OWNER 'nodedb';

nodedbwithtable.sql

CREATE ROLE nodedb WITH PASSWORD 'nodedb' LOGIN CREATEDB
REPLICATION;

DROP DATABASE IF EXISTS nodedb;

CREATE DATABASE nodedb OWNER 'nodedb';

\c nodedb

CREATE TABLE node (

node_name     VARCHAR(255) PRIMARY KEY,

pods_info     TEXT NOT NULL,

timestamp    VARCHAR(25) NOT NULL

);

**## postgresql 서비스 실행**

$ sudo docker run -d --name postgresql15 --restart unless-stopped \

-p 5432:5432 \

-e POSTGRES_PASSWORD=postgres \

-v ${PWD}/data:/var/lib/postgresql/data \

-v ${PWD}/share:/tmp/share \

postgres:15.1

$ docker ps

$ docker exec -it <container> bash

**## postgresql에 user, db, table 생성**

$ psql -h localhost -U postgres -a -f /tmp/share/nodedb.sql

$ psql -h localhost -U nodedb

nodedb=> \d+ node

**## go module 생성**

$ mkdir todo

$ cd $_

$ go mod init todo

**## ent download**

$ go get entgo.io/ent/cmd/ent

**## make schema dirs**

syntax$ go run [-mod=mod] entgo.io/ent/cmd/ent init [--target
<path to schema dir>] [entity ...]

**## make empty schema dir (entimport 사용시 사용)**

하위 경로(ent/scheme)가 생성됨

$ go run entgo.io/ent/cmd/ent init

**## make initial entity schema (schema 직접 구현시 사용)**

하위 경로(ent/scheme) 경로와 entity로 지정한 schema 파일이 생성

주의) entity의 첫번째 문자는 대문자 이어야만 한다

$ go run entgo.io/ent/cmd/ent init --target ent/scratch/schema Person
Car Customer

**## (option) entity의 schema를 직접 구현**

node entity 구현의 예)

```go
package schema

import (

"entgo.io/ent"

"entgo.io/ent/dialect/entsql"

"entgo.io/ent/schema"

"entgo.io/ent/schema/field"

)

type Node struct {

ent.Schema

}

func (Node) Fields() []ent.Field {

return []ent.Field{

field.String("id").StorageKey("node_name"),

field.String("timestamp"),

field.String("pod_info"),

}

}

func (Node) Edges() []ent.Edge {

return nil

}

func (Node) Annotations() []schema.Annotation {

return []schema.Annotation{entsql.Annotation{Table: "node"}}

}
```

**## entimport 다운로드**

entimport? postgresql의 기존 database 내용을 이용해 ent에 맞는 schema
파일을 자동으로 생성해 주는 도구

$ go get ariga.io/entimport/cmd/entimport

**## entimport 이용하여 기존 postgresql database에 맞는 ent용 schema
자동 생성**

syntax$ go run ariga.io/entimport/cmd/entimport -dsn
"postgres://<id>:<password>@<host>:<port>/<database>?sslmode=disable"

$ go run ariga.io/entimport/cmd/entimport -dsn
"postgres://postgres:postgres@localhost:5432/nodedb?sslmode=disable"

**## ent orm 코드 생성**

$ go generate ./ent

go generate? 지정한 명령 실행 또는 지정한 dir 이하에서 generate.go
파일을 참조하여 내부에 명시된 명령 실행. 어떠한 프로세스도 실행될 수
있으나 generate command의 의도는 go source 파일의 생성 또는
업데이트이다.

**@ 테스트 코드**

agentdb_test.go

```go
package test

import (

"context"

"log"

"math/rand"

"testing"

"time"

"entsampler/ent" // generated package for ent

_ "github.com/lib/pq" // SHOULD import postgresql driver

)

func init() {

rand.Seed(time.Now().UnixNano())

}

func makeRandStr(n int) string {

letterRunes :=
[]rune("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ")

b := make([]rune, n)

for i := range b {

b[i] = letterRunes[rand.Intn(len(letterRunes))]

}

return string(b)

}

func getClient() *ent.Client {

client, err := ent.Open("postgres", "host=localhost port=5432
user=nodedb password=nodedb dbname=nodedb sslmode=disable")

if err != nil {

log.Fatalf("failed connecting to postgresql: %v", err)

}

//defer client.Close()

// Run the auto migration tool.

ctx := context.Background()

if err = client.Schema.Create(ctx); err != nil {

log.Fatalf("failed creating schema resources: %v", err)

}

return client

}

func TestCreateNode(t *testing.T) {

ctx := context.Background()

client := getClient()

defer client.Close()

n, err := client.Node.

Create().

SetID(makeRandStr(10)).

SetTimestamp("11:11:11").

SetPodInfo("mypod").

Save(ctx)

if err != nil {

t.Error("failed creating node: ", err)

}

t.Log("node was created: ", n)

}

func TestReadNode(t *testing.T) {

ctx := context.Background()

client := getClient()

defer client.Close()

nodes, err := client.Node.Query().All(ctx)

if err != nil {

t.Error("failed read node: ", err)

}

for _, n := range nodes {

log.Printf("%v\n", n)

}

}
```
