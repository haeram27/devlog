# SAM(Single Abstract Method) Conversion

SAM 변환(SAM Conversion, Single Abstract Method 변환)은 Java 8에서 도입된 람다 표현식과 함수형 인터페이스의 기반이 되는 컴파일러 기술입니다.

---

## 1. SAM 변환이란?

- **SAM**: Single Abstract Method. 단 하나의 추상 메서드(Single Abstract Method, SAM). `SAM`로 구성된 인터페이스를 함수형 인터페이스(Functional Interface)라고 합니다. 인터페이스에서 SAM 여부 판단 시, 다음의 메소드들은 메서드 카운트에서 제외 됩니다.
  - Object 클래스의 public 메서드 제외: 앞서 말씀하신 Comparator처럼 equals(Object obj)나 toString(), hashCode()와 같은 Object 클래스의 메서드를 인터페이스 내에 추상 메서드로 다시 선언하더라도, 이는 추상 메서드 개수에 포함되지 않습니다.
  - default 및 static 메서드 제외: 인터페이스 내에 default 메서드나 static 메서드가 아무리 많아도 추상 메서드가 하나뿐이라면 함수형 인터페이스로 인정됩니다.
  - @FunctionalInterface 어노테이션이 없더라도 추상 메서드가 하나뿐인 인터페이스는 함수형 인터페이스로 취급됩니다. 하지만 이 어노테이션을 붙여주면 컴파일러가 해당 규칙을 지키고 있는지 검사해주므로 사용이 권장됩니다.
- **SAM 변환**: 람다식(`() -> { … }`) 또는 메서드 참조(Method Reference)를 그 인터페이스의 `추상 메서드 구현체`로 자동으로 바꿔 주는 과정입니다.
- 결과적으로 익명 클래스(anonymous class)를 더 간결한 람다 문법으로 대체할 수 있게 해 줍니다.

---

## 2. 작동 원리

1. **컴파일 시**  
   - 람다식이 등장하면, 컴파일러는 “이 람다식이 어떤 타입(함수형 인터페이스)으로 쓰였는가”를 확인합니다.  
   - 그 인터페이스에 추상 메서드가 하나인지 검사한 뒤, 람다 본문을 그 메서드의 구현부로 간주하고 내부적으로 익명 클래스나 메서드 핸들링 코드(`invokedynamic` + `LambdaMetafactory`)를 생성하도록 합니다.

2. **바이트코드 수준**  
   - **`invokedynamic`** 인스트럭션을 통해 런타임에 실제 구현 객체(함수형 인터페이스를 구현하는 인스턴스)를 만들어 냅니다.  
   - 익명 클래스처럼 중간에 별도 클래스 파일(.class)을 만들지 않고, JVM 내부에서 동적으로 구현체를 생성합니다.

---

## 3. 요건

- **추상 메서드가 정확히 하나**  
  - `default` 메서드나 `static` 메서드는 수에 포함되지 않습니다.  
  - `java.lang.Object` 에 정의된 `equals`, `hashCode`, `toString` 도 무시합니다.
- **메서드 시그니처**  
  - 람다 `파라미터 리스트`와 `반환 타입`이 그 추상 메서드의 `시그니처`와 일치해야 합니다.
- **타입 정보가 명확해야**  
  - 변수 선언, 메서드 파라미터, 캐스트 등 “어떤 함수형 인터페이스 타입으로 변환할지” 컴파일러가 알아야 합니다.

---

## 4. 예시

### 4.1 Runnable (표준 함수형 인터페이스)

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}

// Runnable 은 void run() 단 하나의 추상 메서드만 가짐
Runnable r1 = () -> System.out.println("Hello, SAM!");
new Thread(r1).start();

// 익명 클래스와 비교
Runnable r2 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, Anonymous Class!");
    }
};
new Thread(r2).start();
```

### 4.2 커스텀 함수형 인터페이스

```java
@FunctionalInterface
interface Calculator {
    int calc(int x, int y);       // 단 하나의 추상 메서드
    default void info() {         // default 메서드는 SAM 카운트에 포함되지 않음
        System.out.println("Calculator");
    }
}

Calculator sum = (a, b) -> a + b;  // SAM 변환으로 Calculator.calc 구현
int result = sum.calc(3, 5);      // 결과: 8
```

---

## 5. @FunctionalInterface 어노테이션

SAM 인터페이스에 @FunctionalInterface 어노테이션을 명시 하는 것은 **선택 사항**입니다. @FunctionalInterface 어노테이션이 명시 되어 있지 않더라도 SAM 인터페이스를 만족하는 인터페이스는 SAM 변환이 될 수 있습니다. 

- 효과
  - 해당 인터페이스가 “함수형 인터페이스”용으로 설계되었음을 문서화하고,  
  - 추상 메서드를 하나 이상 추가하면 컴파일 오류를 발생시켜 실수를 방지해 줍니다.

---

## 6. 장·단점

| 장점                                                        | 단점                                         |
|-----------------------------------------------------------|--------------------------------------------|
| • 코드가 훨씬 간결해지고 가독성이 좋아짐                      | • 복잡한 구현(여러 메서드, 상태 보유)은 부적합 |
| • 익명 클래스 대비 보일러플레이트가 대폭 감소               | • 디버깅 시 스택트레이스가 다소 덜 직관적일 수 있음 |
| • JVM 레벨에서 최적화된 `invokedynamic`을 통한 생성         |                                            |

---

### 결론

SAM 변환은 “단일 추상 메서드 인터페이스”에 대한 람다식을 간결하게 적용하도록 해 주는 핵심 메커니즘이며, Java 8 이후 함수형 프로그래밍 스타일을 가능하게 만든 중요한 언어 기능입니다.

---

네, 공식적으로는 **람다 표현식(lambda expression)** 이라고 부릅니다.  

좀 더 정확히 말하면, Java 8에서 도입된 **함수형 인터페이스(functional interface)** 와 **SAM 변환(SAM conversion, Single Abstract Method)** 기능을 활용한 것입니다.  

- **함수형 인터페이스(functional interface)**:  
  추상 메서드를 딱 하나만 가지는 인터페이스로, `@FunctionalInterface` 어노테이션을 붙여 표시할 수도 있습니다.  
- **람다 표현식(lambda expression)**:  
  그 함수형 인터페이스의 추상 메서드를 구현하는 익명 함수를 간결하게 표현하는 문법입니다.  
- **SAM 변환(SAM conversion)**:  
  함수형 인터페이스에 대해 `factory -> { … }` 형태의 람다식을 쓰면, 컴파일러가 곧바로 해당 인터페이스의 유일한 추상 메서드(`customize`) 구현으로 “변환”해 주는 과정입니다.

즉, “익명 클래스 대신 람다를 쓴 것”은 Java 언어 사양에서 **람다 표현식**이자 **SAM 변환** 기법입니다.