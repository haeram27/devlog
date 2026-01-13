# Stream.mapMulti()

- [Stream.mapMulti()](#streammapmulti)
  - [평탄화](#평탄화)
    - [mapMulti() 의  default definition](#mapmulti-의--default-definition)
      - [java.util.stream/Stream.class](#javautilstreamstreamclass)
      - [java.util.function/BiConsumer.class](#javautilfunctionbiconsumerclass)
      - [java.util.function/Consumer.class](#javautilfunctionconsumerclass)
    - [예제: `List<Map<String, String>` -\> `List<String>`](#예제-listmapstring-string---liststring)
      - [코드](#코드)

---

## 평탄화

원본 stream의 각 요소로 부터 중간 stream을 생성한 후 중간 stream의 모든 요소를 합쳐 최종 반환 stream을 생성한다.  
`mapMulti()에 인자로 전달되는 BiConsumer의 두번째 인자인 Consumer.accept()로 전달되는 오브젝트만이 신규 반환 Stream의 element가 된다.`

***원본 stream의 각 요소는 작은 단위로 분해가 가능한 자료이어야 한다. (String, Array, Collection(list, set, map) 등)***

### mapMulti() 의  default definition

#### java.util.stream/Stream.class

```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
 default <R> Stream<R> mapMulti(BiConsumer<? super T, ? super Consumer<R>> mapper) {
        Objects.requireNonNull(mapper);
        return flatMap(e -> {
            SpinedBuffer<R> buffer = new SpinedBuffer<>();
            mapper.accept(e, buffer);
            return StreamSupport.stream(buffer.spliterator(), false);
        });
    }
}
```

mapMulti() 메소드는 BiConsumer 구현체를 인자로 받는다.  
mapMulti()는 실행시 BiConsumer 메소드에 두 개의 인자를 전달한다.
첫 번째 인자는 원본 Stream의 각 element 이다.  
두 번째 인자는 Consumer(mapper) interface이다.  
mapMulti()는 두 번째 인자인 `Comsumer.accept(\<new-element\>)에 전달되는 인자를 모아서 신규 Stream을 반환하는 메소드` 이다.

#### java.util.function/BiConsumer.class

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);
    ...
}
```

BiConsumer interface는 accept() abstract method를 구현해야 하며, 두 개의 인자를 받음

#### java.util.function/Consumer.class

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);
    ...
}
```

Consumer interface는 accept() abstract method를 구현해야 하며, 한한 개의 인자를 받음

### 예제: `List<Map<String, String>` -> `List<String>`

```java
List<Map<String, String> mapList = ...
mapList.stream().<String>mapMulti((map, consumer) -> map.values().forEach(v -> consumer.accept((String) v))).collect(Collectors.toList());
```

- `<String>mapMulti()`
  : `<String>`은 mapMulti() 실행 반환 stream의 요소 타입
- `map.values().forEach(v -> consumer.accept((String) v)))`
  : 원본 stream의 각 요소 map을 values()로 분해하여 consumer.accept()로 전달
    consumer.accept()로 전달된 데이터는 mapMulti() 반환 stream의 요소가 된다.
- `collect(Collectors.toList())`
  : mapMulti()의 반환 stream을 List로 변환

#### 코드

```java
// rs = [{"ip": "1.1.1.1"}, {"ip": "2.2.2.2"}, {"ip": "3.3.3.3"}]
// ipSet = {"1.1.1.1", "2.2.2.2", "3.3.3.3"}
List<Map<String, Object>> rs = ...;
Set<String> ipSet = rs.stream()
 .<String>mapMulti((map, consumer) -> map.values().forEach(v -> consumer.accept((String) v)))
 .collect(Collectors.toSet());
ipSet.forEach(t -> log.debug("{}", t));
```

이 예제는 `Map<String, String>` 의 원본 Stream으로 부터 value String만 추출하여 신규 Stream을 생성하는 예제이다.  
`<String>mapMulti((map, consumer) -> map.values().forEach(v -> consumer.accept((String) v)))` 가 핵심 코드이다.  
mapMulti() 앞에 `<String>` 제너릭이 사용되었는데, 이는 반환 되는 신규스트림의 내부 타입을 지정하는 메서드 제너릭이다.
일반적으로 메서드 제너릭의 경우 컴파일러러 의한 유추가 가능하여 명시하지 않지만, mapMulti()의 경우 복잡한 BiConsumer 인터페이스 구현으로 인해서 컴파일러 유추가 되지 않으므로 명시해주는 것이 좋다.
