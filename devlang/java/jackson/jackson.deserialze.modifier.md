# Jackson Deserialization Modifier: StdConverter and JsonDeserializer

- `StdConverter`: JsonMapper가 변환 완료한 객체에 대해서 추가 수정(post process) 수행
- `JsonDeserializer` : JsonMapper가 해당 아이템을 deserialize할 때 기본이 아닌 지정된 Deserializer 사용

```java
    class PostProcess extends com.fasterxml.jackson.databind.util.StdConverter<JsonDoc, JsonDoc> {
        @Override public JsonDoc convert(JsonDoc doc) {
            doc.setData("postProcessed")
            return doc;
        }
    }

    // convert emptySting to null
    public class EmptyToNullStringDeserializer extends com.fasterxml.jackson.databind.JsonDeserializer<String> {
        @Override
        public String deserialize(com.fasterxml.jackson.core.JsonParser p,
                                  com.fasterxml.jackson.databind.DeserializationContext ctxt) throws java.io.IOException {
            String v = p.getValueAsString();
            return (v == null || v.isEmpty()) ? null : v;
        }
    }

    @Data
    // converter = using StdConverter, post-process after jackson convets Object
    @JsonDeserialize(converter = JsonDoc.PostProcess.class)
    public class JsonDoc {



        @JsonProperty("data")
        // using JsonDeserializer, json deserialize this item using assigned JsonDeserializer
        @JsonDeserialize(using = EmptyToNullStringDeserializer.class)
        private String data;
    }
```
