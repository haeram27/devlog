# Trailing Lambda

- 코틀린의 후행 람다(Trailing Lambda)는 함수의 마지막 매개변수가 함수 타입(람다)일 때, 해당 람다를 함수 호출 괄호 `()` 바깥으로 빼서 작성할 수 있는 문법입니다.
- 이 문법을 사용하면 코드가 마치 제어 구조(if, for 등)처럼 보여 가독성이 크게 향상됩니다.

## 1. 기본 원리

함수의 마지막 파라미터가 람다라면 다음과 같이 호출할 수 있습니다.

- 일반적인 호출: `func(a, { ... })`
- 후행 람다 사용: `func(a) { ... }`

만약 함수에 매개변수가 람다 하나뿐이라면, 호출 시 괄호 ()를 아예 생략할 수도 있습니다.

## 2. 코드 예제

### 매개변수가 여러 개인 경우

마지막 매개변수가 람다일 때 괄호 뒤에 붙여 씁니다. [5] 

```kotlin
// 함수 정의: 마지막 파라미터가 함수 타입임
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

fun main() {
    // 1. 일반적인 호출 방식
    val result1 = calculate(10, 5, { x, y -> x + y })

    // 2. 후행 람다 방식 (권장)
    val result2 = calculate(10, 5) { x, y -> 
        x + y 
    }
}
```

### 매개변수가 람다 하나인 경우

호출 시 괄호 ()를 생략할 수 있어 매우 간결해집니다.

```kotlin
// 함수 정의
fun runAction(action: () -> Unit) {
    action()
}

fun main() {
    // 괄호 없이 중괄호만 사용하여 호출 가능
    runAction {
        println("액션을 실행합니다!")
    }
}
```

## 3. 왜 사용하나요?

- 가독성: 람다 내용이 길어질 때 괄호 안에 가두는 것보다 밖으로 빼는 것이 코드를 읽기 훨씬 편합니다.
- DSL(Domain Specific Language) 구축: 코틀린의 Type-safe Builders나 안드로이드의 Jetpack Compose처럼 선언형 UI를 작성할 때 핵심적인 역할을 합니다.
