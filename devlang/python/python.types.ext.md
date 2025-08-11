# python extended types

좋죠! `collections` 모듈의 **확장 자료구조**들을 한눈에 볼 수 있게 표로 정리했습니다. (추상 베이스 클래스 `collections.abc`는 제외하고, 실사용 컨테이너 위주로 정리했어요.)

## 핵심 컨테이너

| 타입 | 핵심 목적 | 주요 메서드/속성 | 시간 복잡도(대표) | 흔한 활용 |
| --- | --- | --- | --- | --- |
| `deque`       | 양쪽 끝에서 빠른 삽입/삭제가 가능한 큐 | `append`, `appendleft`, `pop`, `popleft`, `extend`, `extendleft`, `rotate`, `maxlen` | `append/popleft` **O(1)**, 인덱싱/삽입(중간) **O(n)**, `rotate(k)` **O(k)** | BFS/큐, 최근 N개 버퍼, 슬라이딩 윈도우 |
| `Counter`     | 해시 가능한 항목의 개수(빈도) 계산 | `most_common`, `elements`, 산술연산(`+`, `-`, `&`, `\|`), 딕트처럼 인덱싱 | 카운트 업데이트 **O(n)**, `most_common(m)` **O(n log m)** | 로그/단어·토큰 빈도, 상위 k 빈도 추출 |
| `OrderedDict` | (파이썬 3.7+에서 `dict`도 삽입순서 유지하지만) 순서 조작 메서드 제공 | `move_to_end(last=True/False)`, `popitem(last=True/False)` | 키 조회/삽입/삭제 평균 **O(1)** | LRU/LFU의 기본 구성, 양끝 pop 제어 |
| `defaultdict` | 키가 없을 때 자동으로 기본값 생성 | 생성자 `default_factory`, 누락 키 접근 시 자동 생성 | 조회/삽입 평균 **O(1)** | 카운팅(예: `int`), 리스트 누적(`list`), 집합 누적(`set`) |
| `ChainMap`    | 여러 매핑을 **읽기 전용으로** 겹쳐 보이게 하는 뷰 | `.maps`(리스트), `new_child`, `parents` | 조회: 체인 길이를 `k`라 하면 최악 **O(k)** | 다중 스코프(환경변수 + 디폴트), 설정 오버레이 |

* ChainMap은 여러 dict를 겹쳐서 구성 후 값의 뷰를 조회/수정하기 위한 컨테이너이다. 키 조회시 조회대상 dict의 우선순위를 정하기 위하여 사용. 조회시 `first to last dict`로 순서 검색하며 가장 먼저 찾은 키의 값을 반환한다.

## 래퍼/서브클래싱 보조

| 타입 | 핵심 목적 | 특징 | 사용 시점 |
| --- | --- | --- | --- |
| `UserDict`   | 딕셔너리를 쉽게 서브클래싱 | 내부에 실제 딕트 `data`를 보유(내장 `dict` 직접 상속보다 안전) | 커스텀 딕트 동작을 정의할 때     |
| `UserList`   | 리스트 서브클래싱 보조   | 내부에 `data` 리스트 | 커스텀 리스트 동작을 정의할 때      |
| `UserString` | 문자열 서브클래싱 보조   | 불변 문자열을 래핑   | 문자열 유사 객체에 메서드 추가할 때 |

## 레코드/튜플 팩토리

| 타입/팩토리 | 핵심 목적 | 주요 포인트 | 예시 |
| --- | --- | --- | --- |
| `namedtuple(typename, field_names)` | 필드 이름이 있는 가벼운 불변 레코드 | 튜플 기반(메모리 효율), 필드 접근은 속성으로, `_replace`, `_asdict` 제공 | `Point = namedtuple('Point', 'x y'); p = Point(1,2); p.x` |

---

### 간단 예시 모음

```python
from collections import deque, Counter, OrderedDict, defaultdict, ChainMap, namedtuple, UserDict, UserList, UserString

# deque
q = deque(maxlen=3)
for x in [1,2,3,4]: q.append(x)   # -> deque([2,3,4], maxlen=3)
q.rotate(1)                       # -> deque([4,2,3], maxlen=3)

# Counter
cnt = Counter("abracadabra")
cnt.most_common(2)                # -> [('a', 5), ('b', 2)]

# OrderedDict
od = OrderedDict([('a',1), ('b',2), ('c',3)])
od.move_to_end('a', last=False)   # 'a'를 맨 앞으로

# defaultdict
d = defaultdict(list)
for k, v in [('a',1), ('a',2), ('b',3)]: d[k].append(v)
# d -> {'a': [1,2], 'b':[3]}

# ChainMap
defaults = {'host':'localhost', 'port':8000}
env = {'port': 8080}
cm = ChainMap(env, defaults)      # 조회 시 env 우선

# namedtuple
Point = namedtuple('Point', 'x y')
p = Point(1, 2); p.x              # 1

# UserDict/UserList/UserString (간단 커스텀)
class LowerDict(UserDict):
    def __setitem__(self, key, value):
        super().__setitem__(key.lower(), value)

ld = LowerDict()
ld['Key'] = 1                     # 실제로는 'key'로 저장
```

