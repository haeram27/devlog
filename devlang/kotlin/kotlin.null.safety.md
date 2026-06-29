# [NUll Safety](https://kotlinlang.org/docs/null-safety.html)

1. [NUll Safety](#null-safety)
   1. [? (nullable)](#-nullable)
   2. [?. (safe call)](#-safe-call)
   3. [?: (elvis operator)](#-elvis-operator)
   4. [!! (not NULL assertion Operator)](#-not-null-assertion-operator)

---

- kotlin의 변수는 기본 적으로 Non-null 속성 가짐
- `?` 는 Nullable 타입으로 변수 선언(연산자는 타입에 사용)
- `?.`, `?:`, `!!`은 Nullable 타입의 변수 참조시 null 핸들링 방법 지정(연산자는 변수에 사용), `?` 타입으로 생성된 변수에만 사용하는 연산자

## ? ([nullable](https://kotlinlang.org/docs/null-safety.html#nullable-types-and-non-nullable-types))

purpose:

- indicate variable CAN be `nullable`, kotlin basically every variable is `non-null`
- following three expressions ***(safe-call, elvis operator, not NULL asserttion)*** can be apply only for `nullable` object

syntax :

```text
<type>?
```

example :

```kotlin
val str: String? = "ABC"
```

## ?. ([safe call](https://kotlinlang.org/docs/null-safety.html#safe-call-operator))

action:

- run `<member>` if `<nullable-var>` is ***NOT*** null, throw NPE if `<nullable-var>` is null.

purpose:

- use `safe-call` for `<nullable-var>` and want NPE when `<nullable-var>` is null

syntax :

```text
<nullable-var>?.<member>
```

example :

```kotlin
val str: String? = "ABC"
str.length   // NPE error
str?.length  // ok
```

---

## ?: ([elvis operator](https://kotlinlang.org/docs/null-safety.html#elvis-operator))

action:

- use `<value>` if `<value-expr>` is null

purpose:

- `elvis operator` uses default value when source is not valid
- use `elvis operator` for `<nullable-var>.<value-expr>` and do NOT want `NPE` when `<nullable-var >` is null

syntax :

```text
<nullable-var>.<value-expr> ?: <defaul-value>
```

example :

```kotlin
val str: String? = "ABC"
str?.length ?: 0
```

example :

```kotlin
fun starsWith(str: String?): Boolean {
    return str?.startsWith("A") ?: false
}
```

---

## !! ([not NULL assertion Operator](https://kotlinlang.org/docs/null-safety.html#not-null-assertion-operator))

action:

- compiler handles `<nullable-var>!!` as `non-nullable`.  
- if `<nullable-var>!!` is null then `NPE` occurs.

purpose :

- suppress `NPE` warning.

syntax :

```plain
<nullable-var>!!
```

example :

```kotlin
fun starsWith(str: String?): Boolean {
    return str!!.startsWith("A")
}
```
