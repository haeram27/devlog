# jackson JsonNode를 java object로 mapping 하기

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ArrayNode;
import com.fasterxml.jackson.databind.node.ObjectNode;

ObjectMapper mapper = JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
            .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
            .build();

JsonNode jsonNode = ...

if (node.isObject()) {
    ObjectNode node = (ObjectNode) jsonNode;
    Map<String, Object> map = mapper.convertValue(node,
        new com.fasterxml.jackson.core.type.TypeReference<Map<String,Object>>() {});
    // var map = restClientObjectMapper.treeToValue(node, Map.class);
    list = new ArrayList<Map<String, Object>>();
    list.add(map);
} else if (node.isArray()) {
    ArrayNode node = (ArrayNode) jsonNode;
    list = mapper.convertValue(node,
        new com.fasterxml.jackson.core.type.TypeReference<List<Map<String,Object>>>() {});
}
```
