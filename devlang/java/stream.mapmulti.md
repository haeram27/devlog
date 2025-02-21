# Stream.mapMulti()

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