JPA를 처음 접할 때 가장 헷갈리는 부분 중 하나입니다. 둘의 역할은 완전히 다릅니다.

- **Entity = 데이터(객체)**
- **Repository = 데이터베이스와 통신하는 객체**

비유하면 다음과 같습니다.

> **Entity**는 "회원 정보가 적힌 서류"이고,
> **Repository**는 "그 서류를 보관하고 꺼내주는 서류 보관소"입니다.

---

## 1. Entity

Entity는 **DB의 테이블 한 행(Row)*- 을 객체로 표현한 것입니다.

예를 들어 회원 테이블이 있다면

| id | name | age |
| -- | ---- | --- |
| 1  | Kim  | 20  |
| 2  | Lee  | 25  |

이를 객체로 표현하면

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    public Member(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void changeName(String name) {
        this.name = name;
    }
}
```

이 객체 하나가 테이블의 한 행을 의미합니다.

```java
Member member = new Member("Kim", 20);
```

이 시점에는 그냥 Java 객체일 뿐입니다.

---

## 2. Repository

Repository는 **Entity를 DB에 저장하거나 조회하는 역할**을 합니다.

```java
public interface MemberRepository
        extends JpaRepository<Member, Long> {
}
```

Repository가 하는 일은

- INSERT
- SELECT
- UPDATE
- DELETE

같은 DB 작업입니다.

예를 들어

```java
Member member = new Member("Kim", 20);

memberRepository.save(member);
```

`save()`가 실행되면

```sql
INSERT INTO member(name, age)
VALUES ('Kim', 20);
```

가 수행됩니다.

---

조회도 Repository가 합니다.

```java
Member member =
    memberRepository.findById(1L).orElseThrow();
```

그러면

```sql
SELECT *
FROM member
WHERE id = 1;
```

을 실행하고

```java
Member
```

객체를 반환합니다.

---

## 역할을 비교하면

### Entity

```java
Member member = new Member("Kim", 20);

member.changeName("Lee");
```

- 데이터를 가지고 있음
- 상태를 변경함
- 비즈니스 규칙을 가질 수 있음

DB와 직접 통신하지 않습니다.

---

### Repository

```java
memberRepository.save(member);

memberRepository.findById(id);

memberRepository.delete(member);
```

- DB에 저장
- DB에서 조회
- DB에서 삭제

데이터를 어떻게 저장하는지만 담당합니다.

---

## 함께 사용하는 흐름

예를 들어 회원을 생성하는 서비스

```java
@Service
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional
    public void join(String name, int age) {

        Member member = new Member(name, age);

        memberRepository.save(member);
    }
}
```

흐름은

```
Controller
      ↓
Service
      ↓
Entity 생성
      ↓
Repository.save()
      ↓
DB
```

입니다.

---

## 쉽게 기억하는 방법

| Entity       | Repository      |
| ------------ | --------------- |
| 테이블 한 행(Row) | 테이블 접근 도구       |
| 데이터를 표현      | 데이터를 저장/조회      |
| 상태를 가짐       | SQL 실행          |
| `@Entity`    | `JpaRepository` |

즉, **Entity는 "무엇을 저장할지"를 표현하는 객체이고, Repository는 "어떻게 저장하고 가져올지"를 담당하는 객체**입니다. 이 역할을 분리하는 것이 JPA와 도메인 모델 설계의 핵심입니다.

## Repository 생성 기준

**일반적으로는 Entity 하나당 Repository 하나를 만드는 것이 가장 흔한 방식**입니다.

다만 **"테이블 기준"이라기보다는 "Aggregate(도메인 루트) 기준"**이라는 점을 이해하면 더 정확합니다. 간단한 프로젝트에서는 Entity와 Aggregate가 거의 1:1인 경우가 많아서 결과적으로 Entity마다 Repository를 하나씩 두게 됩니다.

예를 들어 회원과 게시글이 있다면

```text
Member
    ↓
MemberRepository

Post
    ↓
PostRepository
```

처럼 구성합니다.

```java
@Entity
public class Member {
    ...
}
```

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Optional<Member> findByEmail(String email);

    boolean existsByNickname(String nickname);
}
```

그리고 게시글은 별도의 Repository를 둡니다.

```java
@Entity
public class Post {
    ...
}
```

```java
public interface PostRepository extends JpaRepository<Post, Long> {

    List<Post> findByAuthorId(Long authorId);
}
```

---

### 왜 하나씩 만들까?

Repository는 해당 Entity에 대한 영속성(저장, 조회)을 담당합니다.

예를 들어 `MemberRepository`에

```java
findByEmail(...)
existsByNickname(...)
findByLoginId(...)
```

같은 메서드가 있는 것은 자연스럽습니다.

하지만 여기에

```java
findPostsByCategory(...)
```

같은 게시글 관련 메서드가 들어가면 역할이 섞이게 됩니다.

---

### 그럼 테이블이 20개면 Repository도 20개?

네, 대부분의 Spring Boot + JPA 프로젝트는 그렇게 구성합니다.

```text
Member
MemberRepository

Post
PostRepository

Comment
CommentRepository

Order
OrderRepository

Product
ProductRepository
```

이렇게 Repository가 여러 개 있는 것이 오히려 관리하기 쉽습니다.

---

### Repository가 없는 Entity도 있을까?

있습니다.

예를 들어 `Order`와 `OrderItem`이 다음처럼 관계를 가진다고 해봅시다.

```text
Order
 ├── OrderItem
 ├── OrderItem
 └── OrderItem
```

`OrderItem`이 **`Order`에 종속된 엔티티**라면, 보통은 `OrderRepository`만 두고 `OrderItemRepository`는 만들지 않는 경우가 많습니다.

```java
Order order = orderRepository.findById(id).orElseThrow();

order.addItem(...);

order.removeItem(...);
```

`Order`를 저장하면 `OrderItem`도 함께 저장되도록(`cascade`) 설계할 수 있기 때문입니다.

---

### 실무 기준

실무에서는 보통 다음 기준으로 생각합니다.

- **독립적으로 조회·저장되는 Entity*- → Repository를 만든다.
- **항상 다른 Entity에 포함되어 관리되는 Entity*- → 별도 Repository를 만들지 않는 경우도 많다.

처음 JPA를 배우는 단계라면 **"Entity 하나당 Repository 하나"**라고 생각해도 대부분의 상황에서 맞습니다. 이후 도메인 주도 설계(DDD)를 접하게 되면 "Aggregate Root만 Repository를 가진다"는 개념을 배우게 되는데, 그때 지금의 이해를 조금 확장하면 됩니다.
