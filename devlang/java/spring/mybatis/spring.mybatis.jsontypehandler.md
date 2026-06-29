# Mybatis Type Handler for jsonb

이 문서는 mybatis + spring jackson 환경에서 사용자정의 Class 또는 Generic Type을 RDB에 `json/jsonb` 형식으로 읽고(deserialize) 쓰기(serialize) 위한 mybatis용 `typehandler`를 설명한다.

## Postgresql `json/jsonb` type

`json/jsonb` type은 사실상 `json` 문서에 대응 하는 타입이다.
Postgresql에서 어떤 데이터의 타입이 `json/jsonb`라고 하면 단지 TEXT인데 `json`문서 형식을 담은 타입이다 라는 의미이다.
`json` 문서는 Java 등의 프로그래밍 언어에서는 String으로 다룰수 있고, Postgresql도 기본적으로는 사실상 TEXT type처럼 기록하고 다루게 된다. 그래서 Postgresql의 `json/jsonb` 데이터는 JDBC에서 `String`이나 `List<String>` 으로 맵핑해도 아무런 문제가 발생하지 않는다.

하지만 Postgresql은 `json` 문서 데이터를 다루기 위한 좀 더우 유연한 방식을 지원하기 위하여 `jsonb` 타입을 만들었고 `jsonb` 타입에만 사용할 수 있는 다양한 json 핸들링 용 함수를 제공한다.

정리하면, Postgresql 입장에서 `json` 문서는 TEXT와 마찬가지다, 하지만 `json` 문서를 좀더 효율적으로 탐색하고 변경하기 위한 목적으로 `jsonb` 타입을 만들었으며, Postgresql에 `jsonb` 타입으로 `json` 문서가 저장되면 json 데이터의 node 검색을 위한 인덱스를 자동 생성하며, `json` 문서를 다루기 위한 다양한 함수를 사용할 수 있게 된다.

Postgresql의 `json` 타입은 `jsonb` 타입 보다 이전의 방식이며, Postgresql 내에서 지원 되는 기능이 많이 부족하다. 거의 '`json` 문서를 저장한 TEXT 타입이다'의 방식으로 다루어 지게 된다.

## mybatis TypeHandler 

- mybatis의 `TypeHander`는 Java의 사용자 정의 DataType(Class 또는 Generic Type)을 DB의 Object(data item) 타입과 상호 변환하는데 사용되는 도구이다.
- Java와 DB 간에 데이터의 타입이 유사하지 않을 때 이 둘을 상호 변환하기 위해서 사용한다.

mybatis의 `TypeHander`가 필요한 경우

- Java의 데이터와 DB의 데이터를 기본적인 맵핑 방식으로 데이터를 전환하지 못할때(type miss match 발생 등)
- Java의 사용자 정의 Class 또는 Generic Type을 DB의 json/jsonb type으로 저장

### mybatis TypeHandler 주의 사항

- mybatis는 typehandler를 이용해 Java의 data를 RDB의 data로 serialize/deserialize 한다.
- typehandler는 mybatis용 `mapper.xml`에서 보통 참조되는 class이다.
- mybatis는 `mapper.xml`을 보고 현재 RDB 데이터가 Java의 어떤 데이터로 변환이 될지 타입 추론(Type Inference)이 가능하다.
- mybatis의 타입 추론은 Java의 Type이 non-generic 단일 Class 인 경우에만 정상 동작한다.
- Java의 Type이 GenericType(`List<MyDto>` 등)인 경우에 mybatis는 타입 추론이 불가능하므로(`List`까지만 이해 가능하며 그 내부의 타입은 추론이 불가) 반드시 `TypeHandler`를 통해서 Java의 Type을 개발자가 개입하여 변환해 주어야 한다.

## json type handler 구현

