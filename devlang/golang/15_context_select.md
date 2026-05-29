# [go] context

[Context isn't for cancellation | Dave
Cheney](https://dave.cheney.net/2017/08/20/context-isnt-for-cancellation)

<https://jaehue.github.io/post/how-to-use-golang-context/>

<https://www.sohamkamani.com/golang/context-cancellation-and-values/>

**<> context의 용도**

context는 두 가지 용도로 사용된다.

용도 1, context를 소유하는 모든 함수에서 context에 기록된 <key, value>
형태의 data를 공유할 수 있다.

용도 2, 같은 parent context를 가진 모든 go-routine 실행 흐름들을 동시에
종료(취소)할 수 있다.

context를 이용하여 취소 이벤트(context.Done() channel)를 발생시키는
방법은 두 가지가 있다.

2-1, context의 CancelFunc()을 호출하여 context의 Done
channel(context.Done())에 이벤트를 송신하는 방법,

Done 채널 이벤트를 수신하면 context.Err()를 체크하고 함수를 종료(취소에
의한 에러 상태로 종료)하면 된다.

2-2, go routine에 전달할 child context 생성 할 때 종료할 시간(Timeout)을
명시한다.

이 경우 context의 CancelFunc()가 호출된 것과 마찬가지로

context의 Done channel(context.Done())에 이벤트가 송신된다.

Done 채널 이벤트를 수신하면 context.Err()를 체크하고 함수를 종료(취소에
의한 에러 상태로 종료)하면 된다.

**<> context를 생성해야 하는 상황 (추천 사용법)**

상황 1, application context(root context): application이 시작할 때
하나를 생성한다.

application context는 context의 두 가지 용도 모두를 목적으로 사용 된다.

application context 용도 1, application 전체에서 공유되어야 할 메타
데이터를 가진다.

application context 용도 2, application 내에서 go routine을 실행할
때마다 application context로 부터 child context를 생성하여 전달한다.
application의 종료 직전 application context의 CancelFunc() 호출 하면
실행중인 go routine이 취소 되도록 조치하여 resource를 안정적으로 정리할
수 있다.

상황 2, application 내에서 go routine을 실행할 때마다 application
context로 부터 child context를 생성하여 전달

이 상황은 application context 용도 2번의 내용에서 설명 되었다.

application context 생성 예)

**<> base context를 생성하는 함수들**

func **Background**() Context

func **TODO**() Context

실제 context 인스턴스를 생성하는 모든 함수(WithValue(), WithCancel(),
WithDeadline(), WithTimeout())는 parent context가 필요하다.

parent로 사용할 기존에 생성된 context가 없다면 새로운 context 인스턴스를
생성할 때 그 base로 사용할 context가 필요하다.

이때 base context를 생성하기 위해서 사용하는 함수가 Background()와
TODO()이다. 사실 Background()와 TODO()는 실제 context 인스턴스가 아닌
int의 포인터인 fake 인스턴스 주소를 반환하여 context 인스턴스 체인의
시작점을 만들수 있게 해준다.

같은 타입을 반환하는 함수가 Background()와 TODO() 의 두 개 이름을로
구분되어 있는 이유는 개발자들에게 관습적 사용 용도를 구분해 주기
위해서이다.

Background()는 실제로 context의 시작점(fake parent context instance)을
생성하기 위해서 사용한다.

TODO()는 context의 임시적인 사용 그리고 나중에 바꾸어 주어야 하는
context를 의미한다.

TODO()는 context를 필요로 하는 함수를 호출해야할 상황에서 적절한
context가 없는 경우에 임시로 입력할 context를 생성하기 위해서 사용한다.

**<> data를 공유하기 위해 conetext를 생성하는 함수들**

func context.**WithValue**(parent context.Context, key any, val any)
context.Context

context.WithValue() 함수를 중첩 사용하여 여러 쌍의 <key>, <value>
data를 갖는 context를 생성할 수 있다.

context.WithValue()를 한번 호출할 때마다 새로운 child context가 생성되며
parent context는 child context에 추가된 <key>, <value> data는 알 수
없다.

Value Context 사용예)

```go
func TestValueContext(t *testing.T) {

baseCtx := context.WithValue(context.Background(), "key1",
"value1")

paretCtx := context.WithValue(baseCtx, "key2", "value2")

childCtx := context.WithValue(paretCtx, "key3", "value3")

t.Log(childCtx.Value("key1")) // value1

t.Log(childCtx.Value("key2")) // value2

t.Log(childCtx.Value("key3")) // value3

t.Log(baseCtx.Value("key2"))  // <nil>

t.Log(paretCtx.Value("key3")) // <nil>

}
```

**<> go-routine을 취소 시키기 위한 conetext를 생성하는 함수들**

func context.**WithCancel**(parent context.Context) (ctx
context.Context, cancel context.CancelFunc)

func context.**WithDeadline**(parent context.Context, d time.Time)
(context.Context, context.CancelFunc)

func context.**WithTimeout**(parent context.Context, timeout
time.Duration) (context.Context, context.CancelFunc)

위 세가지 context 생성 함수들은 공통적으로 cancelFunc를 반환한다.

WithDeadline()와 WithTimeout()의 경우 CancelFunc()를 직접 호출하지
않더라도 지정한 시간이 지나면 내부적으로 CancelFunc()가 호출되어
context의 Done() channel에 수신이벤트가 발생한다.

WithTimeout(Context, time.Duration)는 WithDeadline(Context,
time.Now().Add(time.Duration)) 과 같다.

CancelFunc() ?

ctx :: context

Done() channel :: context.Done() 함수에서 반환 되는 channel

ctx.cancelFunc()를 호출하면 즉시 해당 ctx의 Done channel(Done())에
반복적으로 수신 이벤트가 발생한다.

