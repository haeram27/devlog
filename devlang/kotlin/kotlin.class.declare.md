# 코틀린에서 클래스 선언

## 1. 일반 클래스 (Final Class)

기본적으로 상속이 불가능한 상태입니다.

```kotlin
class Person(val name: String)
```

## 2. 개방 클래스 (Open Class)

open 키워드를 붙여야 다른 클래스가 상속할 수 있습니다.

```kotlin
open class Vehicle(val speed: Int)class Car(speed: Int) : Vehicle(speed)
```

## 3. 데이터 클래스 (Data Class)

데이터 보관이 주 목적으로, toString(), equals(), hashCode(), copy()가 자동 생성됩니다.

```kotlin
data class User(val id: Int, val name: String)
```

## 4. 추상 클래스 (Abstract Class)

직접 인스턴스화할 수 없으며, 하위 클래스에서 구현해야 할 가이드라인을 제시합니다.

```kotlin
abstract class Shape {
    abstract fun draw()
}
```

## 5. 인터페이스 (Interface)

추상 메서드와 기본 구현이 포함된 메서드를 가질 수 있습니다.

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}
```

## abstract와 interface 차이

Kotlin에서 abstract 클래스와 interface는 모두 추상화를 위한 도구이지만, 그 목적과 능력치에 확실한 차이가 있습니다.

| 구분 | 추상 클래스 (abstract) | 인터페이스 (interface) |
|---|---|---|
| 상속 (상속 개수) | 단일 상속만 가능 (하나만 골라!) | 다중 구현 가능 (여러 개 조합 가능!) |
| 상태 저장 (Property) | Backing Field 가짐 (실제 값을 저장함) | Backing Field 없음 (값 저장 불가, 커스텀 Getter만 가능) |
| 생성자 | 생성자 가질 수 있음 | 생성자 가질 수 없음 |
| 목적 | "무엇인가?" (본질적인 정체성 확장) | "무엇을 할 수 있는가?" (기능 추가/능력 부여) |

### Abstract 클래스: "유전자를 물려주는 부모"

실제 변수를 가지고 있고, 생성자를 통해 초기화가 가능합니다.

```kotlin
// 생성자 가능, 변수 저장 가능
abstract class Animal(val name: String) {
    abstract fun sound() // 자식이 반드시 구현
    fun breathe() = println("숨을 쉽니다") // 공통 기능 물려주기
}

class Dog(name: String) : Animal(name) {
    override fun sound() = println("멍멍")
}
```

### Interface: "장착하는 액세서리/자격증"

변수를 선언해도 실제 값을 저장(field)하지 못하며, 여러 개를 동시에 가질 수 있습니다.

```kotlin
interface Runnable {
    // 실제 값이 저장되는 게 아니라, 하위에서 결정해야 함
    val speed: Int
    fun run() = println("${speed}km/h로 달립니다") // 기본 구현 가능
}

interface Swimmable {
    fun swim()
}

