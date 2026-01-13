
https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/stream/package-summary.html
https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/stream/Stream.html


# Stream이란
어떤 객체(요소, elements) 집합을 효율적으로 처리하기 위한 작업용 컨테이너
기존의 Java 자료구조인 Array, Collection과 같이 요소를 갖는 집합 자료의 각 요소들에 대한 반복처리 작업을 효율적으로 하기 위해서 개발된 컨테이너 클래스

실상 Array/List/Collection 자료구조와 같이 요소를 갖는 컨테이너 이지만 요소들에 대한 반복 처리를 위한 여러 메소드를 지원한다.
(Array/Collection > Stream 변환) > Stream 요소에 대한 반복작업 > (Stream > Array/ Collection 변환)의 방식으로 처리한다. 

일반 순차 stream외에 대규모 데이터 병렬 처리를 위한 parallel(병렬) stream 도 사용이 가능하다. 일반적으로 10000개 이상의 데이터에서 pararrel stream이 순차 stream보다 처리가 빠르다. 


## Stream Method 정리
### Stream generate
#### primitive
```java
IntStream IntStream.of(int t)
IntStream IntStream.of(int... values)
LongStream LongStream.of(long t)
LongStream LongStream.of(long... values)
DoubleStream DoubleStream.of(double t)
DoubleStream DoubleStream.of(double... values)
```

#### String
```java
IntStream {String}.chars()
IntStream {String}.codePoints()
```

#### Arrays
```java
Stream<T> Arrays.stream(T[] array)
Stream<T> Arrays.stream(T[] array, int startInclusive, int endExclusive)
IntStream Arrays.stream(int[] array)
IntStream Arrays.stream(int[] array, int startInclusive, int endExclusive)
LongStream Arrays.stream(long[] array)
LongStream Arrays.stream(long[] array, int startInclusive, int endExclusive)
DoubleStream Arrays.stream(double[] array)
DoubleStream Arrays.stream(double[] array, int startInclusive, int endExclusive)
```

#### Collection(List, Set)
```java
Stream<E> {Collection}.stream()
Stream<E> {Collection}.parallelStream()
Stream<E> {Map}.entrySet().stream()
```

#### Stream

```java
Stream<T> Stream.of(T t)
Stream<T> Stream.of(T... values)
Stream<T> Stream.ofNullable(T t)
Stream<T> Stream.empty()
Stream<T> Stream.iterate(final T seed, final UnaryOperator<T> f)
Stream<T> Stream.iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next)
Stream<T> Stream.generate(Supplier<? extends T> s)
Stream<T> Stream.builder().add(T t).build()
Stream<T> Stream.concat(Stream<? extends T> a, Stream<? extends T> b)
```

### Stream.intermediate operations
```java
<R> Stream<R>  map(Function<? super T,? extends R> mapper)
DoubleStream   mapToDouble(ToDoubleFunction<? super T> mapper)
IntStream      mapToInt(ToIntFunction<? super T> mapper)
LongStream     mapToLong(ToLongFunction<? super T> mapper)
<R> Stream<R>  flatMap(Function<? super T,? extends Stream<? extends R>> mapper)
DoubleStream   flatMapToDouble(Function<? super T,? extends DoubleStream> mapper)
IntStream      flatMapToInt(Function<? super T,? extends IntStream> mapper)
LongStream     flatMapToLong(Function<? super T,? extends LongStream> mapper)
<R> Stream<R>  mapMulti(BiConsumer<? super T,? super Consumer<R>> mapper)
DoubleStream   mapMultiToDouble(BiConsumer<? super T,? super DoubleConsumer> mapper)
IntStream      mapMultiToInt(BiConsumer<? super T,? super IntConsumer> mapper)
LongStream     mapMultiToLong(BiConsumer<? super T,? super LongConsumer> mapper)
```

#### map() ?
소스 스트림의 요소 기반으로 새 요소를 생성하여 신규 스트림 생성
flatMap() ?
소스 스트림의 요소가 분리 가능한 데이터(List / String 등)일 때 데이터를 분리하여 신규 요소들(s)을 생성하고 이를 담은 신규 Stream을 생성
mapMulti() ?
flatMap()과 동일한 용도지만 flatMap()의 mapper가 소스 스트림의 각 요소 별로 신규 스트림을 생성하여 반환해야 하는 반면 mapMulti()의 mapper는 소스 요소별 신규 스트림 생성을 하지 않아도 됨
요소별 신규 스트림 부하 제거를 위해 java 16 부터 추가되었으며 java 16 이상에서는 flatMap() 대신 mapMulti() 사용 권장

```java
Stream<T>      sorted()
Stream<T>      sorted(Comparator<? super T> comparator)

Stream<T>      distinct()
               중복제거
Stream<T>      filter(Predicate<? super T> predicate)
               조건 충족 요소 선택
Stream<T>      skip(long n)
               head 부터 버림
Stream<T>      limit(long maxSize)
               tail 부터 버림

Stream<T>      peek(Consumer<? super T> action)
               모든 요소에 대해 Consumer(반환값 없음)를 수행한다.
               주로 중간 처리 단계에서 Consumer를 통해 처리 결과를 디버깅하는데 사용한다.

Stream<T>      takeWhile(Predicate<? super T> predicate)
Stream<T>      dropWhile(Predicate<? super T> predicate)
```

# Stream.terminal operations

```java
<R,A> R        collect(Collector<? super T,A,R> collector)
<R> R          collect(Supplier<R> supplier, BiConsumer<R,? super T> accumulator, BiConsumer<R,R> combiner)

boolean        allMatch(Predicate<? super T> predicate)
boolean        anyMatch(Predicate<? super T> predicate)
boolean        noneMatch(Predicate<? super T> predicate)

Optional<T>    findAny()
Optional<T>    findFirst()

void           forEach(Consumer<? super T> action)
void           forEachOrdered(Consumer<? super T> action)

long           count()
Optional<T>    max(Comparator<? super T> comparator)
Optional<T>    min(Comparator<? super T> comparator)

Optional<T>    reduce(BinaryOperator<T> accumulator)
               여러 개 요소들을 하나의 값(객체)로 연산
T              reduce(T identity, BinaryOperator<T> accumulator)
<U> U          reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)

Object[]       toArray()
<A> A[]        toArray(IntFunction<A[]> generator)
default List<T>    toList()
```

# map(), flatmap()의 차이

```java
<R> Stream<R> map(Function<? super T,? extends R> mapper)
```
since java 8
용도: 스트림 요소의 변환 (타입, 값 모두 가능)
소스 스트림의 요소 기반으로 새 요소를 생성하여 신규 스트림 생성

```java
<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)
```
since java 8
용도: 스트림 요소의 분리 및 평탄화

e.g.
```java
List<List<T>> -> Stream<T>
List<String> -> Stream<word>
```

