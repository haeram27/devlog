# spring annotations

## lombok

### @AllArgsConstructor

- 주요 대상: 클래스
- 기능: 클래스의 모든 필드를 매개변수로 받는 생성자를 생성
- 사용 목적: 주로 의존성 주입이나 객체 생성 시 모든 필드를 초기화해야 할 때 사용

### @NoArgsConstructor

- 주요 대상: 클래스
- 기능: 기본 생성자(매개변수가 없는 생성자)를 생성
- 사용 목적: 주로 JPA 같은 ORM 프레임워크에서 엔티티를 생성할 때 필요하거나, 빈 객체를 생성해야 할 때 사용

### @RequiredArgsConstructor

- 주요 대상: 클래스
- 기능: `private final` 속성의 필드를 매개 변수로 받는 생성자를 생성
- 사용 목적: 필요한 필드만 매개변수로 받는 생성자를 생성

### @ToString

- 주요 대상: 클래스
- 기능: 클래스내 모든 필드를 human readble 형식으로 출력하는 toString() 메소드 생성