- 이 `JsonTypeHandler` 구현은 java의 `Class<T>`, `TypeReference<T>`(jackson), `JavaType`(jackson) 타입을 RDB의 `json/jsonb` type과 맵핑(변환)을 하는데 사용하는 mybatis용 `BaseTypeHandler` 구현체이다.
- `JsonTypeHandler`이 하는 역할은 `Java Obejct(app) - String(Jackson) - PGObject(JDBC/postgresql)` 단계로 상호간 데이터를 맵핑하는 것이다.
- ⚠️ 기본적으로 RDB의 `json/jsonb` 타입은 Java의 `String` 타입으로 자동 맵핑(JDBC가 `String`으로 변환)된다.
- RDB의 type은 `getPgType()` 메서드로 지정하는데, 리턴 값을 변경하면 해당 Type에 맞는 TypeHandler로 변환하여 사용 가능하다.

```java
package com.example.typehandler;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.postgresql.util.PGobject;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class JsonTypeHandler<T> extends BaseTypeHandler<T> {

    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

    private final JavaType explicitJavaType;
    private final Class<T> explicitClassType;

    /** default constructor, when MyBatis uses default constructor */
    public JsonTypeHandler() {
        this.explicitJavaType = null;
        this.explicitClassType = null;
    }

    /** Class<T> Constructor
        for (non generic) class type, ex: MyDto.class
        JsonTypeHandler(MyDto.class)
    */
    public JsonTypeHandler(Class<T> classType) {
        if (classType == null) {
            throw new IllegalArgumentException("classType cannot be null");
        }
        this.explicitClassType = classType;
        this.explicitJavaType = null;
    }

    /** TypeReference Constructor
        for GenericType, ex: List<MyDto> 
        JsonTypeHandler(new TypeReference<List<MyDto>>() {})
    */
    public JsonTypeHandler(TypeReference<T> typeReference) {
        if (typeReference == null) {
            throw new IllegalArgumentException("typeReference cannot be null");
        }
        this.explicitJavaType = OBJECT_MAPPER.getTypeFactory().constructType(typeReference.getType());
        this.explicitClassType = null;
    }

    /** JavaType Constructor
        for GenericType, ex: List<MyDto> 
        JsonTypeHandler(mapper.getTypeFactory().constructCollectionType(List.class, MyDto.clss))
    */
    public JsonTypeHandler(JavaType javaType) {
        if (javaType == null) {
            throw new IllegalArgumentException("javaType cannot be null");
        }
        this.explicitJavaType = javaType;
        this.explicitClassType = null;
    }

    /** serialize: transform object to jsonb (postgresql)
     * Java Object -> Java String (by jackson) -> jsonb
     */
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
        PGobject object = new PGobject();
        try {
            object.setType(getPgType());
            object.setValue(OBJECT_MAPPER.writeValueAsString(parameter));
        } catch (JsonProcessingException e) {
            throw new IllegalArgumentException("Failed to serialize parameter to JSON", e);
        }
        ps.setObject(i, object);
    }

    /** deserialize: transform jsonb (postgresql) to object
     * Java String(json) -> Java Object (by jackson)
     */
    @Override
    public T getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return toObject(rs.getString(columnName));
    }

    /** deserialize: transform jsonb (postgresql) to object
     * Java String(json) -> Java Object (by jackson)
    */
    @Override
    public T getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return toObject(rs.getString(columnIndex));
    }

    /** deserialize: transform jsonb (postgresql) to object
     * Java String(json) -> Java Object (by jackson)
    */
    @Override
    public T getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return toObject(cs.getString(columnIndex));
    }

    /**
     * can be OVERRIDE
     * default jsonb, if need, ovrride this method as "json" at child class
     */
    protected String getPgType() {
        return "jsonb";
    }

    /**
     * can be OVERRIDE
     * when works with default constuctor and it needs type assign then override this method at child class
     * ex: return new TypeReference<List<Ip4Range>>() {};
     */
    protected TypeReference<T> getTypeReference() {
        return new TypeReference<T>() {};
    }

    private T toObject(String content) {
        if (content == null || content.isEmpty()) {
            return null;
        }
        try {
            return OBJECT_MAPPER.readValue(content, resolveJavaType());
        } catch (Exception e) {
            throw new IllegalArgumentException("Failed to deserialize JSON to target type", e);
        }
    }

    private JavaType resolveJavaType() {
        if (explicitJavaType != null) {
            return explicitJavaType;
        }
        if (explicitClassType != null) {
            return OBJECT_MAPPER.getTypeFactory().constructType(explicitClassType);
        }
        return OBJECT_MAPPER.getTypeFactory().constructType(getTypeReference().getType());
    }
}
```

