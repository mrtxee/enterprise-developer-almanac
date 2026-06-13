---
aliases:
  - Fluent API
  - Fluent
  - API
---
**Fluent API** — это стиль построения интерфейса, при котором методы возвращают сам объект (`this`), что позволяет вызывать их цепочкой (method chaining). Это делает код более читаемым и выразительным.

### Пример

```java
// Без Fluent API
client.setUrl("https://api.com");
client.setMethod("GET");
client.addHeader("Accept", "application/json");
Response response = client.execute();

// С Fluent API
Response response = client.get()
    .uri("https://api.com")
    .header("Accept", "application/json")
    .retrieve()
    .body(Response.class);
```

### Где используется в Spring

- **`RestClient`**: `restClient.get().uri(...).retrieve().body(...)`
- **`WebClient`**: `webClient.post().uri(...).body(...).retrieve().bodyToMono(...)`
- **`MockMvc`**: `mockMvc.perform(get("/api").param("id", "1")).andExpect(status().isOk())`

### Преимущества

- ✅ Читаемость (код читается как предложение)
- ✅ Меньше кода (без временных переменных)
- ✅ IDE автодополнение ведёт по цепочке
- ✅ Легко комбинировать и переиспользовать

### Суть

Цепочка методов, где каждый метод возвращает тот же или новый объект, позволяя вызывать следующий метод.