소스 스트림의 각 요소가 분리 가능한 데이터(List, String 등)일 때 각 요소를 분리하여 신규 요소들(s)을 생성하고 생성된 요소들을 갖는 신규 Stream을 생성

```java
<R> Stream<R> mapMulti(BiConsumer<? super T,? super Consumer<R>> mapper)
```
since java 16
용도: 스트림 요소의 분리 및 평탄화
e.g.
```java
List<List<T>> -> Stream<T>
List<String> -> Stream<word>
```
flatMap()과 동일한 용도지만 flatMap()이 소스 스트림의 각 요소 별로 신규 스트림을 생성하여 반환해야 하는 반면 multiMap()은 소스 요소별 신규 스트림 생성을 하지 않아도 됨
flatMap() 사용시 수반되는 요소별 신규 스트림 생성 부하를 제거를 위해 java 16 부터 추가됨
java 16 이상에서는 flatMap() 대신 mapMulti() 사용 권장

# map 메소드의 공통점 ?
소스 스트림의 요소(element) 기반으로 새 요소를 생성하여 신규 스트림 생성
새로운 Stream이 생성되면, 기존 Stream은 닫힌다(close)

# map() vs flatMap()/mapMulti() ?
@ 메소드의 용도 차이
map()은 소스 Stream의 각 요소를 변경하는 용도이며 신규 Stream의 요소 수는 기존과 같다.

flatMap()과 mapMulti()는 소스 스트림의 요소가 분리 가능한 데이터일 때 각 요소를 분리하여 신규 요소들(s)을 생성하고 이를 담은 신규 Stream을 생성하는 용도이며 신규 Stream의 요소 수는 기존 Stream과 다를수 있다. 보통 분리 가능한 요소는 집합 객체(Collection, String 등)이며 이 요소의 데이터를 분리하여 집합의 평탄화(다차원 집합 -> 일차원 집합)를 하는 것이 목적이다.

@ 신규 생성 요소의 수 차이
map()은 소스 스트림의 각 요소당 신규 요소 1개(one to one) 생성
flatMap()과 mapMulti()는 소스 스트림의 각 요소당 신규 요소 여러 개(one to Many) 생성

@ 각 메소드는 mapper 함수 차이
map()의 mapper 메소드 반환값은 신규 Stream의 요소(element) 객체이며, 신규 요소의 타입은 기존 Stream 요소의 타입과 달라도 된다.
flatMap()의 mapper 메소드 반환값은 신규 Stream 객체이며, 신규 요소의 타입은 기존 Stream 요소의 타입과 달라도 된다.
mapMulti()의 mapper 메소드는 반환값이 없다. 소스 스트림의 요소를 기반으로 새로 생성되는 요소들은 mapper의 두 번째 인자로 주어지는 consumer(소스 스트림으로 부터 주입)에 의해 신규 Stream에 삽입된다. 그래서 mapMulti()는 평탄화를 위한 중간 Stream 생성이 필요하지 않다.

# map() 예제
```java
List<String> words = List.of("hello", "world");
List<Integer> wordLengths = words.stream()
  .map(String::length)
  .collect(Collectors.toList());

System.out.println(wordLengths); // [5, 5] 출력
List<Integer> numbers = List.of(1, 2, 3, 4);
List<Integer> squares = numbers.stream()
   .map(n -> n * n)
   .collect(Collectors.toList());

System.out.println(squares); // [1, 4, 9, 16] 출력
```

# flatmap() 예제 

```java
<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)
Function type interface를 인자로 받음

@FunctionalInterface
public interface Function<T, R> {
    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
}

List<String> strings = List.of("hello world", "java stream");

List<String> words = strings.stream()
  .flatMap(sentence -> Stream.of(sentence.split("\\s+")))
  .collect(Collectors.toList());

System.out.println(words); // [hello, world, java, stream] 출력
List<List<String>> listOfLists = List.of(
        List.of("a", "b", "c"),
        List.of("d", "e"),
        List.of("f", "g", "h")
);

List<String> flatList = listOfLists.stream()
        .flatMap(List::stream)
        .collect(Collectors.toList());

System.out.println(flatList); // [a, b, c, d, e, f, g, h] 출력
```

# mapMulti() 예제
아래 예제에서 flatMap()과 mapMulti()는 같은 결과를 출력한다.

```java
List<String> -> Stream<String>
List<String> sentences = List.of("hello world", "java stream");

// flatMap SHOULD return new stream instance per element of source stream
sentences.stream()
         .flatMap(sentence -> Stream.of(sentence.split("\\s+")))
         .forEach(System.out::println); // "hello" "world" "java" "stream"

// mapMulti() is advanced version of flatMap()
// mapMulti do NOT need to create new stream instance
sentences.stream()
         .mapMulti((sentence, consumer) -> {
             for (String word : sentence.split("\\s+")) {
                 consumer.accept(word);
              }
          })
          .forEach(System.out::println); // "hello" "world" "java" "stream"

List<List<Integer>> -> Stream<Integer>
List<Integer> firstList = List.of(3, 4);
List<Integer> secondList = List.of(1, 2);
List<List<Integer>> listlist = List.of(firstList, secondList);

// flatMap SHOULD return new stream instance per element of source stream
listlist.stream()
        .flatMap(List::stream)
        .forEach(System.out::println); // 3 4 1 2

// mapMulti() is advanced version of flatMap()
// mapMulti do NOT need to create new stream instance
listlist.stream()
        .mapMulti((list, consumer) -> list.forEach(consumer))
        .forEach(System.out::println); // 3 4 1 2

listlist.stream()
        .mapMulti((list, consumer) -> {
                for (Integer i : list) { consumer.accept(i); }
        })
        .forEach(System.out::println); // 3 4 1 2
```


# map() vs flatmap() 정리
특징	map()	flatMap()	mapMulti (java 16+)
변환 함수(mapper)	각 요소를 하나의 요소로 변환	평탄화
소스 스트림의 요소를 처리하여 여러 개의 신규 요소를 생성하고 스트림으로 변환한 후, 이 스트림들을 다시 하나의 최종 스트림으로 병합	평탄화:
중간 Stream을 생성하지 않도록 하는 flatMap의 개선 버전
결과 스트림 크기	원래 스트림과 동일	원래 스트림과 다를 수 있음	원래 스트림과 다를 수 있음
용도	개별 요소의 변환	평탄화: 다중 Collection을 단일 Collection으로 변환

e.g.	평탄화: 다중 Collection을 단일 Collection으로 변환

        List<List<T>> -> Stream<T>	e.g.
    e.g.	List<String> -> Stream<word>	List<List<T>> -> Stream<T>
    타입 변환		List<String> -> Stream<word>
    속성 추출

## Java 스트림 Stream

