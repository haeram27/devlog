## Unit과 Any

- Unit
  - "빈 객체"을 의미, null은 아니지만 빈 객체를 사용해야하는 자리(주로 반환 타입이나 값)에 사용
  - 반환 타입 또는 값: 반환 값이 없음을 의미
  - override한 메소드의 파라미터 타입에 사용: 파라미터의 값이 함수내에서 사용되지 않을 것임을 의미
  - 자식 클래스의 제너릭에 사용: 해당 타입은 클래스나 함수에서 사용되지 않음을 의미
- Any
  - 모든 객체의 조상, java generic의 `?`와 같은 의미로 사용 

| 구분 | Unit | Any |
|---|---|---|
| 의미 | "함수가 반환할 값이 없다" | "모든 타입의 조상이다" |
| Java 대응 | void (또는 Void 클래스) | Object |
| 인스턴스 수 | 단 하나 (Unit 객체) | 무수히 많음 (모든 객체) |
| 주요 사용처 | 반환 타입이 없는 함수 정의 | 다양한 타입을 수용해야 할 때 |


## 1. Unit (반환값이 없는 경우)
Java의 void와 유사하지만, 실제로는 하나의 인스턴스만 존재하는 객체입니다.

- 용도: 함수가 유의미한 값을 반환하지 않을 때 사용합니다.
- 특징: 생략 가능하며, 함수 끝에 return Unit이나 값을 명시하지 않아도 컴파일러가 자동으로 처리합니다.
- Java void와의 차이: void는 아무것도 없음을 뜻하는 '상태'지만, Unit은 그 자체로 '객체'입니다. 따라서 제네릭 등에서 타입 인자로 사용할 수 있습니다.

```kotlin
fun printHello(): Unit { // Unit은 생략 가능
    println("Hello")
}
```

## 2. Any (모든 타입의 최상위 클래스)
Java의 Object와 대응되는 개념으로, Kotlin의 모든 클래스가 상속받는 루트 클래스입니다.

- 용도: 어떤 타입의 객체든 담을 수 있는 변수를 선언하거나, 공통 메서드(equals, hashCode, toString)를 정의할 때 사용합니다.
- 특징: Int, String 같은 기본 타입뿐만 아니라 사용자 정의 클래스까지 모두 Any의 자식입니다.
- Java Object와의 차이: Java의 Object는 참조 타입만 포함하지만, Kotlin의 Any는 Int 같은 원시 타입(Primitive)까지 모두 포괄하는 더 넓은 개념입니다.

```kotlin
// 서로 다른 타입을 한꺼번에 담을 수 있음
val list: List<Any> = listOf("Text", 123, true)
```


## Unit과 Generic

Unit이 Generic에 사용되는 경우

### 1. 결과값이 없는 비동기/콜백 처리
가장 흔한 케이스입니다. 작업의 성공 여부만 알면 되고, 결과 데이터는 필요 없을 때 Unit을 사용합니다.

- Any를 쓰면: 무언가 반환해야 할 것 같은 오해를 주며, 호출부에서 불필요한 값을 처리해야 할 수도 있습니다.
- Unit을 쓰면: "이 작업은 값을 반환하지 않는다"는 의도를 명확히 전달합니다.

```kotlin
// 데이터가 필요 없는 네트워크 응답 처리
interface Callback<T> {
    fun onSuccess(result: T)
}

// 결과를 받을 필요가 없는 경우 Unit 지정val simpleCallback = object : Callback<Unit> {
    override fun onSuccess(result: Unit) {
        println("작업 완료!") // result를 다룰 필요가 없음
    }
}
```

### 2. 특정 기능을 수행만 하는 '명령(Command)' 객체
디자인 패턴 중 커맨드 패턴처럼, 무언가를 실행(execute)하기만 하고 반환값이 없는 클래스를 제네릭으로 추상화할 때 사용합니다.

```kotlin
abstract class Task<out T> {
    abstract fun execute(): T
}

// 아무것도 반환하지 않는 할 일
class LoggingTask : Task<Unit>() {
    override fun execute() {
        println("로그 기록 중...")
        // 명시적인 return이 없어도 자동으로 Unit 반환
    }
}
```

### 3. Java의 Void 제네릭 대응
Java에서는 제네릭에 원시 타입 void를 쓸 수 없어 Void 클래스를 사용하고 항상 return null;을 해줘야 했습니다. Kotlin의 Unit은 실제 객체이므로 return 문 없이도 Java의 Void 역할을 깔끔하게 대체합니다.

## `<T>`와 `<T: Any>` 차이

`<T>`는 `Any?` 의미, null 수용

`<T: Any>`는 `Any` 의미, null 수용하지 않음

## 1. T만 사용했을 때 (기본값)
제네릭에서 상한(Upper Bound)을 지정하지 않고 `<T>`라고만 쓰면, 암묵적으로 `<T : Any?>`가 됩니다.

* 의미: "어떤 타입이든 올 수 있고, null도 허용한다."
* Unit 사용 시: `<Unit>`으로 타입을 명시하여 쓸 수 있습니다.
* Any 사용 시: `<Any>` 또는 `<Any?>` 모두 가능합니다.

## 2. T : Any라고 명시했을 때
이것은 "상한 제한(Upper Bound Constraint)"이라고 부릅니다.

* 의미: "반드시 null이 될 수 없는(Non-null) 타입만 올 수 있다."
* Unit 사용 시: Unit은 그 자체로 null이 아니므로 당연히 사용 가능합니다.
* 차이점: 만약 누가 이 제네릭에 String?이나 Int?처럼 null이 가능한 타입을 넣으려고 하면 컴파일 에러를 내뱉습니다.

------------------------------
## 왜 T : Any라고 명시하나요?
주로 Null-safety(널 안정성)를 강제하고 싶을 때 사용합니다.

```kotlin
// T는 null일 수도 있음 (T : Any? 와 같음), class Box<T>(val item: T) 
// T는 절대 null일 수 없음, class StrictBox<T : Any>(val item: T)
fun main() {
    val b1 = Box<Int?>(null)       // OK
    // val b2 = StrictBox<Int?>(null) // 에러! null 허용 타입은 못 들어옴

    val b3 = StrictBox<Unit>(Unit) // OK (Unit은 null이 아니니까요)
    val b4 = StrictBox<Any>(123)   // OK (Any는 null이 아니니까요)
}
```


