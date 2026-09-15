---
aliases:
  - Domain-Specific Language
  - DSL
  - Fluent API
  - Fluent Builder
  - Fluent Interface
  - Method Chaining
  - fluent-Interface
  - jOOQ
  - Предметно-ориентированный язык
  - Текучий интерфейс
  - Цепочка вызовов
---

## Fluent Interface (текучий интерфейс)

**Суть**

> **Fluent Interface** — это подход к проектированию API, при котором код читается как **связный текст на естественном языке** (или предметно-ориентированном языке, DSL).
> Технически он опирается на **цепочку вызовов (Method Chaining)**, где каждый метод возвращает объект, позволяя сразу вызвать следующий метод.

Термин ввели Эрик Эванс #👨 и Мартин Фаулер #👨 в 2005 году.

## Суть и техническая реализация

Главная цель Fluent Interface — **снизить когнитивную нагрузку** при чтении кода. Разработчик не должен держать в голове состояние объекта; API сам «подсказывает», что можно сделать дальше.

### Технический фундамент — `return this`

```java
public class Email {
    private String to;
    private String subject;

    // Методы возвращают сам объект (this)
    public Email to(String to) {
        this.to = to;
        return this;
    }

    public Email subject(String subject) {
        this.subject = subject;
        return this;
    }

    public void send() { /* отправка */ }
}
```

## Сравнение: обычный API vs Fluent API

### Обычный API

Много шума, разорванный контекст.

```java
Email email = new Email();
email.setTo("john@example.com");
email.setSubject("Hello");
email.setBody("How are you?");
email.send();
```

**Проблема:** глазу приходится «прыгать» по коду. Много повторяющегося кода (`email.`).

### Fluent API

Читается как предложение.

```java
new Email()
    .to("john@example.com")
    .subject("Hello")
    .body("How are you?")
    .send();
```

**Результат:** код читается слева направо, как инструкция на английском языке.

## Продвинутый уровень: контекстный Fluent Interface

Настоящий Fluent Interface — это **не просто** `return this;`. Это **управление контекстом**, когда API *запрещает* вызывать методы в неправильном порядке, направляя разработчика.

### Пример: конструктор SQL-запросов (как в jOOQ)

```java
// API само подсказывает, что после SELECT нужно указать поля,
// а после FROM — таблицу. Вы не сможете вызвать WHERE до FROM.

dslContext.select(USERS.NAME, USERS.EMAIL)
          .from(USERS)
          .where(USERS.AGE.gt(18))
          .orderBy(USERS.NAME)
          .fetch();
```

Здесь методы возвращают **не просто `this`**, а **специальные объекты-контексты** (например, `SelectJoinStep`, `SelectWhereStep`), в которых доступны только те методы, которые имеют смысл на данном этапе построения запроса.

## Примеры Fluent Interface в Java

**Java Stream API**

```java
List<String> names = users.stream()
    .filter(u -> u.isActive())
    .map(User::getName)
    .sorted()
    .collect(Collectors.toList());
```

**StringBuilder**

```java
String sql = new StringBuilder()
    .append("SELECT * FROM users ")
    .append("WHERE id = ?")
    .toString();
```

**Mockito (тестирование)**

```java
when(userService.findById(1L))
    .thenReturn(Optional.of(new User("John")));
```

**AssertJ (проверки в тестах)**

```java
assertThat(user.getName())
    .isNotNull()
    .startsWith("J")
    .endsWith("n")
    .hasSize(4);
```

**Spring MockMvc**

```java
mockMvc.perform(get("/api/users/1"))
       .andExpect(status().isOk())
       .andExpect(jsonPath("$.name").value("John"));
```

## Особенности

- ✅ **Читаемость:** код выглядит как предметно-ориентированный язык (DSL)
- ✅ **Компактность:** меньше временных переменных и дублирования
- ✅ **Направляемость:** контекстные интерфейсы не дают написать «глупый» код
- ✅ **Иммутабельность:** часто используется для создания неизменяемых объектов (как в Builder)
- ❌ **Сложность отладки:** если в середине цепочки падает `NullPointerException`, сложно понять, какой именно метод вернул `null` (в Java 14+ эта проблема частично решена)
- ❌ **Сложность проектирования:** требует глубокого продумывания API и использования дженериков для контекстов
- ❌ **Не для всех задач:** если методы не логически связаны, цепочка выглядит неестественно
- ❌ **Проблемы с наследованием:** возврат `this` в базовом классе может сломать типизацию в классах-наследниках (решается через дженерики `<T extends BaseBuilder<T>>`)

## Fluent Interface vs Fluent Builder

Часто эти понятия путают, но они находятся на разных уровнях:

| Характеристика | [[fluent-builder\|Fluent Builder]] | Fluent Interface |
| :------------- | :--------------------------------- | :--------------- |
| **Что это?**   | Паттерн **создания** объекта (порождающий паттерн) | Паттерн **проектирования API** (архитектурная идиома) |
| **Цель**       | Собрать сложный объект шаг за шагом | Сделать API максимально читаемым и похожим на язык |
| **Связь**      | Fluent Builder *использует* принципы Fluent Interface | Fluent Interface *шире*, чем Builder (используется в Stream, AssertJ, SQL-билдерах) |

## Чек-лист применения

- Методы логически связаны и относятся к одной сущности/процессу? → ✅ Да, Fluent
- Порядок вызовов важен или его нужно ограничить? → ✅ Да (контекстный Fluent Interface)
- Методов больше 3-х и они создают «простыню» кода? → ✅ Да
- Это разовая утилита с 2 методами? → ❌ Нет, обычный API проще
- Методы возвращают разные несвязанные сущности? → ❌ Нет, будет каша

## Итог

- **Цель:** код читается как естественный язык (DSL)
- **Механика:** Method Chaining (возврат `this` или контекста)
- **Эволюция:** от простого `return this` до контекстных интерфейсов, которые скрывают ненужные методы
- **Примеры в Java:** Stream API (`.filter().map().collect()`), AssertJ (`.assertThat().isNotNull().isEqualTo()`), Mockito (`.when().thenReturn()`), StringBuilder (`.append().append()`)
- **Правило:** если API можно прочитать вслух как связное предложение — получился отличный Fluent API.
