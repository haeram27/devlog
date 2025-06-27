# Method References (TODO)
[Oracle Java 튜토리얼: 메서드 참조](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

람다 표현식을 사용하면 익명 메서드를 생성할 수 있습니다. 하지만 때로는 람다 표현식이 기존 메서드를 호출하는 것 외에는 아무 작업도 하지 않을 때가 있습니다. 이러한 경우 기존 메서드를 이름으로 참조하는 것이 더 명확할 수 있습니다. 메서드 참조를 사용하면 이미 이름이 있는 메서드에 대해 간결하고 읽기 쉬운 람다 표현식을 작성할 수 있습니다.

예를 들어, 이전에 람다 표현식 섹션에서 다룬 `Person` 클래스를 다시 살펴보겠습니다:

```java
public class Person {
    LocalDate birthday;

    public int getAge() {
        // ...
    }

    public LocalDate getBirthday() {
        return birthday;
    }   

    public static int compareByAge(Person a, Person b) {
        return a.birthday.compareTo(b.birthday);
    }
}
```

소셜 네트워킹 애플리케이션의 회원들이 배열에 저장되어 있고, 이 배열을 나이순으로 정렬하고 싶다고 가정해 봅시다. 다음과 같은 코드를 사용할 수 있습니다:

```java
Person[] rosterAsArray = roster.toArray(new Person[roster.size()]);

class PersonAgeComparator implements Comparator<Person> {
    public int compare(Person a, Person b) {
        return a.getBirthday().compareTo(b.getBirthday());
    }
}

Arrays.sort(rosterAsArray, new PersonAgeComparator());
```

`sort` 메서드의 시그니처는 다음과 같습니다:

```java
static <T> void sort(T[] a, Comparator<? super T> c)
```

`Comparator` 인터페이스는 함수형 인터페이스이므로, `Comparator`를 구현하는 새 클래스를 정의하고 인스턴스를 생성하는 대신 람다 표현식을 사용할 수 있습니다:([Oracle Docs](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html?utm_source=chatgpt.com "Default Methods - Interfaces and Inheritance - Oracle Help Center"))

```java
Arrays.sort(rosterAsArray,
    (Person a, Person b) -> {
        return a.getBirthday().compareTo(b.getBirthday());
    }
);
```

그러나 두 `Person` 인스턴스의 생일을 비교하는 메서드인 `Person.compareByAge`가 이미 존재하므로, 람다 표현식의 본문에서 이 메서드를 호출할 수 있습니다:

```java
Arrays.sort(rosterAsArray,
    (a, b) -> Person.compareByAge(a, b)
);
```

이 람다 표현식은 기존 메서드를 호출하므로, 람다 표현식 대신 메서드 참조를 사용할 수 있습니다:

```java
Arrays.sort(rosterAsArray, Person::compareByAge);
```

`Person::compareByAge` 메서드 참조는 람다 표현식 `(a, b) -> Person.compareByAge(a, b)`와 의미적으로 동일합니다. 각각의 특징은 다음과 같습니다:

- 형식 매개변수 목록은 `Comparator<Person>.compare`에서 복사된 `(Person, Person)`입니다.
    
- 본문은 `Person.compareByAge` 메서드를 호출합니다.
    

---

## 🔹 메서드 참조의 종류

메서드 참조에는 네 가지 종류가 있습니다:

|종류|문법|예시|
|---|---|---|
|정적 메서드 참조|`ContainingClass::staticMethodName`|`Person::compareByAge`|
|특정 객체의 인스턴스 메서드 참조|`containingObject::instanceMethodName`|`myComparisonProvider::compareByName`|
|특정 타입의 임의 객체의 인스턴스 메서드 참조|`ContainingType::methodName`|`String::compareToIgnoreCase`|
|생성자 참조|`ClassName::new`|`ArrayList::new`|

---

이러한 메서드 참조를 사용하면 코드가 더 간결하고 읽기 쉬워지며, 기존 메서드를 재사용하여 코드의 재사용성과 유지보수성을 향상시킬 수 있습니다.

---