### postgresql의 `json`과 `jsonb` 타입

- postgresql은 `json` 문자열을 저장하기 위해서 `json`과 `jsonb`라는 타입을 지원한다.
- `json` 타입은 json을 공백등의 불필요한 부분만 최적화하여 문자열 그대로 저장하는 방식이다.
- `jsonb` 타입은 json을 deserialize하여 내부적으로 연산 가능한 tree node 형식으로 저장하는 방식이다.
- `jsonb`가 최신 방식이며 `json` 타입에 비해 postgreql 자체에서 많은 연산 함수들을 지원한다.
- 최근 실무에서는 대부분의 경우 `jsonb`를 사용을 권장하고 있으며, 단순 저장 용도 이외에는 특별히 `json` 타입을 써야할 이유는 없다.

### `TypeReference<T>`와 `JavaType` 차이

- `TypeReference<T>`와 `JavaType` 는 둘 모두 jackson에서 `Generic Type` 정보를 저장하는데 사용 Class이다.
- 개발자가 실제 사용시에도 `TypeReference`나 `JavaType` 모두 동일한 목적(제너릭 타입 정보 보존)으로 사용할 수 있다.
- `TypeReference<T>` 사람이 읽기 쉬운 표현식이 필요할 때 사용
- `JavaType`은 jackson library 내부적으로 사용하는 기능적으로 좀 더 유연한 지원을 하는 type이다.
- jackson은 사용자 코드에서 TypeReference로 type을 전달받으면 내부적으로 이를 JavaType으로 저장해서 처리하는 방식을 사용한다.

- TypeReference가 더 좋은 경우
  - 소스 코드에 타입이 고정돼 있을 때
  - 사람이 읽기 쉬운 선언이 필요할 때
  - 예: `List<Ip4Range>`, `Map<String, Object>`

- JavaType이 더 좋은 경우
  - 타입을 런타임에 조합해야 할 때
  - 프레임워크/공통 유틸을 만들 때
  - 중첩 제네릭을 동적으로 만들 때

## 사용예제

- RDB의 `json/jsonb` 타입을 Java의 `String` 또는 `List<String>`으로 변환하는 경우에는 별도의 type 명시를 위한 wrapper 클래스를 작성하지 않고 `JsonTypeHandler`를 `mapper.xml`에서 지접 지정이 가능하다.

### mybatis

#### Class type

- 목적 `type` 전용 `JsonTypeHandler` wrapper class 선언
  - 명시적(explicit) 타입 정보 전달 (생성자 사용), recommended

    ```java
    package com.example.typehandler;

    import java.util.List;

    import com.example.model.MyDto;
    import com.fasterxml.jackson.core.type.TypeReference;

    public class MyDtoJsonTypeHandler extends JsonTypeHandler<MyDto> {

        public MyDtoJsonTypeHandler() {
            super(MyDto.class);
        }
    }
    ```

  - 암시적 타입 정보 전달 (child class 생성자 미사용하고 JsonTypeHandler 기본 생성자 사용시), method OVERRIDE 방식

    ```java
    package com.example.typehandler;

    import java.util.List;

    import com.example.model.MyDto;
    import com.fasterxml.jackson.core.type.TypeReference;

    public class MyDtoJsonTypeHandler extends JsonTypeHandler<MyDto> {
         @Override
        protected TypeReference<MyDto> getTypeReference() {
            return new TypeReference<MyDto>() {};
        }

        @Override
        protected String getPgType() {
            return "jsonb";
        }
    }
    ```