이번 포스트에서는 Java 8의 스트림(Stream)을 살펴봅니다.
총 두 개의 포스트로, 기본적인 내용을 총정리하는 이번 포스트와 좀 더 고급 내용을 다루는 다음 포스트로 나뉘어져 있습니다.
    · Java 스트림 Stream (1) 총정리
    · Java 스트림 Stream (2) 고급

Java 스트림 Stream (1) 총정리

살펴볼 내용
이번 포스트에서 다루는 내용은 다음과 같습니다. 아는 내용이라면 다음 포스트를 살펴보시는게 좋습니다.

    · 생성하기
        § 배열 / 컬렉션 / 빈 스트림
        § Stream.builder() / Stream.generate() / Stream.iterate()
        § 기본 타입형 / String / 파일 스트림
        § 병렬 스트림 / 스트림 연결하기
     
    · 가공하기
        § Filtering
        § Mapping
        § Sorting
        § Iterating
     
    · 결과 만들기
        § Calculating
        § Reduction
        § Collecting
        § Matching
        § Iterating


스트림 Streams

자바 8에서 추가한 스트림(Streams)은 람다를 활용할 수 있는 기술 중 하나입니다. 자바 8 이전에는 배열 또는 컬렉션 인스턴스를 다루는 방법은 for 또는 foreach 문을 돌면서 요소 하나씩을 꺼내서 다루는 방법이 었습니다. 간단한 경우라면 상관없지만 로직이 복잡해질수록 코드의 양이 많아져 여러 로직이 섞이게 되고, 메소드를 나눌 경우 루프를 여러 번 도는 경우가 발생합니다.

스트림은 '데이터의 흐름’입니다. 배열 또는 컬렉션 인스턴스에 함수 여러 개를 조합해서 원하는 결과를 필터링하고 가공된 결과를 얻을 수 있습니다. 또한 람다를 이용해서 코드의 양을 줄이고 간결하게 표현할 수 있습니다. 즉, 배열과 컬렉션을 함수형으로 처리할 수 있습니다.

또 하나의 장점은 간단하게 병렬처리(multi-threading)가 가능하다는 점입니다. 하나의 작업을 둘 이상의 작업으로 잘게 나눠서 동시에 진행하는 것을 병렬 처리(parallel processing)라고 합니다. 즉 쓰레드를 이용해 많은 요소들을 빠르게 처리할 수 있습니다.

스트림에 대한 내용은 크게 세 가지로 나눌 수 있습니다.
    · 생성하기 : 스트림 인스턴스 생성.
    · 가공하기 : 필터링(filtering) 및 맵핑(mapping) 등 원하는 결과를 만들어가는 중간 작업(intermediate operations).
    · 결과 만들기 : 최종적으로 결과를 만들어내는 작업(terminal operations).
전체 -> 맵핑 -> 필터링 1 -> 필터링 2 -> 결과 만들기 -> 결과물


생성하기

보통 배열과 컬렉션을 이용해서 스트림을 만들지만 이 외에도 다양한 방법으로 스트림을 만들 수 있습니다.
하나씩 살펴보겠습니다.

배열 스트림
스트림을 이용하기 위해서는 먼저 생성을 해야 합니다.
스트림은 배열 또는 컬렉션 인스턴스를 이용해서 생성할 수 있습니다.
배열은 다음과 같이 Arrays.stream 메소드를 사용합니다.
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
Stream<String> streamOfArrayPart = 
  Arrays.stream(arr, 1, 3); // 1~2 요소 [b, c]

컬렉션 스트림
컬렉션 타입(Collection, List, Set)의 경우 인터페이스에 추가된 디폴트 메소드 stream 을 이용해서 스트림을 만들 수 있습니다.
public interface Collection<E> extends Iterable<E> {
  default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
  } 
  // ...
}

그러면 다음과 같이 생성할 수 있습니다.
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream(); // 병렬 처리 스트림


비어 있는 스트림
비어 있는 스트림(empty streams)도 생성할 수 있습니다. 언제 빈 스트림이 필요할까요?
빈 스트림은 요소가 없을 때 null 대신 사용할 수 있습니다.
public Stream<String> streamOf(List<String> list) {
  return list == null || list.isEmpty()
    ? Stream.empty()
    : list.stream();
}


Stream.builder()
빌더(Builder)를 사용하면 스트림에 직접적으로 원하는 값을 넣을 수 있습니다. 마지막에 build 메소드로 스트림을 리턴합니다.
Stream<String> builderStream = 
  Stream.<String>builder()
    .add("Eric").add("Elena").add("Java")
    .build(); // [Eric, Elena, Java]


Stream.generate()
generate 메소드를 이용하면 Supplier<T> 에 해당하는 람다로 값을 넣을 수 있습니다. Supplier<T> 는 인자는 없고 리턴값만 있는 함수형 인터페이스죠. 람다에서 리턴하는 값이 들어갑니다.
public static<T> Stream<T> generate(Supplier<T> s) { ... }

이 때 생성되는 스트림은 크기가 정해져있지 않고 무한(infinite)하기 때문에 특정 사이즈로 최대 크기를 제한해야 합니다.
Stream<String> generatedStream = 
  Stream.generate(() -> "gen").limit(5); // [el, el, el, el, el]

5개의 “gen” 이 들어간 스트림이 생성됩니다.


Stream.iterate()
iterate 메소드를 이용하면 초기값과 해당 값을 다루는 람다를 이용해서 스트림에 들어갈 요소를 만듭니다. 다음 예제에서는 30이 초기값이고 값이 2씩 증가하는 값들이 들어가게 됩니다. 즉 요소가 다음 요소의 인풋으로 들어갑니다. 이 방법도 스트림의 사이즈가 무한하기 때문에 특정 사이즈로 제한해야 합니다.
Stream<Integer> iteratedStream = 
  Stream.iterate(30, n -> n + 2).limit(5); // [30, 32, 34, 36, 38]


기본 타입형 스트림
물론 제네릭을 사용하면 리스트나 배열을 이용해서 기본 타입(int, long, double) 스트림을 생성할 수 있습니다. 하지만 제네릭을 사용하지 않고 직접적으로 해당 타입의 스트림을 다룰 수도 있습니다. range 와 rangeClosed 는 범위의 차이입니다. 두 번째 인자인 종료지점이 포함되느냐 안되느냐의 차이입니다.
IntStream intStream = IntStream.range(1, 5); // [1, 2, 3, 4]
LongStream longStream = LongStream.rangeClosed(1, 5); // [1, 2, 3, 4, 5]

제네릭을 사용하지 않기 때문에 불필요한 오토박싱(auto-boxing)이 일어나지 않습니다. 필요한 경우 boxed 메소드를 이용해서 박싱(boxing)할 수 있습니다.
Stream<Integer> boxedIntStream = IntStream.range(1, 5).boxed();

