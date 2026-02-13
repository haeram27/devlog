# Jackson으로 json 수정하기

프로그램으로 json 내의 임의의 값을 변경하는 기본 컨셉은 다음의 세 단계를 진행하는 것이다.

```text
역직렬화(읽기) > 값 수정 > 직렬화(쓰기)
```

1. json srting을 java object로 역직렬화(deserialize)
1. java object의 값을 변경
1. java object를 json string으로 직렬화(serialize)

## 읽기(역직렬화)

- readValue

## 쓰기(직렬화)

- writeValue

## 주의: JsonMapper.readValue() 호출시 `Class<T>` vs `TypeReference<T>` 사용 차이

- Generic 파라미터를 가지는 ParamterizedType에 매핑하는 경우 TypeReference를 사용하라
  - TypeReference를 사용하면 jackson은 런타임에도 역직렬화 중 databind시 ParameterizedType이 가져야하는 제너릭 타입에 대한 type check를 해준다.
  - 그래서 런타임에 json 데이터가 개발자가 의도한 타입이 아니라면 Exception을 발생시킨다.

## 단순 구분 조건 (추천)

T가 ParameterizedType이면 `TypeReference<T>` 사용
T가 Non-ParameterizedType이면 `Class<T>` 사용

## 상세 구분 조건

## `TypeReference<T>`를 사용해야 하는 경우: (다음 모든 조건 만족)

1. T가 ParameterizedType (Generic 파라미터를 사용하는 타입 - List, Map, 사용자 정의 ParameterizedType)
1. Generic 파라미터가 Object 타입이 아님
1. databinding 시 강한 type check 원함(실제 값이 Generic 타입과 불일치할 경우 Exception 발생)

## `Class<T>`를 사용하는 경우: TypeReference를 사용하지 않는 거의 모든 경우

T가 Non-ParameterizedType(Generic 파라미터를 사용하지 않는 타입 - 사전 정의된 value object나 DTO 등) 인 경우

## `TypeReference<T>` 나 `Class<T>` 모두 사용 가능한 경우

T가 ParameterizedType 일지라도 Generic 타입을 Object로 처리해야 하는 경우(데이터 바인딩시 타입 체크를 하지 않아야 하는 경우)

예) List<Object>: json array이고 값이 단일 타입인 경우

json data: ["hello", 1, 2.4, true, "message"]

```java
JsonMapper.readValue(jsonString, List.class) ==
JsonMapper.readValue(jsonString, new TypeReference<List<Object>>(){})
```

예) Map<String, Object>: json object 이고 값이 단일 타입이 아닌 경우 
json data:

```json
{
  "id": 1111,
  "name": "John Doe",
  "isEmployee": true
}
```

```java
JsonMapper.readValue(jsonString, Map.class) ==
JsonMapper.readValue(jsonString, new TypeReference<Map<String, Object>>(){})
```

## Non-ParameterizedType (제너릭 미사용 타입)의 readValue() 호출

```java
UserVO userVO = mapper.readValue(jsonString, userVO.class);
UserVO userVO = mapper.readValue(jsonString, Map.class); // Generic 타입을 Object로 처리
UserVO userVO = mapper.readValue(jsonString, List.class); // Generic 타입을 Object로 처리
```

## ParameterizedType (제너릭 사용 타입)의 readValue() 호출

### Map (json Object에 대응)

```java
Map<String, Object> map = jsonMapper.readValue(jsonString,
    new TypeReference<Map<String, Object>>(){});
Map<String, Integer> map = jsonMapper.readValue(jsonString,
    new TypeReference<Map<String, UserVO>>(){});
```

### List (json array에 대응)

```java
List<Map<String, Object>> list = jsonMapper.readValue(jsonArrayString,
    new TypeReference<List<Map<String, Object>>>(){});
List<String> list = jsonMapper.readValue(jsonArrayString,
    new TypeReference<List<Map<String, UserVO>>>(){});
```

- jackson의 JsonMapper 클래스에는 여러 버전의 readValue() Overload가 정의 되어 있다.
- 일반적으로 Json String 역질렬화시 아래의 두 가지 readValue()가 사용된다.

```java
public <T> T readValue(String content, Class<T> valueType)
public <T> T readValue(String content, TypeReference<T> valueTypeRef)
```

`Class<T>`와 `TypeReference<T>` 파라미터 사용의 차이는 T가 ParameterizedType 인가 아닌가의 차이 이다.

Non-ParameterizedType은 Generic 파라미터를 사용하지 않는 일반 클래스를 말하고
ParameterizedType은 Collection(List, Set), Map 등 Generic 파라미터를 사용하는 복합 클래스를 말한다.

