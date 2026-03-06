# spring cache

spring은 외부 캐시 솔루션(redis 등)과 편리하게 연동하기 위해서 Cache 관련 어노테이션을 제공함

## 주요 캐시 어노테이션 설명

- @EnableCaching
  - Spring에서 캐싱을 활성화
  - 별도의 Cache Configuration 클래스나 메인 클래스(@SpringBootApplicaiton)에 지정
- @Cacheable(value = `<logical-db-name>`, key = `<cache-key>`)
  - 캐시 읽기
  - 캐시에서 먼저 데이터를 찾고 만약 캐시에 없으면 메서드를 실행한 후 그 결과를 캐시에 저장하고 반환함
  - 주의: 캐시에 값이 있는 경우 메서드는 실행되지 않음
  - 메서드는 캐시가 없는 경우 반환 값을 생성하도록 구현함(주로 데이터 베이스의 값을 읽어오는 용도)
- @CachePut(value = `<logical-db-name>`, key = `<cache-key>`)
  - 캐시 갱신(update)
  - 메서드가 호출되면 메서드를 항상 먼저 실행하고 그 결과를 캐시에 저장한 후 값을 반환함
  - 메서드는 항상 실행됨
- @CacheEvict(value = `<logical-db-name>`, key = `<cache-key>`)
  - 캐시 삭제
  - 메소드 호출시 지정된 키의 캐시 또는 캐시 전체를 무효화(삭제)함
  - 메서드는 항상 실행됨

## 이해를 위한 지식

### Redis란

1. 저장 위치: 인 메모리 (In-Memory) 

    주 저장소: 데이터를 디스크가 아닌 RAM(주기억장치)에 저장하여 읽기/쓰기 속도가 매우 빠릅니다.
    영속성 보장: 메모리는 휘발성이지만, Redis는 스냅샷(RDB)이나 로그 기록(AOF) 방식을 통해 데이터를 디스크에 백업하여 복구할 수 있는 기능을 제공합니다

2. DB 구조: 키-값 (Key-Value)

    구조: 모든 데이터는 고유한 Key와 그에 대응하는 Value로 저장됩니다.
    특징: 관계형 DB(MySQL 등)처럼 복잡한 쿼리를 사용하지 않고, Key를 통해 즉시 데이터를 조회하는 단순한 구조로 설계되어 지연 시간이 매우 짧습니다

3. 다양한 데이터 타입 (Value)

    단순 문자열 외에도 다양한 자료 구조를 지원하는 것이 Redis의 큰 장점

    Strings: 텍스트, 숫자, 바이너리 데이터 저장
    Lists: 데이터가 들어간 순서대로 저장되는 연결 리스트
    Hashes: Field-Value 쌍을 포함하는 맵 (객체 형태 저장에 용이)
    Sets: 중복되지 않는 값들의 집합
    Sorted Sets: 점수(Score)에 따라 정렬된 상태를 유지하는 집합

4. 주요 용도

    캐시(Cache): 자주 조회되는 데이터를 임시 저장하여 DB 부하를 줄임.
    세션 관리: 웹 서비스의 로그인 세션 정보 저장.
    실시간 순위표: 게임 점수나 실시간 검색어 랭킹 관리. 

### 추가

- Redis는 `Logical Database`라는 논리적 구분된 공간을 제공한다.
- 

### 사용 예제

```java
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.stereotype.Service;

@EnableCaching // 캐싱 기능 활성화
@Configuration
public class CacheConfig {
    // 캐싱 관련 설정이 필요하면 추가로 정의할 수 있습니다.
}

@Service
@Slf4j
public class ProductService {

    // 캐시에 저장된 값(반환값)이 있으면 읽어오고 없으면 메서드 실행 후 결과 반환값을 캐시에 저장
    // 캐시에 저장된 값이 있는 경우 메서드는 실행되지 않음
    @Cacheable(value = "users", key = "#id")
    public Product getProductById(Long id) {
        // DB에서 데이터 가져오는 로직
        Product product = rdbDay.selectProductById(id);
        log.info("Fetching product with ID: " + id);
        return product;
    }

    // 메서드 호출시 리모트 캐시를 삭제, 캐시값 초기화 또느 refresh(삭제 후 다시 설정) 필요시 사용
    // 'users' cache db에서 'id 인자 값'에 해당하는 cache-key를 가진 항목을 삭제 
    @CacheEvict(value = "users", key = "#id")
    public void removeProductFromCache(Long id) {
        productRepository.deleteById(id);
        log.info("Removing product with ID: " + id + " from cache.");
    }

    @CacheEvict(value = "users", allEntries = true)
    public void deleteAllUsers() {
        userRepository.deleteAll();
        log.info("Removing users from cache.");
    }

    // 메서드 호출시 메서드를 항상 먼저 실행하고 그 결과를 캐시에 저장하고 값을 반환함
    @CachePut(value = "users", key = "#product.id")
    public Product updateProduct(Product product) {
        log.info("Updating product with ID: " + product.getId());
        return product;
    }
}
```

## 캐시 사용 시 주의사항

- 캐시 일관성:
  - 캐싱된 데이터가 변경될 경우 적절히 무효화(@CacheEvict)하거나 갱신(@CachePut)하는 전략을 사용해야 합니다.
- 캐시 저장소 설정:
  - Spring Boot에서는 기본적으로 ConcurrentMap 기반의 캐시를 사용하지만 Redis, Ehcache 등 외부 캐시 저장소와 통합하여 사용 가능
  - gradle이나 maven 설정에 spring-boot-starter-data-redis 의존성이 있으면 자동으로 Redis를 cache로 사용(수동 설정도 가능)