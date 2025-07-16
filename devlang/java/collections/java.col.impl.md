# Collections

## implementations

| Interfaces | Hash table Implementations | Resizable array Implementations | Tree Implementations | Linked list Implementations | Hash table + Linked list Implementations |
| --- | --- | --- | --- | --- | --- |
| Set   | HashSet | | TreeSet | | LinkedHashSet |
| List  | | ArrayList | | LinkedList | |
| Deque | | ArrayDeque | | LinkedList | |
| Map   | HashMap | | TreeMap | | LinkedHashMap |

### List Interface

- ArrayList :
  - 가변길이 array. 상대적으로 빠르고 요소에 대해 순차적으로 접근할 수 있다.
- LinkedList :
  - 순서가 변경되는 경우 노드 링크만 변경하면 되므로 삽입, 삭제가 빈번할 때 빠르다

### Set Interface

- HashSet :
  - set, random access. 빠른 접근 속도를 가지고 있으나 순서를 예측할 수 없다
- TreeSet :
  - set, key 기준 정렬. 요소들의 정렬 방법을 직접 지정할 수 있다
- LinkedHashSet :
  - set, 삽입 순서 유지
  - key로 random-access 가능
  - iterator()으로 삽입순서의 Iterator 획득

### Map Interface

- TreeMap :
  - map의 key 기준 정렬. 정렬된 순서대로 Key와 Value를 저장하므로 빠른 검색이 가능하지만 요소를 추가할 때 정렬로 인해 오래걸린다.

- HashMap :
  - map, random access.
  - key 중복을 허용하지 않고 순서를 보장하지 않으며 null 값을 허용한다

- LinkedHashMap :
  - map, 삽입 순서 유지
  - key로 random-access 가능
  - keySet()/entrySet().iterator 이용하여 입력 순서로 순회가능
  - accessOrder=true 설정을 이용하여 LRU Cache 로써 사용 가능

### Deque Interface