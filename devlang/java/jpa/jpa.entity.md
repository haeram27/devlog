# [JPA] 엔티티(Entity) 핵심 요약 노트

## 1. 엔티티(Entity) 개념

* **정의**: 데이터베이스 테이블과 1:1로 매핑되는 자바 클래스
* **역할**: SQL을 직접 작성하지 않고 객체 지향적으로 CRUD(생성, 조회, 수정, 삭제)를 수행하는 기본 단위

---

## 2. 엔티티 작성 규칙 (필수 제약사항)

* ⚠️ **기본 생성자 필수**: JPA가 리플렉션을 사용하므로 `public` 또는 `protected` 기본 생성자가 반드시 있어야 함
* ⚠️ **클래스 제약**: `final` 클래스, `enum`, `interface`, `inner` 클래스는 엔티티로 사용 불가
* ⚠️ **필드 제약**: 저장할 필드에 `final` 키워드 사용 불가

---

## 3. 핵심 매핑 어노테이션 요약

| 어노테이션 | 설명 | 주요 옵션 / 특징 |
| :--- | :--- | :--- |
| `@Entity` | 해당 클래스를 JPA 엔티티로 지정 | 필수 지정 |
| `@Table` | 매핑할 DB 테이블 명 지정 | 생략 시 클래스명 사용 (`name="테이블명"`) |
| `@Id` | 테이블의 기본 키(PK) 지정 | 필수 지정 |
| `@GeneratedValue`| 기본 키 생성 전략 설정 | `IDENTITY`(DB 위임), `SEQUENCE`, `AUTO` |
| `@Column` | 필드와 컬럼 매핑 | `name`, `nullable=false`, `length=50` 등 |
| `@Transient` | DB 매핑에서 제외할 필드 | 메모리 내에서만 임시로 사용할 변수에 지정 |

---

## 4. 엔티티 기본 템플릿 코드

```java
import jakarta.persistence.*;

@Entity
@Table(name = "users") // 생략 시 클래스명인 User 테이블과 매핑
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // MySQL 등에서 Auto Increment 사용 시
    private Long id;

    @Column(name = "username", nullable = false, length = 50)
    private String name;

    // 기본 생성자 (JPA 필수 사항)
    protected User() {}

    // 사용자 정의 생성자
    public User(String name) {
        this.name = name;
    }

    // Getter (필요 시 Setter 추가 가능하나 지양 권장)
    public Long getId() { return id; }
    public String getName() { return name; }
    
    // 비즈니스 메서드 (Setter 대신 의미 있는 메서드 사용)
    public void changeName(String newName) {
        this.name = newName;
    }
}
```

lombok 사용시

```java
import java.time.OffsetDateTime;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.FetchType;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.JoinColumn;
import jakarta.persistence.ManyToOne;
import jakarta.persistence.Table;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // MySQL 등에서 Auto Increment 사용 시
    private Long id;

    @Column(name = "username", nullable = false, length = 50)
    private String name;

    @Column(name = "created_at", nullable = false)
    private OffsetDateTime createdAt;

    @Column(name = "updated_at", nullable = false)
    private OffsetDateTime updatedAt;
}
```

### @Id에 @Column이 생략된 이유 (필드 - 컬럼명 기본 맵핑 관례)

위 예제 코드의 `private Long id;` 필드에 `@Column` 어노테이션이 없는 이유는 다음과 같습니다.

#### 1) 필드 이름과 테이블 컬럼 이름의 일치 (기본 매핑 관례)

- JPA는 엔티티의 필드 위에 `@Column`이 없으면, **필드 이름(id)을 그대로 데이터베이스 테이블의 컬럼 이름(id)으로 매핑**합니다.
- 예제의 `users` 테이블 역시 기본 키(PK) 컬럼명이 `id`이기 때문에 별도의 `@Column(name = "id")`을 지정하지 않아도 자동으로 연결됩니다.

#### 2) @Column 명시가 필요한 경우와 비교

반면 `private String name;` 필드는 자바 객체에서는 `name`이지만, DB 테이블 컬럼명은 `username`으로 서로 다릅니다. 이처럼 **이름이 일치하지 않거나, `nullable = false` 같은 추가 제약 조건을 넣어야 할 때만 `@Column`을 명시**합니다.

```java
// 생략 가능: 필드명 'id'와 DB 컬럼명 'id'가 같음
@Id
private Long id; 

// 생략 불가: 필드명 'name'과 DB 컬럼명 'username'이 다르고, 추가 제약조건이 필요함
@Column(name = "username", nullable = false, length = 50)
private String name;
```

### 💡 요약 및 실무 팁

- JPA에서 `@Column`은 **필수가 아닌 선택** 사항입니다.
- 코드를 깔끔하게 유지하기 위해 테이블 컬럼명과 필드명이 일치할 때는 관례적으로 `@Column`을 생략합니다.
- 만약 테이블의 PK 컬럼명이 `id`가 아니라 `user_id`라면 다음과 같이 작성해야 합니다.

  ```java
  @Id
  @Column(name = "user_id") // 이름이 다를 때는 필수 작성
  private Long id;
  ```

