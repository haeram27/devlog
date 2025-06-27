# Instance Method Reference

참고:
[MethodReference](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

**Java에서 인스턴스 메서드 (`ClassName::instanceMethod`)에 대해 참조가 사용되면 첫 번째 인자(a)를 메소드의 인스턴스로 간주한다.**  
예:
`Integer::compareTo`를 `(a, b) -> a.compareTo(b)`로 해석

instance method란 class내에서 static이 아닌 멤버 메소드를 의미한다.


Table. Kinds of Method References

| Kind |Syntax | Examples |
|--- |--- |--- |
| Reference to a static method | containingClass::staticMethodName | Person::compareByAge <br> MethodReferencesExamples::appendStrings |
| Reference to an instance method of a particular object | containingObject::instanceMethodName | myComparisonProvider::compareByName <br> myApp::appendStrings2 |
| Reference to an instance method of an arbitrary object of a particular type | ContainingType::methodName | String::compareToIgnoreCase <br> String::concat |
| Reference to a constructor | ClassName::new | HashSet::new |

## 예: Comparator interface를 타입에 Integer의 compare/compareTo 메소드 모두 맵핑 가능

다음 두 라인의 메소드 레퍼런스는 모두 유효하다.
Integer::compare와 달리 Integer::compareTo는 Comparator::compare와 메소드 시그니처가 다른데 어떻게 정상적으로 맵핑이 되는가?
이를 설명하는 것이 instance method reference이다.

```java
Stream.of(1,2,3).max(Integer::compare).orElse(0);
Stream.of(1,2,3).max(Integer::compareTo).orElse(0);
```

```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
    Optional<T> min(Comparator<? super T> comparator);
    Optional<T> max(Comparator<? super T> comparator);
}
```

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

```java
public final class Integer extends Number
        implements Comparable<Integer>, Constable, ConstantDesc {

    public int compareTo(Integer anotherInteger) {
        return compare(this.value, anotherInteger.value);
    }

    public static int compare(int x, int y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }
}
```

---

## 핵심 규칙: `ClassName::instanceMethod`는 `(T t, U u) -> t.instanceMethod(u)`로 해석된다

자바에서 다음과 같은 메서드 참조가 있을 때:

```java
SomeType::instanceMethod
```

이것은 **해당 메서드를 갖는 객체의 인스턴스를 첫 번째 매개변수로 받아서 그 객체에 대해 메서드를 호출하는 람다식**으로 변환됩니다.

즉:

```java
Integer::compareTo
```

는 실제로는:

```java
(a, b) -> a.compareTo(b)
```

로 해석됩니다.

---

## 예시로 설명

### 1. 일반 람다 표현식

```java
Comparator<Integer> cmp = (a, b) -> a.compareTo(b);
```

### 2. 동일한 기능의 메서드 레퍼런스

```java
Comparator<Integer> cmp = Integer::compareTo;
```

컴파일러는 이 문맥에서 `Integer::compareTo`가 **두 개의 Integer 인자를 받아**, 첫 번째 인자에 대해 `compareTo()`를 호출해야 한다는 것을 파악합니다.
왜냐하면 `Comparator<T>`는 다음과 같은 함수형 인터페이스이기 때문입니다:

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

이 타입에 맞춰 `Integer::compareTo`는 자동으로 다음과 같이 변환됩니다:

```java
(a, b) -> a.compareTo(b)
```

---

## 기술적 근거: 메서드 레퍼런스의 대상 타입(Contextual Target Typing)

Java의 메서드 레퍼런스는 \*\*타겟 타입(target type)\*\*에 따라 해석됩니다.
`Integer::compareTo`는 여러 방식으로 해석될 수 있지만, **해당 위치(예: `Stream.min(...)`)에서 요구하는 함수형 인터페이스의 매칭 시그니처와 비교해서 일치하는 방식으로 해석**합니다.

따라서:

* `Integer::compareTo`는 `Comparator<Integer>`처럼 `(Integer a, Integer b) -> int`인 문맥에서
* `a.compareTo(b)`로 **자동 변환됩니다.**

---

## 정리

| 표현                   | 해석 방식                                     | 이유                                                             |
| --- | --- | --- |
| `Integer::compareTo` | `(a, b) -> a.compareTo(b)` | `Comparator<Integer>`는 `(T, T) -> int` 시그니처를 요구하므로, 해당 방식으로 해석 |
| 메서드 타입 | 인스턴스 메서드 참조 (`ClassName::instanceMethod`) | 컴파일러가 첫 번째 인자를 대상 객체로 매핑 |

---