- mapper.xml의 resultSet type binding

```xml
<resultMap id="MySuperDto"
    type="com.example.model.MySuperDto">
    <result column="mydto" property="myDto"
        jdbcType="OTHER"
        typeHandler="com.example.typehandler.MyDtoJsonTypeHandler"/>
</resultMap>

<select id="selectMySuperDto" resultMap="MySuperDto">
    SELECT
        mydto
    FROM
        tb_table
    ORDER BY
        id DESC
</select>
```

- mapper.xml의 query parameter binding

```xml
<insert id="updateMySuperDto"
    parameterType="com.example.model.MySuperDto"
    useGeneratedKeys="true">
    INSERT INTO tb_table
    (
        mydto
    )

    VALUES (
        <!-- assign typehandler here -->
        #{mydto,
            jdbcType=OTHER,
            typeHandler=com.example.typehandler.MyDtoJsonTypeHandler}
    )
</insert>

<update id="updateMySuperDtoById"
    parameterType="com.example.model.MySuperDto">
    UPDATE tb_table
    SET
        <!-- assign typehandler here -->
        mydto=#{myDto,
            jdbcType=OTHER,
            typeHandler=com.example.typehandler.MyDtoJsonTypeHandler}
    WHERE
        idx=#{id}
</update>
```

#### TypeReference (Generic Type)

- 목적 `type` 전용 `JsonTypeHandler` wrapper class 선언
  - 명시적(explicit) 타입 정보 전달 (생성자 사용), recommended

    ```java
    package com.example.typehandler;

    import java.util.List;

    import com.example.model.MyDto;
    import com.fasterxml.jackson.core.type.TypeReference;

    public class MyDtoListJsonTypeHandler extends JsonTypeHandler<List<MyDto>> {

        public MyDtoListJsonTypeHandler() {
            super(new TypeReference<List<MyDto>>() {});
        }
    }
    ```

  - 암시적 타입 정보 전달 (chile class 생성자 미사용하고 JsonTypeHandler 기본 생성자 사용시), method OVERRIDE 방식

    ```java
    package com.example.typehandler;

    import java.util.List;

    import com.example.model.MyDto;
    import com.fasterxml.jackson.core.type.TypeReference;

    public class MyDtoListJsonTypeHandler extends JsonTypeHandler<List<MyDto>> {
         @Override
        protected TypeReference<List<MyDto>> getTypeReference() {
            return new TypeReference<List<MyDto>>() {};
        }

        @Override
        protected String getPgType() {
            return "jsonb";
        }
    }
    ```

- mapper.xml의 resultSet type binding

```xml
<resultMap id="MySuperDto"
    type="com.example.model.MySuperDto">
    <result column="mydto_list" property="myDtoList"
        jdbcType="OTHER"
        typeHandler="com.example.typehandler.MyDtoListJsonTypeHandler"/>
</resultMap>

<select id="selectMySuperDto" resultMap="MySuperDto">
    SELECT
        mydto_list
    FROM
        tb_table
    ORDER BY
        id DESC
</select>
```

- mapper.xml의 query parameter binding

```xml
<insert id="updateMySuperDto"
    parameterType="com.example.model.MySuperDto"
    useGeneratedKeys="true">
    INSERT INTO tb_table
    (
        mydto_list
    )

    VALUES (
        <!-- assign typehandler here -->
        #{myDtoList,
            jdbcType=OTHER,
            typeHandler=com.example.typehandler.MyDtoListJsonTypeHandler}
    )
</insert>

<update id="updateMySuperDtoById"
    parameterType="com.example.model.MySuperDto">
    UPDATE tb_table
    SET
        <!-- assign typehandler here -->
        mydto_list=#{myDtoList,
            jdbcType=OTHER,
            typeHandler=com.example.typehandler.MyDtoListJsonTypeHandler}
    WHERE
        idx=#{id}
</update>
```