// 여러 인터페이스를 동시에 구현(다중 상속 효과) 가능
class Duck : Swimmable, Runnable {
    override val speed = 5
    override fun swim() = println("오리가 수영합니다")
}
```

### 언제 무엇을 쓸까?

- abstract 클래스를 써야 할 때:
- 여러 클래스 간에 공통된 변수(상태)를 공유해야 할 때.
  - "고양이, 강아지는 동물이다"처럼 강한 계층 구조(Is-A 관계)를 만들 때.
  - 부모 클래스에서 생성자를 통해 초기화 로직을 수행해야 할 때.
- interface를 써야 할 때:
- 클래스 계층과 상관없이 특정 기능(Behavior)을 여러 곳에 적용하고 싶을 때.
  - 이미 다른 클래스를 상속받고 있는 클래스에 추가 능력을 주고 싶을 때 (다중 구현).
  - 느슨한 결합으로 나중에 교체하기 쉬운 설계를 하고 싶을 때.

요약하자면:
"이것은 본질적으로 무엇인가?"를 정의하려면 abstract, "이것이 어떤 기능을 하는가?"를 정의하려면 interface를 선택하세요!

## 6. 봉인된 클래스 (Sealed Class)
상속받는 하위 클래스 종류를 제한합니다. when 식에서 모든 케이스를 처리할 때 유용합니다.

```kotlin
sealed class Resultdata class Success(val data: String) : Result()data class Error(val message: String) : Result()
```

## 7. 열거형 클래스 (Enum Class)
정해진 상수 집합을 정의합니다.

```kotlin
enum class Direction {
    NORTH, SOUTH, EAST, WEST
}
```

## 8. 값 클래스 (Value Class / Inline Class)
하나의 값을 감싸서 성능 최적화를 꾀할 때 사용합니다.

```kotlin
@JvmInlinevalue class Password(val value: String)
```

## 9. 중첩 및 내부 클래스 (Nested & Inner Class)

- Nested: 외부 클래스 참조 불가
- Inner: inner 키워드로 외부 클래스 참조 가능

```kotlin
class Outer {
    class Nested // Outer.Nested()
    inner class Inner // Outer().Inner()
}
```

## 10. 객체 선언 (Object - Singleton)

클래스 정의와 동시에 단 하나의 인스턴스(싱글톤)를 생성합니다.

- 싱글톤 객체 생성에 사용 (유틸리티(Utility) 함수나 공통 상수를 관리할 때 가장 흔하게 사용)
- 네임스페이스로서의 사용
  - kotlin은 전역 함수/변수(파일 최상위에 정의 - 프로젝트 전반에서 아주 범용적 사용 가능)를 선언할 수 있음
  - 전역 함수/변수를 어떤 그룹으로 묶어서 관리할 때 object 내에 선언하여 사용
- object를 클래스 내부에서 사용시, 부모의 이름을 참조하여 접근해야 합니다.


### 최상위 class 선언에 사용시

- kotlin

```kotlin
object DatabaseConfig {
    val url = "jdbc:mysql://localhost:3306"
}
```

- java 변환

```java
public final class DatabaseConfig {
    // 1. 자기 자신을 상수로 가짐 (싱글톤)
    public static final DatabaseConfig INSTANCE = new DatabaseConfig();

    private final String url = "jdbc:mysql://localhost:3306";

    // 2. 외부에서 인스턴스를 생성하지 못하도록 생성자를 private으로 설정
    private DatabaseConfig() {
    }

    // 3. 필드 접근을 위한 Getter 메서드
    public final String getUrl() {
        return this.url;
    }
}

```

### inner class 선언에 사용시

- kotlin

```kotlin
class Outter {
    object Inner {
        val url = "jdbc:mysql://localhost:3306"
    }
}
```

- java 변환

```java
public final class Outter {

    public static final class Inner {
        // 1. 싱글톤 인스턴스 생성
        public static final Outter.Inner INSTANCE = new Outter.Inner();

        private final String url = "jdbc:mysql://localhost:3306";

        // 2. 생성자를 private으로 제한
        private Inner() { }

        // 3. 필드 접근을 위한 Getter
        public final String getUrl() {
            return this.url;
        }
    }
}
```

## 11. 동반 객체 (Companion Object)

outter 클래스에 속한 단일 싱글톤 객체를 생성합니다.
클래스 당 내부에 단 하나의 `compnion object`만 선언할 수 있습니다.
outter 클래스가 클래스 로더에 로딩 될 때 `compnion object`는 단 한번 인스턴스화 됩니다.
outter 클래스는 여러 개의 객체로 인스턴스화 될 수 있지만, companion object는 단 한번만 인스턴스화 됩니다.
컴파일된 bytecode를 보면 companion object는 `public static final class`가 됩니다. 

```kotlin
class MyClass {
    companion object {
        fun create() = MyClass()
    }
}
```

kotlin에는 static 키워드가 없으므로 클래스의 정적(static) 특성의 필드를 클래스에 직접 선언할 수 없습니다.

`compnion object`는 어떤 클래스 내부에서 선언되며, outter 클래스의 정적(static) 멤버를 담는 컨테이너 역할의 클래스 입니다.

이름 유무에 따라 두 가지 세부 형태로 나뉩니다: 

- 이름 없는 companion object: 가장 일반적인 형태로, `companion object { ... }`와 같이 작성합니다. 내부적으로는 Companion이라는 기본 이름을 가집니다.
- 이름 있는 companion object: `companion object Factory { ... }`처럼 이름을 붙일 수 있습니다. 특정 인터페이스를 구현하거나 용도를 명확히 하고 싶을 때 사용합니다. 

왜 companion 혼자 쓰지 않고 object와 함께 쓰나요?

- 객체 지향적 설계: Kotlin에는 Java의 static 키워드가 없습니다. 대신 '클래스에 속한 실제 객체(Object)'를 하나 생성하여 정적 멤버처럼 보이게 만듭니다.
- 다양한 기능 활용: 단순한 정적 변수 모음이 아니라, 하나의 '객체'이기 때문에 인터페이스를 구현하거나 다른 클래스를 상속받는 등 일반 객체처럼 다룰 수 있습니다.
- 싱글톤 보장: 클래스가 로드될 때 단 하나만 생성되는 싱글톤 인스턴스임을 object 키워드가 명시해 줍니다.

java 코드와 비교

- kotlin

```kotlin
class MyClass {
    companion object {
        const val CONSTNAME = "constKotlin"
        val NAME = "Kotlin"
        fun sayHello() = println("Hello")
    }
}
```

- java

```java
public final class MyClass {
    // 외부 클래스의 static 필드로 승격
    public static final String CONSTNAME = "constKotlin";

