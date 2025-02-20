# [NUll Safety](https://kotlinlang.org/docs/null-safety.html)
---
- [NUll Safety](#null-safety)
  - [? (nullable)](#-nullable)
  - [?. (safe call)](#-safe-call)
  - [?: (elvis operator)](#-elvis-operator)
  - [!! (not NULL assertion Operator)](#-not-null-assertion-operator)

---


## ? ([nullable](https://kotlinlang.org/docs/null-safety.html#nullable-types-and-non-nullable-types)) 
syntax :
```
<type>?
```

example : 
```kotlin
val str: String? = "ABC"
```


## ?. ([safe call](https://kotlinlang.org/docs/null-safety.html#safe-call-operator))
run `<member>` if `<nullable-var>` is ***NOT*** null, throw NPE if `<nullable-var>` is null.

syntax :
```
<nullable-var>?.<member>
```

example :
```kotlin
val str: String? = "ABC"
str.length   // error
str?.length  // ok
```


## ?: ([elvis operator](https://kotlinlang.org/docs/null-safety.html#elvis-operator))
use `<value>` if `<value-expr>` is null

syntax :
```
<value-expr> ?: <value>
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


## !! ([not NULL assertion Operator](https://kotlinlang.org/docs/null-safety.html#not-null-assertion-operator))
suppress NPE warning.  
compiler handles `<nullable-var>!!` as non-nullable.  
if `<nullable-var>!!` is null then NPE occurs.

example :
```kotlin
fun starsWith(str: String?): Boolean {
    return str!!.startsWith("A")
}
```