# redis java에서 사용

## DB 테이블 전체를 Redis에 캐시하는 방법

### 자료구조 선택 기준

테이블 전체를 캐시할 때 가장 중요한 질문은: **"어떤 방식으로 조회할 것인가?"**

---

### 방식별 비교

#### 1. `Hash` — 행(row)을 ID로 빠르게 조회할 때 (권장)

Redis Hash를 "테이블처럼" 사용하는 방식. `key = 테이블명`, `field = PK`, `value = JSON`

```
Redis Key: "product:all"
  field "1" → { "id":1, "name":"사과", "price":1000 }
  field "2" → { "id":2, "name":"배",   "price":2000 }
  field "3" → { "id":3, "name":"감",   "price":1500 }
```

```java
// 전체 로드
Map<Object, Object> rows = redisTemplate.opsForHash().entries("product:all");

// 단건 조회 (O(1))
String json = (String) redisTemplate.opsForHash().get("product:all", "1");

// 단건 갱신
redisTemplate.opsForHash().put("product:all", "2", newJson);

// 단건 삭제
redisTemplate.opsForHash().delete("product:all", "3");
```

**장점**: 단건 CRUD가 O(1), 부분 업데이트 가능  
**단점**: 전체를 List로 반환할 때 순서 보장 안 됨

---

#### 2. `String + JSON 배열` — 전체를 통째로 저장 (단순한 경우)

```java
// 저장
List<Product> products = productRepository.findAll();
String json = objectMapper.writeValueAsString(products);
redisTemplate.opsForValue().set("product:all", json, Duration.ofMinutes(30));

// 조회
String cached = (String) redisTemplate.opsForValue().get("product:all");
List<Product> products = objectMapper.readValue(cached,
    new TypeReference<List<Product>>() {});
```

**장점**: 구현 단순, 전체 조회에 최적  
**단점**: 1건 수정해도 전체를 다시 저장해야 함 → **데이터가 자주 변하면 비효율**

---

#### 3. `String + 개별 키` — 단건 조회가 매우 빈번할 때

```
"product:1" → { "id":1, "name":"사과", ... }
"product:2" → { "id":2, "name":"배",   ... }
```

→ 사실상 `@Cacheable(key = "#id")`와 동일한 패턴  
**단점**: 전체 목록 조회 시 `KEYS product:*` 가 필요 — **운영환경에서 KEYS 명령 금지**

---

#### 4. `ZSet (Sorted Set)` — 정렬이 필요할 때

가격순, 인기순 등 **순서가 중요한 경우**

```java
// 가격 기준으로 저장
products.forEach(p ->
    redisTemplate.opsForZSet().add("product:byPrice", p.getId(), p.getPrice())
);

// 가격 100~2000 범위 조회
Set<Object> ids = redisTemplate.opsForZSet().rangeByScore("product:byPrice", 100, 2000);
```

---

### 추천 조합 (실무 패턴)

```
목적별로 두 가지를 함께 사용
  - product:all         (Hash)   → ID로 단건 조회/수정
  - product:list        (String) → 전체 목록 페이지 렌더링용
  - product:byPrice     (ZSet)   → 정렬/필터링용
```

```java
@Service
public class ProductCacheService {

    private static final String HASH_KEY = "product:all";
    private final RedisTemplate<String, String> rt;
    private final ObjectMapper om;

    // 테이블 전체 로드 (앱 시작 시 or 스케줄러로 주기 갱신)
    public void loadAll(List<Product> products) throws Exception {
        Map<String, String> map = new LinkedHashMap<>();
        for (Product p : products) {
            map.put(String.valueOf(p.getId()), om.writeValueAsString(p));
        }
        rt.opsForHash().putAll(HASH_KEY, map);
        rt.expire(HASH_KEY, Duration.ofHours(1));
    }

    // 단건 조회
    public Optional<Product> get(long id) throws Exception {
        String json = (String) rt.opsForHash().get(HASH_KEY, String.valueOf(id));
        if (json == null) return Optional.empty();
        return Optional.of(om.readValue(json, Product.class));
    }

    // 전체 조회
    public List<Product> getAll() {
        return rt.opsForHash().values(HASH_KEY).stream()
            .map(v -> { try { return om.readValue((String)v, Product.class); } catch (Exception e) { throw new RuntimeException(e); } })
            .collect(Collectors.toList());
    }

    // DB 변경 시 캐시 동기화
    public void update(Product p) throws Exception {
        rt.opsForHash().put(HASH_KEY, String.valueOf(p.getId()), om.writeValueAsString(p));
    }

    public void delete(long id) {
        rt.opsForHash().delete(HASH_KEY, String.valueOf(id));
    }
}
```

---

### 최종 결론

| 상황 | 추천 |
|------|------|
| ID로 단건 조회가 많음 | **Hash** |
| 전체 목록만 통째로 쓰면 됨 | **String (JSON 배열)** |
| 변경이 없거나 드문 정적 데이터 | **String + `@Cacheable`** |
| 정렬/범위 검색 필요 | **ZSet** |
| 실시간 변경이 잦은 테이블 | Redis 캐시 대신 DB 인덱스 재검토 권장 |

**데이터가 자주 변하지 않는 코드성 테이블** (공통코드, 카테고리, 설정값 등)이라면 **Hash 방식**이 가장 실용적입니다.