---

## 5. 엔티티 라이프사이클 및 CRUD 동작

엔티티는 `EntityManager`와 **영속성 컨텍스트(Persistence Context)**를 통해 관리됩니다.

### ① 저장 (Create)
* `persist()` 메서드를 사용하여 객체를 영속성 컨텍스트에 저장합니다.
* 트랜잭션 커밋 시점에 `INSERT SQL`이 실행됩니다.
```java
User user = new User("홍길동");
em.persist(user); 
```

### ② 조회 (Read)
* `find()` 메서드를 통해 PK 기준으로 데이터를 조회합니다.
* 영속성 컨텍스트(1차 캐시)에 있으면 DB를 거치지 않고 바로 가져옵니다.
```java
User user = em.find(User.class, 1L);
```

### ③ 수정 (Update)
* **변경 감지(Dirty Checking)** 기능 덕분에 별도의 update() 메서드가 없습니다.
* 데이터를 조회한 뒤 필드 값만 변경하고 트랜잭션을 커밋하면, JPA가 자동으로 `UPDATE SQL`을 생성하여 실행합니다.
```java
User user = em.find(User.class, 1L);
user.changeName("이순신"); // 값 변경 시 자동 수정됨
```

### ④ 삭제 (Delete)
* `remove()` 메서드를 호출하여 데이터를 삭제합니다.
```java
User user = em.find(User.class, 1L);
em.remove(user);
```

## 8. 엔티티에서 Setter 사용을 지양하는 이유

자바 빈 규약(JavaBean Specification)에 익숙하면 관습적으로 모든 필드에 Getter/Setter를 만듭니다. 하지만 **JPA 엔티티에서는 Setter 작성을 엄격히 제한**합니다. 

### 1) Setter를 지양하는 구체적인 이유

* **의도의 모호함 (데이터 변경 목적을 알 수 없음)**
  `user.setName("이름")`이라는 코드를 보면, 이것이 단순 회원가입인지, 개명인지, 혹은 시스템 오류 수정인지 코드 자체만으로는 알 수 없습니다.
* **객체의 일관성(안전성) 붕괴**
  Setter가 public으로 열려 있으면 애플리케이션 어디서나 엔티티의 값을 임의로 변경할 수 있습니다. 엔티티 인스턴스의 값이 언제, 어디서, 왜 바뀌었는지 추적하기가 매우 어려워집니다.
* **JPA 변경 감지(Dirty Checking)와의 위험한 시너지**
  JPA는 트랜잭션 안에서 엔티티의 값이 바뀌면 커밋 시점에 자동으로 `UPDATE` 쿼리를 날립니다. 개발자가 의도하지 않고 실수로 Setter를 호출해도 DB 값이 조용히 변경되는 대형 사고가 날 수 있습니다.

---

## 9. Setter는 아예 필요 없을까? 어떻게 대체하나?

결론부터 말씀드리면, **Setter는 필요 없습니다.** 값 변경이 필요할 때는 Setter 대신 **'생성자'**와 **'의미 있는 비즈니스 메서드'**를 사용합니다.

### 데이터 변경 및 생성 프로세스 대체 방법

#### ① 엔티티 생성 시점: '생성자' 또는 '빌더(Builder) 패턴' 사용
생성 시점에 딱 한 번만 데이터를 채우고, 이후에는 수정할 수 없도록 Setter 대신 생성자를 사용합니다.
```java
// 빌더 패턴(Lombok @Builder) 사용 예시 (가장 추천)
User user = User.builder()
                .name("홍길동")
                .email("hong@test.com")
                .build();
```

#### ② 데이터 수정 시점: '의미 있는 메서드(도메인 메서드)' 사용
수정이 필요한 경우 `setXXX`가 아니라, **실제 비즈니스 요구사항의 이름을 딴 메서드**를 엔티티 내부에 직접 만들어 사용합니다.

```java
// 안 좋은 예 (Setter 남용)
user.setName("이순신"); 
user.setStatus("ACTIVE");

// 좋은 예 (비즈니스 의미를 담은 메서드)
user.changeName("이순신"); // 이름 변경이라는 목적이 명확함
user.activate();          // 계정 활성화라는 비즈니스 로직이 드러남
```

### 💡 개정된 엔티티 템플릿 코드 예시
```java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private boolean isActive;

    protected User() {} // JPA용 기본 생성자

    // 생성자로만 초기 데이터 주입 (Setter 원천 차단)
    public User(String name) {
        this.name = name;
        this.isActive = true;
    }

    // Getter는 안전하므로 자유롭게 생성
    public Long getId() { return id; }
    public String getName() { return name; }

    // [대체재] 수정이 필요할 땐 의미 있는 메서드 작성
    public void changeName(String newName) {
        if (newName == null || newName.isBlank()) {
            throw new IllegalArgumentException("이름은 빈 값일 수 없습니다."); // 검증 로직도 포함 가능
        }
        this.name = newName;
    }

    public void deactivate() {
        this.isActive = false; // 계정 비활성화 로직
    }
}
```
