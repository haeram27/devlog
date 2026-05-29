# [go] signal handling

signal 캐치하기

```go
package main

import "fmt"

import "os"

import "os/signal"

import "syscall"

func main() {

quit := make(chan os.Signal, 1)

signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

<-quit

fmt.Println("to confirm shutdown, please send interrupt again")

<-quit

fmt.Println("terminating application")

apps.Close() // graceful shutdown, remove resources of application

fmt.Println("Bye")

}
```

캐치된 signal 확인하기

```go
package main

import "fmt"

import "os"

import "os/signal"

import "syscall"

func main() {

sigs := make(chan os.Signal, 1)

done := make(chan bool, 1)

signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)

go func() {

sig := <-sigs

fmt.Println()

fmt.Println(sig)

done <- true

}()

fmt.Println("awaiting signal")

<-done

fmt.Println("exiting")

}
```

---

# [go] channel

[참고](https://velog.io/@moonyoung/golang-channel-with-select-%ED%97%B7%EA%B0%88%EB%A6%AC%EB%8A%94-%EC%BC%80%EC%9D%B4%EC%8A%A4-%EC%A0%95%EB%A6%AC%ED%95%98%EA%B8%B0)

**<> channel 이란?**

channel은 기본적으로 1개의 저장공간을 갖는 blocking queue(FIFO) 구현체
이다.

단, 생성시 옵션을 통해 추가 buffer 사이즈를 지정 할 수 있다.

Blocking queue란 queue의 데이터 상태에 따라 producer와 consumer thread를
wait 시키는 queue이다.

consumer thread가 queue의 item을 take()할 때 queue가 비어 있다면
consumer thread가 wait상태가 되도록함.

producer thread가 삽입시 queue가 이미 가득찬 경우 공간이 생길때 까지
producer thread가 wait 상태가 된다.

**<> channel 용도**

1, go routine 간 데이터 공유

2, go routine 간 이벤트 송수신

3, go routine waiting(대기) 상태 만들기

channel은 goroutine의 실행을 일시 정지(wait) 시키는 blocking queue이므로
사용시 기본적으로 channel에 송신하는 Producer goroutine과 channel로 부터
수신을 하는 consumer goroutine의 역할을 하는 두 개의 groutine 사이에서
메시지(task, response 등) 전달 용도로 사용하는 것이 올바르다.

채널 송신 코드 실행 시 채널 버퍼에 element를 삽입한 후 버퍼가 꽉차면
채널 버퍼에 여유가 생길 때 까지 송신측 고루틴이 정지된다.(goroutine
blocking)

채널 수신 코드 실행 시 채널 버퍼에 element가 있을 때 까지 수신측
고루틴이 정지 된다.(goroutine blocking)

**<> channel 생성 방법**

#1, make(chan <type>[, <size>])

channel을 생성하기 위한 가장 기본적인 방법 channel 인스턴스를 반환한다.

channel은 별도의 <size>지정이 없거나 0으로 설정하면 capacity가 0이지만
실제 저장공간 사이즈는 1인 채널이 생성된다.

channel은 golang에서 len/cap() 함수를 사용할 때 매우 혼란을 주는
자료구조 이다.

왜냐하면 channel의 실제 저장 공간 사이즈는 cap(channel) + 1 사이즈 이기
때문이다.

#2, new(chan <type>)

nil channel(저장 공간의 크기가 0인 실제로는 사용할 수 없는 channel)을
생성하고 그 포인터를 반환한다.

new 키워드로 생성한 nil channel에 대해 push나 pop action을 실행 하는
경우 go routine이 waiting 상태가 되며, waiting을 해제할 방법도 없다. nil
channel의 경우 사실상 사용처가 없다.

**<> channel 실제 크기와 명목 크기: 실체 크기 == 명목크기+1**

```go
func TestChannelSize(t *testing.T) {

ch := make(chan int) //== make(chan int,0)

t.Log(cap(ch)) // 0

t.Log(len(ch)) // 0

go func() {

time.Sleep(1 * time.Second)

t.Log("cap: ", cap(ch)) // 0

t.Log("len: ", len(ch)) // 0

t.Log("rcv: ", <-ch)    // 1, release main goroutine

}()

ch <- 1  // block main goroutine

t.Log("BYE")

}
```

위 예제에서 보듯이 <size>가 0인 채널을 생성 했고 cap()과 len()이 모두
0이지만 해당 channel에는 1개 데이터의 push가 가능하다. 1개 데이터를
추가한 channel에 대해 cap()과 len()을 다시 체크해 보아도 그 값은 0
이지만 channel에서는 1개 데이터를 빼는 것이 가능하다. 이 현상이 channel
관련 함수에 대한 혼란을 가져온다. <size>를 0으로 하여 생성한
channel에는 사실 내부에 1개의 저장공간이 있는 것이다.

nil이 아닌 정상 channel의 실제 저장 공간 크기는 "cap(channel)+1" 이다.

make(chan int, <size>)의 <size>, cap(channel)과 len(channel) 함수로
부터 출력되는 값은 channel의 "명목 크기"라고 하고 "명목크기+1" 값을
채널의 "실제 크기"라고 정의하자

이렇듯 "make(chan int, <size>)" 구문으로 channel을 생성하는 경우
여기서 <size>의 실제 의미는 channel에 추가할 버퍼 사이즈(<size of
buffer to be added>)라고 생각하는 것이 올바르다. "make(chan int,
<size>)" 구문으로 생성되는 channel에는 실제로 <size>+1 만큼의
데이터를 삽입할 수 있다. 다시말해 <size>에 0을 넣으면 cap()으로 부터
출력되는 명목 값은 <size> 값인 0 이지만 실제 channel이 가진 cap은
1이다. 다만 실제 capacity(명목값 +1) 만큼 데이터가 삽입되면 되면 삽입
코드를 실행하는 goroutine 이 wait이 걸리게 된다.(wait이 되었다는 것은
이미 channel에 cap(channel)+1 사이즈의 데이터가 삽입된 상태 라는 것)

