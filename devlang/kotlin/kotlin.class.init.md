## 클래스 초기화 블럭

초기화 실행 순서:
- companion object init {} : static inner class 디폴트 생성자 블럭, 클래스 당 단일 선언 가능
- init {} : 클래스 디폴트 생성자 블럭, 여러개 선언시 순서대로 호출
- constructor() {} : 클래스 서브 생성자 블럭, 여러개 선언시 순서대로 호출

### 예제
- code:

```kotlin
open class Parent(val name: String) { // 주 생성자

    companion object {
        init {
            println("0. Parent class companion object init")
        }
    }

    init {
        println("2. Parent class 1st init block: $name")
    }

    // 부 생성자: 추가 파라미터(age)를 받고, name은 주 생성자로 전달
    constructor(name: String, age: Int) : this(name) {
        println("4. Parent class 2nd constructor body (age: $age)")
    }

    init {
        println("3. Parent class 2nd init block")
    }
}

class Child(name: String) : Parent(name) { // 주 생성자

    companion object {
        init {
            println("1. Child class companion object init")
        }
    }

    init {
        println("5. Child class 1st init block: $name")
    }

    // 부 생성자: 추가 파라미터(age)를 받고, name은 주 생성자(this)로 전달
    // 주 생성자는 다시 부모의 주 생성자(Parent(name))를 호출합니다.
    constructor(name: String, age: Int) : this(name) {
        println("7. Child class 2nd constructor body (age: $age)")
    }

    init {
        println("6. Child class 2nd init block")
    }
}

fun main() {
    println("--- Starting Instance Creation via Secondary Constructor ---")
    // 부 생성자를 호출하여 'name'과 'age'를 전달
    val child = Child(name = "foo", age = 25)
}
```

- result:

```plain
--- Starting Instance Creation ---
1. Parent class companion object init
2. Child class companion object init
3. Parent class 1st init block
4. Parent class 2nd init block
5. Parent class 2nd constructor body
6. Child class 1st init block
7. Child class 2nd init block
8. Child class 2nd constructor body
```