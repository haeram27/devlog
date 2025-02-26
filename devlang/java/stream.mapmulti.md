# Stream.mapMulti()


## 평탄화
원본 stream의 각 요소로 부터 중간 stream을 생성한 후 중간 stream의 모든 요소를 합쳐 최종 반환 stream을 생성

***원본 stream의 각 요소는 작은 단위로 분해가 가능한 자료이어야 한다. (String, Array, Collection(list, set, map) 등)***


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


### 예제
```java
    <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
    default <R> Stream<R> mapMulti(BiConsumer<? super T, ? super Consumer<R>> mapper) {
        Objects.requireNonNull(mapper);
        return flatMap(e -> {
            SpinedBuffer<R> buffer = new SpinedBuffer<>();
            mapper.accept(e, buffer);
            return StreamSupport.stream(buffer.spliterator(), false);
        });
    }



var ipSet = rs.stream()
	.<String>mapMulti((map, consumer) -> map.values().forEach(v -> consumer.accept((String) v)))
	.collect(Collectors.toSet());
ipSet.forEach(t -> log.debug("{}", t));


// rs = [{"ip": "1.1.1.1"}, {"ip": "2.2.2.2"}, {"ip": "3.3.3.3"}, {"ip": "4.4.4.4"}]
// ipset = {"1.1.1.1", "2.2.2.2", "3.3.3.3", "4.4.4.4"}
List<Map<String, Object>> rs = ...;
Set<String> ipSet = rs.stream()
	.<String>mapMulti((map, consumer) -> map.values().forEach(v -> consumer.accept((String) v)))
	.collect(Collectors.toSet());
ipSet.forEach(t -> log.debug("{}", t));
```