Java 8 의 Random 클래스는 난수를 가지고 세 가지 타입의 스트림(IntStream, LongStream, DoubleStream)을 만들어낼 수 있습니다. 쉽게 난수 스트림을 생성해서 여러가지 후속 작업을 취할 수 있어 유용합니다.
DoubleStream doubles = new Random().doubles(3); // 난수 3개 생성


문자열 스트링
스트링을 이용해서 스트림을 생성할수도 있습니다. 다음은 스트링의 각 문자(char)를 IntStream 으로 변환한 예제입니다. char 는 문자이지만 본질적으로는 숫자이기 때문에 가능합니다.
IntStream charsStream = 
  "Stream".chars(); // [83, 116, 114, 101, 97, 109]

다음은 정규표현식(RegEx)을 이용해서 문자열을 자르고, 각 요소들로 스트림을 만든 예제입니다.
Stream<String> stringStream = 
  Pattern.compile(", ").splitAsStream("Eric, Elena, Java");
  // [Eric, Elena, Java]


파일 스트림
자바 NIO 의 Files 클래스의 lines 메소드는 해당 파일의 각 라인을 스트링 타입의 스트림으로 만들어줍니다.
Stream<String> lineStream = 
  Files.lines(Paths.get("file.txt"), 
              Charset.forName("UTF-8"));


병렬 스트림 Parallel Stream
스트림 생성 시 사용하는 stream 대신 parallelStream 메소드를 사용해서 병렬 스트림을 쉽게 생성할 수 있습니다. 내부적으로는 쓰레드를 처리하기 위해 자바 7부터 도입된 Fork/Join framework 를 사용합니다.
// 병렬 스트림 생성
Stream<Product> parallelStream = productList.parallelStream();
 
// 병렬 여부 확인
boolean isParallel = parallelStream.isParallel();

따라서 다음 코드는 각 작업을 쓰레드를 이용해 병렬 처리됩니다.
boolean isMany = parallelStream
  .map(product -> product.getAmount() * 10)
  .anyMatch(amount -> amount > 200);

다음은 배열을 이용해서 병렬 스트림을 생성하는 경우입니다.
Arrays.stream(arr).parallel();

컬렉션과 배열이 아닌 경우는 다음과 같이 parallel 메소드를 이용해서 처리합니다.
IntStream intStream = IntStream.range(1, 150).parallel();
boolean isParallel = intStream.isParallel();

다시 시퀀셜(sequential) 모드로 돌리고 싶다면 다음처럼 sequential 메소드를 사용합니다.
뒤에서 한번 더 다루겠지만 반드시 병렬 스트림이 좋은 것은 아닙니다.
IntStream intStream = intStream.sequential();
boolean isParallel = intStream.isParallel();


스트림 연결하기
Stream.concat 메소드를 이용해 두 개의 스트림을 연결해서 새로운 스트림을 만들어낼 수 있습니다.
Stream<String> stream1 = Stream.of("Java", "Scala", "Groovy");
Stream<String> stream2 = Stream.of("Python", "Go", "Swift");
Stream<String> concat = Stream.concat(stream1, stream2);
// [Java, Scala, Groovy, Python, Go, Swift]



가공하기

전체 요소 중에서 다음과 같은 API 를 이용해서 내가 원하는 것만 뽑아낼 수 있습니다. 이러한 가공 단계를 중간 작업(intermediate operations)이라고 하는데, 이러한 작업은 스트림을 리턴하기 때문에 여러 작업을 이어 붙여서(chaining) 작성할 수 있습니다.
List<String> names = Arrays.asList("Eric", "Elena", "Java");
아래 나오는 예제 코드는 위와 같은 리스트를 대상으로 합니다.


Filtering

필터(filter)은 스트림 내 요소들을 하나씩 평가해서 걸러내는 작업입니다. 인자로 받는 Predicate 는 boolean 을 리턴하는 함수형 인터페이스로 평가식이 들어가게 됩니다.
Stream<T> filter(Predicate<? super T> predicate);

간단한 예제입니다.
Stream<String> stream = 
  names.stream()
  .filter(name -> name.contains("a"));
// [Elena, Java]

스트림의 각 요소에 대해서 평가식을 실행하게 되고 ‘a’ 가 들어간 이름만 들어간 스트림이 리턴됩니다.


Mapping

맵(map)은 스트림 내 요소들을 하나씩 특정 값으로 변환해줍니다.
이 때 값을 변환하기 위한 람다를 인자로 받습니다.
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

스트림에 들어가 있는 값이 input 이 되어서 특정 로직을 거친 후 output 이 되어 (리턴되는) 새로운 스트림에 담기게 됩니다. 이러한 작업을 맵핑(mapping)이라고 합니다.
간단한 예제입니다. 스트림 내 String 의 toUpperCase 메소드를 실행해서 대문자로 변환한 값들이 담긴 스트림을 리턴합니다.
Stream<String> stream = 
  names.stream()
  .map(String::toUpperCase);
// [ERIC, ELENA, JAVA]

다음처럼 요소 내 들어있는 Product 개체의 수량을 꺼내올 수도 있습니다. 각 ‘상품’을 ‘상품의 수량’으로 맵핑하는거죠.
Stream<Integer> stream = 
  productList.stream()
  .map(Product::getAmount);
// [23, 14, 13, 23, 13]

map 이외에도 조금 더 복잡한 flatMap 메소드도 있습니다.
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
인자로 mapper를 받고 있는데, 리턴 타입이 Stream 입니다. 즉, 새로운 스트림을 생성해서 리턴하는 람다를 넘겨야합니다. flatMap 은 중첩 구조를 한 단계 제거하고 단일 컬렉션으로 만들어주는 역할을 합니다. 이러한 작업을 플래트닝(flattening)이라고 합니다.

다음과 같은 중첩된 리스트가 있습니다.
List<List<String>> list = 
  Arrays.asList(Arrays.asList("a"), 
                Arrays.asList("b"));
// [[a], [b]]

이를 flatMap을 사용해서 중첩 구조를 제거한 후 작업할 수 있습니다.
List<String> flatList = 
  list.stream()
  .flatMap(Collection::stream)
  .collect(Collectors.toList());
// [a, b]

이번엔 객체에 적용해보겠습니다.
students.stream()
  .flatMapToInt(student -> 
                IntStream.of(student.getKor(), 
                             student.getEng(), 
                             student.getMath()))
  .average().ifPresent(avg -> 
                       System.out.println(Math.round(avg * 10)/10.0));

위 예제에서는 학생 객체를 가진 스트림에서 학생의 국영수 점수를 뽑아 새로운 스트림을 만들어 평균을 구하는 코드입니다. 이는 map 메소드 자체만으로는 한번에 할 수 없는 기능입니다.




