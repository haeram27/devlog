# Java Singleton 구현

- 싱글톤은 class로 부터 단 하나의 인스턴스 생성만을 보장하여 동일 인스턴스 다중 생성으로 인한 메모리 낭비를 방지하는 클래스 정의 패턴
- `Enum 싱글톤` 방식 사용을 권장하며 상속 또는 인터페이스구현이 필요한 경우에만 `LazyHolderSingleton 패턴`을 사용

|항목|Enum 싱글톤|Lazy Holder 패턴|
|---|---|---|
|지연 로딩|o|o|
|Reflection을 이용한 인스턴스 중복 생성 가능|x|o|
|직렬화/역직렬화를 이용한 인스턴스 중복 생성 가능|x|o|
|상속/인터페이스 구현 가능|x|o|

- 지연 로딩: 인스턴스 참조 시점에 인스턴스 생성, 미리 인스턴스 생성을 하지 않아 어플리케이션 실행 초기에 메모리 낭비 방지
- 리플렉션과 직렬화/역직렬화를 통한 인스턴스 복제: `LazyHolderSingleton 패턴`은 이 인스턴스 복제 가능 하므로 방어 코드 추가 해야함
- `Enum 싱글톤` 방식은 상속이 불가해서 유연성이 떨어지지만, 클래스 싱글톤(`LazyHolderSingleton 패턴`)은 구조 설계가 자유로움

## Enum 싱글톤 (가장 권장)

````java
public enum EnumSingleton {
    INSTANCE;

    public void doSomething() {
        System.out.println("Doing something...");
    }
}
````

- EnumSingleton은 JVM이 직렬화/역직렬화 안전성과 단일 인스턴스 보장을 제공해 더 안전하고 간결함

## Lazy Holder 패턴 (지연 로딩, 스레드 안전)

기본형:

````java
// final: 상속 불가
public final class LazyHolderSingleton implements Serializable {
    // private: 외부에서 생성자 접근 불가
    private LazyHolderSingleton() {
    }

    // private: 외부에서 접근 불가
    // static inner class : 지연 초기화,`Holder` Inner class는 outter class가 로드될 때 인스턴스화 되지 않음, 직접 호출되었을 때만 인스턴스화 됨
    private static class Holder {
        private static final LazyHolderSingleton INSTANCE = new LazyHolderSingleton();
    }

    public static LazyHolderSingleton getInstance() {
        return Holder.INSTANCE;
    }
}
````

방어형:

- 리플렉션과 직렬화/역직렬화를 통한 인스턴스 중복 생성을 방어하는 버전

````java
public final class LazyHolderSingleton implements Serializable {
    private LazyHolderSingleton() {
        // 리플렉션 방어
        if (Holder.INSTANCE != null) {
            throw new IllegalStateException("Already initialized");
        }
    }

    private static class Holder {
        private static final LazyHolderSingleton INSTANCE = new LazyHolderSingleton();
    }

    public static LazyHolderSingleton getInstance() {
        return Holder.INSTANCE;
    }

    // 역직렬화 시 기존 인스턴스 반환
    private Object readResolve() {
        return Holder.INSTANCE;
    }
}
````

## Lazy Holder 패턴 우회(breaking) 방법

### 리플렉션을 이용한 LazyHolderSingleton 중복 인스턴스 생성

```java
// 정상적인 인스턴스
LazyHolderSingleton instance1 = LazyHolderSingleton.getInstance();

// 리플렉션으로 private 생성자 접근
Constructor<LazyHolderSingleton> constructor = 
    LazyHolderSingleton.class.getDeclaredConstructor();
constructor.setAccessible(true);
LazyHolderSingleton instance2 = constructor.newInstance();

// 다른 인스턴스 생성됨!
System.out.println(instance1 == instance2); // false
```

### 직렬화/역직렬활를 이용한 LazyHolderSingleton 중복 인스턴스 생성

```java
// 직렬화
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("singleton.ser"));
out.writeObject(LazyHolderSingleton.getInstance());

// 역직렬화 - 새로운 인스턴스 생성!
ObjectInputStream in = new ObjectInputStream(new FileInputStream("singleton.ser"));
LazyHolderSingleton deserializedInstance = (LazyHolderSingleton) in.readObject();

// 다른 인스턴스!
System.out.println(LazyHolderSingleton.getInstance() == deserializedInstance); // false
```
