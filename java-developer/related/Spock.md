---
aliases:
  - Spock
  - Spock Framework
  - JUnit
  - BDD-стиль
  - BDD
  - Behavior-Driven Development
---
## Spock Framework
> `spock.lang.Specification`

**Specification** — базовый класс для всех тестов в **Spock Framework** (альтернатива JUnit, написанная на Groovy). Он предоставляет [[BDD|BDD]]-стиль написания тестов с богатой DSL и выразительным синтаксисом.

---

### 🧱 Структура теста (блоки)

```groovy
class MySpec extends Specification {

    def "should return user by id"() {
        given: "подготовка данных"
        def user = new User(id: 1, name: "Alex")
        def repo = Mock(UserRepository)
        repo.findById(1) >> Optional.of(user)

        and: "создаём сервис"
        def service = new UserService(repo)

        when: "вызываем метод"
        def result = service.getUser(1)

        then: "проверяем результат"
        result.get() == user
        result.get().name == "Alex"
    }
}
```

---

### 📦 Основные блоки

| Блок | Назначение |
|------|------------|
| `given:` | Подготовка данных (аналог `@Before` + инициализация) |
| `when:` | Вызов тестируемого метода |
| `then:` | Проверки (assertions) |
| `expect:` | Альтернатива `when` + `then` (для простых проверок) |
| `where:` | Табличное тестирование (параметризация) |
| `cleanup:` | Освобождение ресурсов (аналог `@After`) |

---

### 🧪 Базовый пример с параметризацией

```groovy
class CalculatorSpec extends Specification {

    def "should add two numbers"() {
        expect:
        calculator.add(a, b) == result
        where:
        a | b || result
        2 | 3 || 5
        0 | 5 || 5
       -1 | 1 || 0
    }
}
```

---

### 🎭 Мокирование (Mocking)

Spock имеет встроенный синтаксис для моков:

```groovy
def repo = Mock(UserRepository)
repo.findById(1) >> Optional.empty()          // возвращаем пустой Optional
repo.findById(2) >> { throw new RuntimeException() } // имитация исключения

// Проверка взаимодействия (вызов метода)
1 * repo.save(_)   // ожидаем ровно один вызов save с любым аргументом
0 * repo.delete(_) // ни одного вызова delete
```

---

### ✅ Spock vs JUnit

| Характеристика | Spock                           | JUnit                               |
| -------------- | ------------------------------- | ----------------------------------- |
| Синтаксис      | BDD-стиль (given/when/then)     | Императивный + аннотации            |
| Параметризация | `where:` блок + таблицы         | `@ParameterizedTest` + `@CsvSource` |
| Моки           | Встроенные (`Mock()`, `Stub()`) | Требует Mockito, EasyMock           |
| Язык           | Groovy (более лаконичный)       | Java (более многословный)           |

---

### 📌 Зависимость для Maven

```xml
<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-core</artifactId>
    <version>2.3-groovy-4.0</version>
    <scope>test</scope>
</dependency>
```

Spock активно используется для тестирования Spring-приложений, в том числе через `spring-spock` модуль. Это отличная альтернатива классическим JUnit + Mockito, если команда готова использовать Groovy в тестах.