# Method References Example

[Oracle Java 튜토리얼: 메서드 참조](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

## Method Reference (메서드 참조)란

- `@FuntionalInterface`는 람다를 담는 타입이다.
- `@FuntionalInterface`를 타입으로 하는 파라미터에는 `@FuntionalInterface`의 SAM(Single Abstract Method)과 동일한 람다 시그니처를 갖는 람다를 할당할 수 있다. 
- `@FuntionalInterface` 타입의 파라미터에 람다 대신 클래스의 메소드를 지정 할 수 있으며, 이 문법을 `Method Reference`라고 한다.
- `Method Reference`는 컴파일러가 메소드를 람다 인스턴스로 변환해 주기 때문에 가능하다.
- ***`메서드 참조`를 람다 표현식으로 변환했을때 그 람다 시그니처가 `@FuntionalInterface`의 람다 시그니처와 같으면 유효한 문법이 된다.***
- 람다 시그니처는 파라미터 목록(개수, 순서, 타입)과 반환 타입이다
   ```text
      (param1-type, param2-type, ...) -> return-type
   ```

- `@FunctionalInterface` 예제
   ```java
   @FunctionalInterface
   public interface Function<T, R> {

      /**
       * Applies this function to the given argument.
       *
       * @param t the function argument
       * @return the function result
       */
      R apply(T t);
   }

   // Function의 람다 시그니처 `(T) -> R`
   ```

## `@Functional Interface`와 Method Reference에 사용된 메소드의 매칭 허용 규칙

- `@Functional Interface`의 람다 시그니처와 메소드의 람다 변환 결과 비교하여 두 람다의 시그니처가 동일하면 메소드 참조가 허용된다.
- 람다 시그니처의 매칭 비교 기준은 파라미터 목록(개수, 순서, 타입), 반환 타입이다.
- 주의: 단, `@Functional Interface`의 SAM의 반환 값이 void인 경우 참조에 사용된 메소드의 반환 타입은 무시되며, void로 처리될 수 있다.

## 메소드 참조의 형태

메소드 참조에 사용된 메소드에 대해 컴파일러는 다음과 같이 변환을 한다.

1. **Constructor method reference**
   ```java
   ClassName::new
   // 람다 변환: () -> new ClassName()
   ```
   - 주의: 인자를 받는 생성자의 경우 `메서드 참조` 대신 람다 표현을 사용해야 한다.
   ```java
   인자가 있는 생성자 호출 람다식:
   (param) -> new ClassName(param);
   (n) -> new int[n];
   ```

2. **Static method reference**

   ```java
   ClassName::staticMethod(param)
   // 람다 변환: (param) -> ClassName.staticMethod(param)
   ```

3. **Bound Instance method reference**
   - 인스턴스의 메소드 지정
   ```java
   instance::instanceMethod(param)
   // 람다 변환: (param) -> instance.instanceMethod(param)
   ```

4. **Unbound Instance method reference**
   - 인스턴스가 아닌 class 또는 type의 메소드 지정
   - 주의: 언바운드 메소드 레퍼런스 사용시 함수형 인터페이스(람다)의 첫 파라미터로 클래스의 인스턴스가 전달되며, 첫 인자로 전달되는 인스턴스를 파라미터를 `리시버` 라고 함
   - `Unbound Instance method reference`는 주로 첫 번째 파라미터를 어떤 객체의 인스턴스로 받는 `@FunctionalInterface`에 사용된다.
   - 대부분의 경우 단일 파라미터(이 파라미터가 객체의 인스턴스)를 받는 `@FunctionalInterface`에 사용되는 경우가 많다.
   - `리시버` 때문에 **함수형 인터페이스에 선언된 파라미터 개수는 언바운드 메서드 레퍼런스의 파라미터 수보다 항상 1개 많다.** (함수형 인터페이스의 첫번째 인자에는 메서드의 인스턴스를 지정해야 하므로)
   - 만약 함수형 인터페이스의 파라미터가 3개라면 파라미터 2개인 언바운드 메서드 레퍼런스만 대입이 가능하다.
   ```java
   ClassName::instanceMethod()
   // 람다 변환: (instance) -> instance.instanceMethod()
   ClassName::instanceMethod(param)
   // 람다 변환: (instance, param) -> instance.instanceMethod(param)
   ```

- Instance method reference에서 bound와 unbound의 차이는 ** 인스턴스에 메소드가 묶여(bound) 있는지** 여부의 차이다

### Bound/Unbound Instance Method Reference 비교

| 구분 | 형태 | 인스턴스 결정 시점 | 함수형 인터페이스 |
|---|---|---|---|
| Bound | `instance::method` | 메소드 참조 생성 시 고정 | 인스턴스 파라미터 없음 |
| Unbound | `ClassName::instanceMethod` | 호출 시 첫 파라미터로 전달 | 함수형 인터페이스의 첫 파라미터에 인스턴스가 전달됨|

## 함수형 인터페이스 타입과 메소드 참조의 매칭 기준(시그니처)

컴파일러는 `함수형 인터페이스`와 `메소드 참조`를 파라미터 타입과 반환 타입을 매칭의 기준으로 사용한다. 이 매칭의 기준이 되는 형식을 `시그니처`라고 한다. 람다 표현식이 파라미터와 반환값을 나타내는 표현식이란 것을 생각해보면 매칭의 시그니처가 람다 형식과 유사한것이 이해가 간다.

***결국, `메서드 참조`를 람다 형식으로 변환했을때 시그니처가 `함수형 인터페이스`와 같으면 유효한 문법이 된다.***

다음의 예제로 확인해보면,

```java
@FunctionalInterface
public interface Predicate<T> {
   boolean test(T t);
}
// 람다 시그니처: (T) -> boolean
```

```java
public sealed interface HttpStatusCode {
   boolean is2xxSuccessful();
}
// 람다 변환: (HttpStatusCode instance) -> instance.is2xxSuccessful()
// 람다 시그니처: (HttpStatusCode) -> boolean
```

```java
// `@FunctionalInterface` Predicate를 파라미터로 받는 메소드 onStatus() 정의
interface ResponseSpec {
   ResponseSpec onStatus(Predicate<HttpStatusCode> statusPredicate);
}
```

```java
// Unbound Instance method reference 사용
refSpec.onStatus(HttpStatusCode::is2xxSuccessful());
```

이 예제는 `Predicate<HttpStatusCode>` 타입의 `statusPredicate`파라미터에 `HttpStatusCode::is2xxSuccessful()` 메서드 참조를 사용하는 예제이다.

- `statusPredicate` 파라미터의 타입인 `Predicate<HttpStatusCode>` 의 시그니처는 `(HttpStatusCode) -> boolean` 이다.
- `HttpStatusCode::is2xxSuccessful()`의 메서드 참조 시그니처는 `(HttpStatusCode) -> boolean` 이다. 이 시그니처의 인자는 리시버(HttpStatusCode의 인스턴스)이다.
- 두 시그니처가 같으므로 `statusPredicate` 파라미터에 `HttpStatusCode::is2xxSuccessful()` 메서드 레퍼런스 사용이 가능하다.

### SAM의 반환 타입이 void인 경우, Method Reference의 반환 타입이 무시되며 할당 가능함

- `Consumer` 함수형 인터페이스에 Method Reference 사용 예제
- `Consumer` 함수형 인터페이스 타입 파라 미터에 `builder::vertion`을 사용

```java
// java.net.http.HttpRequest$Builder.class
public static Builder newBuilder(HttpRequest request, BiPredicate<String, String> filter) {
   ...
   request.version().ifPresent(builder::version);
   ...
}
```

```java
   // Optional.class
   private final T value;
   public void ifPresent(Consumer<? super T> action) {
      if (value != null) {
         action.accept(value);
      }
   }
```

- `Consumer` Functional Interface
```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    // SAM lambda: t -> void
}
```

- Method Reference로 사용된 `HttpRequest$Builder::version(v)` 메소드

```java
// java.net.http.HttpRequest$Builder.class
   public Builder version(HttpClient.Version version);

// method reference lambda(bounded): v -> builder.version(v)
```

- 파라미터는 개수와 타입이 타깃 SAM의 파라미터와 호환되어야 합니다(상속, 박싱/언박싱, 원시↔래퍼 등 허용). 바운드 참조(instance::method)의 경우 리시버는 이미 고정되어 SAM의 파라미터와 직접 매칭됩니다.
- 함수형 인터페이스(SAM)의 반환 타입과 참조된 메서드의 반환 타입은 서로 할당/변환 가능해야 합니다.
단, SAM의 반환 타입이 void인 경우(예: Consumer.accept) 참조된 메서드가 값을 반환하더라도 그 반환값은 무시되므로 허용됩니다. 그래서 `Builder.version(Version) -> Builder`가 `Consumer<Version>`에 들어갈 수 있습니다.