**<> channel에 의해 실행중인 go routine에서 waiting이 발생하는 경우:**

1, push(chan<-) waiting

channel에 data push action에 의해 channel에 data가 push된 직후 buffer에
여유 공간이 없는 경우에 현재 goroutine은 waiting 상태가 된다.

push waiting이 풀리려면 chan의 버퍼에 삽입 할 수 있는 여유 공간이 생겨야
하고 이는 channel에 대한 pop 동작에 의해 가능하다.

2, pop(<-chan) waiting

channel내 buffer에 대해 pop action을 시도 했을 때 pop 할 수 있는
element가 없는 경우에 현재 goroutine은 waiting 상태가 된다.

pop waiting이 풀리려면 chan의 버퍼에 pop 할 수 있는 element가 생겨야
하고 이는 다른 go routine에서 channel에 대한 push 동작에 의해 가능하다.

**<> closed channel**

```go
c := make(chan int, 3)

close(c)
```

channel을 close한다는 것은 채널에 element push를 금지 한다는 의미(panic
발생)이다.

channel을 close하면 해당 channel은 무한히 pop만 가능한 상태가 된다.

close 된 channel에 대해 push를 하면 pannic이 발생한다.

close 된 channel의 경우 pop 실행시 go routine이 wait 상태로 전환되지
않으며 그러므로 호출 횟수나 element의 존재 여부와 관계없이 pop을 호출할
수 있다.

close 된 channel에서 pop을 하면 channel내에 element이 있는 경우
element를 반환하고 element가 없는 경우 channel이 갖는 element type의
default 값(예-int의 경우 0)이 반환된다.

closed된 channel에 대해 무한 pop 호출을 할 수 있으므로 closed된
channel에 대해 range를 사용하여 pop을 하는 경우

for-range를 사용하여 channel로 부터 수신하는 경우 송신측(channel에 push
하는 goroutine 측)에서는 모든 데이터를 push 한 후에 반드시 channel을
close해 주어야만 한다.

range 문을 사용해 channel을 pop하는 경우, channel의 상태가 "non-closed
and non-empty" 상태인 경우 channel로 부터 pop을 수행하고 "non-closed
and empty" 상태인 경우 pop action이 blocking(deadlock)되며, "closed
and empty"가 되면 for loop를 escaping한다.

수신측이 for-range를 사용하지 않는다면 close(ch) 사용 여부를 고민해야
한다.

수신측이 for-range를 사용하지 않는다면 close(ch)를 사용하지 않을 수
있다.

반대로 for-range를 사용하지 않는 수신측에 blocking이 걸리지 않게 하려는
경우 close(ch)를 사용할 수 있다.

"closed and empty" 상태의 channel로 부터 수신은 무한 횟수로 default
값을 pop할 수 있으며 pop 성공 여부는 false이다.

close(ch)는 수신측에서 range <channel> 문법을 사용할 때 반드시 그
사용이 필요하지만 .

range가 아닌 for문으로 channel를 일정 횟수나 무한 pop하는 경우 close된
channel로 부터 default값 이벤트를 수신할 수 있으므로 두번째 bool
반환값(pop 성공여부)를 반드시 체크해야 한다.

**\
### channel에서 pop 하기

**pop#1, <-chan (pop expression) 사용하여 pop하기**

channel로 부터 element를 하나씩 pop할 때 사용한다.

보통 channel의 buffer가 1개 이하 일때 사용한다.

syntax)

  -----------------------------------------------------------------------
  v, ok := <-ch\
  v := <-ch
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

v : channel의 element type, chnannel element의 값

ok : bool, pop 성공 여부, 생략 가능,

송신 측에서 close(ch) 사용하고 수신측이 range가 아닌 방식으로 수신하는
경우 반드시 체크 해야 함.

ok 변수 값이 true로 반환 된 경우, channel에 pop 할 수 있는 element가
존재 했으며 정상적으로 element pop 했음을 의미

ok 변수 값이 false로 반환 된 경우, channel에 pop 할 수 있는 element가
존재 하지 않았으며 element 값(v)에는 element type의 기본값이 반환됨

