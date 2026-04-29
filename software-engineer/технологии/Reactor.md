---
aliases:
  - Project Reactor
  - Reactor
---
Отличный вопрос! **WebFlux** и **Project Reactor** — это **разные уровни одной и той же экосистемы**, и их часто путают. Давайте разберёмся чётко.

---

## 🧩 Краткий ответ:

> - **Project Reactor** — это **библиотека реализации Reactive Streams** (даёт `Flux`, `Mono`, операторы).  
> - **Spring WebFlux** — это **реактивный веб-фреймворк**, который **использует Project Reactor** под капотом.

👉 **WebFlux зависит от Reactor, но не наоборот.**

---

## 🔍 Подробное сравнение

| Характеристика | **Project Reactor** | **Spring WebFlux** |
|----------------|---------------------|--------------------|
| **Что это?** | Реактивная библиотека (реализация Reactive Streams) | Веб-фреймворк для создания HTTP-сервисов |
| **Основные классы** | `Flux<T>`, `Mono<T>`, операторы (`map`, `flatMap`, `filter`) | `WebClient`, `RouterFunction`, `@RestController` с `Flux`/`Mono` |
| **Зависимости** | Только `reactive-streams` | Зависит от **Project Reactor** + Spring Core |
| **Где используется?** | В любом реактивном Java-коде (не только веб) | Только для **веб-приложений** (HTTP, WebSocket) |
| **Сервер** | Не предоставляет сервер | Работает на **Netty**, **Undertow**, **Tomcat (неблокирующем)** |
| **Аналог в других экосистемах** | RxJava, Akka Streams | Node.js, Vert.x, Quarkus Reactive Routes |

---

## 🧱 Как они связаны?

### Project Reactor — **фундамент**
- Предоставляет **реактивные типы**:
  ```java
  Flux<String> stream = Flux.just("a", "b", "c");
  Mono<User> user = Mono.just(new User("Alice"));
  ```
- Реализует **операторы**, **backpressure**, **асинхронные цепочки**.
- Это **низкоуровневая библиотека** — как «реактивный `java.util.stream`».

### Spring WebFlux — **надстройка для веба**
- Использует `Flux`/`Mono` **везде**:
  - В контроллерах:
    ```java
    @GetMapping("/users")
    public Flux<User> getUsers() { ... }
    ```
  - В клиенте:
    ```java
    WebClient.create().get().retrieve().bodyToFlux(User.class);
    ```
  - В репозиториях (с R2DBC):
    ```java
    Flux<User> findAll();
    ```
- Предоставляет **интеграцию с HTTP**, маршрутизацию, сериализацию JSON и т.д.

> 💡 WebFlux — это **Spring MVC, но реактивный**, и он **выбирает Project Reactor как реализацию Reactive Streams**.

---

## 🌐 Архитектура приложения

```
[Клиент] 
    → HTTP → [Spring WebFlux (контроллеры, маршруты)]
        → [Ваша бизнес-логика на Flux/Mono]
            → [Project Reactor (операторы, backpressure)]
                → [R2DBC / WebClient / Kafka] → внешние сервисы
```

→ **Project Reactor** работает **внутри** вашего кода и WebFlux.  
→ **WebFlux** отвечает за **взаимодействие с HTTP**.

---

## ❓ Почему Spring выбрал именно Project Reactor?

1. **Создан той же командой** (Pivotal → VMware → Broadcom).
2. **Оптимизирован под Spring** (интеграция с `@Transactional`, `@Controller` и др.).
3. **Без дополнительных зависимостей** — только `reactive-streams`.
4. **Производительность** — один из самых быстрых реализаций Reactive Streams.

> ⚠️ WebFlux **можно заставить работать с другими `Publisher`** (например, RxJava), но **по умолчанию — Reactor**.

---

## 🛠️ Можно ли использовать Project Reactor без WebFlux?

**Да!** Например:

- Реактивная обработка данных в **batch-приложении**:
  ```java
  Flux.from(fileReader)
      .map(parseLine)
      .filter(valid)
      .buffer(1000)
      .subscribe(batch -> saveToDb(batch));
  ```
- **Микросервис на Vert.x** с использованием `Flux` для логики.
- **Тестирование реактивных цепочек** без веба.

→ Reactor — это **универсальная библиотека**, не привязанная к вебу.

---

## 🌐 А можно ли использовать WebFlux без Project Reactor?

**Технически — да**, но **на практике — нет**.

WebFlux:
- Возвращает `Flux`/`Mono` в контроллерах.
- Использует Reactor в `WebClient`.
- Ожидает `Publisher<T>` везде — но **реализация по умолчанию — Reactor**.

Если вы попытаетесь подставить `RxJava Flowable`:
```java
@GetMapping("/data")
public Flowable<String> getData() { ... } // ← не сработает напрямую
```
→ Придётся **конвертировать в `Flux`**:
```java
return Flux.from(flowable);
```

> 💡 Spring Boot **автоматически подключает Project Reactor**, если в classpath есть `spring-boot-starter-webflux`.

---

## 📊 Сравнение по функциональности

| Задача | Project Reactor | Spring WebFlux |
|--------|------------------|----------------|
| Создать поток данных | ✅ `Flux.just(...)` | ❌ |
| Обработать backpressure | ✅ `onBackpressureBuffer()` | ✅ (через Reactor) |
| Написать HTTP-контроллер | ❌ | ✅ `@RestController` |
| Сделать HTTP-запрос | ❌ (нет клиента) | ✅ `WebClient` |
| Подключиться к БД | ❌ | ✅ (с R2DBC) |
| Маршрутизация запросов | ❌ | ✅ `RouterFunction` |

---

## 🎯 Итог

|                     | **Project Reactor** | **Spring WebFlux** |
|---------------------|---------------------|--------------------|
| **Роль**            | Реактивная библиотека | Реактивный веб-фреймворк |
| **Аналогия**        | Как `java.util.stream` | Как Spring MVC, но реактивный |
| **Зависимость**     | Независимая | Зависит от Reactor |
| **Используется для**| Любая реактивная логика | Только веб и HTTP |

> ✅ **Вы используете Project Reactor, когда пишете реактивный код.**  
> ✅ **Вы используете WebFlux, когда этот код — часть веб-сервиса.**

---

## 💡 Совет

- Если вы учитесь реактивному программированию — **начните с Project Reactor** (`Flux`, `Mono`, операторы).
- Если строите веб-API — **добавляйте WebFlux** поверх.

---

Хочешь пример, где Reactor используется без WebFlux, или как WebFlux обрабатывает backpressure от медленного клиента? Напиши — покажу! 😊