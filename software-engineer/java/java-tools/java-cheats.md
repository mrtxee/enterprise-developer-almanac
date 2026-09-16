---
aliases:
  - java cheats
  - java code shortcuts
  - java shortcuts
  - java tricks
  - java-cheats
---
## master object mapper
```java
public static final ObjectMapper OBJECT_MAPPER = new ObjectMapper()
    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
    .configure(SerializationFeature.INDENT_OUTPUT, true);
public static final ObjectWriter OBJECT_WRITER = OBJECT_MAPPER.writerWithDefaultPrettyPrinter();
```