```go
func TestChannelPop(t *testing.T) {

ch := make(chan int)

go func() {

for i := 0; i < 2; i++ {

ch <- i

t.Log("send: ", i)

}

close(ch)

}()

for cnt := 0; cnt < 10; cnt++ {

v, ok := <-ch

if ok != false {

t.Log("recv: ", v, ", ", ok)

}

}

t.Log("finish")

}
```

=== RUN TestChannelPop

/channel_test.go:35: send: 0

/channel_test.go:35: send: 1

/channel_test.go:42: recv: 0 , true

/channel_test.go:42: recv: 1 , true

/channel_test.go:45: finish

--- PASS: TestChannelPop (0.00s)

PASS

ok gosampler/test 0.004s

**pop#2, range 사용하여 pop 하기**

goroutine context change 없이 channel로 부터 buffer 사이즈 만큼
element를 반복하여 pop할 때 사용한다.

channel의 버퍼는 channel에 push/pop할 때 마다 발생하는 goroutine context
change의 횟수를 줄임으로써 performance를 향상 시키기 위해서 사용한다.

보통 channel의 buffer size가 1 이상인 channel에 대해 range를 사용할 수
있다.

channel의 buffer가 1 이상인 go는 경우 최선으로 한번에 최대 buffer
사이즈만큼의 횟수로 push 또는 pop action을 goroutine context change 없이
연속 실행해 준다.

  -----------------------------------------------------------------------
  for v := range ch {}
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

for-range를 사용하는 경우 **송신측(channel에 push 하는 goroutine
측)에서** 모든 데이터 push를 완료하고 더 이상 push할 데이터가 없다면
**반드시 channel을 close**해 주어야만 한다.

range 문을 사용해 channel을 pop하는 경우, channel의 상태가 "non-closed
and non-empty" 상태인 경우 channel로 부터 pop을 수행하고 "non-closed
and empty" 상태인 경우 pop action이 blocking(새 데이터가 올때까지
무기한 대기)되며, "closed and empty"가 되면 for loop를 escaping한다.

for-range 구문에서 channel을 사용하는 것는 무한 루프에서 channel을
사용하는 것과 같다.

다음 세가지 구문은 모두 같은 역할을 수행한다.

```go
receiver1 := func(name string) {

defer log.Printf(" [%s] out", name)

for msg := range msgs {

log.Printf(" [%s] %d", name, msg)

}

}

go receiver1("receiver1")
```

```go
receiver2 := func(name string) {

defer log.Printf(" [%s] out", name)

for {

msg := <-msgs

log.Printf(" [%s] %d", name, msg)

}

}

go receiver2("receiver2")
```

```go
receiver3 := func(name string) {

defer log.Printf(" [%s] out", name)

for {

select {

case msg := <-msgs:

log.Printf(" [%s] %d", name, msg)

}

}

}

go receiver3("receiver3")
```

range를 사용하나 <- 기호를 사용하나 channel로 부터 element 하나씩을
가져오게된다.

"for msg := range msgs"

구문은 for문 안에서

"msg := <- msgs"

를 실행하는 것과 같다.

close(ch)는 수신측에서 range를 이용하여 channel을 수신하는 경우에만
사용한다.

close(ch)는 수신측에서 range의 종료 지점을 지정하기 위해서 사용되는
문법이다.

range가 아닌 for문으로 channel를 일정 횟수나 무한 pop하는 경우 close된
channel로 부터 무한 default값을 수신하게 된다.

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

ch := make(chan int)

go func() {

for i := 0; i < 1000; i++ {

ch <- i

t.Log("send: ", i)

}

close(ch)

}()

for v := range ch {

t.Log("recv: ", v)

}

t.Log("finish")

}
```

**<> select 문에서 channel 사용**

select는 여러 수신 channel 중에서 가장 먼저 도착하는 이벤트 하나를
대기하기 위해서 사용된다.

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

check.5, select문의 case는 channel에 대한 대기상태(blocking)가 가능하다.

check.6, case에 함수가 있다면 그 함수가 끝난 후, 다른 case를 검사한다.

예제. range는 channel이 close되어야 끝난다.

channel을 close해주지 않는다면 영원히 기다리면서 deadlock이 발생한다.

```go
ch := make(chan int, 1)

ch <- 101

for value := range ch {

fmt.Println(value)

}
```

Output:

101

fatal error: all goroutines are asleep - deadlock!

예제. close된 channel에 send 할 수 없다.

```go
ch := make(chan int)

close(ch)

ch <- 1 // panic: send on closed channel
```

예제. close된 channel에 send 할 수는 없지만 receive 할 수는 있다.

버퍼에 저장된 값이 있다면 읽고 값이 없다면 type의 default value를
가져온다.

```go
ch := make(chan int, 2)

var wg sync.WaitGroup

wg.Add(1)

go func() {

ch <- 10

ch <- 11

wg.Done()

}()

wg.Wait()

close(ch)

fmt.Println(<-ch) // 10

fmt.Println(<-ch) // 11

