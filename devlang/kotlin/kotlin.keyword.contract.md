# kotlin keyword - contract

Kotlin의 contract는 컴파일러에게 함수의 동작 방식을 명시적으로 전달하여, 컴파일러가 코드를 더 똑똑하게 분석하고 스마트 캐스트(Smart Casting) 등을 수행할 수 있게 돕는 기능입니다.
일반적으로 컴파일러는 함수 내부의 로직을 완벽히 이해하지 못해 호출부에서 불필요한 null 체크나 타입 변환을 요구할 때가 있는데, contract를 통해 이를 해결할 수 있습니다.

⚠️ 주의: 현재 실험적(Experimental) 단계이므로 개발자가 직접 사용할 필요는 없습니다.

## 핵심 개념과 주요 키워드

contract 블록 내에서 주로 사용되는 "효과(Effect)"들은 다음과 같습니다

- returns(value) implies [condition]: 함수가 특정 값(예: true, false)을 반환한다면, 특정 조건이 참임을 보장합니다.
- 예: isNullOrBlank()가 false를 반환하면 해당 변수는 null이 아니다.
- returnsNotNull() implies [condition]: 함수가 null이 아닌 값을 반환하면 특정 조건이 참임을 보장합니다.
- callsInPlace(lambda, kind): 전달된 람다가 함수 내부에서 언제, 몇 번 실행되는지를 명시합니다.
- InvocationKind.EXACTLY_ONCE: 딱 한 번만 실행됨을 보장.
  - InvocationKind.AT_LEAST_ONCE: 최소 한 번 이상 실행됨을 보장.

## 실제 사용 예시

코틀린 표준 라이브러리의 isNullOrBlank() 같은 함수는 내부적으로 contract를 사용합니다.

```kotlin
@OptIn(ExperimentalContracts::class)
public inline fun CharSequence?.isNullOrBlank(): Boolean {
    contract {
        // 이 함수가 false를 반환하면, receiver(this)는 null이 아님을 컴파일러에게 알림
        returns(false) implies (this != null)
    }
    return this == null || this.blank
}

fun main() {
    val name: String? = "Kotlin"
    if (!name.isNullOrBlank()) {
        // contract 덕분에 여기서 name은 자동으로 Non-null 타입으로 스마트 캐스트됨
        println(name.length) 
    }
}
```

## 특징 및 주의사항

- 실험적 기능: 현재 실험적(Experimental) 단계이므로 @OptIn(ExperimentalContracts::class) 어노테이션이 필요합니다. (개발자가 contract 직접 사용시에만 필요)
- 컴파일 타임 전용: 런타임 오버헤드가 없으며, 컴파일 과정에서 분석용으로만 쓰인 후 제거됩니다.
- 제한 사항: 함수의 최상단(첫 줄)에 위치해야 하며, inline 함수와 함께 자주 사용됩니다.
