# Anonymous Class (TODO)

Java의 **익명 클래스(Anonymous Class)** 는 이름이 없는 1회용 클래스 구현체를 정의할 때 사용하는 문법입니다. 주로 `인터페이스나 추상 클래스를 바로 구현(extend/implement)하면서 인스턴스를 생성`해야 할 때 간편하게 쓰입니다.

단, JAVA 8+ 에서는 `SAM(Single Abstract Method) Conversion` 이 도입되어 Functional Interface(단 하나의 추상메소드를 갖는 인터페이스)에 대해서 lambda expressino으로 인스턴스 생성이 가능하다. 그러므로 `Anonymous Class 방식의 인스턴스 생성은 최신 JAVA에서는 잘 사용되지 않는다`.

---

## 1. 익명 클래스의 정의

익명 클래스는  

- 기존 클래스나 인터페이스를 **상속(extends)** 또는 **구현(implements)**  
- 클래스 몸체(`{ … }`)를 즉석에서 작성  
- 이름 없이(`new Type() { … }`) 바로 인스턴스를 생성  

하는 형태로 표현됩니다.

```java
// 인터페이스 Runnable 익명 클래스 구현
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("익명 클래스 실행");
    }
};
new Thread(r).start();
```

위 예제에서 `new Runnable() { … }` 은  

1. `Runnable` 인터페이스를 구현하는 **이름 없는 클래스**를  
2. 즉시 정의하고,  
3. 그 객체를 생성하여 `r` 에 할당  

하는 코드입니다.

---

## 2. 기본 문법

```java
new SuperType(constructorArgs…) {
    // 필드 정의 가능
    private int count = 0;

    // 메서드 오버라이드 또는 추가
    @Override
    public void someMethod() {
        // 구현부
    }
    
    // 추가 메서드
    public void helper() { … }
};
```

- **SuperType** 은 `인터페이스` 또는 `클래스`  
- 괄호 `(…)` 안에는 슈퍼클래스의 생성자 인자(추상 인터페이스는 빈 괄호)  
- `{ … }` 블록 안에 **오버라이드할 메서드**, **필드**, **추가 메서드** 등을 정의 가능

---

## 3. 예시

### 3.1 인터페이스 구현

```java
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("버튼 클릭됨");
    }
});
```

### 3.2 추상 클래스 상속

```java
abstract class Animal {
    abstract void sound();
}

Animal cat = new Animal() {
    @Override
    void sound() {
        System.out.println("야옹");
    }
};
cat.sound();
```

---

## 4. 익명 클래스의 특징

- **한 번만** 사용할 구현체에 적합  
- **추가 필드**와 **여러 메서드** 정의 가능  
- 람다식보다 **더 많은 자유도** (여러 메서드, 상태 보유)  
- 다만 **코드가 장황**해질 수 있음

---

## 5. 람다와의 차이

| 익명 클래스                          | 람다식                        |
|------------------------------------|----------------------------|
| 여러 메서드 오버라이드 가능            | 함수형 인터페이스 1개 메서드만  |
| `this` 는 **익명 클래스 자신** 참조      | `this` 는 **바깥 클래스** 참조    |
| 필드와 생성자 정의 가능                | 필드 정의 불가, 선언만 가능     |
| 문법이 비교적 **장황**                 | 문법이 **간결**                |

익명 클래스는 복잡한 커스터마이징이 필요할 때, 람다는 단일 로직을 간단히 표현할 때 주로 선택하면 좋습니다.