fmt.Println(<-ch) // 0
```

예제. select문의 case는 channel에 대한 대기상태(wait())가 가능하다.

select는 무한루프가 아니기 때문에 for-select 형식으로 사용된다.

하지만, case가 발생할 때까지 대기한다.

```go
ch := make(chan int)

takeSomeTime := func() {

go func() {

time.Sleep(time.Second * 2)

ch <- 1

}()

}

start := time.Now()

takeSomeTime()

select {

case <-ch:

fmt.Println(time.Since(start)) // 2.004948752s

}
```

예제. case에 함수가 있다면 그 함수가 끝난 후, 다른 case를 검사한다.

select {

case <-ch:

case ch <- doSomething:

}

---

# [go][channel] send messages with multiple goroutine

실행:

$ go run sendmsgwithchan.go

종료:

terminal에서 ctrl+c 키 입력(SIGINT signal)시 프로그램 종료

<sendmsgwithchan.go>

```go
package main

import (

"context"

"fmt"

"log"

"os"

"os/signal"

"sync/atomic"

"syscall"

"time"

)

func main() {

ctx, shutdown := context.WithCancel(context.Background())

msgs := make(chan uint64, 100)

var msgNumber atomic.Uint64

receiver := func(name string) {

defer log.Printf(" [%s] out", name)

for msg := range msgs {

log.Printf(" [%s] %d", name, msg)

}

// for {

//         msg := <-msgs

//         log.Printf(" [%s] %d", name, msg)

// }

// for {

//         select {

//         case msg := <-msgs:

//                 log.Printf(" [%s] %d", name, msg)

//         }

// }

}

go receiver("r(1)")

go receiver("r(2)")

go receiver("r(3)")

sender := func(name string) {

defer log.Printf("[%s] out", name)

timeout := time.NewTimer(time.Second * 5)

// timeoutAfter := time.After(time.Second * 5)

repeat := time.NewTimer(time.Second * 2)

for {

select {

case <-repeat.C:

log.Printf("[%s] timer.repeat", name)

repeat.Reset(time.Second)

case <-timeout.C:

// case <-timeoutAftera:

log.Printf("[%s] timer.timeout", name)

return

case <-ctx.Done():

log.Printf("[%s] context done", name)

return

default:

msgs <- msgNumber.Add(1)

time.Sleep(time.Second)

}

}

}

go sender("s(1)")

go sender("s(2)")

go sender("s(3)")

log.Printf(" [*] Waiting for logs. To exit press CTRL+C")

quit := make(chan os.Signal, 1)

signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

<-quit

shutdown()

/ for graceful shutdown /

if true {

log.Printf(" [*] Application is Interrupted. Please Wait 3
Second for graceful shutdown")

<-time.After(time.Second * 3)

} else {

log.Printf(" [*] Application is Interrupted. To exit press
CTRL+C")

<-quit

}

fmt.Println("Bye")

}
```

---

# [go] goroutine

**<> life cycle**

goroutine은 thread가 아니다.

goroutine은 어플리케이션 레벨에서 컨트롤 되는 실행 흐름으로써 OS에 의해
조절되는 thread와 달라서

모든 goroutine은 main() 함수가 종료될 때(프로그램의 종료) 함께 종료된다.

(goroutine은 모두 java의 daemon thread와 같은 life cycle을 갖는다.)

goroutine의 이러한 특성은 다른 언어들에서 존재하는 thread에 의한
memmory-leak 생성을 방지한다.

main 함수 이외에 다른 함수들은 goroutine의 life-cycle과 아무런 관련이
없다.

goroutine을 실행한 함수가 종료되어도 goroutine은 종료되지 않는다.

**<> runtime**

**runtime.NumGoroutine() int**

현재 존재하는 goroutine의 수를 반환

**runtime.NumCPU() int**

cpu 전체 thread 수(전체 core 수 x core당 thread수) 반환

cpu 전체 core 수 아님에 주의

**runtime.GOMAXPROCS(n int) int**

goroutine에 사용되는 cpu의 thread 수를 지정

default는 runtime.NumCPU()의 값과 같으므로 이 함수는 주로 제한된 thread
사용이 필요할 때 사용

runtime.GOMAXPROCS(runtime.NumCPU()/2)

n이 1보다 작으면 현재 설정을 변경하지 않음

**runtime.GoSched()**

현재 goroutine의 실행을 대기상태로 변경하고 다른 goroutine에게 실행
흐름을 양보

**runtime.Goexit()**

이 함수를 호출한 goroutine을 종료하고 다른 goroutine들에는 영향을 끼치지
않음

종료 전에 모든 deferred 호출을 실행함

main() 함수에서 호출 하지 말 것

일반적으로 프로세스는 main 함수가 return되면 실행중인 모든 goroutine을
exit 시키고 프로그램을 종료함

만약 main() 함수에서 Goexit 호출하면 main 함수는 return 없이 main
goroutine만 종료되며 다른 실행중인 goroutine은 영향을 받지 않고 계속
실행됨. main 함수가 return 되지 않고 모든 goroutine이 종료되면 그
프로그램은 crash 발생

---

# [go][sync] Mutex 와 Goshched

sync package

1. [Mutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35)
: 상호배제/ 여러 스레드에서 공유되는 데이터를 보호할때 사용

2.
[RWMutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/02)
: 읽기/쓰기 동작을 나누어서 Lock 을 걸수 있음

3.
[Cond](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/03) :
조건 변수. 대기하고 있는 하나의 객체를 깨울 수도 있고 여러개를 동시에
깨울수도 있다.

4.
[Once](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/04) :
특정 함수를 딱 한번만 실행할때

5.
[Pool](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/05) :
멀티 스레트(고루틴)에서 사용할 수 있는 객체의 풀.

자주 사용하는 객체를 풀에 보과냈다가 다시 사용

6.
[W
aitGroup](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/06)
: 고루틴이 모두 끝날 때까지 기다리는 기능

7.
[Atomic](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/07)
: 원자적 연산이라고도 하며 더이상 쪼갤수 없는 연산.

멀티 스레드, 멀티코어 환경에서 안전하게 값을 연산하는 기능

<https://go.dev/tour/concurrency/9>

<https://gobyexample.com/mutexes>

<https://mingrammer.com/gobyexample/mutexes/>

공유 데이터에 한번에 하나의 Goroutine만 접근하도록 하기위해서 Mutext를
사용한다.

* runtime.Gosched() - 다른 고루틴으로 제어권을 넘깁니다.

```go
var sharedData = []int{} // int 슬라이스 생성

go func(){

for i:=0; i<1000; i++{

data = append( sharedData, 1)

runtime.Gosched() // 다른 고루틴이 CPU를 사용할 수 있도록 양보

}

}()

go func(){

for i:=0; i<1000;i++ {

data = append( sharedData, i)

runtime.Gosched()

}

}()

time.Sleep( 2 * time.Second)

fmt.Println( len(sharedData))
```

import sync

sync.Mutex

func(m *Mutex) Lock() - 뮤텍스 잠금

func(m *Mutex) Unlock() - 뮤텍스 잠금 해제

```go
var sharedData = []int{} // int 슬라이스 생성

var mutex sync.Mutex

go func(){

for i:=0; i<1000; i++{

mutex.Lock()

data = append( sharedData , 1)

mutex.Unlock()

runtime.Gosched() // 다른 고루틴이 CPU를 사용할 수 있도록 양보

}

}()

go func(){

for i:=0; i<1000;i++ {

mutex.Lock()

data = append( sharedData, i)

mutex.Unlock()

runtime.Gosched()

}

}()

time.Sleep( 2 * time.Second)

fmt.Println( len(sharedData))
```

---

# [go][sync] RWLock

sync package

1. [Mutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35)
: 상호배제/ 여러 스레드에서 공유되는 데이터를 보호할때 사용

2.
[RWMutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/02)
: 읽기/쓰기 동작을 나누어서 Lock 을 걸수 있음

3.
[Cond](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/03) :
조건 변수. 대기하고 있는 하나의 객체를 깨울 수도 있고 여러개를 동시에
깨울수도 있다.

4.
[Once](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/04) :
특정 함수를 딱 한번만 실행할때

5.
[Pool](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/05) :
멀티 스레트(고루틴)에서 사용할 수 있는 객체의 풀.

자주 사용하는 객체를 풀에 보과냈다가 다시 사용

6.
[W
aitGroup](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/06)
: 고루틴이 모두 끝날 때까지 기다리는 기능

7.
[Atomic](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/07)
: 원자적 연산이라고도 하며 더이상 쪼갤수 없는 연산.

멀티 스레드, 멀티코어 환경에서 안전하게 값을 연산하는 기능

RWMutex는 Read와 Write의 Action을 구분하여 Lock을 사용하는 Mutex이다.

Read 중에 다른 Read는 할 수 있지만 Write 액션은 할 수 없음

Write 중에 다른 Read와 Write 액션 모두 할 수 없음

Read Critical Section 생성 함수 :

sync.RWMutex.RLock();

sync.RWMutex.RUnlock();

Rlock()과 Runlock()은

Read action을 위한 Read critical section을 생성하는 함수.

하나의 Read critical section이 진입 되었다면 다른 위치에 동일 Lock으로
구현된 Read critical section도 여전히 진입이 가능하다. 하지만 동일
Lock으로 구현되는 Write critical section들은 진입이 불가능해진다.

사실상 RLock()은 Write critical section 실행만을 막는 Lock이다.

Write Critical Section 생성 함수 :

sync.RWMutex.Lock();

sync.RWMutex.Unlock();

Lock()과 Unlock()은

Write action을 위한 Write critical section을 생성하는 함수.

하나의 Write Critical Section이 진입된 상황에서는 다른 위치에 동일
Lock으로 구현된 Read 또는 Write Critical Section은 모두 진입될 수 없다.

RWLock 미사용시

```go
var data int = 0

go func(){

for i:=0; i<3; i++{

data += 1

fmt.Println("write:" , data)

time.Sleep( 10 * time.Millisecond)

}

}()

go func(){

for i:=0 ; i <3 ; i++ {

fmt.Println("read 1 " , data)

time.Sleep( 1 * time.Second)

}

}()

go func(){

for i:=0 ; i <3 ; i++ {

fmt.Println("read 2 " , data)

time.Sleep( 2 * time.Second)

}

}()

time.Sleep( 10*time.Second)
```

read 1 1

write: 1

read 2 1

write: 2

write: 3

read 1 3

read 2 3

read 1 3

read 2 3

RWLock 사용시

```go
var rwMutex = new(sync.RWMutex)

var data int = 0

go func(){

for i:=0; i<3; i++{

rwMutex.Lock() // 쓰기 뮤텍스 잠금, 쓰기 보호 시작

data += i

fmt.Println("write:" , data)

time.Sleep( 10 * time.Millisecond)

rwMutex.Unlock() // 쓰기 뮤텍스 잠금 해제, 쓰기 보호 종료

}

}()

go func(){

for i:=0 ; i <3 ; i++ {

rwMutex.RLock() // 읽기 뮤텍스 잠금, 읽기 보호 시작

fmt.Println("read 1 " , data)

time.Sleep( 1 * time.Second)

rwMutex.RUnlock() // 읽기 뮤텍스 잠금 해제, 읽기 보호 종료

}

}()

go func(){

for i:=0 ; i <3 ; i++ {

rwMutex.RLock() // 읽기 뮤텍스 잠금, 읽기 보호 시작

fmt.Println("read 2 " , data)

time.Sleep( 2 * time.Second)

rwMutex.RUnlock() // 읽기 뮤텍스 잠금 해제, 읽기 보호 종료

}

}()

time.Sleep(10*time.Second)
```

write: 0\
read 2 0\
read 1 0\
write: 1\
read 2 1\
read 1 1\
write: 3\
read 2 3\
read 1 3

---

# [go][sync] Cond

sync package

1. [Mutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35)
: 상호배제/ 여러 스레드에서 공유되는 데이터를 보호할때 사용

2.
[RWMutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/02)
: 읽기/쓰기 동작을 나누어서 Lock 을 걸수 있음

3.
[Cond](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/03) :
조건 변수. 대기하고 있는 하나의 객체를 깨울 수도 있고 여러개를 동시에
깨울수도 있다.

4.
[Once](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/04) :
특정 함수를 딱 한번만 실행할때

5.
[Pool](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/05) :
멀티 스레트(고루틴)에서 사용할 수 있는 객체의 풀.

자주 사용하는 객체를 풀에 보관했다가 다시 사용

6.
[W
aitGroup](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/06)
: 고루틴이 모두 끝날 때까지 기다리는 기능

7.
[Atomic](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/07)
: 원자적 연산이라고도 하며 더이상 쪼갤수 없는 연산.

멀티 스레드, 멀티코어 환경에서 안전하게 값을 연산하는 기능

조건변수(conditional variable) :

조건변수는 wait 상태인 고루틴을 하나씩 또는 모두를 동시에 깨울 때 사용

보통 고루틴 pool을 만드는데 사용한다.

​

- sync.Cond

- func NewCond(I Locker) *Cond: 조건변수 생성

- func (c *Cond) Wait() - 고루틴 실행을 멈추고 대기

- func (c *Cond) Signal() - 대기하고 있는 고루틴 하나만 깨움

- func (c *Cond) Broadcase() - 대기하고 있는 고루틴을 전부 깨움

```go
var mutex = new(sync.Mutex)

var cond = sync.NewCond( mutex)

c := make(chan bool, 3)

for i:=0; i<3; i++{

go func(n int){

mutex.Lock()

c<-true

fmt.Println("wait begin: " , n)

cond.Wait() // 조건 변수 대기

fmt.Println("wait end : " , n)

mutex.Unlock()

}(i)

}

for i:=0; i<3; i++{

<-c

}

for i:=0; i<3; i++{

mutex.Lock()

fmt.Println("signal: ",i)

cond.Signal() // 대기하는 고루틴을 하나씩 깨움

mutex.Unlock()

}
```

wait begin: 2

wait begin: 0

wait begin: 1

signal: 0

signal: 1

signal: 2

wait end : 2

wait end : 0

wait end : 1

* Wait()함수는 Lock, Unlock함수로 보호해야 함

​

Broadcast() 로 한번에 모두 깨우는 예제

```go
var mutex = new(sync.Mutex)

var cond = sync.NewCond( mutex)

c := make(chan bool, 3)

for i:=0; i<3; i++{

go func(n int){

mutex.Lock()

c<-true

fmt.Println("wait begin: " , n)

cond.Wait()

fmt.Println("wait end : " , n)

mutex.Unlock()

}(i)

}

for i:=0; i<3; i++{

<-c

}

mutex.Lock()

fmt.Println("broadcase")

cond.Broadcast()

mutex.Unlock()
```

wait begin: 2

wait begin: 0

wait begin: 1

broadcase

wait end : 2

wait end : 0

wait end : 1

---

# [go][sync] Once

sync package

1. [Mutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35)
: 상호배제/ 여러 스레드에서 공유되는 데이터를 보호할때 사용

2.
[RWMutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/02)
: 읽기/쓰기 동작을 나누어서 Lock 을 걸수 있음

3.
[Cond](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/03) :
조건 변수. 대기하고 있는 하나의 객체를 깨울 수도 있고 여러개를 동시에
깨울수도 있다.

4.
[Once](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/04) :
특정 함수를 딱 한번만 실행할때

5.
[Pool](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/05) :
멀티 스레트(고루틴)에서 사용할 수 있는 객체의 풀.

자주 사용하는 객체를 풀에 보과냈다가 다시 사용

6.
[W
aitGroup](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/06)
: 고루틴이 모두 끝날 때까지 기다리는 기능

7.
[Atomic](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/07)
: 원자적 연산이라고도 하며 더이상 쪼갤수 없는 연산.

멀티 스레드, 멀티코어 환경에서 안전하게 값을 연산하는 기능

sync.Once 객체를 사용한다.

sync.Once는 단 한번만 실행할 수 있는 Do() 메소드를 제공하는 객체이다.

sync.Once는 Singleton 객체를 만드는데 사용한다.

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

# [go][sync] Pool

sync package

1. [Mutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35)
: 상호배제/ 여러 스레드에서 공유되는 데이터를 보호할때 사용

2.
[RWMutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/02)
: 읽기/쓰기 동작을 나누어서 Lock 을 걸수 있음

3.
[Cond](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/03) :
조건 변수. 대기하고 있는 하나의 객체를 깨울 수도 있고 여러개를 동시에
깨울수도 있다.

4.
[Once](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/04) :
특정 함수를 딱 한번만 실행할때

5.
[Pool](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/05) :
멀티 스레트(고루틴)에서 사용할 수 있는 객체의 풀.

자주 사용하는 객체를 풀에 보과냈다가 다시 사용

6.
[W
aitGroup](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/06)
: 고루틴이 모두 끝날 때까지 기다리는 기능

7.
[Atomic](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/07)
: 원자적 연산이라고도 하며 더이상 쪼갤수 없는 연산.

멀티 스레드, 멀티코어 환경에서 안전하게 값을 연산하는 기능

- 풀은 객체를 사용한 후 보관했다가 다시 사용.

- 객체를 반복해서 할당하면메모리 사용량도 늘어나고, 메모리를 해제하는
가비지 컬렉토도 부담

- 메모리 할당과 해제 횟수를 줄여 설능을 높이고자 할때 사용

- 풀은 여러 고루틴에서 동시에 사용이 가능

​

- sync.Pool

- func (p *Pool) Get() interface{}: 풀에 보관된 객체를 가져옴

- func (p *Pool) Pub( x interface{}) : 풀에 객체를 보관

​

```go
type Data struct{

tag string

buffer []int

}

func pool(){

p := sync.Pool{ // 풀을 할당

New: func() interface{}{ // GET 함수사용으로 호출될 함수를 정의

data := new(Data)                 // 새 메모리 할당

data.tag = "new"                 // 태그 설정

data.buffer = make([]int, 10)         // 슬라이스 공간 할당

return data // 할당한 메모리(객체) 리턴ㄴ

},

}

// 고루틴을 10개 생성함

for i:=0; i<10; i++ {

go func(){

data := p.Get().(Data) // 풀에서 Data 타입으로 데이터
가져옴

for index := range data.buffer {

data.buffer[index] = rand.Intn(100) // 슬라이스에 랜덤값 저장

}

fmt.Println(data)

data.tag = "used" // 사용된 객체라는 증거로 값을 넣음

p.Put(data) // 풀에 객체를 보관

}()

}

for i:=0; i<10; i++ {

go func(){

data := p.Get().(*Data)

n := 0

for index := range data.buffer {

data.buffer[index] = n

n += 2

}

fmt.Println(data)

data.tag = "used"

p.Put(data)

}()

}

fmt.Scanln()

}
```

​

실행을 해보면 새로운 할당은 2번이고..

나머지는 할당된 데이터를 꺼내서 사용을 한것을 알수 있습니다.

&{new [81 87 47 59 81 18 25 40 56 0]}

&{used [94 11 62 89 28 74 11 45 37 6]}

&{used [95 66 28 58 47 47 87 88 90 15]}

&{used [41 8 87 31 29 56 37 31 85 26]}

&{used [13 90 94 63 33 47 78 24 59 53]}

&{used [57 21 89 99 0 5 88 38 3 55]}

&{used [51 10 5 56 66 28 61 2 83 46]}

&{used [63 76 2 18 47 94 77 63 96 20]}

&{used [23 53 37 33 41 59 33 43 91 2]}

&{used [78 36 46 7 40 3 52 43 5 98]}

&{used [0 2 4 6 8 10 12 14 16 18]}

&{used [0 2 4 6 8 10 12 14 16 18]}

&{new [0 2 4 6 8 10 12 14 16 18]}

&{used [0 2 4 6 8 10 12 14 16 18]}

---

# [go][sync] WaitGroup

sync package

1. [Mutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35)
: 상호배제/ 여러 스레드에서 공유되는 데이터를 보호할때 사용

2.
[RWMutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/02)
: 읽기/쓰기 동작을 나누어서 Lock 을 걸수 있음

3.
[Cond](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/03) :
조건 변수. 대기하고 있는 하나의 객체를 깨울 수도 있고 여러개를 동시에
깨울수도 있다.

4.
[Once](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/04) :
특정 함수를 딱 한번만 실행할때

5.
[Pool](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/05) :
멀티 스레트(고루틴)에서 사용할 수 있는 객체의 풀.

자주 사용하는 객체를 풀에 보과냈다가 다시 사용

6.
[W
aitGroup](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/06)
: 고루틴이 모두 끝날 때까지 기다리는 기능

7.
[Atomic](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/07)
: 원자적 연산이라고도 하며 더이상 쪼갤수 없는 연산.

멀티 스레드, 멀티코어 환경에서 안전하게 값을 연산하는 기능

- 대기그룹은 고루틴이 모두 끝날때까지 기다릴때 사용

- sync.WaitGroup

- func ( wg *WaitGroup) **Add**(delta int) - 대기그룹에 고루틴 개수
추가

- func ( wg *WaitGroup) **Wait**() - 모든 고루틴이 끝날 때 까지 기다림

- func ( wg *WaitGroup) **Done**() - 고루틴이 끝났다는 것을 알려줄 때
사용

**Add, Wait, Done 암기하자**

```go
wg := new(sync.WaitGroup) // 대기그룹 생성

for i:=0 ; i<10 ; i++{

wg.Add(1) // 반복할때 마다 wg.Add 함수로 1씩 추가

// 고루틴 10개 생성

go func(n int){

fmt.Println(n)

wg.Done() // 고루틴이 끝났다는 것을 알려줌

}(i)

}

wg.Wait() // 모든 고루틴이 끝날때 까지 기다림

fmt.Println("the end")
```

- 고루틴을 생성시 Add함수로 고루틴 개수를 추가

- 고루틴 안에서 Done함수를 사용하여 고루틴이 끝났다는 것을 알려줌

- Wait 함수를 사용하여 모든 고루틴이 끝날때까지 기다린다.

* Add함수에 설정한 값과 Done함수가 호출되는 횟수는 같아야 함.

---

# [go][sync] Atomic action

sync package

1. [Mutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35)
: 상호배제/ 여러 스레드에서 공유되는 데이터를 보호할때 사용

2.
[RWMutex](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/02)
: 읽기/쓰기 동작을 나누어서 Lock 을 걸수 있음

3.
[Cond](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/03) :
조건 변수. 대기하고 있는 하나의 객체를 깨울 수도 있고 여러개를 동시에
깨울수도 있다.

4.
[Once](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/04) :
특정 함수를 딱 한번만 실행할때

5.
[Pool](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/05) :
멀티 스레트(고루틴)에서 사용할 수 있는 객체의 풀.

자주 사용하는 객체를 풀에 보과냈다가 다시 사용

6.
[W
aitGroup](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/06)
: 고루틴이 모두 끝날 때까지 기다리는 기능

7.
[Atomic](https://pyrasis.com/book/GoForTheReallyImpatient/Unit35/07)
: 원자적 연산이라고도 하며 더이상 쪼갤수 없는 연산.

멀티 스레드, 멀티코어 환경에서 안전하게 값을 연산하는 기능

- 고루틴, CPU, 코어에서 같은 메모리를 수정할때 서로 영향을 받지 않고
안전하게 연산

- CPU명령어를 직접 사용하여 구현

​

```go
var data int32 = 0

wg := new(sync.WaitGroup)

for i:=0 ; i < 2000 ; i++{

wg.Add(1)

go func(){

data += 1

wg.Done()

}()

}

for i:=0 ; i < 1000 ; i++{

wg.Add(1)

go func(){

data -= 1

wg.Done()

}()

}

wg.Wait()

fmt.Println(data)
```

결과는 -> 1000이 아니고 997 등등..

분명히 1000이 나와야 하지만..

여러 변수에 고루틴이 동시에 접근하면서 정확하게 연산이 되지 않았음..

​

```go
var data int32 = 0

wg := new(sync.WaitGroup)

for i:=0 ; i < 2000 ; i++{

wg.Add(1)

go func(){

atomic.AddInt32( &data, 1)

wg.Done()

}()

}

for i:=0 ; i < 1000 ; i++{

wg.Add(1)

go func(){

atomic.AddInt32( &data, -1)

wg.Done()

}()

}

wg.Wait()

fmt.Println(data)
```

--> 결과 1000

​

sync/atomic 패키지에서 제공하는 원자적 연산으로 처리가 됨

​

- Add 계열: 변수에 값을 더하고 결과를 리턴

- CompareAndSwap 계역: 변수 AB를 비교하여 같으면 C를 대입, 같으면 true,
다르면 false를 리턴

- Load 계열: 변수에서 값을 가져옵니다.

- Store 계열: 변수에 값을 저장

- Swap 계열: 변수에 새 값을 대입하고 이전 값을 리턴
