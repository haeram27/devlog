# **FunctionalInterface와 Lambda 구현체의 매칭 기준**

- [**FunctionalInterface와 Lambda 구현체의 매칭 기준**](#functionalinterface와-lambda-구현체의-매칭-기준)
  - [**결론**](#결론)
    - [**람다가 함수형 인터페이스와 매칭되는 기준**](#람다가-함수형-인터페이스와-매칭되는-기준)
  - [**1. 매칭 기준**](#1-매칭-기준)
    - [**1.1. 단일 추상 메서드(SAM, Single Abstract Method) 여부**](#11-단일-추상-메서드sam-single-abstract-method-여부)
    - [**1.2. 람다 표현식의 시그니처가 추상 메서드와 일치해야 함**](#12-람다-표현식의-시그니처가-추상-메서드와-일치해야-함)
  - [**2. 매칭이 실패하는 경우**](#2-매칭이-실패하는-경우)
    - [**2.1. 추상 메서드가 두 개 이상 존재할 경우**](#21-추상-메서드가-두-개-이상-존재할-경우)
    - [**2.2. 람다 표현식의 시그니처 불일치**](#22-람다-표현식의-시그니처-불일치)
    - [**2.3. 반환 타입 불일치**](#23-반환-타입-불일치)
  - [**3. 메서드 레퍼런스와의 매칭**](#3-메서드-레퍼런스와의-매칭)
  - [**4. 타입 추론과 명시적 형변환**](#4-타입-추론과-명시적-형변환)
  - [**5. `FunctionalInterface`의 대표적인 예시**](#5-functionalinterface의-대표적인-예시)
    - [**예제: `Function<T, R>` 사용**](#예제-functiont-r-사용)

---

Java의 **Functional Interface**(함수형 인터페이스)는 단 하나의 추상 메서드(SAM, **Single Abstract Method**)를 가지는 인터페이스입니다. 이러한 인터페이스를 구현하는 람다 표현식이 올바르게 매칭되려면 몇 가지 기준이 있습니다.

## **결론**

### **람다가 함수형 인터페이스와 매칭되는 기준**

1. **추상 메서드가 정확히 하나(Single Abstract Method, SAM)**여야 한다.
2. **람다의 매개변수 타입 및 개수**가 추상 메서드와 동일해야 한다.
3. **람다의 반환 타입**이 추상 메서드의 반환 타입과 일치해야 한다.
4. **메서드 레퍼런스와 클래스 레퍼런스도 람다와 동일한 방식으로 매칭된다.**
5. **타입 추론이 가능하지만, 명시적 형변환이 필요한 경우가 있다.**

---

## **1. 매칭 기준**

### **1.1. 단일 추상 메서드(SAM, Single Abstract Method) 여부**

- 인터페이스 내 **추상 메서드가 하나만 존재해야** 합니다.
- `@FunctionalInterface` 어노테이션은 필수는 아니지만, 선언하면 컴파일러가 단일 추상 메서드인지 검증합니다.

```java
@FunctionalInterface
interface MyFunction {
    int apply(int x); // 하나의 추상 메서드만 존재해야 함
}
```

### **1.2. 람다 표현식의 시그니처가 추상 메서드와 일치해야 함**

- 람다 표현식의 **매개변수 타입 및 개수**가 인터페이스의 추상 메서드와 동일해야 합니다.
- 람다 표현식의 **반환 타입**도 추상 메서드와 동일해야 합니다.

```java
MyFunction square = (x) -> x * x;  // apply(int x)의 구현
System.out.println(square.apply(5)); // 25
```

위 코드에서 `square`의 타입은 `MyFunction`이고, 람다 표현식 `(x) -> x * x`는 `apply(int x)` 메서드의 시그니처와 일치합니다.

---

## **2. 매칭이 실패하는 경우**

다음과 같은 경우는 매칭이 실패하여 컴파일 오류가 발생합니다.

### **2.1. 추상 메서드가 두 개 이상 존재할 경우**

```java
@FunctionalInterface
interface InvalidInterface {
    void method1();
    void method2(); // 오류: FunctionalInterface는 하나의 추상 메서드만 가능
}
```

**해결 방법**: 하나의 추상 메서드만 남기거나, 함수형 인터페이스로 사용할 수 없도록 변경해야 합니다.

---

### **2.2. 람다 표현식의 시그니처 불일치**

```java
@FunctionalInterface
interface StringToInt {
    int convert(String s);
}

// 올바른 예시
StringToInt valid = (s) -> s.length();

// 오류 예시 (매개변수 타입 불일치)
StringToInt invalid = (int x) -> x * 2; // String을 받아야 하는데 int를 받음
```

**해결 방법**: 인터페이스의 추상 메서드 시그니처와 동일하게 맞춰야 합니다.

---

### **2.3. 반환 타입 불일치**

```java
@FunctionalInterface
interface IntToString {
    String convert(int x);
}

// 올바른 예시
IntToString valid = (x) -> "Number: " + x;

// 오류 예시 (반환 타입 불일치)
IntToString invalid = (x) -> x * 2; // int 반환하지만, String 반환해야 함
```

**해결 방법**: 반환 타입을 올바르게 맞춰야 합니다.

---

## **3. 메서드 레퍼런스와의 매칭**

람다 표현식은 **메서드 레퍼런스**로 대체할 수도 있으며, 동일한 매칭 규칙이 적용됩니다.

```java
@FunctionalInterface
interface StringParser {
    int parse(String s);
}

// 람다 표현식 사용
StringParser lambdaParser = s -> Integer.parseInt(s);

// 메서드 레퍼런스 사용 (Integer.parseInt는 String을 받고 int 반환)
StringParser methodRefParser = Integer::parseInt;

System.out.println(methodRefParser.parse("100")); // 100
```

**조건**: `Integer::parseInt`는 `(String) -> int`의 형태를 가지므로, `parse(String s)` 메서드와 매칭됩니다.

---

## **4. 타입 추론과 명시적 형변환**

Java의 람다는 **타입 추론**을 지원하지만, 필요한 경우 명시적으로 형을 지정해야 할 수도 있습니다.

```java
// 타입 추론 가능
MyFunction func = x -> x + 10;

// 명시적 형 변환 필요 (인터페이스가 모호할 경우)
MyFunction func2 = (int x) -> x + 10;
```

타입 추론은 **컨텍스트에 따라 결정**됩니다. 예를 들어, 다중 인터페이스 중 하나로 람다를 적용할 경우 명시적 형변환이 필요할 수도 있습니다.

```java
interface A {
    int process(int x);
}

interface B {
    int process(int x);
}

// 모호성 오류 발생
// var myLambda = x -> x * 2; // 어떤 인터페이스인지 알 수 없음

// 해결 방법: 명시적 타입 지정
A myLambda = (A) (x -> x * 2);
```

---

## **5. `FunctionalInterface`의 대표적인 예시**

Java의 기본 제공 함수형 인터페이스는 `java.util.function` 패키지에 포함되어 있으며, 다음과 같은 것들이 있습니다.

| 인터페이스 | 메서드 시그니처 | 설명 |
|-----------|----------------|------|
| `Function<T, R>` | `R apply(T t)` | 입력 `T`를 받아 출력 `R` 반환 |
| `Consumer<T>` | `void accept(T t)` | 입력을 소비하고 반환값 없음 |
| `Supplier<T>` | `T get()` | 입력 없이 결과 생성 |
| `Predicate<T>` | `boolean test(T t)` | 입력 `T`를 받아 `boolean` 반환 |
| `BiFunction<T, U, R>` | `R apply(T t, U u)` | 두 개의 입력을 받아 하나의 결과 반환 |

### **예제: `Function<T, R>` 사용**

```java
import java.util.function.Function;

Function<String, Integer> lengthFunction = s -> s.length();
System.out.println(lengthFunction.apply("hello")); // 5
```
