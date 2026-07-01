# Spring JPA 구조

한 줄로 정리하면,

> **JPA는 명세(인터페이스)이고, Hibernate는 JPA를 구현한 구현체이며, Spring Data JPA는 JPA를 더 편하게 사용할 수 있도록 만든 Spring의 라이브러리입니다.**

관계를 그림으로 표현하면 다음과 같습니다.

```text
내 코드
    │
    ▼
Spring Data JPA
    │
    ▼
JPA (Jakarta Persistence API)
    │
    ▼
Hibernate (구현체)
    │
    ▼
Database
```

## 1. JPA란?

JPA(Java Persistence API)는 **ORM(Object Relational Mapping)에 대한 표준 명세**입니다.

즉, "Entity는 이렇게 정의하고, EntityManager는 이런 기능을 제공해야 한다"와 같은 **규칙**만 정의합니다.

예를 들어

```java
@Entity
public class Member { ... }
```

```java
EntityManager em = ...
```

이런 API들은 JPA 표준입니다.

하지만 JPA 자체는 SQL을 실행하지 않습니다.

---

## 2. Hibernate란?

Hibernate는 JPA 명세를 실제로 구현한 라이브러리입니다.

예를 들어

```java
em.persist(member);
```

를 호출하면

실제로는 Hibernate가

```sql
INSERT INTO member ...
```

를 생성해서 실행합니다.

즉,

- JPA = 설계도
- Hibernate = 실제 제품

이라고 생각하면 됩니다.

Hibernate 외에도 EclipseLink 같은 구현체가 있지만, **Spring Boot에서는 기본적으로 Hibernate를 사용합니다.**

---

## 3. Spring Data JPA란?

JPA를 그대로 사용하면 `EntityManager`를 직접 다뤄야 합니다.

예를 들어 JPA만 사용한다면

```java
@Repository
public class MemberRepository {

    @PersistenceContext
    private EntityManager em;

    public void save(Member member) {
        em.persist(member);
    }

    public Member find(Long id) {
        return em.find(Member.class, id);
    }
}
```

이처럼 Repository를 직접 구현해야 합니다.

---

Spring Data JPA를 사용하면

```java
public interface MemberRepository
        extends JpaRepository<Member, Long> {
}
```

이 한 줄만 작성하면

- save()
- findById()
- findAll()
- delete()

등의 메서드를 자동으로 제공합니다.

즉, **반복적인 Repository 구현을 없애주는 라이브러리**입니다.

---

## 4. Spring Data JPA도 결국 JPA를 사용한다

예를 들어

```java
memberRepository.save(member);
```

를 호출하면 내부적으로는

```java
entityManager.persist(member);
```

또는

```java
entityManager.merge(member);
```

가 호출됩니다.

그리고 EntityManager는 Hibernate를 통해 SQL을 실행합니다.

즉,

```text
memberRepository.save()
        ↓
EntityManager.persist()
        ↓
Hibernate
        ↓
INSERT SQL
```

이렇게 이어집니다.

---

## 역할을 비교하면

| 기술                | 역할                                             |
| ------------------- | ------------------------------------------------ |
| **JPA*-             | ORM 표준 명세(API와 규칙)                        |
| **Hibernate*-       | JPA를 구현한 ORM 프레임워크                      |
| **Spring Data JPA*- | JPA를 더 쉽게 사용하도록 Repository 기능을 제공하는 Spring 라이브러리 |

---

## 비유하면

자동차에 비유하면 이해하기 쉽습니다.

- **JPA*- → 자동차 설계 규격
- **Hibernate*- → 그 규격에 맞춰 만든 실제 자동차
- **Spring Data JPA*- → 운전을 더 쉽게 해주는 자동변속기, 내비게이션 같은 편의 기능

즉, **Spring Data JPA는 JPA를 대체하는 기술이 아니라, JPA 위에서 동작하는 생산성 향상 라이브러리**입니다.

### 실무에서는?

대부분의 Spring Boot 프로젝트에서는 다음 조합을 사용합니다.

- **Spring Data JPA**로 `Repository`를 작성하고,
- 내부에서는 **JPA API**를 사용하며,
- 실제 SQL 생성과 실행은 **Hibernate**가 담당합니다.

그래서 실무에서는 `EntityManager`를 직접 다루는 경우는 복잡한 쿼리나 특수한 기능이 필요할 때 정도이고, 대부분의 CRUD는 `JpaRepository`만으로 충분히 구현합니다.
