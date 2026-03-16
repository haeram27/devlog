# Mybatis Type Handler

mybatis + spring jackson 환경에서 RDB(postgresql)와 사용자정의 Java Object의 type mapping을 위한 typehandler 정의 및 사용법 

- RDB Table내 데이터의 type이 jsonb(json binary)일 때, mybatis가 java \<-\> RDB 간 해당 데이터의 상호 변환을 위해서 `Json` 변환용 typehandler를 구현해 사용해야 한다.
- 다시말해, mybatis의 `TypeHander`는 특정 data 아이템을 변환할 때 사용하는 변환 도구이다.

## json type handler 구현

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

    /** serialize: transform object to jsonb (postgresql) */
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

    /** deserialize: transform jsonb (postgresql) to object */
    @Override
    public T getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return toObject(rs.getString(columnName));
    }

    /** deserialize: transform jsonb (postgresql) to object */
    @Override
    public T getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return toObject(rs.getString(columnIndex));
    }

    /** deserialize: transform jsonb (postgresql) to object */
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
- `JavaType`은 jackson library 내부적으로 사용하는 기능적으로 좀 더 유연한 지원을하는 type이다.
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

  - 암시적 타입 정보 전달 (chile class 생성자 미사용하고 JsonTypeHandler 기본 생성자 사용시), method OVERRIDE 방식

    ```java
    package com.example.typehandler;

    import java.util.List;

    import com.example.model.MyDto;
    import com.fasterxml.jackson.core.type.TypeReference;

    public class MyDtoJsonTypeHandler extends JsonTypeHandler<MyDto> {
         @Override
        protected TypeReference<List<Ip4Range>> getTypeReference() {
            return new TypeReference<List<Ip4Range>>() {};
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

