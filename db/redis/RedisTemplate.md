`RedisTemplate`은 Spring에서 제공하는 Redis 데이터 조작을 위한 핵심 클래스입니다. 다양한 메서드를 통해 Redis에 데이터를 저장, 검색, 삭제할 수 있습니다.

---

## **1. RedisTemplate 주요 메서드 리스트 및 설명**

### **1.1. Key 관련 메서드**

|메서드|설명|
|---|---|
|`delete(K key)`|특정 키를 삭제|
|`delete(Collection<K> keys)`|여러 개의 키를 삭제|
|`hasKey(K key)`|특정 키가 존재하는지 확인|
|`keys(K pattern)`|주어진 패턴에 맞는 모든 키 검색 (예: `user:*`)|
|`randomKey()`|Redis에서 랜덤한 키를 반환|
|`rename(K oldKey, K newKey)`|특정 키의 이름 변경|
|`renameIfAbsent(K oldKey, K newKey)`|기존 키가 존재하지 않을 경우 새로운 키로 변경|
|`expire(K key, long timeout, TimeUnit unit)`|특정 키의 만료 시간 설정|
|`getExpire(K key, TimeUnit unit)`|특정 키의 남은 만료 시간 조회|

---

### **1.2. Value (String) 관련 메서드**

|메서드|설명|
|---|---|
|`opsForValue().set(K key, V value)`|특정 키에 값 저장|
|`opsForValue().set(K key, V value, long timeout, TimeUnit unit)`|특정 키에 값을 저장하고 만료 시간 설정|
|`opsForValue().get(K key)`|특정 키의 값 가져오기|
|`opsForValue().getAndSet(K key, V value)`|현재 값을 가져온 후 새로운 값으로 설정|
|`opsForValue().increment(K key, long delta)`|특정 키의 값을 `delta` 만큼 증가|
|`opsForValue().decrement(K key, long delta)`|특정 키의 값을 `delta` 만큼 감소|

---

### **1.3. Hash 관련 메서드**

|메서드|설명|
|---|---|
|`opsForHash().put(H key, HK hashKey, HV value)`|Hash에 특정 필드 저장|
|`opsForHash().putAll(H key, Map<HK, HV> map)`|여러 필드를 한 번에 저장|
|`opsForHash().get(H key, HK hashKey)`|특정 Hash 필드의 값 가져오기|
|`opsForHash().entries(H key)`|Hash의 모든 필드 가져오기 (Map 형태)|
|`opsForHash().hasKey(H key, HK hashKey)`|특정 필드가 존재하는지 확인|
|`opsForHash().delete(H key, Object... hashKeys)`|특정 필드 삭제|
|`opsForHash().increment(H key, HK hashKey, long delta)`|Hash 필드 값 증가|
|`opsForHash().size(H key)`|Hash에 저장된 필드 개수 조회|

---

### **1.4. List 관련 메서드**

|메서드|설명|
|---|---|
|`opsForList().leftPush(K key, V value)`|리스트의 왼쪽(Head) 에 값 추가|
|`opsForList().leftPushAll(K key, V... values)`|여러 개의 값을 왼쪽에 추가|
|`opsForList().rightPush(K key, V value)`|리스트의 오른쪽(Tail) 에 값 추가|
|`opsForList().rightPushAll(K key, V... values)`|여러 개의 값을 오른쪽에 추가|
|`opsForList().rightPop(K key)`|리스트의 오른쪽(Tail) 값 가져오고 삭제|
|`opsForList().leftPop(K key)`|리스트의 왼쪽(Head) 값 가져오고 삭제|
|`opsForList().size(K key)`|리스트의 길이 반환|
|`opsForList().index(K key, long index)`|리스트의 특정 인덱스 값 조회|
|`opsForList().range(K key, long start, long end)`|리스트 범위 조회 (ex: `0` to `-1` 은 전체 리스트)|

---

### **1.5. Set 관련 메서드**

|메서드|설명|
|---|---|
|`opsForSet().add(K key, V... values)`|Set에 값 추가|
|`opsForSet().members(K key)`|Set의 모든 요소 조회|
|`opsForSet().isMember(K key, V value)`|특정 값이 Set에 존재하는지 확인|
|`opsForSet().remove(K key, Object... values)`|Set에서 특정 값 삭제|
|`opsForSet().size(K key)`|Set의 요소 개수 조회|
|`opsForSet().intersect(K key, K otherKey)`|두 Set의 교집합 조회|
|`opsForSet().union(K key, K otherKey)`|두 Set의 합집합 조회|
|`opsForSet().difference(K key, K otherKey)`|두 Set의 차집합 조회|

---

### **1.6. Sorted Set (ZSet) 관련 메서드**

|메서드|설명|
|---|---|
|`opsForZSet().add(K key, V value, double score)`|Sorted Set에 값 추가|
|`opsForZSet().incrementScore(K key, V value, double delta)`|특정 값의 점수 증가|
|`opsForZSet().range(K key, long start, long end)`|특정 범위 내의 요소 조회 (점수 기준 오름차순)|
|`opsForZSet().rangeByScore(K key, double min, double max)`|특정 점수 범위 내의 요소 조회|
|`opsForZSet().remove(K key, Object... values)`|특정 요소 삭제|
|`opsForZSet().score(K key, V value)`|특정 요소의 점수 조회|
|`opsForZSet().rank(K key, V value)`|특정 값의 랭킹 조회 (0부터 시작, 낮을수록 순위 높음)|

---

## **2. RedisTemplate 예제 코드**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.concurrent.TimeUnit;

@Service
public class RedisService {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    // String 저장
    public void saveString(String key, String value) {
        redisTemplate.opsForValue().set(key, value);
        redisTemplate.expire(key, 10, TimeUnit.MINUTES); // 10분 후 만료
    }

    // String 조회
    public String getString(String key) {
        return redisTemplate.opsForValue().get(key);
    }

    // Hash 저장
    public void saveHash(String key, String field, String value) {
        redisTemplate.opsForHash().put(key, field, value);
    }

    // Hash 조회
    public String getHash(String key, String field) {
        return (String) redisTemplate.opsForHash().get(key, field);
    }

    // List 저장
    public void pushToList(String key, String value) {
        redisTemplate.opsForList().rightPush(key, value);
    }

    // List 조회
    public List<String> getList(String key, long start, long end) {
        return redisTemplate.opsForList().range(key, start, end);
    }

    // Set 저장
    public void addToSet(String key, String value) {
        redisTemplate.opsForSet().add(key, value);
    }

    // Set 조회
    public boolean isMemberOfSet(String key, String value) {
        return redisTemplate.opsForSet().isMember(key, value);
    }

    // Sorted Set 저장
    public void addToZSet(String key, String value, double score) {
        redisTemplate.opsForZSet().add(key, value, score);
    }

    // Sorted Set 조회
    public Double getZSetScore(String key, String value) {
        return redisTemplate.opsForZSet().score(key, value);
    }
}
```

---

## **3. 결론**

`RedisTemplate`을 사용하면 **String, Hash, List, Set, Sorted Set** 등 다양한 Redis 데이터 타입을 쉽게 다룰 수 있습니다. 위에서 설명한 메서드를 활용하면 원하는 방식으로 데이터를 저장하고 관리할 수 있습니다! 🚀