parent ctx의 cancelFunc()을 호출하면 parent ctx로 부터 파생된 child
ctx의 Done channel에 수신 이벤트가 발생하며,

child chain이 있는 경우 parect -> child 순서로 모두 차례대로 Done
channel에 수신 이벤트가 발생한다.

반대로 child ctx의 cancelFunc() 호출한다고 해서 parent ctx의 Done
channel에 수신 이벤트가 발생 하진 않는다.

사실 context.CancelFunc()를 호출한다고 해서 실제로 go routine의 실행
흐름이 취소되는 것은 아니다.

context.CancelFunc()은 그 context와 모든 child context의 Done()
channel에 parent->child 순서대로 송신을 할 뿐이다.

go-routing에 대한 취소 처리는 개발자가 이 이벤트를 수신 받아서 함수
종료를 처리해 주어야 한다.

예) CancelFunc()를 이용한 고루틴의 종료

```go
func BlockingFn(ctx context.Context, printCh <-chan int, caller
string) {

for {

select {

case <-ctx.Done(): // receive context Done channel by invoking
CancelFunc()

fmt.Printf("%s :: (P)%s: ctx.Done() is called\n", caller,
ctx.Value("parentFn"))

if err := ctx.Err(); err != nil {

fmt.Printf("%s :: err: %s\n", caller, err)

}

fmt.Printf("%s :: finished\n", caller)

return // terminate go-routine

case num := <-printCh:

fmt.Printf("%s :: %d\n", caller, num)

default: // check select in every second than nano time in case
there is no channel event

            time.Sleep(1 * time.Second)

}

}

}

func TestCancelContext(t *testing.T) {

baseCtx := context.WithValue(context.Background(), "parentFn",
"TestCancelContext")

ctx, cancelCtx := context.WithCancel(baseCtx)

printCh := make(chan int)

go BlockingFn(ctx, printCh, "BlockingFn1") // run go-routine

for num := 1; num <= 3; num++ {

printCh <- num

}

cancelCtx() // send data into ctx.Done() channel immediately

time.Sleep(4 * time.Second)

fmt.Println("TestCancelContext:: finished")

}

func TestCancelContextM(t *testing.T) {

baseCtx := context.WithValue(context.Background(), "parentFn",
"TestCancelContext")

ctx, cancelCtx := context.WithCancel(baseCtx)

printCh := make(chan int)

go BlockingFn(ctx, printCh, "BlockingFn1") // run go-routine

go BlockingFn(ctx, printCh, "BlockingFn2") // run go-routine

go BlockingFn(ctx, printCh, "BlockingFn3") // run go-routine

for num := 1; num <= 3; num++ {

printCh <- num

}

cancelCtx() // send data into ctx.Done() channel immediately

time.Sleep(4 * time.Second)

fmt.Println("TestCancelContext:: finished")

}
```

---

# [go] select

select 문은 여러 case 문에 정의된 expression중 blocking 되지 않은
case/default context를 실행하고 block(select context)을 종료하는 지시어
이다.

**<> select 문에서 case와 default 실행 순서**

select문은 자신 이하의 모든 case, default문을 체크한다.

모든 case -> default 순서로 expression을 체크하며 모든 case 문이
blocking인 경우에만 default 문이 실행된다.

case 문에는 channel 송/수신 중 하나의 expression이 반드시 명시 되어야
한다.

case 문에 대해서는 random 순서로 체크하되 체크 중인 case가 bloking이면
다른 case를 체크하는 방식으로 모든 case문을 한번씩 체크하게 된다. 체크
중인 case에 blocking이 아닌 expression이 있으면 해당 case context를
실행하고 select문은 종료된다.

이 과정을 반복하고 싶으면 select block을 for문으로 감싸야 하고 for
loop를 종료할 조건을 break <LABEL>을 통해서 명시해야한다.

아래의 예제는 case가 램덤 순서로 처리 되는 것을 확인 할 수 있으며

wg 사용을 주석 처리하면 channel에 element가 송신될 때 까지 default문이
먼저 실행되는 것을 볼 수 있다.

```go
func TestSelectRace(t *testing.T) {

ch1 := make(chan int, 1000)

ch2 := make(chan int, 1000)

var wg sync.WaitGroup

wg.Add(1)

go func() {

for i := 0; i < 1000; i++ {

if i%2 == 0 {

ch2 <- i

} else {

ch1 <- i

}

}

wg.Done()

}()

wg.Wait()

LOOP:

for {

select {

case v := <-ch1:

t.Log("ch1: ", v)

if v == 999 {

break LOOP

}

case v := <-ch2:

t.Log("ch2: ", v)

default:

t.Log("default")

}

}

t.Log("finish")

}
```

**<> for-select 문에서 <LABEL>을 사용한 for loop escaping**

for-select 구문을 사용한 경우 "break <LABEL>" 지시어를 사용하여 for
loop 구간(for block)을 빠져나올수 있다.

break문에 사용되는 <LABEL>은 반드시 break 사용전에 정의 되어야 하며
보통 for 지시어 직전에 선언한다.

break <LABEL>을 구문을 통해서 for문을 escape하게 되면 for block의 종료
curly bracket(})의 다음 라인이 실행된다.

```go
func TestChannelForSelect(t *testing.T) {

ch1 := make(chan int, 10)

ch2 := make(chan int, 10)

var wg sync.WaitGroup

wg.Add(1)

go func() {

for i := 0; i < 10; i++ {

ch1 <- i

}

wg.Done()

}()

wg.Wait()

Loop:

for {

select {

case val := <-ch1:

t.Log(val)

if val == 3 {

break Loop

}

case <-ch2:

}

}

t.Log("finish")

}
```