Sorting
정렬의 방법은 다른 정렬과 마찬가지로 Comparator 를 이용합니다.
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);

인자 없이 그냥 호출할 경우 오름차순으로 정렬합니다.
IntStream.of(14, 11, 20, 39, 23)
  .sorted()
  .boxed()
  .collect(Collectors.toList());
// [11, 14, 20, 23, 39]

인자를 넘기는 경우와 비교해보겠습니다. 스트링 리스트에서 알파벳 순으로 정렬한 코드와 Comparator 를 넘겨서 역순으로 정렬한 코드입니다.
List<String> lang = 
  Arrays.asList("Java", "Scala", "Groovy", "Python", "Go", "Swift");
 
lang.stream()
  .sorted()
  .collect(Collectors.toList());
// [Go, Groovy, Java, Python, Scala, Swift]
 
lang.stream()
  .sorted(Comparator.reverseOrder())
  .collect(Collectors.toList());
// [Swift, Scala, Python, Java, Groovy, Go]

Comparator 의 compare 메소드는 두 인자를 비교해서 값을 리턴합니다.
int compare(T o1, T o2)

기본적으로 Comparator 사용법과 동일합니다.
이를 이용해서 문자열 길이를 기준으로 정렬해보겠습니다.
lang.stream()
  .sorted(Comparator.comparingInt(String::length))
  .collect(Collectors.toList());
// [Go, Java, Scala, Swift, Groovy, Python]
 
lang.stream()
  .sorted((s1, s2) -> s2.length() - s1.length())
  .collect(Collectors.toList());
// [Groovy, Python, Scala, Swift, Java, Go]


Iterating
스트림 내 요소들 각각을 대상으로 특정 연산을 수행하는 메소드로는 peek 이 있습니다. ‘peek’ 은 그냥 확인해본다는 단어 뜻처럼 특정 결과를 반환하지 않는 함수형 인터페이스 Consumer 를 인자로 받습니다.
Stream<T> peek(Consumer<? super T> action);

따라서 스트림 내 요소들 각각에 특정 작업을 수행할 뿐 결과에 영향을 미치지 않습니다. 다음처럼 작업을 처리하는 중간에 결과를 확인해볼 때 사용할 수 있습니다.
int sum = IntStream.of(1, 3, 5, 7, 9)
  .peek(System.out::println)
  .sum();



결과 만들기

가공한 스트림을 가지고 내가 사용할 결과값으로 만들어내는 단계입니다. 따라서 스트림을 끝내는 최종 작업(terminal operations)입니다.

Calculating
스트림 API 는 다양한 종료 작업을 제공합니다. 최소, 최대, 합, 평균 등 기본형 타입으로 결과를 만들어낼 수 있습니다.
long count = IntStream.of(1, 3, 5, 7, 9).count();
long sum = LongStream.of(1, 3, 5, 7, 9).sum();

만약 스트림이 비어 있는 경우 count 와 sum 은 0을 출력하면 됩니다. 하지만 평균, 최소, 최대의 경우에는 표현할 수가 없기 때문에 Optional 을 이용해 리턴합니다.
OptionalInt min = IntStream.of(1, 3, 5, 7, 9).min();
OptionalInt max = IntStream.of(1, 3, 5, 7, 9).max();

스트림에서 바로 ifPresent 메소드를 이용해서 Optional 을 처리할 수 있습니다.
DoubleStream.of(1.1, 2.2, 3.3, 4.4, 5.5)
  .average()
  .ifPresent(System.out::println);

이 외에도 사용자가 원하는대로 결과를 만들어내기 위해 reduce 와 collect 메소드를 제공합니다. 이 두 가지 메소드를 좀 더 알아보겠습니다.


Reduction
스트림은 reduce라는 메소드를 이용해서 결과를 만들어냅니다. 람다 예제에서 살펴봤듯이 스트림에 있는 여러 요소의 총합을 낼 수도 있습니다.
다음은 reduce 메소드는 총 세 가지의 파라미터를 받을 수 있습니다.
    · accumulator :
각 요소를 처리하는 계산 로직. 각 요소가 올 때마다 중간 결과를 생성하는 로직.
    · identity :
계산을 위한 초기값으로 스트림이 비어서 계산할 내용이 없더라도 이 값은 리턴.
    · combiner :
병렬(parallel) 스트림에서 나눠 계산한 결과를 하나로 합치는 동작하는 로직.
// 1개 (accumulator)
Optional<T> reduce(BinaryOperator<T> accumulator);
 
// 2개 (identity)
T reduce(T identity, BinaryOperator<T> accumulator);
 
// 3개 (combiner)
<U> U reduce(U identity,
  BiFunction<U, ? super T, U> accumulator,
  BinaryOperator<U> combiner);

먼저 인자가 하나만 있는 경우입니다. 여기서 BinaryOperator<T> 는 같은 타입의 인자 두 개를 받아 같은 타입의 결과를 반환하는 함수형 인터페이스입니다. 다음 예제에서는 두 값을 더하는 람다를 넘겨주고 있습니다. 따라서 결과는 6(1 + 2 + 3)이 됩니다.
OptionalInt reduced = 
  IntStream.range(1, 4) // [1, 2, 3]
  .reduce((a, b) -> {
    return Integer.sum(a, b);
  });

이번엔 두 개의 인자를 받는 경우입니다. 여기서 10은 초기값이고, 스트림 내 값을 더해서 결과는 16(10 + 1 + 2 + 3)이 됩니다. 여기서 람다는 메소드 참조(method reference)를 이용해서 넘길 수 있습니다.
int reducedTwoParams = 
  IntStream.range(1, 4) // [1, 2, 3]
  .reduce(10, Integer::sum); // method reference

마지막으로 세 개의 인자를 받는 경우입니다. Combiner 가 하는 역할을 설명만 봤을 때는 잘 이해가 안갈 수 있는데요, 코드를 한번 살펴봅시다. 그런데 다음 코드를 실행해보면 이상하게 마지막 인자인 combiner 는 실행되지 않습니다.
Integer reducedParams = Stream.of(1, 2, 3)
  .reduce(10, // identity
          Integer::sum, // accumulator
          (a, b) -> {
            System.out.println("combiner was called");
            return a + b;
          });

Combiner 는 병렬 처리 시 각자 다른 쓰레드에서 실행한 결과를 마지막에 합치는 단계입니다. 따라서 병렬 스트림에서만 동작합니다.
Integer reducedParallel = Arrays.asList(1, 2, 3)
  .parallelStream()
  .reduce(10,
          Integer::sum,
          (a, b) -> {
            System.out.println("combiner was called");
            return a + b;
          });

