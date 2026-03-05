# java record

- JAVA 14 부터 지원
- Immutable VO(Value Obejct)를 정의하는 키워드

- 내용
  - 모든 인스턴스 변수는 final
  - public static 변수와 메소드 선언 가능

선언:

```java
public record Person (String name, Integer age) {}
```

사용:

```java
Person person = new Person("Joe", 22);
```

## static variable/method 정의

```java
public record Person (String name, Integer age) {
    public static String HELLO = "hello";
    public static Person foo(Integer age) {
        return new Person ("foo", age);
    }
}
```

## 인스턴스 생성시 자동 생성 항목

- getter()
  - 각 필드의 이름과 동일한 getter 메소드
- public constructor
  - 사용자 정의도 가능
- equals()
- hashCode()
- toString()

## 사용자 정의 가능 항목

- public constructor
- static variable
- static method

## 제한 사항

- 다른 클래스를 상속 불가능
- abstract class 로 정의 불가능
- static 을 제외한 다른 field와 method 는 사용자 정의할 수 없음