---

## 추가 메모

* **왜 `OrderedDict`가 아직 유용한가?**
  파이썬 3.7+의 `dict`도 **삽입 순서 유지**는 하지만, `OrderedDict`는 `move_to_end` 같은 **순서 조작**과 **양끝 pop 제어**가 편리합니다. 또한 `OrderedDict`의 **동등성 비교는 순서를 고려**합니다.
* **`defaultdict` 주의점**: 존재하지 않는 키를 **접근만 해도** 엔트리가 생성될 수 있어, 단순 존재 확인에는 `in` 연산을 쓰는 것이 안전합니다.
* \*\*`ChainMap`\*\*은 상위 맵을 **변경 없이 겹쳐 보는 용도**에 적합합니다(쓰기 연산은 첫 맵에만 반영).

좋습니다! `collections` 주요 타입별로 **사용 패턴/안티패턴**과 **시간 복잡도 상세표**를 정리했습니다. (CPython 기준의 실무적 평균 복잡도이며, 최악/상수는 구현·데이터 형태에 따라 달라질 수 있어요.)

## 사용 패턴 / 안티패턴

| 타입 | 이런 때 쓰세요 (패턴) | 이렇게 쓰진 마세요 (안티패턴) | 이유/메모 |
| --- | --- | --- | --- |
| `deque`       | 슬라이딩 윈도우, 최근 N개 버퍼(`maxlen`), 양쪽 끝 큐(BFS), `rotate`로 라운드로빈 | 중간 인덱스에 잦은 삽입/삭제, 빈번한 임의 인덱싱 | 양끝 연산 O(1)이 장점. 중간 삽입/삭제/인덱싱은 O(n) |
| `Counter`     | 빈도 집계, 상위 k 추출(`most_common(k)`), 다중 카운트 합/교집합/합집합 | 부동소수/가변 키를 카운트 키로 사용, 음수·0 카운트 정리 없이 방치 | 키는 해시 가능해야 안정적. `+/-/&/\|`로 자연스러운 다중 집계 가능. `+cnt`로 음수·0 제거 |
| `OrderedDict` | 순서 조작(`move_to_end`, `popitem(last=…)`), 명시적 순서가 의미인 매핑 | 단순히 “삽입 순서 유지용”으로만 사용(`dict`면 충분) | 파이썬 3.7+의 `dict`도 삽입순서 유지. `OrderedDict`는 **순서 조작**과 **순서 기반 비교**가 강점 |
| `defaultdict` | 그룹핑 누적(`list`, `set`), 카운팅(`int`), 누락 키 처리 간소화             | 존재 여부 확인 목적으로 접근(사이드이펙트로 키 생성), 복잡한 팩토리(예: DB콜) | 누락 키 접근만으로 엔트리가 생김. 존재 확인은 `k in d` 사용 권장 |
| `ChainMap`    | 설정 레이어 오버레이(환경변수 > 사용자 > 기본), 다중 스코프 조회 | 상위 매핑을 “병합/복제”하려는 용도, 체인이 긴데 고빈도 조회 | 읽기 뷰에 가까움(쓰기 시 첫 맵에만). 체인이 길면 조회 비용 증가 |
| `namedtuple`  | 가벼운 불변 레코드, 튜플과 호환되는 구조적 데이터 | 필드를 자주 바꾸는 가변 레코드(그땐 `dataclass`/`attrs`) | 메모리·속도 유리하지만 불변. 변경은 `_replace` 비용 발생 |
| `UserDict`    | 커스텀 딕셔너리 동작을 안전하게 오버라이드 | 내장 `dict` 직접 상속으로 매직메서드 누락/미묘한 버그 | 내부 `data`에 위임해 상속 함정 줄임 |
| `UserList`    | 리스트 동작 확장(검증, 로깅 등) | 내장 `list` 직접 상속으로 일부 연산 누락 | `UserList.data` 래핑으로 일관된 커스터마이즈 |
| `UserString`  | 문자열 유사 타입에 동작 추가 | 직접 `str` 서브클래싱 후 불변/슬라이싱 동작 누락 | 불변 래퍼라 안전하게 오버라이드 가능 |

---

## 시간 복잡도 상세

