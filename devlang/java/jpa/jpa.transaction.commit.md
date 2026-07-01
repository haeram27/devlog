## 6. 트랜잭션 커밋(Transaction Commit) 사용법

JPA의 모든 데이터 변경(C, U, D)은 반드시 트랜잭션 안에서 이루어져야 합니다. 트랜잭션이 커밋되는 순간 JPA의 **변경 감지(Dirty Checking)**가 작동하며 실제 DB로 SQL이 전송(Flush)됩니다.

### 방식 1. 스프링 환경 (Spring Boot / Spring Data JPA)

실무에서 가장 많이 사용하는 방식입니다. 메서드나 클래스 위에 `@Transactional` 어노테이션만 붙이면 **스프링이 자동으로 트랜잭션을 시작하고, 메서드가 정상 종료되면 커밋**합니다. (예외 발생 시 롤백)

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Transactional // 1. 트랜잭션 시작 -> 3. 메서드 정상 종료 시 자동 커밋
    public void updateUser(Long id, String newName) {
        // 2. 비즈니스 로직 수행
        User user = userRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("유저 없음"));
        
        user.changeName(newName); // 변경 감지로 인해 커밋 시점에 UPDATE SQL 실행됨
    }
}
```

### 방식 2. 순수 자바 환경 (JPA 원천 기술)

스프링 없이 순수 자바와 `EntityManager`를 사용할 때는 개발자가 코드로 직접 트랜잭션을 열고 닫아야 합니다.

```java
import jakarta.persistence.*;

public class Main {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();

        // 1. 트랜잭션 API 획득 및 시작
        EntityTransaction tx = em.getTransaction();
        try {
            tx.begin(); // 트랜잭션 시작!

            // 2. 비즈니스 로직 실행
            User user = new User("홍길동");
            em.persist(user);

            // 3. 트랜잭션 커밋! (이때 DB에 실제 저장됨)
            tx.commit(); 
        } catch (Exception e) {
            // 예외 발생 시 안전하게 롤백
            tx.rollback();
            e.printStackTrace();
        } finally {
            // 엔티티 매니저 종료 (필수)
            em.close();
        }
        emf.close();
    }
}
```

### 💡 핵심 요약 (커밋 시 일어나는 일)

1. **`flush()` 자동 호출**: 영속성 컨텍스트의 변경 내용과 DB를 동기화(SQL 쿼리를 DB로 전송)합니다.
2. **DB 커밋**: 데이터베이스에 실제로 물리적인 커밋 명령을 내려 데이터를 영구 저장합니다.
