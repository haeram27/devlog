# Method References Example(TODO)


메소드 참조가 함수형 인터페이스로 자동 변환되기 때문입니다.

JavaHttpStatusCode::is2xxSuccessful 은 다음과 같이 동작합니다:

1. HttpStatusCode 인터페이스에 정의된 메소드 시그니처:

```java
boolean is2xxSuccessful();
```

2. 메소드 참조 변환:
`HttpStatusCode::is2xxSuccessful` 은 다음처럼 해석됩니다:

```java
(HttpStatusCode statusCode) -> statusCode.is2xxSuccessful()
```

3. Predicate 매칭:
```java
// Predicate<HttpStatusCode>의 추상 메소드
boolean test(HttpStatusCode t);

// 메소드 참조의 시그니처
(HttpStatusCode) -> boolean
```

두 시그니처가 일치하므로, Java 컴파일러가 자동으로 메소드 참조를 Predicate로 변환합니다.

핵심 개념

메소드 참조는 3가지 형태가 있습니다:

1. **Static method reference**

   ```java
   ClassName::staticMethod
   // (param) -> ClassName.staticMethod(param)
   ```

2. **Instance method reference (bound)**

   ```java
   instance::instanceMethod
   // () -> instance.instanceMethod()
   ```

3. **Instance method reference (unbound)**

   ```java
   ClassName::instanceMethod
   // (instance, param) -> instance.instanceMethod(param)
   ```

현재 경우는 **3번 unbound** 형태입니다:

- `HttpStatusCode::is2xxSuccessful`은 첫 번째 파라미터로 HttpStatusCode 인스턴스를 받습니다
- 결과: `(HttpStatusCode statusCode) -> statusCode.is2xxSuccessful()`
- 이것은 `Predicate<HttpStatusCode>`의 `test(HttpStatusCode t)` 시그니처와 정확히 일치합니다

따라서 "형식이 다르다"고 보이지만, Java의 메소드 참조 변환 규칙에 의해 완벽하게 호환되는 것이 정상입니다.

## Instance method reference에서 bound와 unbound의 차이

**어떤 인스턴스에 메소드가 묶여(bound) 있는지** 여부의 차이입니다.

---

## Bound (특정 인스턴스에 묶임)

```java
String str = "hello";
Supplier<String> s = str::toUpperCase;
// () -> str.toUpperCase()
```

- 메소드 참조 생성 시점에 이미 **특정 인스턴스(`str`)가 고정**됩니다.
- 나중에 어떤 인스턴스를 쓸지 고민할 필요 없이, 항상 `str`에 대해 실행됩니다.
- 함수형 인터페이스에서 **인스턴스 파라미터가 필요 없습니다**.
  - `Supplier<String>` → `String get()` (파라미터 없음)

---

## Unbound (인스턴스가 묶이지 않음)

```java
Function<String, String> f = String::toUpperCase;
// (String s) -> s.toUpperCase()
```

- 어떤 인스턴스에 실행할지가 **호출 시점에 결정**됩니다.
- 함수형 인터페이스에서 **첫 번째 파라미터가 인스턴스 역할**을 합니다.
  - `Function<String, String>` → `String apply(String s)` (인스턴스를 파라미터로 받음)

---

## 비교 요약

| 구분 | 형태 | 인스턴스 결정 시점 | 함수형 인터페이스 |
|---|---|---|---|
| Bound | `instance::method` | 메소드 참조 생성 시 고정 | 인스턴스 파라미터 없음 |
| Unbound | `ClassName::instanceMethod` | 호출 시 첫 파라미터로 전달 | 첫 파라미터가 인스턴스 |

---

## 앞서 나온 실제 예시

```java
// Unbound - HttpStatusCode 인스턴스가 호출 시점에 Predicate의 파라미터로 전달됨
Predicate<HttpStatusCode> p = HttpStatusCode::is2xxSuccessful;
// (HttpStatusCode code) -> code.is2xxSuccessful()

// 만약 Bound였다면 (특정 인스턴스 고정)
HttpStatusCode specific = HttpStatus.OK;
Supplier<Boolean> s = specific::is2xxSuccessful;
// () -> specific.is2xxSuccessful()
```

한 줄 요약:

- **Bound** = "이 인스턴스로 실행해"
- **Unbound** = "어떤 인스턴스로 실행할지는 나중에 알려줘"