List(json array 대응)나 Map(json pair 대응)의 경우에도 그냥 List.class, Map.class를 사용해도 오류는 없다.
다만 컴파일 후에도 제너릭 타입의 정보를 컴파일된 코드에 남겨서 역직렬화시 "개발자가 의도한 제너릭 타입 정보"를 갖는ParameterizedType(List, Map 등)으로 정확히 databinding(mapping) 되도록 하기 위해서 사용된다.
TypeReference를 사용하면 jackson은 런타임에도 역직렬화 중 databind시 ParameterizedType이 가져야하는 제너릭 타입에 대한 type check를 해준다. 그래서 런타임에 json 데이터가 개발자가 의도한 타입이 아니라면 Exception을 발생시킨다.

제너릭 표현은 컴파일 과정에서 Type Erasure(타입 소거)된다.
Type Erasure는 컴파일러가 제너릭 파라미터를 실제 클래스로 변환하는 것을 의미하며 그래서 제너릭 정보는 컴파일 후에 손실된다. 그래서 런타임에는 ParameterizedType을 변수에 받으려면 Type Casting(형변환)을 해야만 한다.

## Type Erasure 전/후 비교

## bound(super, extends)가 없는 제네릭 (T -> Object)

타입 소거 전

```java
public class Box<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

타입 소거 후

```java
public class Box {
    private Object value;

    public void set(Object value) {
        this.value = value;
    }

    public Object get() {
        return value;
    }
}
```

## bound(super, extends)가 있는 제너릭 (T -> Number)

타입 소거 전

```java
public class Box<T extends Number> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

타입 소거 후

```java
public class Box {
    private Number value;

    public void set(Number value) {
        this.value = value;
    }

    public Number get() {
        return value;
    }
}
```

## JsonMapper를 이용한 직렬화/역직렬화 사용법

### 역직렬화(deserialize) : Json(String) -> Java Object

- JsonMapper.readValue()

```java
JsonMapper mapper = new JsonMapper();

// JSON 파일에서 읽기
UserVO userVO = mapper.readValue(new File("data.json"), userVO.class);

//  URL 에서 읽기
UserVO userVO = mapper.readValue(new URL("요청 URL"), userVO.class);

// String 으로 읽기
UserVO userVO = mapper.readValue("{\"id\":\"abc\", \"pw\":1234}", userVO.class);
```

### 역직렬화(deserialize) : Json(String) -> JsonNode(Object)

- JsonMapper.readTree()

```java 
JsonMapper mapper = new JsonMapper();

// JSON String 에서 읽기
JsonNode rootNode = mapper.readTree(jsonString);
```

### 직렬화(serialize, write): Java Object -> Json(String)

- JsonMapper.writeValue(Model)
- JsonMapper.writeValueAsString(Model)

```java
JsonMapper mapper = JsonMapper.builder()
    .addModule(new JavaTimeModule())
    .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
    .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
    .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
    .build();

// json 파일로 저장
mapper.writeValue(new File("result.json"), userVO);

// byte[] 로 저장
byte[] jsonBytes = mapper.writeValueAsBytes(userVO);

// string 으로 저장
String jsonString = mapper.writeValueAsString(userVO);

// 포맷팅하여 스트링으로 변환
String jsonString = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(userVO);
  
// 포맷팅하여 파일로 저장
mapper.writerWithDefaultPrettyPrinter().writeValue(new File("result.json"), userVO);
```

## json array 를 java.util.List 으로 읽기

