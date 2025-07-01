# [NUll Safety](https://kotlinlang.org/docs/null-safety.html)

- [NUll Safety](#null-safety)
  - [? (nullable)](#-nullable)
  - [?. (safe call)](#-safe-call)
  - [?: (elvis operator)](#-elvis-operator)
  - [!! (not NULL assertion Operator)](#-not-null-assertion-operator)

---

## ? ([nullable](https://kotlinlang.org/docs/null-safety.html#nullable-types-and-non-nullable-types))

purpose:
indicate variable CAN be `nullable`
***following three expressions(safe-call, elvis operator, not NULL asserttion) can be apply only for `nullable` object***

syntax :

```text
<type>?
```

example :

```kotlin
val str: String? = "ABC"
```

## ?. ([safe call](https://kotlinlang.org/docs/null-safety.html#safe-call-operator))

run `<member>` if `<nullable-var>` is ***NOT*** null, throw NPE if `<nullable-var>` is null.

purpose:
use `safe-call` for `<nullable-var>` and want NPE when `<nullable-var>` is null

syntax :

```text
<nullable-var>?.<member>
```

example :

```kotlin
val str: String? = "ABC"
str.length   // error
str?.length  // ok
```

---

## ?: ([elvis operator](https://kotlinlang.org/docs/null-safety.html#elvis-operator))

use `<value>` if `<value-expr>` is null

purpose:
`elvis operator` use default value when source is not valid
use `elvis operator` for `<nullable-var>.<value-expr >` and do NOT want `NPE` when `<nullable-var >` is null

syntax :

```text
<nullable-var>.<value-expr> ?: `<value>
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

compiler handles `<nullable-var>!!` as `non-nullable`.  
if `<nullable-var>!!` is null then `NPE` occurs.

purpose :
suppress `NPE` warning.  

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
