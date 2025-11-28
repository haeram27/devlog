# Mybatis Type Handler

## json type handler

mapper.xml

```xml
    <resultMap id="myResultMap" type="com.example.MyResultType">
        <result property="id" column="id" />
        <result property="name" column="name" />
        <result property="jsonData" column="json_data" typeHandler="com.example.typehandler.JsonTypeHandler" />
    </resultMap>
```

JsonTypeHandler.java

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonMapper;
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.postgresql.util.PGobject;

import java.io.IOException;
import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class JsonTypeHandler<T> extends BaseTypeHandler<T> {
    private final JsonMapper mapper = JsonMapper.builder()
                    .addModule(new JavaTimeModule())
                    .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
                    .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
                    .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                    .build();

    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType)
            throws SQLException {
        String stringValue;
        try {
            stringValue = mapper.writeValueAsString(parameter);
        } catch (JsonProcessingException e) {
            throw new IllegalArgumentException(e);
        }
        PGobject object = new PGobject();
        object.setValue(stringValue);
        object.setType("json");
        ps.setObject(i, object);
    }

    @Override
    public T getNullableResult(ResultSet resultSet, String columnName) throws SQLException {
        String s = resultSet.getString(columnName);
        return s == null ? null : readValue(s);
    }

    @Override
    public T getNullableResult(ResultSet resultSet, int columnIndex) throws SQLException {
        String s = resultSet.getString(columnIndex);
        return s == null ? null : readValue(s);
    }

    @Override
    public T getNullableResult(CallableStatement callableStatement, int columnIndex)
            throws SQLException {
        String s = callableStatement.getString(columnIndex);
        return s == null ? null : readValue(s);
    }

    private T readValue(String jsonString) {
        try {
            return mapper.readValue(jsonString, getTypeReference());
        } catch (IOException e) {
            throw new IllegalArgumentException(e);
        }
    }

    protected TypeReference<T> getTypeReference() {
        return new TypeReference<T>() {};
    }
}
```