결과는 다음과 같이 36이 나옵니다. 먼저 accumulator 는 총 세 번 동작합니다. 초기값 10에 각 스트림 값을 더한 세 개의 값(10 + 1 = 11, 10 + 2 = 12, 10 + 3 = 13)을 계산합니다. Combiner 는 identity 와 accumulator 를 가지고 여러 쓰레드에서 나눠 계산한 결과를 합치는 역할입니다. 12 + 13 = 25, 25 + 11 = 36 이렇게 두 번 호출됩니다.
combiner was called
combiner was called
36

병렬 스트림이 무조건 시퀀셜보다 좋은 것은 아닙니다. 오히려 간단한 경우에는 이렇게 부가적인 처리가 필요하기 때문에 오히려 느릴 수도 있습니다.


Collecting
collect 메소드는 또 다른 종료 작업입니다. Collector 타입의 인자를 받아서 처리를 하는데요, 자주 사용하는 작업은 Collectors 객체에서 제공하고 있습니다.
이번 예제에서는 다음과 같은 간단한 리스트를 사용합니다. Product 객체는 수량(amout)과 이름(name)을 가지고 있습니다.
List<Product> productList = 
  Arrays.asList(new Product(23, "potatoes"),
                new Product(14, "orange"),
                new Product(13, "lemon"),
                new Product(23, "bread"),
                new Product(13, "sugar"));


Collectors.toList()
스트림에서 작업한 결과를 담은 리스트로 반환합니다. 다음 예제에서는 map 으로 각 요소의 이름을 가져온 후 Collectors.toList 를 이용해서 리스트로 결과를 가져옵니다.
List<String> collectorCollection =
  productList.stream()
    .map(Product::getName)
    .collect(Collectors.toList());
// [potatoes, orange, lemon, bread, sugar]


Collectors.joining()
스트림에서 작업한 결과를 하나의 스트링으로 이어 붙일 수 있습니다.
String listToString = 
 productList.stream()
  .map(Product::getName)
  .collect(Collectors.joining());
// potatoesorangelemonbreadsugar
Collectors.joining 은 세 개의 인자를 받을 수 있습니다. 이를 이용하면 간단하게 스트링을 조합할 수 있습니다.
    · delimiter : 각 요소 중간에 들어가 요소를 구분시켜주는 구분자
    · prefix : 결과 맨 앞에 붙는 문자
    · suffix : 결과 맨 뒤에 붙는 문자
String listToString = 
 productList.stream()
  .map(Product::getName)
  .collect(Collectors.joining(", ", "<", ">"));
// <potatoes, orange, lemon, bread, sugar>


Collectors.averageingInt()
숫자 값(Integer value )의 평균(arithmetic mean)을 냅니다.
Double averageAmount = 
 productList.stream()
  .collect(Collectors.averagingInt(Product::getAmount));
// 17.2


Collectors.summingInt()
숫자값의 합(sum)을 냅니다.
Integer summingAmount = 
 productList.stream()
  .collect(Collectors.summingInt(Product::getAmount));
// 86

IntStream 으로 바꿔주는 mapToInt 메소드를 사용해서 좀 더 간단하게 표현할 수 있습니다.
Integer summingAmount = 
  productList.stream()
  .mapToInt(Product::getAmount)
  .sum(); // 86


Collectors.summarizingInt()
만약 합계와 평균 모두 필요하다면 스트림을 두 번 생성해야 할까요? 이런 정보를 한번에 얻을 수 있는 방법으로는 summarizingInt 메소드가 있습니다.
IntSummaryStatistics statistics = 
 productList.stream()
  .collect(Collectors.summarizingInt(Product::getAmount));

이렇게 받아온 IntSummaryStatistics 객체에는 다음과 같은 정보가 담겨 있습니다.
IntSummaryStatistics {count=5, sum=86, min=13, average=17.200000, max=23}
    · 개수 getCount()
    · 합계 getSum()
    · 평균 getAverage()
    · 최소 getMin()
    · 최대 getMax()
이를 이용하면 collect 전에 이런 통계 작업을 위한 map 을 호출할 필요가 없게 됩니다. 위에서 살펴본 averaging, summing, summarizing 메소드는 각 기본 타입(int, long, double)별로 제공됩니다.


Collectors.groupingBy()
특정 조건으로 요소들을 그룹지을 수 있습니다. 수량을 기준으로 그룹핑해보겠습니다. 여기서 받는 인자는 함수형 인터페이스 Function 입니다.
Map<Integer, List<Product>> collectorMapOfLists =
 productList.stream()
  .collect(Collectors.groupingBy(Product::getAmount));

결과는 Map 타입으로 나오는데요, 같은 수량이면 리스트로 묶어서 보여줍니다.
{23=[Product{amount=23, name='potatoes'}, 
     Product{amount=23, name='bread'}], 
 13=[Product{amount=13, name='lemon'}, 
     Product{amount=13, name='sugar'}], 
 14=[Product{amount=14, name='orange'}]}


Collectors.partitioningBy()
위의 groupingBy 함수형 인터페이스 Function 을 이용해서 특정 값을 기준으로 스트림 내 요소들을 묶었다면, partitioningBy 은 함수형 인터페이스 Predicate 를 받습니다. Predicate 는 인자를 받아서 boolean 값을 리턴합니다.
Map<Boolean, List<Product>> mapPartitioned = 
  productList.stream()
  .collect(Collectors.partitioningBy(el -> el.getAmount() > 15));

따라서 평가를 하는 함수를 통해서 스트림 내 요소들을 true 와 false 두 가지로 나눌 수 있습니다.
{false=[Product{amount=14, name='orange'}, 
        Product{amount=13, name='lemon'}, 
        Product{amount=13, name='sugar'}], 
 true=[Product{amount=23, name='potatoes'}, 
       Product{amount=23, name='bread'}]}


Collectors.collectingAndThen()
특정 타입으로 결과를 collect 한 이후에 추가 작업이 필요한 경우에 사용할 수 있습니다. 이 메소드의 시그니쳐는 다음과 같습니다. finisher 가 추가된 모양인데, 이 피니셔는 collect 를 한 후에 실행할 작업을 의미합니다.
public static<T,A,R,RR> Collector<T,A,RR> collectingAndThen(
  Collector<T,A,R> downstream,
  Function<R,RR> finisher) { ... }

다음 예제는 Collectors.toSet 을 이용해서 결과를 Set 으로 collect 한 후 수정불가한 Set 으로 변환하는 작업을 추가로 실행하는 코드입니다.
Set<Product> unmodifiableSet = 
 productList.stream()
  .collect(Collectors.collectingAndThen(Collectors.toSet(),
                                        Collections::unmodifiableSet));


