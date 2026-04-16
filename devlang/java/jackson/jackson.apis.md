# Jackson api

## Api Docs

- [Jackson Api Docs](https://javadoc.io/doc/tools.jackson.core/jackson-databind/latest/tools.jackson.databind/module-summary.html)
- [ObjectMapper API Docs](https://javadoc.io/doc/tools.jackson.core/jackson-databind/latest/tools.jackson.databind/tools/jackson/databind/ObjectMapper.html)
- [JsonNode API Docs](https://javadoc.io/doc/tools.jackson.core/jackson-databind/latest/overview-tree.html)

## ObjectMapper

- Parameter Terms:
  - JSONBIN = String, byte[], Input/Output Stream
  - TREE = ObjectNode, JsonNode, YamlNode
  - VALUE = JavaType, Class, Generic Type and includes JsonNode or String

### JsonMapper

`JsonMapper` and `YamlMapper` is extends `ObjectMapper` 

How to build JsonMapper

```java
JsonMapper.builder()
    .addModule(new JavaTimeModule())
    .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
    .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
    .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES) // prevent UnknownPropertyException
    .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
    .build();
```

### Serialize

#### writeTree(): TREE to JSONBIN

```text
void    writeTree(JsonGenerator jgen, JsonNode rootNode)
void    writeTree(JsonGenerator jgen, TreeNode rootNode)
```

#### writeValue():  VALUE to JSONBIN

```text
void    writeValue(File resultFile, Object value)
void    writeValue(JsonGenerator jgen, Object value)
void    writeValue(OutputStream out, Object value)
void    writeValue(Writer w, Object value)
byte[]    writeValueAsBytes(Object value)
String    writeValueAsString(Object value)
```

### Deserialize

#### readTree(): JSONBIN to TREE

```text
JsonNode    readTree(byte[] content)
JsonNode    readTree(File file)
JsonNode    readTree(InputStream in)
JsonNode    readTree(Reader r)
JsonNode    readTree(String content)
JsonNode    readTree(URL source)
```

#### readValue(): JSONBIN to VALUE

```text
<T> T    readValue(byte[] src, Class<T> valueType)
<T> T    readValue(byte[] src, int offset, int len, Class<T> valueType)
<T> T    readValue(byte[] src, int offset, int len, JavaType valueType)
<T> T    readValue(byte[] src, int offset, int len, TypeReference valueTypeRef)
<T> T    readValue(byte[] src, JavaType valueType)
<T> T    readValue(byte[] src, TypeReference valueTypeRef)
<T> T    readValue(File src, Class<T> valueType)
<T> T    readValue(File src, JavaType valueType)
<T> T    readValue(File src, TypeReference valueTypeRef)
<T> T    readValue(InputStream src, Class<T> valueType)
<T> T    readValue(InputStream src, JavaType valueType)
<T> T    readValue(InputStream src, TypeReference valueTypeRef)
<T> T    readValue(JsonParser jp, Class<T> valueType)
<T> T    readValue(JsonParser jp, JavaType valueType)
<T> T    readValue(JsonParser jp, ResolvedType valueType)
<T> T    readValue(JsonParser jp, TypeReference<?> valueTypeRef)
<T> T    readValue(Reader src, Class<T> valueType)
<T> T    readValue(Reader src, JavaType valueType)
<T> T    readValue(Reader src, TypeReference valueTypeRef)
<T> T    readValue(String content, Class<T> valueType)
<T> T    readValue(String content, JavaType valueType)
<T> T    readValue(String content, TypeReference valueTypeRef)
<T> T    readValue(URL src, Class<T> valueType)
<T> T    readValue(URL src, JavaType valueType)
<T> T    readValue(URL src, TypeReference valueTypeRef)
<T> MappingIterator<T>    readValues(JsonParser jp, Class<T> valueType)
<T> MappingIterator<T>    readValues(JsonParser jp, JavaType valueType)
<T> MappingIterator<T>    readValues(JsonParser jp, ResolvedType valueType)
<T> MappingIterator<T>    readValues(JsonParser jp, TypeReference<?> valueTypeRef)
```

#### convertValue(): JsonObject(JsonNode, ObjectNode etc) to Type(User Defined Class, Generic Type)

```text
<T> T    convertValue(Object fromValue, Class<T> toValueType)
<T> T    convertValue(Object fromValue, JavaType toValueType)
<T> T    convertValue(Object fromValue, TypeReference<?> toValueTypeRef)
```

#### exchange VALUE with TREE (IMPORTANT)

```text
<T extends JsonNode> T valueToTree(Object fromValue)
<T> T treeToValue(TreeNode n, Class<T> valueType)
```

#### How to map json string to java object

To map json string to object or JsonNode

- jsonString is only allowed empty("") or json string

```java
String jsonString = """
    {
        "key1": "value1",
        "key2": "value2",
    }
    """;

T obj = JsonMapper.readValue(jsonString, new TypeReference<T>(){})
JsonNode = JsonMapper.readTree(jsonString)
```

## JsonNode

### find Node

```text
# Shallow Search
JsonNode        JsonNode.path(String fieldName)

# Deep Search
JsonNode        JsonNode.at(String jsonPtrExpr)
JsonNode        JsonNode.findPath(String fieldName)
List<JsonNode>  JsonNode.findValues(String fieldName)
JsonNode        JsonNode.findParent(String fieldName)
List<JsonNode>  JsonNode.findValues(String propertyName, List<JsonNode> foundSoFar)
List<JsonNode>  JsonNode.findParents(String fieldName)
```

#### Shallow Search (find direct child node)

| method | not existing key | purpose |
|---|---|---|
| get(String fieldName) | null | search direct child node |
| path(String fieldName) | MissingNode | search direct child node |

#### Deep Search

⚠️ Strongly recommend to use methods which returns `MissingNode` than null. 

| method | not existing key | purpose |
|---|---|---|
| at(String jsonPtrExpr) | MissingNode | access using JsonPointer(/a/b/c) |
| findValue(String fieldName) | null | returns first matched child node |
| findPath(String fieldName) | MissingNode | returns first matched child node |
| findValues(String fieldName) | empty List | returns all matched child node |
| findValues(String fieldName, List<JsonNode> foundSoFar) | empty List | returns all child node from subtree except nodes in foundSoFar list |
| findParent(String fieldName) | null | returns first matched parent node |
| findParents(String fieldName) | null | returns all matched parent node |

#### Examples

```java
JsonNode node = objectMapper.readTree(jsonString);

JsonNode city = node.get("address");
if (!Objects.isNull(city) && StringUtils.hasText(city.asString("");)) {
    // do something with city
}

String name = node.path("address").path("owner").asText();
if (StringUtils.hasText(name)) {
    // do something with name
}

String name = node.findPath("owner").asText();
if (StringUtils.hasText(name)) {
    // do something with name
}

Integer number = node.at("/address/number").asInt();
if (number != 0) {
    // do something with number
}
```
