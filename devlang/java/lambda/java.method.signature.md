# **메서드 시그니처(Method Signature)란?**

- [**메서드 시그니처(Method Signature)란?**](#메서드-시그니처method-signature란)
  - [요약](#요약)
    - [메소드 시그니처](#메소드-시그니처)
    - [람다 시그니처](#람다-시그니처)
  - [**1. 메서드 시그니처의 예시**](#1-메서드-시그니처의-예시)
    - [**위 코드에서 메서드 시그니처**](#위-코드에서-메서드-시그니처)
  - [**2. 메서드 시그니처와 오버로딩**](#2-메서드-시그니처와-오버로딩)
  - [**3. 메서드 시그니처와 오버라이딩**](#3-메서드-시그니처와-오버라이딩)
  - [**4. 메서드 시그니처에 포함되지 않는 요소**](#4-메서드-시그니처에-포함되지-않는-요소)
    - [**반환 타입**](#반환-타입)
    - [**`throws` 예외**](#throws-예외)
  - [**결론**](#결론)

---

## 요약

시그니처란 메소드나 람다를 고유하게 식별하는 기준

시그니처를 기준으로 메소드의 중복 선언 여부(오버로딩 등)나 람다의 매칭 허용 여부(메소드 참조)를 판단함

### 메소드 시그니처

- 메소드의 이름
- 파라미터 목록(타입, 개수, 순서)의 조합
- 미포함 항목: 반환 타입, 접근 제어자

```java
void print(int a, String b) -> 시그니처: print(int, String)
```

### 람다 시그니처

- 파라미터 목록(타입, 개수, 순서)의 조합
- 반환 타입

```java
BiConsumer<String, Integer> -> 시그니처: (String, Integer) -> void
BiConsumer<Integer, String> -> 시그니처: (Integer, String) -> void
```

## **1. 메서드 시그니처의 예시**

```java
public class Example {
    public void greet(String name) { } // 시그니처: greet(String)
    public int add(int a, int b) { return a + b; } // 시그니처: add(int, int)
    public double multiply(double x, double y) { return x * y; } // 시그니처: multiply(double, double)
}
```

### **위 코드에서 메서드 시그니처**

- `greet(String)`
- `add(int, int)`
- `multiply(double, double)`

---

## **2. 메서드 시그니처와 오버로딩**

**오버로딩(Overloading)**은 같은 이름을 가진 메서드를 **매개변수 리스트(타입, 개수, 순서)** 를 다르게 하여 정의하는 것을 말합니다.  

```java
public class OverloadingExample {
    public void display(int a) { }        // 시그니처: display(int)
    public void display(double a) { }     // 시그니처: display(double)
    public void display(int a, int b) { } // 시그니처: display(int, int)
}
```

 **각 메서드 시그니처가 다르므로 오버로딩이 가능**합니다.

---

## **3. 메서드 시그니처와 오버라이딩**

**오버라이딩(Overriding)** 시에는 **메서드 시그니처가 반드시 동일**해야 합니다.

```java
class Parent {
    void show(int x) { }  // 시그니처: show(int)
}

class Child extends Parent {
    @Override
    void show(int x) { } // 오버라이딩: 시그니처 동일
}
```

다음 코드는 오류가 발생합니다.

```java
class Child extends Parent {
    @Override
    int show(int x) { return x; } // 오류: 반환 타입이 다르면 오버라이딩 불가능
}
```

 **반환 타입은 시그니처에 포함되지 않지만, 오버라이딩 시에는 일치해야 합니다.**

---

## **4. 메서드 시그니처에 포함되지 않는 요소**

### **반환 타입**

```java
class Example {
    public int getValue() { return 0; }
    public double getValue() { return 0.0; } // 오류 (시그니처 중복)
}
```

**반환 타입만 다르면 중복된 메서드로 간주됨** → 메서드 오버로딩이 불가능.

### **`throws` 예외**

```java
class Example {
    public void method(int x) throws IOException { }
    public void method(int x) throws SQLException { } // 오류 (시그니처 동일)
}
```

`throws`는 메서드 시그니처에 포함되지 않음 → 같은 클래스 내에서 같은 이름, 같은 매개변수를 가지면 중복 오류 발생.

---

## **결론**

| 포함 여부 | 요소 | 예시 |
|----------|------|------|
| ✅ 포함 | 메서드 이름 | `add` |
| ✅ 포함 | 매개변수 타입과 개수 | `add(int, int)` vs `add(double, double)` |
| ❌ 미포함 | 반환 타입 | `int add(int, int)` vs `double add(int, int)` (중복 오류 발생) |
| ❌ 미포함 | `throws` 예외 | `method(int) throws IOException` vs `method(int) throws SQLException` (중복 오류 발생) |

즉, **메서드 시그니처는 "같은 클래스 내에서 메서드를 구별하는 요소"이며, 메서드 이름과 매개변수 타입/개수/순서만 고려된다**는 점이 핵심입니다.
