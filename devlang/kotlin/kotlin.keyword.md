# kotlin keyword

## 참고

- [Kotlin 공식 문서 - keyowrd](https://kotlinlang.org/docs/keyword-reference.html)

## 키워드 (Keywords)
Kotlin의 키워드는 용도에 따라 세 가지로 분류됩니다.

- 하드 키워드 (Hard Keywords)
- 소프트 키워드 (Soft Keywords)
- 수식어 키워드 (Modifier Keywords)


### 하드 키워드 (Hard Keywords)

식별자로 사용할 수 없는 고정된 키워드입니다.

- 제어 흐름: if, else, when, for, while, do, break, continue, return, throw, try, catch, finally
- 선언: val, var, fun, class, interface, object, typealias, package, import
- 타입 및 상속: as, is, in, super, this, where
- 기타: true, false, null, typeof

#### kotlin 전용 
  - when : switch 대용
    - when(x) { (boolean || 'x == y') -> ... else -> "no-match" }
    - examples
    ```kotlin
    fun whatToDo(dayOfWeek: Any) = when (dayOfWeek) {
        "Saturday", "Sunday" -> "Relax"
        in listOf("Monday", "Tuesday", "Wednesday", "Thursday") -> "Work hard"
        in 2..4 -> "Work hard"
        "Friday" -> "Party"
        is String -> "What?"
        else -> "No Clue"
    }
    ```
  - val : (value), declares read-only, immutable variable.
    - 선언시 초기화 필요
  - var : (variable), Declares a mutable variable
  - fun : method 선언
  - typealias : 타입 축약어 정의
    ```kotlin
        // 긴 제네릭 타입 축약
        typealias UserMap = Map<String, List<User>>

        // 복잡한 함수 타입 축약
        typealias ClickListener = (String, Int) -> Unit

        // 내부 클래스 접근 축약
        typealias UserAdapter = com.example.app.ui.users.UserFragment.Adapter
    ```
  - as, as? : type casting
    - `val x: String? = y as? String`
    - `as?` 사용시 캐스팅 불가 exception이 발생하면 대신 null을 반환
  - is, !is : 타입 검사(= java instalnceOf) + 타입 캐스팅, 반환 타입은 boolean
  - in : 
    - syntax: `in <range-expr>`
    - examples
    ```kotlin
        for (i in 1..100) { ... }
        for (i in 1 until 100) { ... }
        for (x in 2..10 step 2) { ... }
        for (x in 10 downTo 1) { ... }
        if (x in 1..10) { ... }
    ```



### 소프트 키워드 (Soft Keywords)
특정 문맥에서만 키워드로 작동하며, 그 외에는 식별자로 사용할 수 있습니다.

- field, get, set, it, file, import, where, property, receiver, param, setparam, delegate

### 수식어 키워드 (Modifier Keywords)
선언에 추가적인 의미를 부여할 때 사용합니다.

- 접근 제한자(가시성 변경자): public, private, protected, internal
  - 모든 접근 제한자는 필드, 생성자, 메소드에 지정 가능하며 public, proteced는 class에 추가로 지정 가능
  - public : 접근 제한 없음
  - private : 선언된 클래스 내에서 접근 가능
  - protected : 선언된 클래스 또는 하위 클래스에서만 접근 가능  
  - internal : 동일 모듈(module)내에서 접근 가능
- 클래스 특성: abstract, final, open, sealed, data, enum, inner, companion
- 함수 특성: inline, noinline, crossinline, reified, tailrec, operator, infix, suspend, external
- 기타: lateinit, const, override, expect, actual, annotation

#### kotlin 전용
- sealed
- data
- inner
- companion
- noinline
- crossinline
- reified
- tailrec
- operator
- infix
- suspend
- external
- lateinit
- const
- expect
- actual
- annotation

------------------------------
## 연산자 (Operators)
Kotlin의 연산자는 내부적으로 정해진 이름의 함수 호출로 변환되는 '관례(Convention)'를 따릅니다.

### 산술 및 대입 연산자

- 기본 산술: + (plus), - (minus), - (times), / (div), % (rem)
- 증감: ++ (inc), -- (dec)
- 복합 대입: +=, -=, *=, /=, %= (각각 plusAssign 등)

### 비교 및 논리 연산자 [14, 15] 

- 동등성: == (equals), != (not equals)
- 참조 동등성: === (동일 객체), !== (다른 객체)
- 크기 비교: <, >, <=, >= (compareTo)
- 논리: && (AND), || (OR), ! (NOT)

### Null 관련 및 특수 연산자 [17, 18] 

- 세이프 콜: ?. (객체가 null이 아닐 때만 호출)
- 엘비스 연산자: ?: (null일 경우 기본값 지정)
- 단언 연산자: !! (강제로 non-null 타입으로 변환)
- 범위: .., ..< (rangeTo)
- 인덱스 접근: [] (get/set)
- 함수 호출: () (invoke) [19, 20, 21] 
