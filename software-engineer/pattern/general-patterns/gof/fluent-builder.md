---
aliases:
  - Builder
  - Classic Builder
  - Fluent Builder
  - GoF
---
## 🏗️ Fluent Builder vs Classic Builder (GoF)

Хотя оба паттерна решают проблему создания сложных объектов, они возникли в разные эпохи и преследуют **разные цели**.

> **Краткая суть:**
> *   **Classic Builder (GoF)** — это **структурный паттерн**. Его цель — отделить процесс сборки от представления, чтобы *один и тот же процесс мог создать разные объекты* (например, собрать и деревянный, и каменный дом).
> *   **Fluent Builder** — это **паттерн создания API (идиома)**. Его цель — сделать процесс настройки *одного и того же объекта* максимально читаемым и удобным для разработчика за счет цепочки вызовов (method chaining).

---

## 📊 Сравнительная таблица

| Критерий            | 🏛️ Classic Builder (GoF)                                              | 🌊 Fluent Builder                                                          |
| :------------------ | :--------------------------------------------------------------------- | :------------------------------------------------------------------------- |
| **Главная цель**    | Создание **разных** представлений одного сложного объекта.             | Удобная настройка **одного** объекта с множеством опциональных параметров. |
| **Происхождение**   | Книга "Банда четырех" (GoF), 1994 год.                                 | Современная Java-идиома (эволюция GoF + [[fluent-Interface]]).             |
| **Ключевая фишка**  | Наличие `Director` (Директор) и `Abstract Builder`.                    | **Method Chaining** (возврат `this` / `return this`).                      |
| **Вариативность**   | Может создавать объекты **разных типов** (через общий интерфейс).      | Создает объект **одного типа**, но с разной конфигурацией.                 |
| **Сложность кода**  | Высокая (интерфейсы, абстрактные классы, директоры).                   | Низкая (один статический вложенный класс).                                 |
| **Иммутабельность** | Не гарантирует (зависит от реализации).                                | Почти всегда используется для создания **иммутабельных** объектов.         |
| **Пример из жизни** | Сборка ПК (можно собрать Gaming PC или Office PC по одному алгоритму). | Настройка `User`, `HttpRequest` или `Query` (заполнение полей).            |

---

## 1. 🏛️ Classic Builder Pattern (GoF)

В классическом понимании (по GoF) паттерн Builder нужен, когда у вас есть **сложный продукт**, и вы хотите иметь возможность собирать его **разные вариации**, используя пошаговый алгоритм.

### Структура (UML-подобно)
1. **Builder** (Интерфейс) — описывает шаги сборки (`buildWalls`, `buildRoof`).
2. **ConcreteBuilder** — реализует шаги для конкретного типа (`WoodenHouseBuilder`, `StoneHouseBuilder`).
3. **Director** — управляет порядком шагов (знает, как построить "Стандартный дом").
4. **Product** — итоговый сложный объект.

### Пример кода (Java)

```java
// 1. Продукт
class House {
    private String walls;
    private String roof;
    // ... геттеры, сеттеры
}

// 2. Интерфейс Builder (шаги абстрактны)
interface HouseBuilder {
    void buildWalls();
    void buildRoof();
    House getResult();
}

// 3. Concrete Builders (разные представления)
class WoodenHouseBuilder implements HouseBuilder {
    private House house = new House();
    public void buildWalls() { house.setWalls("Wooden walls"); }
    public void buildRoof() { house.setRoof("Wooden roof"); }
    public House getResult() { return house; }
}

class StoneHouseBuilder implements HouseBuilder {
    private House house = new House();
    public void buildWalls() { house.setWalls("Stone walls"); }
    public void buildRoof() { house.setRoof("Stone roof"); }
    public House getResult() { return house; }
}

// 4. Director (управляет процессом)
class Director {
    public void constructStandardHouse(HouseBuilder builder) {
        builder.buildWalls();
        builder.buildRoof();
    }
}

// Использование:
Director director = new Director();
WoodenHouseBuilder woodenBuilder = new WoodenHouseBuilder();
director.constructStandardHouse(woodenBuilder);
House woodenHouse = woodenBuilder.getResult(); // Готов деревянный дом
```

**Когда применять:** Когда вам нужно собирать **разные по структуре или типу** объекты, используя общий пошаговый алгоритм. В современном Java-бизнес-коде встречается **редко** (чаще в парсерах, генераторах документов, фреймворках).

---

## 2. 🌊 Fluent Builder Pattern

Это адаптация, где мы **выбрасываем** тяжеловесные `Director` и `Abstract Builder`, потому что в 90% задач нам не нужно строить "разные представления". Нам просто нужно удобно заполнить поля одного DTO или доменного объекта.

Главное правило: **каждый метод-сеттер возвращает сам Builder (`return this`)**, что позволяет писать код в одну строку (Fluent Interface).

### Пример кода (Java)

```java
public final class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;

    // Приватный конструктор
    private HttpRequest(Builder builder) {
        this.url = builder.url;
        this.method = builder.method;
        this.headers = builder.headers;
    }

    // Точка входа
    public static Builder builder(String url) {
        return new Builder(url);
    }

    // Вложенный Fluent Builder
    public static class Builder {
        private final String url;
        private String method = "GET";
        private final Map<String, String> headers = new HashMap<>();

        public Builder(String url) { this.url = url; }

        // ⚡ КЛЮЧЕВОЕ ОТЛИЧИЕ: return this
        public Builder method(String method) {
            this.method = method;
            return this; 
        }

        public Builder header(String key, String value) {
            this.headers.put(key, value);
            return this; 
        }

        public HttpRequest build() {
            return new HttpRequest(this);
        }
    }
}

// Использование (читается как предложение на английском):
HttpRequest request = HttpRequest.builder("https://api.com")
    .method("POST")
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer token")
    .build();
```

**Когда применять:** Практически всегда в бизнес-логике, когда у класса много опциональных полей, и вы хотите сделать код читаемым, а итоговый объект — иммутабельным.

---

## 💡 Главные отличия "на пальцах"

| Ситуация | Какой паттерн использовать? |
| :--- | :--- |
| "Мне нужно создать `User`, но у него 15 полей, и половина из них опциональна." | 🌊 **Fluent Builder** |
| "Мне нужно создать `Query` (SQL, NoSQL, GraphQL), и процесс их сборки сильно отличается, но интерфейс вызова должен быть общим." | 🏛️ **Classic Builder (GoF)** |
| "Я хочу, чтобы мой код читался как связный текст: `user.withName().withAge().build()`." | 🌊 **Fluent Builder** |
| "Я пишу парсер HTML, который может собрать из тегов либо DOM-дерево, либо упрощенную текстовую модель." | 🏛️ **Classic Builder (GoF)** |

---

## 🎯 Эволюция: Почему Fluent победил в Java?

В классическом GoF Builder методы часто возвращали `void`:

```java
// Classic GoF style
builder.buildWalls();
builder.buildRoof();
House house = builder.getResult();
```

Но разработчики заметили, что если изменить `void` на `return this`, код становится **намного чище**:

```java
// Fluent style
builder.buildWalls().buildRoof();
```

Поскольку в современном Java **Director** (который управляет шагами) оказался не нужен для большинства задач (разработчики сами хотят решать, какие поля заполнять, а какие нет), от него отказались.

**Итог:** То, что мы сегодня называем "Builder" в Java (например, `@Builder` от Lombok или `Stream.Builder`) — это **Fluent Builder**. Классический GoF Builder остался нишевым инструментом для специфических архитектурных задач.