### deque

| 연산 | 복잡도 | 메모 |
| --- | --- | --- |
| `append`, `appendleft` | O(1) | 양끝 고정 시간 |
| `pop`, `popleft`       | O(1) | - |
| `extend`, `extendleft` | O(m) | m은 추가 항목 수 |
| `rotate(k)`            | O(k) | k가 작은 경우 효율적 |
| 랜덤 인덱싱/중간 삽입  | O(n) | 리스트보다 불리 |

### Counter

| 연산 | 복잡도 | 메모 |
| --- | --- | --- |
| 생성자/업데이트(`update(iter)` ) | O(n)       | n은 항목 수      |
| 조회/증감 `c[x]+=1`              | 평균 O(1)  | 해시 기반        |
| `most_common(k)`                 | O(n log k) | 힙/선별 사용     |
| 산술: `c1 + c2`, `c1 - c2`       | O(u)       | u는 유니크 키 수 |
| 교집합/합집합 `&`, `\`           | O(u)       | 카운트의 min/max |

### OrderedDict

| 연산 | 복잡도 | 메모 |
| --- | --- | --- |
| 조회/삽입/삭제                | 평균 O(1) | 해시 기반             |
| `move_to_end(key, last=True)` | 평균 O(1) | 연결 리스트 링크 이동 |
| `popitem(last=True/False)`    | 평균 O(1) | 양끝 pop 가능         |
| 순회                          | O(n)      | 삽입 순서 유지        |

### defaultdict

| 연산 | 복잡도 | 메모 |
| --- | --- | --- |
| 조회(기존 키)        | 평균 O(1)                | - |
| 조회(신규 키 → 생성) | 평균 O(1) + factory 비용 | factory가 무거우면 병목 |
| 삽입/삭제            | 평균 O(1)                | 해시 기반 |
| 순회                 | O(n)                     | - |

### ChainMap

| 연산 | 복잡도 | 메모 |
| --- | --- | --- |
| 조회            | 최악 O(k) | k는 체인 길이           |
| 삽입/수정/삭제  | 평균 O(1) | **항상 첫 맵에만** 반영 |
| `new_child()`   | O(1)      | 얕은 새 프런트 추가     |
| `parents`       | O(1)      | 첫 맵 제거한 뷰 반환    |

### namedtuple

| 연산 | 복잡도  | 메모 |
| --- | --- | --- |
| 생성               | O(f) | f는 필드 수                         |
| 속성/인덱스 접근   | O(1) | 튜플과 동일                         |
| `_replace(**kw)`   | O(f) | 새 인스턴스 생성                    |
| `_asdict()`        | O(f) | `dict` 변환(파이썬 3.8+: 일반 dict) |

### UserDict / UserList / UserString

> 이들은 “래퍼/보조 클래스”라, 기본 연산은 내부 `data`에 위임됩니다. 복잡도는 보통 내장 컨테이너와 동일하되, **오버라이드한 메서드의 추가 로직** 비용이 더해질 수 있어요.

---

### 실전 팁 (요약)

* **큐/버퍼/윈도우**: `deque(maxlen=N)` → 자동 버림 + O(1) 양끝 연산
* **빈도 분석**: `Counter.update()` + `most_common(k)` → 상위 k만 필요하면 전체 정렬 대신 **O(n log k)**
* **순서 조작 필요**: 단순 보존이면 `dict`, **재배치 필요**이면 `OrderedDict`
* **그룹핑/카운팅 누락 키**: `defaultdict(list/int/set)`
* **구성값 오버레이**: `ChainMap(override, defaults)`
* **가벼운 불변 레코드**: `namedtuple` (자주 바꾸면 `dataclasses.dataclass(frozen=False)` 고려)
* **컨테이너 커스터마이징**: `User*` 계열로 안전하게 래핑

## `collections` 주요 타입별로 미세 최적화 체크리스트

핫패스에만 적용할 것. 과한 마이크로튜닝은 가독성을 저해 가능

### deque

* 슬라이딩 윈도우: `deque(maxlen=N)`으로 자동 버림 처리 → 분기/삭제 코드 제거.
* 합/통계 유지: 윈도우 합은 **증분 갱신**.

  ```python
  win = deque(maxlen=N); s = 0
  for x in stream:
      if len(win) == N: s -= win[0]
      win.append(x); s += x
  ```

* 큐/스택: 양끝 연산만 사용(`append`, `appendleft`, `pop`, `popleft`) → O(1) 보장.
* 회전: `rotate(k)`는 |k|가 작을 때만. 큰 k면 `k %= len(d)`.
* 대량 앞삽입은 `extendleft(reversed(seq))`가 더 빠름(`extendleft`는 역순 삽입).
* 반복 루프에서 메서드 바인딩(속성 조회 최소화):

  ```python
  ap, pl = dq.append, dq.popleft
  for _ in range(m): ap(...); pl()
  ```

* 임의 인덱싱/중간 삽입은 피하기 → 리스트로 전환해서 일괄 처리 후 다시 deque.

### Counter

* 집계는 `update`에 맡기기:

  ```python
  c = Counter(); c.update(iterable)     # 루프에서 +=1보다 빠름(파이썬 레벨 오버헤드↓)
  ```

* 상위 k만 필요하면 `most_common(k)` (전체 정렬 `sorted(c.items())`보다 유리).
* 합치기/교집합/합집합은 연산자 사용: `c1 + c2`, `c1 & c2`, `c1 | c2` → 파이썬 루프보다 빠름.
* 음수·0 카운트 정리: `+c` (유효 항목만 남김) 또는 `c += Counter()` (0/음수 제거).
* 희소 대량 업데이트는 `Counter(mapping)`보다 `Counter().update(mapping)`가 종종 유리.
* 키는 해시 안정 타입 사용(숫자/문자열/튜플). 가변 객체 금지.
* 누적 중간 결과가 거대하면 주기적으로 `c = +c`로 슬림화(메모리/순회 비용↓).

### OrderedDict

* LRU 캐시 뼈대:

  ```python
  od = OrderedDict()
  def use(key, val=None):
      if val is not None or key not in od: od[key] = val
      od.move_to_end(key)
      # 용량 초과 처리
      if len(od) > CAP: od.popitem(last=False)
  ```

* 단순 “삽입 순서 보존”만 필요하면 내장 `dict` 사용(파이썬 3.7+).
  **순서 조작**이 필요할 때만 `OrderedDict`.
* 빈번한 재배치가 있으면 `move_to_end`만 사용(삭제→재삽입보다 덜 비쌈).

### defaultdict

* 그룹핑/버킷화는 `defaultdict(list|set|int)`로 단일 패스 처리:

  ```python
  from collections import defaultdict
  buckets = defaultdict(list)
  ap = buckets.__getitem__  # 로컬 바인딩으로 속성 조회 최소화
  for k, v in data: ap(k).append(v)
  ```

* 존재 확인은 `in` 사용. `d[key]` 접근은 **엔트리 생성**하므로 부작용 주의.
* `default_factory`는 가벼운 콜러블 사용(무거운 I/O나 DB 호출 넣지 않기).

### ChainMap

* 읽기 우선 오버레이: 가장 **핫한 매핑을 앞**에 둬 조회 비용 최소화.
* 쓰기가 필요하면 `cm = ChainMap({}, *maps)`로 **프런트 버퍼** 추가(원본 보호).
* 체인이 길고 조회가 잦으면 한 번 **실제 dict로 플래튼**:

  ```python
  flat = dict(ChainMap(*maps))  # 핫패스에서 반복 조회 시 유리
  ```

### namedtuple

* 대량 생성은 팩토리 바인딩:

  ```python
  Point = namedtuple('Point', 'x y'); P = Point
  pts = [P(x, y) for x, y in pairs]
  ```

* 값 변경이 잦다면 `_replace` 연쇄 대신 **가변 구조(예: dataclass)** 고려.
* 딕트 변환이 잦다면 한 번만 `_asdict()` 후 그 딕트 재사용.

### UserDict / UserList / UserString

* C 최적화 경로 보존: 가능한 한 **메서드 오버라이드 최소화**, 내부 `data` 위임 후 크리티컬 로직만 추가.
* 반복되는 검증/로깅은 **데코레이터**로 감싸고, 핫패스 메서드(`__getitem__`, `append`)는 가벼워야 함.
* 불변语 의미 유지: `UserString`은 **새 인스턴스 반환** 패턴 지키기.

### 공통 미세 튜닝 팁

* **로컬 바인딩**: 루프 전에 `append = lst.append`, `get = d.get` 등으로 속성 조회 줄이기.
* **분기 밖으로 빼기**: 조건문/계산이 루프 내에서 불변이면 루프 외부로.
* **일괄 처리**: 가능하면 컨테이너 변환/확장은 한 번에(`extend`, `update`).
* **제너레이터 활용**: 중간 리스트 생성 피하기(특히 큰 데이터).
* **프로파일링 먼저**: `timeit`, `cProfile`, `line_profiler`로 **병목 확인 후** 적용.

---

원하시면 위 체크리스트를 기반으로 **실측 벤치마크 스크립트**(timeit + 다양한 입력 크기)도 만들어 드릴게요. 어떤 타입/패턴부터 확인할까요?
