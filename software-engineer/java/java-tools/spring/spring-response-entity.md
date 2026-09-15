---
aliases:
  - HTTP Response
  - HTTP-ответ
  - JAX-RS Response
  - Response
  - Response Entity
  - ResponseEntity
---

## ResponseEntity

**Суть**

`ResponseEntity<T>` (`org.springframework.http.ResponseEntity`) — универсальный класс Spring Framework, представляющий полный HTTP-ответ: он объединяет **статус-код**, **заголовки** и **тело ответа** в одном объекте.

**Состав**

- HTTP-статус-код (200, 404, 500 и т. д.);
- заголовки (Content-Type, Location, кастомные и т. п.);
- тело ответа (данные любого типа).

**Ключевые особенности**

- полный контроль над HTTP-ответом;
- типобезопасность (generic-тип для тела ответа);
- встроенное построение ответов через статические фабричные методы (`ok()`, `notFound()`, `created()` и т. д.).

### Ключевые преимущества ResponseEntity

1. **Полный контроль над HTTP-ответом.** Можно явно задать:
   - HTTP-статус (200, 404, 500 и т. д.);
   - кастомные заголовки;
   - тело ответа (данные или сообщение об ошибке).
2. **Гибкость в обработке ошибок.** Можно возвращать разные статусы в зависимости от сценария:
   - `200 OK` — успех;
   - `400 Bad Request` — неверные входные данные;
   - `404 Not Found` — ресурс не найден;
   - `500 Internal Server Error` — внутренняя ошибка сервера.
3. **Возможность добавлять кастомные заголовки.** Полезно для:
   - пагинации (`X-Total-Count`, `Link`);
   - кэширования (`Cache-Control`);
   - CORS-заголовков;
   - метаданных о версии API.
4. **Поддержка условных ответов.** Можно реализовать механизмы:
   - `ETag` / `If-None-Match` (кэширование);
   - `Last-Modified` / `If-Modified-Since`.
5. **Интеграция с бизнес-логикой.** Статус и содержимое ответа можно динамически менять на основе:
   - результатов валидации;
   - прав доступа пользователя;
   - состояния внешних сервисов.

### Примеры использования ResponseEntity

**Пример 1. Успешный ответ с телом**

```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    return ResponseEntity.ok(user); // статус 200 + тело
}
```

**Пример 2. Ответ с кастомными заголовками**

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody User user) {
    User savedUser = userService.save(user);

    return ResponseEntity
        .created(URI.create("/users/" + savedUser.getId())) // статус 201 + Location
        .header("X-API-Version", "1.0") // кастомный заголовок
        .body(savedUser); // тело ответа
}
```

**Пример 3. Ошибочный ответ**

```java
@GetMapping("/products/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    Optional<Product> product = productService.findById(id);
    if (product.isPresent()) {
        return ResponseEntity.ok(product.get());
    } else {
        return ResponseEntity.notFound().build(); // статус 404
    }
}
```

**Пример 4. Условный ответ (кэширование)**

```java
@GetMapping("/products/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id,
                                         @RequestHeader(value = "If-None-Match", required = false) String etag) {
    Product product = productService.findById(id);
    String currentEtag = calculateEtag(product);

    if (currentEtag.equals(etag)) {
        return ResponseEntity.status(HttpStatus.NOT_MODIFIED).build(); // 304
    }

    return ResponseEntity
        .ok()
        .eTag(currentEtag)
        .body(product);
}
```

**Пример 5. Обработка ошибок с деталями**

```java
@PutMapping("/orders/{id}")
public ResponseEntity<?> updateOrder(@PathVariable Long id, @RequestBody Order order) {
    try {
        Order updated = orderService.update(id, order);
        return ResponseEntity.ok(updated);
    } catch (ValidationException e) {
        return ResponseEntity
            .badRequest() // 400
            .body(Map.of(
                "error", "Validation failed",
                "details", e.getMessage()
            ));
    } catch (ResourceNotFoundException e) {
        return ResponseEntity.notFound().build(); // 404
    }
}
```

---

## Response (JAX-RS)

**Суть**

`Response` (`javax.ws.rs.core.Response`) — стандартный класс JAX-RS (Java API for RESTful Web Services), не специфичный для Spring.

**Особенности**

- часть стандарта Java EE (JSR-339);
- менее типобезопасен — тело ответа передаётся как `Object`;
- требует явного указания типа медиа через `MediaType`;
- чаще используется в приложениях на Jersey, RESTEasy и других JAX-RS-реализациях.

### Примеры использования Response

**Пример 1. Простой ответ**

```java
@GET
@Path("/users/{id}")
public Response getUser(@PathParam("id") Long id) {
    User user = userService.findById(id);
    return Response.status(Response.Status.OK)
        .entity(user)
        .type(MediaType.APPLICATION_JSON)
        .build();
}
```

**Пример 2. Ошибочный ответ**

```java
@GET
@Path("/products/{id}")
public Response getProduct(@PathParam("id") Long id) {
    Product product = productService.findById(id);
    if (product == null) {
        return Response.status(Response.Status.NOT_FOUND)
            .entity("Product not found")
            .build();
    }
    return Response.ok(product).build();
}
```

---

## Сравнительная таблица

| Критерий | ResponseEntity | Response |
|--------|--------------|----------|
| **Происхождение** | Spring Framework | JAX-RS (Java EE) |
| **Типобезопасность** | Да (generic) | Нет (`Object` для тела) |
| **Контроль над ответом** | Полный (статус, заголовки, тело) | Полный (статус, заголовки, тело) |
| **Удобство использования в Spring** | Высокая интеграция | Требуется настройка JAX-RS |
| **Фабричные методы** | Есть (`ok()`, `badRequest()` и т. д.) | Нет, всё через `Response.status()` |
| **Указание типа контента** | Автоматически по типу тела | Явно через `.type()` |
| **Типичная среда** | Spring Boot, Spring MVC | Jersey, RESTEasy, Java EE |

---

## Когда что использовать

**Используйте ResponseEntity, когда:**

- работаете в экосистеме Spring (Spring Boot, Spring MVC);
- нужна типобезопасность — чтобы компилятор проверял тип тела ответа;
- нужен максимально лаконичный код благодаря фабричным методам (`ok()`, `created()`, `noContent()`);
- активно используются кастомные заголовки или условные ответы (ETag, Last-Modified);
- разрабатывается REST API с чёткой спецификацией ответов;
- обрабатываются разные HTTP-статусы в одном методе (200, 400, 404 и т. д.);
- нужна интеграция с другими компонентами Spring (например, `@ControllerAdvice` для глобальной обработки ошибок).

**Используйте Response, когда:**

- используется JAX-RS-фреймворк (Jersey, RESTEasy) вместо Spring MVC;
- приложение не зависит от Spring и должно быть переносимым между контейнерами Java EE;
- важно следовать стандартам Java EE и избежать привязки к Spring;
- в проекте уже используется JAX-RS и нужен единообразный код;
- типобезопасность тела ответа не критична.

**Можно обойтись без ResponseEntity, когда:**

- все ответы имеют статус 200;
- не нужны кастомные заголовки;
- достаточно стандартной обработки ошибок Spring;
- метод всегда возвращает данные («ресурс не найден» не бывает).

---

## Сравнение подходов

**Вариант 1. Без ResponseEntity (автоматический ответ)**

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}
```

