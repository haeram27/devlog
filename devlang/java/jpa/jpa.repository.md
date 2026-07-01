# JpaRepository methods

Spring Data JPA에서 `JpaRepository<Entity, ID>`를 상속하면 사용할 수 있는 대표적인 기본 메서드는 다음과 같습니다. (`CrudRepository`, `PagingAndSortingRepository`의 기능도 모두 포함합니다.)

| 메서드                                      | 설명                                               |
| ------------------------------------------- | -------------------------------------------------- |
| `save(S entity)`                            | 새로운 Entity는 INSERT, 기존 Entity는 UPDATE(병합)하여 저장합니다. |
| `saveAll(Iterable<S> entities)`             | 여러 Entity를 한 번에 저장합니다. |
| `findById(ID id)`                           | ID로 Entity를 조회합니다. `Optional`을 반환합니다. |
| `getReferenceById(ID id)`                   | 실제 조회하지 않고 프록시 객체를 반환합니다. 실제 사용 시 DB를 조회합니다. |
| `existsById(ID id)`                         | 해당 ID의 데이터가 존재하는지 확인합니다. |
| `findAll()`                                 | 모든 Entity를 조회합니다. |
| `findAllById(Iterable<ID> ids)`             | 여러 ID에 해당하는 Entity를 조회합니다. |
| `findAll(Sort sort)`                        | 정렬 조건을 적용하여 전체 조회합니다. |
| `findAll(Pageable pageable)`                | 페이징하여 조회합니다. |
| `count()`                                   | 전체 데이터 개수를 반환합니다. |
| `deleteById(ID id)`                         | ID로 Entity를 삭제합니다. |
| `delete(T entity)`                          | Entity를 삭제합니다. |
| `deleteAllById(Iterable<? extends ID> ids)` | 여러 ID를 한 번에 삭제합니다. |
| `deleteAll(Iterable<? extends T> entities)` | 여러 Entity를 삭제합니다. |
| `deleteAll()`                               | 모든 데이터를 삭제합니다. |
| `flush()`                                   | 변경 내용을 즉시 DB에 반영합니다. |
| `saveAndFlush(S entity)`                    | 저장 후 즉시 DB에 반영합니다. |
| `saveAllAndFlush(Iterable<S> entities)`     | 여러 Entity를 저장 후 즉시 DB에 반영합니다. |
| `deleteAllInBatch()`                        | 모든 데이터를 한 번의 SQL로 삭제합니다. |
| `deleteAllInBatch(Iterable<T> entities)`    | 지정한 Entity들을 한 번의 SQL로 삭제합니다. |
| `deleteAllByIdInBatch(Iterable<ID> ids)`    | 여러 ID를 한 번의 SQL로 삭제합니다. |
| `findAll(Example<S> example)`               | Example 객체를 이용해 조건 검색합니다. |
| `findAll(Example<S> example, Sort sort)`    | Example 조건 + 정렬 조회를 수행합니다. |
| `findOne(Example<S> example)`               | Example 조건에 맞는 Entity 하나를 조회합니다. |
| `exists(Example<S> example)`                | Example 조건에 맞는 데이터가 존재하는지 확인합니다. |
| `count(Example<S> example)`                 | Example 조건에 맞는 데이터 개수를 반환합니다. |

### 실무에서 가장 많이 사용하는 메서드

실제로는 아래 메서드들을 가장 많이 사용합니다.

```java
save(entity);                // 저장
findById(id);                // 단건 조회
findAll();                   // 전체 조회
existsById(id);              // 존재 여부 확인
count();                     // 개수 조회
delete(entity);              // 삭제
deleteById(id);              // ID로 삭제
findAll(pageable);           // 페이징
findAll(sort);               // 정렬
```

### 기본 메서드 외에도 자동으로 생성되는 메서드

Spring Data JPA는 **메서드 이름만으로 쿼리를 생성**할 수 있습니다. 예를 들어 `MemberRepository`라면 다음과 같이 선언만 해도 구현 없이 사용할 수 있습니다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Optional<Member> findByEmail(String email);

    List<Member> findByAgeGreaterThan(int age);

    List<Member> findByNameContaining(String name);

    boolean existsByEmail(String email);

    long countByAge(int age);

    void deleteByEmail(String email);

    List<Member> findByAgeBetween(int start, int end);

    List<Member> findByNameOrderByAgeDesc(String name);
}
```

이런 **Query Method**는 Spring Data JPA의 강력한 기능 중 하나이며, 간단한 조회는 별도의 SQL이나 JPQL을 작성하지 않고도 처리할 수 있습니다.