    // 1. Companion 이라는 이름의 static 내부 클래스가 생성됨
    public static final class Companion {
        private Companion() {} // 생성자는 private
        private final String NAME = "Kotlin"; // private 필드

        public final String getNAME() { // getter 메서드 생성됨
            return this.NAME;
        }

        public final void sayHello() {
            System.out.println("Hello");
        }
    }

    // 2. 외부 클래스에 Companion 객체의 static 인스턴스가 생성됨
    public static final MyClass.Companion Companion = new MyClass.Companion();
}
```

## object와 companion object 차이

object와 companion object는 둘 다 싱글톤(인스턴스가 하나만 존재)을 만든다는 공통점이 있지만, 어디에 선언되느냐와 어떻게 접근하느냐가 핵심 차이점입니다.

### 1. Object (객체 선언)

클래스 외부나 내부에 독립적으로 선언하며, 그 자체가 하나의 완성된 인스턴스입니다.

- 정의: object 키워드로 직접 이름을 붙여 만듭니다.
- 접근: 이름.멤버 방식으로 어디서든 접근 가능합니다.
- 용도: 유틸리티 함수 모음, 설정 값 저장, 싱글톤 패턴 구현 시 사용합니다.

```kotlin
object MySingleton {
    val prop = "I am a singleton"
    fun setup() = println("Setting up...")
}

// 사용
MySingleton.setup() 
```

### 2. Companion Object (동반 객체)

특정 클래스 내부에 선언되며, 해당 클래스와 생명주기를 같이 합니다.

- 정의: 클래스 안에 companion object 블록을 만듭니다.
- 접근: 클래스 이름을 통해 접근하므로, Java의 static 멤버처럼 보입니다.
- 용도: 팩토리 메서드(객체 생성 로직), 클래스 관련 상수 정의 시 사용합니다.
- 특징: 클래스의 private 멤버에도 접근할 수 있습니다.

```kotlin
class User {
    companion object {
        val MAX_AGE = 100
        fun create() = User()
    }
}

// 사용 (클래스 이름을 통해 바로 접근)val user = User.create()
println(User.MAX_AGE)
```

### 한눈에 비교하기

| 구분 | Object | Companion Object |
|---|---|---|
| 선언 위치 | 클래스 외부/내부 어디든 가능 | 클래스 내부에서만 가능 |
| 인스턴스 생성 | 선언 시 바로 생성됨 | 클래스가 로드될 때 생성됨 |
| 호출 방식 | 객체이름.멤버 | 클래스이름.멤버 |
| 이름 | 반드시 이름이 필요함 | 이름 생략 가능 (기본값: Companion) |
| 주요 목적 | 전체 프로그램의 단일 인스턴스 | 클래스에 종속된 정적 멤버 구현 |

요약하자면:
혼자서도 잘 노는 독립적인 싱글톤은 object, 특정 클래스의 '단짝 친구'처럼 붙어서 정적 멤버 역할을 하는 것은 companion object입니다.