Collector.of()
여러가지 상황에서 사용할 수 있는 메소드들을 살펴봤습니다. 이 외에 필요한 로직이 있다면 직접 collector 를 만들 수도 있습니다. accumulator 와 combiner 는 reduce 에서 살펴본 내용과 동일합니다.
public static<T, R> Collector<T, R, R> of(
  Supplier<R> supplier, // new collector 생성
  BiConsumer<R, T> accumulator, // 두 값을 가지고 계산
  BinaryOperator<R> combiner, // 계산한 결과를 수집하는 함수.
  Characteristics... characteristics) { ... }

코드를 보시면 더 이해가 쉬우실 겁니다. 다음 코드에서는 collector 를 하나 생성합니다. 컬렉터를 생성하는 supplier 에 LinkedList 의 생성자를 넘겨줍니다. 그리고 accumulator 에는 리스트에 추가하는 add 메소드를 넘겨주고 있습니다. 따라서 이 컬렉터는 스트림의 각 요소에 대해서 LinkedList 를 만들고 요소를 추가하게 됩니다. 마지막으로 combiner 를 이용해 결과를 조합하는데, 생성된 리스트들을 하나의 리스트로 합치고 있습니다.
Collector<Product, ?, LinkedList<Product>> toLinkedList = 
  Collector.of(LinkedList::new, 
               LinkedList::add, 
               (first, second) -> {
                 first.addAll(second);
                 return first;
               });

따라서 다음과 같이 collect 메소드에 우리가 만든 커스텀 컬렉터를 넘겨줄 수 있고, 결과가 담긴 LinkedList 가 반환됩니다.
LinkedList<Product> linkedListOfPersons = productList.stream().collect(toLinkedList);

Matching
매칭은 조건식 람다 Predicate 를 받아서 해당 조건을 만족하는 요소가 있는지 체크한 결과를 리턴합니다. 다음과 같은 세 가지 메소드가 있습니다.
    · 하나라도 조건을 만족하는 요소가 있는지(anyMatch)
    · 모두 조건을 만족하는지(allMatch)
    · 모두 조건을 만족하지 않는지(noneMatch)
boolean anyMatch(Predicate<? super T> predicate);
boolean allMatch(Predicate<? super T> predicate);
boolean noneMatch(Predicate<? super T> predicate);

간단한 예제입니다. 다음 매칭 결과는 모두 true 입니다.
List<String> names = Arrays.asList("Eric", "Elena", "Java");

boolean anyMatch = names.stream()
  .anyMatch(name -> name.contains("a"));
boolean allMatch = names.stream()
  .allMatch(name -> name.length() > 3);
boolean noneMatch = names.stream()
  .noneMatch(name -> name.endsWith("s"));

Iterating
foreach 는 요소를 돌면서 실행되는 최종 작업입니다. 보통 System.out.println 메소드를 넘겨서 결과를 출력할 때 사용하곤 합니다. 앞서 살펴본 peek 과는 중간 작업과 최종 작업의 차이가 있습니다.
names.stream().forEach(System.out::println);

Java 스트림 Stream (2) 고급
이전 포스트에 이어서 Java 8의 스트림(Stream)을 살펴봅니다. 자바 8 스트림은 총 두 개의 포스트로, 기본적인 내용을 총정리하는 이전 포스트와 좀 더 고급 내용을 다루는 이번 포스트로 나뉘어져 있습니다.
    · Java 스트림 Stream (1) 총정리
    · Java 스트림 Stream (2) 고급

살펴볼 내용

이번 포스트에서 다루는 내용은 다음과 같습니다. 이번 내용이 어렵다면 이전 포스트를 참고하시는 것도 좋습니다.
    · 동작 순서
    · 성능 향상
    · 스트림 재사용
    · 지연 처리(Lazy Invocation)
    · Null-safe 스트림 생성하기
    · 줄여쓰기(Simplified)

동작 순서

다음 스트림에서는 최종 작업인 findFirst 메소드를 호출합니다. 과연 출력 결과는 어떨까요?
list.stream()
  .filter(el -> {
    System.out.println("filter() was called.");
    return el.contains("a");
  })
  .map(el -> {
    System.out.println("map() was called.");
    return el.toUpperCase();
  })
  .findFirst();

요소는 3개인데 결과는 다음처럼 filter 두 번, map 이 한 번 출력됩니다.
filter() was called.
filter() was called.
map() was called.

여기서 스트림이 동작하는 순서를 알아낼 수 있습니다. 모든 요소가 첫 번째 중간 연산을 수행하고 남은 결과가 다음 연산으로 넘어가는 것이 아니라, 한 요소가 모든 파이프라인을 거쳐서 결과를 만들어내고, 다음 요소로 넘어가는 순입니다.
좀 더 자세히 살펴보면,
    · 처음 요소인 “Eric” 은 “a” 문자열을 가지고 있지 않기 때문에 다음 요소로 넘어갑니다. 이 때 “filter() was called.” 가 한 번 출력됩니다.
    · 다음 요소인 “Elena” 에서 "filter() was called."가 한 번 더 출력됩니다. "Elena"는 "a"를 가지고 있기 때문에 다음 연산으로 넘어갈 수 있습니다.
    · 다음 연산인 map 에서 toUpperCase 메소드가 호출됩니다. 이 때 "map() was called"가 출력됩니다.
    · 마지막 연산인 findFirst 는 첫 번째 요소만을 반환하는 연산입니다. 따라서 최종 결과는 “ELENA” 이고 다음 연산은 수행할 필요가 없어 종료됩니다.

위와 같은 과정을 통해서 수행됩니다.


성능 향상

위에서 살펴봤듯이 스트림은 한 요소씩 수직적으로(vertically) 실행됩니다.
여기에 스트림의 성능을 개선할 수 있는 힌트가 숨어있습니다.
다음 예제를 살펴보시죠.
list.stream()
  .map(el -> {
    wasCalled();
    return el.substring(0, 3);
  })
  .skip(2)
  .collect(Collectors.toList());

System.out.println(counter); // 3

첫 번째 요소 "Eric"은 먼저 문자열을 잘라내고, 다음 skip 메소드 때문에 스킵됩니다. 다음 요소인 "Elena"도 마찬가지로 문자열을 잘라낸 후 스킵됩니다. 마지막 요소인 “Java” 만 문자열을 잘라내어 “Jav” 가 된 후 스킵되지 않고 결과에 포함됩니다. 여기서 map 메소드는 총 3번 호출됩니다.
여기서 메소드 순서를 바꾸면 어떨까요? skip 메소드가 먼저 실행되도록 해봅시다.
List<String> collect = list.stream()
  .skip(2)
  .map(el -> {
    wasCalled();
    return el.substring(0, 3);
  })
  .collect(Collectors.toList());

System.out.println(counter); // 1

그 결과 스킵을 먼저 하기 때문에 map 메소드는 한 번 밖에 호출되지 않습니다. 이렇게 요소의 범위를 줄이는 작업을 먼저 실행하는 것이 불필요한 연산을 막을 수 있어 성능을 향상시킬 수 있습니다. 이런 메소드로는 skip, filter, distinct 등이 있습니다.


