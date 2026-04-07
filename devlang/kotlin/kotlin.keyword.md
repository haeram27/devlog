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
- when : java switch 대용
  - when(x) { (boolean || 'x == y') -> "match" ... else -> "no-match" }
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
  - 모든 접근 제한자는 필드, 생성자, 메소드에 지정 가능하며 public, proteced는 class에만 추가로 지정 가능
  - public : 접근 제한 없음, default
  - internal : 동일 모듈(module)내에서 접근 가능
  - protected : 선언된 클래스와 자식 클래스에서만 접근 가능
  - private : 선언된 클래스 내에서 접근 가능
- 클래스 특성: abstract, final, open, sealed, data, enum, inner, companion
- 함수 특성: inline, noinline, crossinline, reified, tailrec, operator, infix, suspend, external
- 기타: lateinit, const, override, expect, actual, annotation

#### 접근 제한자

- 선언 위치 별 접근 가능 범위

| 대상 | public | internal | protected | private |
|---|---|---|---|---|
| 최상위 선언 (파일 직속: 클래스, 함수, 변수) | 가능 (기본) | 가능 (모듈 내) | 불가능 | 가능 (파일 내) |
| 클래스 멤버 (함수, 변수) | 가능 (기본) | 가능 (모듈 내) | 가능 (자식까지) | 가능 (클래스 내) |

#### kotlin 전용

- 클래스 특성
  - open: kotlind의 클래스와 메소드는 기본적으로 final 속성을 가져 상속하거나 오버라이드가 불가능 합니다. open 키워드는 클래스와 메소드를 상속 가능하도록 설정합니다. 
  - sealed: 봉인된 클래스. 자기 클래스를 상속받는 자식 클래스 종류를 제한합니다. when 식에서 모든 타입을 체크할 때 유용합니다.
  - data: 데이터를 저장하는 목적의 클래스. equals(), hashCode(), toString(), copy() 등을 자동으로 생성해 줍니다. getter, setter 구현 불필요
  - inner: 내부 클래스. 바깥 클래스의 참조를 가집니다. 바깥 클래스의 멤버에 접근할 수 있습니다. inner class를 참조하려면 부모의 이름이 필요.
  - companion object: 동반 객체. static inner class 선언. 클래스 당 하나의 동반 객체만 선언 가능. 부모 클래스가 클래스 로더에 의해 인스턴스화 될 때 딱 한번 함께 인스턴스화 됨. 부모 참조 없이 자신의 클래스 이름을 통해 접근할 수 있게 합니다.

- 함수 특성
  - open: kotlind의 클래스와 메소드는 기본적으로 final 속성을 가져 상속하거나 오버라이드가 불가능 합니다. open 키워드는 클래스와 메소드를 상속 가능하도록 설정합니다.
  - noinline: inline 함수에서 특정 람다 매개변수만 인라인화되지 않도록 제외할 때 사용합니다.
  - crossinline: inline 함수에 전달된 람다 내에서 return을 사용해 호출 함수를 종료(non-local return)하지 못하도록 제한합니다.
  - reified: inline 함수 내에서 제네릭 타입 T를 런타임에 실제 클래스 정보로 사용할 수 있게 합니다.
  - tailrec: 꼬리 재귀 함수. 재귀 호출을 루프 코드로 변환하여 스택 오버플로우를 방지해 줍니다.
  - operator: 연산자 오버로딩. +, -, * 같은 연산자를 특정 함수(예: plus)로 정의하여 사용할 수 있게 합니다.
  - infix: 중위 함수. obj method arg처럼 점(.)과 괄호 없이 함수를 호출할 수 있게 합니다.
  - suspend: 코루틴 함수. 실행을 일시 중단하고 나중에 다시 시작할 수 있음을 나타냅니다.
  - external: 외부(C, C++ 등) JNI를 통해 구현된 함수임을 나타냅니다.

- 기타
  - lateinit: 가변 변수(var)의 초기화를 나중에 하겠다고 선언합니다. (Primitive 타입은 불가)
  - const: 컴파일 타임 상수. val보다 더 엄격하며 컴파일 시점에 값이 결정됩니다.
  - expect: 멀티플랫폼 프로젝트(KMP)의 공통 모듈에서 선언하는 구현 예정 함수/클래스입니다.
  - actual: expect로 선언된 대상을 각 플랫폼(Android, iOS 등) 모듈에서 실제로 구현한 것입니다.
  - annotation: 주석(애노테이션) 클래스를 선언할 때 사용하며, 코드에 메타데이터를 추가합니다.

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


## Java에는 존재하지만 Kotlin에서는 존재하지 않는 키워드
 
### 완전히 제거된 키워드 (기능 대체) 

- static: Kotlin에서는 companion object, object 선언 또는 패키지 수준의 최상위 함수/변수로 대체되었습니다.
- new: 객체 생성 시 키워드 없이 클래스명() 형태를 사용하므로 더 이상 필요하지 않습니다.
- void: 반환 값이 없음을 나타낼 때 Unit 타입을 사용합니다.
- instanceof: 타입 체크를 위해 is 키워드를 사용합니다.
- extends, implements: 클래스 상속과 인터페이스 구현 모두 : (콜론) 기호로 통합되었습니다.
- native: JNI 메서드를 정의할 때 external 키워드를 사용합니다. 

### 다른 키워드로 변경된 경우

- final (변수): 읽기 전용 변수 선언 시 val을 사용합니다.
- final (클래스/메서드): Kotlin은 모든 클래스가 기본적으로 final이므로 별도 키워드가 필요 없습니다. 반대로 상속을 허용하려면 open을 붙여야 합니다.
- switch: 더 강력한 기능을 가진 when 식으로 대체되었습니다.
- default: switch 문 내에서는 else로, 인터페이스 내 기본 메서드는 별도 키워드 없이 메서드 몸체를 직접 작성하여 구현합니다. 

### 기타 제어 키워드 및 연산자

- synchronized, volatile: Kotlin에서는 키워드 대신 @Synchronized, @Volatile과 같은 어노테이션으로 사용하거나 synchronized() 함수 블록을 사용합니다.
- transient: 직렬화 제외를 위해 @Transient 어노테이션을 사용합니다.
- strictfp: 부동 소수점 정밀도를 제어하는 이 키워드는 Kotlin에 존재하지 않습니다.
- throws: Kotlin은 Checked Exception(체크 예외)을 강제하지 않으므로 이 키워드가 없습니다. Java와 호환을 위해 @Throws 어노테이션을 쓰기도 합니다.
- assert: 키워드가 아닌 assert() 함수 형태로 제공됩니다.
- const: Java에서는 상수를 정의할 때 쓰이지만, Kotlin에서는 const val 형태로 함께 쓰이며 컴파일 타임 상수에만 한정됩니다. 