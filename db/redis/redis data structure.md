## **정리. Redis 자료구조 비교**

- Redis에서 Key는 각 data를 구분하기 위한 redis db내 단일 키이며, redis의 value의 데이터 형식은 `단일 value`, `hash`, `set`, `sorted set`, `list`를 사용할 수 있다. 


| 자료구조 | 데이터 형태 | 키-값 관계 | 중복 허용 여부 | 정렬 여부 | 주요 활용 |
|:---|:---|:---|:---|:---|:---|
| **Value** | `value` (단일 값) | X | NA | NA | 집합이 아닌 단일 값 저장 |
| **Hash** | `{field: value}` (필드-값) | O (Key-(Field-Value)) | X (Field 중복 불가) | X | Map(딕셔너리) 형태로 저장, 사용자 프로필, 세션 데이터, 실시간 로그 |
| **Set** | `{value1, value2, ...}` (단일 값) | X (Key-(Value)) | X (중복 불가) | X | 유일한 값 저장, 태그, 회원 목록 |
| **Sorted Set (ZSet)** | `{value: score}` | X (Key-(Value-Score)) | X (중복 불가) | O (Score 기준 정렬) | 리더보드, 랭킹 시스템 |
| **List** | `[value1, value2, ...]` (값의 리스트) | X (Key-(Value)) | O (중복 허용) | O (삽입 순서 유지) | 큐/스택, 메시지 큐 |

---

## **정리. Redis Hash vs Set 차이점**

|비교항목|Hash|Set|
|:---|:---|:---|
|구조|`Key → {Field: Value}`|`Key → {Value}`|
|용도|**Map (필드-값 저장)**|**집합 (중복 없는 값 저장)**|
|중복 허용|Field 중복 ❌, Value 중복 ✅|중복 ❌|
|데이터 타입|`String: String`|`String`|
|값 변경|특정 Field 값 변경 가능 (`HSET`)|변경 불가 (삭제 후 추가)|
|정렬|X|X|
|사용 예|`user:1001 -> {name: "Alice", age: "25"}`|`online_users -> {"Alice", "Bob", "Charlie"}`|

---

## **Redis 자료구조 선택 가이드**

- **Hash**: Map(딕셔너리) 저장 (`user:1001 -> {name: "Alice", age: "25"}`)
- **Set**: 순서없는 집합, 중복 없는 유일한 값 저장 (`online_users -> {"Alice", "Bob", "Charlie"}`)
- **Sorted Set (ZSet)**: 점수 기반 정렬이 필요한 경우 (`leaderboard -> {"Alice": 100, "Bob": 90}`)
- **List**: 순서있는 집합, 값 중복 허용 (`message_queue -> ["msg1", "msg2", "msg3"]`)

---

## **1. Redis Hash (Key->{Field: Value})**

Redis의 **Hash**는 Java의 `Map<String, String>`과 유사한 구조입니다.

### **특징**

- 하나의 키 아래에 여러 필드-값을 저장하는 **딕셔너리(Map) 구조**
- 각 필드는 **중복 불가**, 단 **값(value)**은 중복 가능
- `opsForHash()`를 이용하여 관리

### **예제**
```java
// Hash에 데이터 추가
redisTemplate.opsForHash().put("user:1001", "name", "Alice");
redisTemplate.opsForHash().put("user:1001", "age", "25");

// Hash 데이터 조회
String name = (String) redisTemplate.opsForHash().get("user:1001", "name"); // "Alice"

// Hash 전체 필드 조회
Map<Object, Object> user = redisTemplate.opsForHash().entries("user:1001");
```
`opsForHash().put(H key, HK hashKey, HV value)`


### **Redis 내부 저장 방식**

```
HSET user:1001 name Alice
HSET user:1001 age 25
```

**Key: `user:1001`**  
**Field-Value: `name: Alice`, `age: 25`**

---

## **2. Redis Set (Key->(Value))**
Redis의 **Set**은 `HashSet<String>`과 유사하며 **중복을 허용하지 않는 집합 구조**입니다.