Что вернётся:

- всегда статус 200, если нет исключений;
- тело — сериализованный объект `User`;
- стандартные заголовки от Spring.

**Вариант 2. С ResponseEntity (полный контроль)**

```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    Optional<User> user = userService.findById(id);

    if (user.isPresent()) {
        return ResponseEntity.ok(user.get()); // 200 OK + тело
    } else {
        return ResponseEntity.notFound().build(); // 404 Not Found
    }
}
```

---

## Практические рекомендации

Для Spring-приложений почти всегда выбирается `ResponseEntity`. Это «родной» способ Spring возвращать HTTP-ответы. Он:

- лучше интегрирован с Spring MVC;
- поддерживает все возможности фреймворка;
- упрощает тестирование;
- соответствует общепринятым практикам Spring Boot.

**Когда может понадобиться Response в Spring**

- миграция старого JAX-RS кода в Spring;
- использование Jersey как движка REST в Spring-приложении;
- гибридное решение с поддержкой и Spring MVC, и JAX-RS.

---

## Альтернативы и компромиссы

1. **@ResponseStatus** — для фиксированных статусов:

```java
@ResponseStatus(HttpStatus.CREATED)
@PostMapping("/users")
public User createUser(@RequestBody User user) { /* ... */ }
```

1. **ResponseStatusException** — для быстрой обработки ошибок:

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id)
        .orElseThrow(() -> new ResponseStatusException(
            HttpStatus.NOT_FOUND,
            "User not found"
        ));
}
```

1. **@ControllerAdvice + @ExceptionHandler** — для глобальной обработки исключений (позволяет избежать `ResponseEntity` в контроллерах).

---

## Итог

- `ResponseEntity` — стандартный и предпочтительный выбор для Spring-приложений. Обеспечивает типобезопасность, лаконичность и полную интеграцию с экосистемой Spring.
- `Response` — стандартный класс JAX-RS, используемый в Java EE-приложениях и фреймворках типа Jersey/RESTEasy. Менее удобен в Spring без дополнительной настройки.

`ResponseEntity` особенно полезен для:

- REST API с чёткой спецификацией;
- обработки ошибок с детализацией;
- добавления метаданных в заголовки;
- реализации механизмов кэширования и пагинации.

В 95 % случаев при разработке на Spring Boot/Spring MVC следует использовать `ResponseEntity`. `Response` имеет смысл только при работе с JAX-RS или миграции старого кода. Для простых CRUD-операций можно использовать более лаконичные подходы, но `ResponseEntity` остаётся стандартом для сложных сценариев.
