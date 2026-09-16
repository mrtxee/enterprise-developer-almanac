---
aliases:
  - Protobuf
  - Protocol Buffers
  - протобуф
---

## Protocol Buffers (Protobuf)

**Protobuf** — это язык, независимый от языка программирования, для описания структур данных, который компилируется в эффективный бинарный код для сериализации/десериализации.

**Пример .proto файла**

```protobuf
syntax = "proto3";

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
  bool is_active = 4;
}
```

Компиляция через `protoc` даёт классы на Java, Go, Python и т. д.

**Как работает**

```python
user = User(id=123, name="Alice", email="alice@example.com", is_active=True)
serialized = user.SerializeToString()  # Бинарные данные (~30 байт)

deserialized = User()
deserialized.ParseFromString(serialized)
print(deserialized.name)  # Alice
```

**Особенности**

- ✅ Сверхкомпактный формат — в 3–5 раз меньше, чем [[config-formats|JSON]].
- ✅ Быстрая сериализация — в 5–10 раз быстрее JSON.
- ✅ Строгая типизация — компилятор проверяет поля, типы, обязательность.
- ✅ Обратная совместимость — можно добавлять новые поля без ломки старых клиентов.
- ✅ Поддержка многих языков — Java, Go, Python, C#, Rust, JavaScript и др.
- ✅ Генерация кода — классы создаются автоматически, без ручного парсинга.
- ❌ Не человекочитаем — бинарный формат.
- ❌ Требует компиляции `.proto` в код.
- ❌ Меньше гибкости, чем JSON — нет динамических полей.

**Идеален для** [[RPC|RPC]] (gRPC), событий в [[kafka|Kafka]], внутренних API.

---

[[avro|avro]]