### **특징**
- 하나의 키 아래 **중복 없는 값(value) 목록**을 저장
- **순서 없음** (랜덤 조회)
- **교집합, 합집합, 차집합 연산 가능**
- `opsForSet()`을 이용하여 관리

### **예제**
```java
// Set에 데이터 추가
redisTemplate.opsForSet().add("user:active", "Alice", "Bob", "Charlie");

// 특정 값이 존재하는지 확인
boolean exists = redisTemplate.opsForSet().isMember("user:active", "Alice"); // true

// Set 전체 값 조회
Set<Object> users = redisTemplate.opsForSet().members("user:active");
```

### **Redis 내부 저장 방식**
```
SADD user:active Alice
SADD user:active Bob
SADD user:active Charlie
```
**Key: `user:active`**  
**Values: `{Alice, Bob, Charlie}` (중복 불가, 순서 없음)**

---

## **3. Redis Sorted Set (Key->(Value:Score))**
Redis의 **Sorted Set (ZSet)**은 **Set의 중복 없는 값 저장 기능 + Score 기반 정렬 기능**이 추가된 자료구조입니다.

### **특징**
- 하나의 키 아래 **중복 없는 값(value)과 점수(score)를 저장**
- **점수(score) 기준으로 자동 정렬**
- **특정 범위(rank) 내의 값 조회 가능**
- `opsForZSet()`을 이용하여 관리

### **예제**
```java
// Sorted Set에 데이터 추가
redisTemplate.opsForZSet().add("leaderboard", "Alice", 100);
redisTemplate.opsForZSet().add("leaderboard", "Bob", 90);
redisTemplate.opsForZSet().add("leaderboard", "Charlie", 95);

// 특정 사용자 점수 확인
Double score = redisTemplate.opsForZSet().score("leaderboard", "Alice"); // 100

// 점수 순위별 조회 (오름차순)
Set<Object> topPlayers = redisTemplate.opsForZSet().range("leaderboard", 0, -1);

// 점수 순위별 조회 (내림차순)
Set<Object> topPlayersDesc = redisTemplate.opsForZSet().reverseRange("leaderboard", 0, -1);
```

### **Redis 내부 저장 방식**
```
ZADD leaderboard 100 Alice
ZADD leaderboard 90 Bob
ZADD leaderboard 95 Charlie
ZRANGE leaderboard 0 -1 → Bob, Charlie, Alice (점수 오름차순)
ZREVRANGE leaderboard 0 -1 → Alice, Charlie, Bob (점수 내림차순)
```
**Key: `leaderboard`**  
**Values: `{"Alice": 100, "Charlie": 95, "Bob": 90}` (점수 기준 정렬됨)**

---

## **4. Redis List (Key->Value[])**
Redis의 **List**는 Java의 `LinkedList<String>`과 유사한 **순서가 유지되는 데이터 구조**입니다.

### **특징**
- 값들이 **삽입된 순서대로 저장** (FIFO 또는 LIFO 가능)
- **중복 허용** (같은 값 여러 개 저장 가능)
- **왼쪽/오른쪽에서 삽입(push) 및 삭제(pop) 가능** → **큐/스택으로 활용 가능**
- `opsForList()`를 이용하여 관리

### **예제**
```java
// List에 데이터 추가 (왼쪽 삽입)
redisTemplate.opsForList().leftPush("queue:messages", "Message1");
redisTemplate.opsForList().leftPush("queue:messages", "Message2");

// List에서 데이터 가져오기
String msg = redisTemplate.opsForList().rightPop("queue:messages"); // "Message1"

// List 전체 조회
List<Object> messages = redisTemplate.opsForList().range("queue:messages", 0, -1);
```

### **Redis 내부 저장 방식**
```
LPUSH queue:messages "Message2"
LPUSH queue:messages "Message1"
RPOP queue:messages  → "Message1"
```
**Key: `queue:messages`**  
**Values: `["Message2", "Message1"]`** (순서 유지)