스트림 재사용

종료 작업을 하지 않는 한 하나의 인스턴스로서 계속해서 사용이 가능합니다. 하지만 종료 작업을 하는 순간 스트림이 닫히기 때문에 재사용은 할 수 없습니다. 스트림은 저장된 데이터를 꺼내서 처리하는 용도이지 데이터를 저장하려는 목적으로 설계되지 않았기 때문입니다.
Stream<String> stream = 
  Stream.of("Eric", "Elena", "Java")
  .filter(name -> name.contains("a"));
 
Optional<String> firstElement = stream.findFirst();
// IllegalStateException: stream has already been operated upon or closed
Optional<String> anyElement = stream.findAny(); 

위 예제에서 findFirst 메소드를 실행하면서 스트림이 닫히기 때문에 findAny 하는 순간 런타임 예외(runtime exception)이 발생합니다. 컴파일러가 캐치할 수 없기 때문에 Stream 이 닫힌 후에 사용되지 않는지 주의해야 합니다.
위 코드는 아래 코드처럼 바꿀 수 있습니다. 데이터를 List 에 저장하고 필요할 때마다 스트림을 생성해 사용합니다.
List<String> names = 
  Stream.of("Eric", "Elena", "Java")
  .filter(name -> name.contains("a"))
  .collect(Collectors.toList());

Optional<String> firstElement = names.stream().findFirst();
Optional<String> anyElement = names.stream().findAny();


지연 처리 Lazy Invocation

스트림에서 최종 결과는 최종 작업이 이루어질 때 계산됩니다. 호출 횟수를 카운트하는 예제입니다.
private long counter;
private void wasCalled() {
  counter++;
}

다음 예제에서 리스트의 요소가 3개이기 때문에 총 세 번 호출되어 결과가 3이 출력될 것으로 예상됩니다. 하지만 출력값은 0입니다.
List<String> list = Arrays.asList("Eric", "Elena", "Java");
counter = 0;
Stream<String> stream = list.stream()
  .filter(el -> {
    wasCalled();
    return el.contains("a");
  });
System.out.println(counter); // 0 ??

왜냐하면 최종 작업이 실행되지 않아서 실제로 스트림의 연산이 실행되지 않았기 때문입니다. 다음 예제처럼 최종 작업인 collect 메소드를 호출한 결과 3이 출력됩니다.
list.stream().filter(el -> {
  wasCalled();
  return el.contains("a");
}).collect(Collectors.toList());
System.out.println(counter); // 3

Null-safe 스트림 생성하기

NullPointerException 은 개발 시 흔히 발생하는 예외입니다.
Optional 을 이용해서 null에 안전한(Null-safe) 스트림을 생성해보겠습니다.
public <T> Stream<T> collectionToStream(Collection<T> collection) {
    return Optional
      .ofNullable(collection)
      .map(Collection::stream)
      .orElseGet(Stream::empty);
  }

위 코드는 인자로 받은 컬렉션 객체를 이용해 옵셔널 객체를 만들고 스트림을 생성후 리턴하는 메소드입니다. 그리고 만약 컬렉션이 비어있는 경우라면 빈 스트림을 리턴하도록 합니다.
제네릭을 이용해 어떤 타입이든 받을 수 있습니다.
List<Integer> intList = Arrays.asList(1, 2, 3);
List<String> strList = Arrays.asList("a", "b", "c");

Stream<Integer> intStream = 
  collectionToStream(intList); // [1, 2, 3]
Stream<String> strStream = 
  collectionToStream(strList); // [a, b, c]

이제 null 로 테스트를 해보겠습니다. 다음과 같이 리스트에 null 이 있다면 NPE 가 날 수 밖에 없는 상황입니다. 외부에서 인자로 받은 리스트로 작업을 하는 경우에 일어날 수 있는 상황입니다.
List<String> nullList = null;

nullList.stream()
  .filter(str -> str.contains("a"))
  .map(String::length)
  .forEach(System.out::println); // NPE!

하지만 우리가 만든 메소드를 이용하면 NPE 가 발생하는 대신 빈 스트림으로 작업을 마칠 수 있습니다.
collectionToStream(nullList)
  .filter(str -> str.contains("a"))
  .map(String::length)
  .forEach(System.out::println); // []


줄여쓰기 Simplified

스트림 사용 시 다음과 같은 경우에 같은 내용을 좀 더 간결하게 줄여쓸 수 있습니다. IntelliJ 를 사용하면 다음과 같은 경우에 줄여쓸 것을 제안해줍니다. 그 중에서 많이 사용되는 것만 추렸습니다.
collection.stream().forEach()
  → collection.forEach()

collection.stream().toArray()
  → collection.toArray()

Arrays.asList().stream()
  → Arrays.stream() or Stream.of()

Collections.emptyList().stream()
  → Stream.empty()

stream.filter().findFirst().isPresent()
  → stream.anyMatch()

stream.collect(counting())
  → stream.count()

stream.collect(maxBy())
  → stream.max()

stream.collect(mapping())
  → stream.map().collect()

stream.collect(reducing())
  → stream.reduce()

stream.collect(summingInt())
  → stream.mapToInt().sum()

stream.map(x -> {...; return x;})
  → stream.peek(x -> ...)

!stream.anyMatch()
  → stream.noneMatch()

!stream.anyMatch(x -> !(...))
  → stream.allMatch()

stream.map().anyMatch(Boolean::booleanValue)
  → stream.anyMatch()

IntStream.range(expr1, expr2).mapToObj(x -> array[x])
  → Arrays.stream(array, expr1, expr2)

Collection.nCopies(count, ...)
  → Stream.generate().limit(count)

stream.sorted(comparator).findFirst()
  → Stream.min(comparator)

하지만 주의점이 있습니다. 특정 케이스에서 조금 다르게 동작할 수 있습니다.
예를 들면 다음의 경우 stream 을 생략할 수 있지만,
collection.stream().forEach()
  → collection.forEach()

다음 경우에서는 동기화(synchronized)는 차이가 있습니다.
// not synchronized
Collections.synchronizedList(...).stream().forEach()
  
// synchronized
Collections.synchronizedList(...).forEach()

다른 예제는 다음과 같이 collect 를 생략하고 바로 max 메소드를 호출하는 경우입니다.
stream.collect(maxBy())
  → stream.max()

하지만 스트림이 비어서 값을 계산할 수 없을 때의 동작은 다릅니다. 전자는 Optional 객체를 리턴하지만, 후자는 NullPointerExcpetion 이 발생할 가능성이 있습니다.
collect(Collectors.maxBy()) // Optional
Stream.max() // NPE 발생 가능
