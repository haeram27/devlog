# Java 접근 제어자(Access Modifier)

```java
class Example {
    int value;           // package-private (기본, 별도의 키워드 없음)
    public int pub;      // 어디서든 접근 가능
    protected int prot;  // 같은 패키지 + 하위 클래스
    private int priv;    // 해당 클래스 내부만
}
```

| 접근 제어자 | 같은 클래스 | 같은 패키지 | 하위 클래스 | 외부 패키지 |
|------------|:-:|:-:|:-:|:-:|
| `private` | O | X | X | X |
| `package-private`(default) | O | O | X | X |
| `protected` | O | O | O | X |
| `public` | O | O | O | O |