```java
    @Test
    public void parseJsonArrayToObjectList() {

        // @formatter:off
        /*
        [
            {
                "name": "John Doe",
                "age": 30,
                "email": "john.doe@example.com"
            },
            {
                "name": "Jane Smith",
                "age": 25,
                "email": "jane.smith@example.com"
            }
        ]
        */
        // @formatter:on

        String jsonArrayString = "[{\"name\":\"John Doe\",\"age\":30,\"email\":\"john.doe@example.com\"},{\"name\":\"Jane Smith\",\"age\":25,\"email\":\"jane.smith@example.com\"}]";

        JsonMapper jsonMapper = JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
            .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .build();

        try {
            // JSON 배열 문자열을 List로 변환
            List<Map<String, Object>> list = jsonMapper.readValue(jsonArrayString,
                    new TypeReference<List<Map<String, Object>>>(){});

            // List에서 데이터 접근
            for (Map<String, Object> map : list) {
                String name = (String) map.get("name");
                int age = (Integer) map.get("age");
                String email = (String) map.get("email");

                // 출력
                System.out.println("Name: " + name);
                System.out.println("Age: " + age);
                System.out.println("Email: " + email);
                System.out.println();
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

## json object를 java.util.Map 으로 읽기

```java
    @Test
    public void parseJsonObjectToMap() {

        // @formatter:off
        /*
        {
            "name": "John Doe",
            "age": 30,
            "email": "john.doe@example.com",
            "roles": [
                "admin",
                "user"
            ]
        }
        */
        // @formatter:on

        String jsonString = "{\"name\":\"John Doe\",\"age\":30,\"email\":\"john.doe@example.com\",\"roles\":[\"admin\",\"user\"]}";

        JsonMapper jsonMapper = JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
            .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .build();

        try {
            // JSON 문자열을 Map으로 변환
            Map<String, Object> map = jsonMapper.readValue(jsonString, new TypeReference<Map<String, Object>>(){});

            // Map에서 데이터 접근
            String name = (String) map.get("name");
            int age = (Integer) map.get("age");
            String email = (String) map.get("email");
            List<String> roles = (List<String>) map.get("roles");

            // 출력
            System.out.println("Name: " + name);
            System.out.println("Age: " + age);
            System.out.println("Email: " + email);
            System.out.println("Roles: " + roles);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

## json pointer expression을 사용하여 json 내부 값 변경 하기

- json의 특정 노드에 접근하기위한 표현식으로 일반적으로 json path와 json point expression이 있는데, Jackson의 경우 json point expression을 사용한다.

```java
    @Test
    public void upateValueUsingJsonNode() {
        // @formatter:off
        /*
        {
            "store": {
                "book": [
                    {
                        "category": "reference",
                        "author": "Nigel Rees",
                        "title": "Sayings of the Century",
                        "price": 8.95
                    },
                    {
                        "category": "fiction",
                        "author": "Evelyn Waugh",
                        "title": "Sword of Honour",
                        "price": 12.99
                    }
                ],
                "bicycle": {
                    "color": "red",
                    "price": 19.95
                }
            }
        }
        */
        // @formatter:on
        String jsonString = "{"
                + "\"store\": {"
                + "\"book\": ["
                + "{ \"category\": \"reference\", \"author\": \"Nigel Rees\", \"title\": \"Sayings of the Century\", \"price\": 8.95 },"
                + "{ \"category\": \"fiction\", \"author\": \"Evelyn Waugh\", \"title\": \"Sword of Honour\", \"price\": 12.99 }"
                + "],"
                + "\"bicycle\": { \"color\": \"red\", \"price\": 19.95 }"
                + "}"
                + "}";

        JsonMapper jsonMapper = JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
            .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .build();

        try {
            JsonNode rootNode = jsonMapper.readTree(jsonString);

            /*
             - read first category of book
             */
            String firstBookCategory;

            // use json point expression
            firstBookCategory = rootNode.at("/store/book/0/category").asText();
            System.out.println("First book category: " + firstBookCategory); // reference
            // use field(key) name
            firstBookCategory = rootNode.path("store").path("book").get(1).path("category").asText();
            System.out.println("First book category: " + firstBookCategory); // fiction
            System.out.println("----------------------------------------------------------");

            /*
             - read bicycle price
             */
            double bicyclePrice;

            // use json point expression
            bicyclePrice = rootNode.at("/store/bicycle/price").asDouble();
            System.out.println("Bicycle price: " + bicyclePrice); // 19.95

            // use field name (key)
            bicyclePrice = rootNode.path("store").path("bicycle").path("price").asDouble();
            System.out.println("Bicycle price: " + bicyclePrice); // 19.95
            System.out.println("----------------------------------------------------------");

            /*
             - update value using json point expression
             */
            String modifiedJsonString;
            ((ObjectNode) rootNode.at("/store/book/0")).put("price", 10.99);
            ((ObjectNode) rootNode.at("/store/bicycle")).put("color", "blue");
            modifiedJsonString = jsonMapper.writerWithDefaultPrettyPrinter().writeValueAsString(rootNode);
            System.out.println("Modified JSON: " + modifiedJsonString);
            System.out.println("----------------------------------------------------------");

            JsonNode bicycleNode = rootNode.at("/store/bicycle");
            if (bicycleNode.isObject()) {
                ((ObjectNode) bicycleNode).put("color", "green");
            }
            modifiedJsonString = jsonMapper.writerWithDefaultPrettyPrinter().writeValueAsString(rootNode);
            System.out.println("Modified JSON: " + modifiedJsonString);
            System.out.println("----------------------------------------------------------");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
