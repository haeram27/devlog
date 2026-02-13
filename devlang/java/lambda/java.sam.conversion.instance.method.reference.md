# Instance Method Reference

참고:
[MethodReference](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

- **Method Reference**란 `FunctionalInterface`를 인자로 받는 위치에 임의의 클래스의 메서드(`ClassName::Method`)를 사용하는 것을 의미한다.
- **Instance method Reference**란 `Method References` 사용 시 `instance method`를 사용한 것을 의미한다.
- **instance method**란 `class내에서 static이 아닌 멤버 메소드`를 의미한다.
- ***중요:*** **instance method Reference**는 컴파일러에서 `SAM 변환`시 첫 번째 인자(a)를 `instanceMethod`의 클래스 인스턴스로 간주한다. 그래서 Functional Interface의 SAM에서 요구 되는 **인자의 수가 맞지 않아도 `insatanceMethod`를 파리미터로 전달할 수 있게 된다.**
  - 예: Comparator Interface를 인자 위치에 `Integer::compareTo` 사용
    - Comparator Interface는 `int compare(T o1, T o2);`의 SAM을 갖는 반면`Integer::compareTo`는 `int compareTo(Integer anotherInteger)` 형식을 갖는다. 이때 컴파일러는 `Integer::compareTo`를 그대로 사용하는 대신에 Comporator Interface에 맞추어 `(Integer o1, Integer o2) -> { o1.compareTo(o2) }` 형식으로 SAM에 변환을 자동으로 수행하여 인터페이스 구현체 지정해 준다.
    - JAVA 컴파일러는 `SAM 변환`에 의해서 Comparator Interface를 인자 위치에 `Integer::compare(o1, o2)`와 `Integer::compareTo(o1)`를 모두 사용할 수 있도록 지원한다.

Table. Kinds of Method References

| Kind | Syntax | Examples |
| --- | --- | --- |
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

- `Integer::compareTo`는 `Comparator<Integer>`처럼 `(Integer a, Integer b) -> int`인 문맥에서
- `a.compareTo(b)`로 **자동 변환됩니다.**

---

## 정리

| 표현                   | 해석 방식                                     | 이유                                                             |
| --- | --- | --- |
| `Integer::compareTo` | `(a, b) -> a.compareTo(b)` | `Comparator<Integer>`는 `(T, T) -> int` 시그니처를 요구하므로, 해당 방식으로 해석 |
| 메서드 타입 | 인스턴스 메서드 참조 (`ClassName::instanceMethod`) | 컴파일러가 첫 번째 인자를 대상 객체로 매핑 |

---
