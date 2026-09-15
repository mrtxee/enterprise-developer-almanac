---
aliases:
  - Argument validation
  - Assert
  - Assert.hasText
  - Assert.isFalse
  - Assert.isNull
  - Assert.isTrue
  - Assert.notEmpty
  - Assert.notNull
  - Assert.state
  - Bean Validation
  - IllegalArgumentException
  - IllegalStateException
  - org.springframework.util.Assert
  - Spring Assert
  - Spring Framework
  - State checking
  - Валидация аргументов
  - Проверка состояния
---

## Spring Assert

**`org.springframework.util.Assert`** — это утилитный класс в **Spring Framework**, предназначенный для **валидации аргументов и состояний** в коде. Он используется для проверки условий и выбрасывания исключений, если условие не выполнено.

## Назначение

`Assert` используется для **защиты от некорректных данных** и **раннего выявления ошибок**. Вместо ручных проверок `if (x == null) throw ...` используются лаконичные методы.

| Задача | Пример |
|--------|--------|
| ✅ Проверка, что объект **не null** | `Assert.notNull(obj, "obj не должен быть null")` |
| ✅ Проверка, что строка **не пустая** | `Assert.hasText(str, "строка не должна быть пустой")` |
| ✅ Проверка, что условие **истинно** | `Assert.isTrue(x > 0, "x должен быть положительным")` |
| ✅ Проверка, что коллекция **не пустая** | `Assert.notEmpty(list, "список не должен быть пустым")` |
| ✅ Проверка состояния объекта | `Assert.state(isReady, "объект не готов")` |

## Основные методы Assert

| Метод | Описание | Пример |
|-------|----------|--------|
| `notNull(Object, String)` | Проверяет, что объект **не null** | `Assert.notNull(user, "user не должен быть null")` |
| `hasText(String, String)` | Проверяет, что строка **не null и не пустая** | `Assert.hasText(name, "name не должен быть пустым")` |
| `isTrue(boolean, String)` | Проверяет, что условие **true** | `Assert.isTrue(age > 0, "age должен быть > 0")` |
| `notEmpty(Collection, String)` | Проверяет, что коллекция **не пустая** | `Assert.notEmpty(items, "items не должен быть пустым")` |
| `notEmpty(Object[], String)` | Проверяет, что массив **не пустой** | `Assert.notEmpty(arr, "массив не должен быть пустым")` |
| `state(boolean, String)` | Проверяет **состояние объекта** | `Assert.state(isInitialized, "объект не инициализирован")` |
| `isNull(Object, String)` | Проверяет, что объект **null** | `Assert.isNull(obj, "obj должен быть null")` |
| `isFalse(boolean, String)` | Проверяет, что условие **false** | `Assert.isFalse(flag, "flag должен быть false")` |

## Примеры использования

### Проверка аргументов метода

```java
public void createUser(String username, String email) {
    Assert.hasText(username, "Username не должен быть пустым");
    Assert.hasText(email, "Email не должен быть пустым");

    // Дальнейшая логика
}
```

### Проверка на null

```java
public void processUser(User user) {
    Assert.notNull(user, "User не должен быть null");

    // Дальнейшая логика
}
```

### Проверка состояния

```java
public void save() {
    Assert.state(isConnected, "Сначала нужно подключиться к БД");

    // Дальнейшая логика
}
```

### Проверка коллекции

```java
public void processItems(List<Item> items) {
    Assert.notEmpty(items, "Список items не должен быть пустым");

    // Дальнейшая логика
}
```

## Исключения

| Метод | Исключение |
|-------|------------|
| `notNull`, `hasText`, `notEmpty`, `isTrue`, `isFalse` | `IllegalArgumentException` |
| `state` | `IllegalStateException` |
| `isNull` | `IllegalArgumentException` |

## Assert vs ручные проверки

| С `Assert` | Без `Assert` |
|------------|--------------|
| ✅ `Assert.notNull(user, "user не должен быть null")` | ❌ `if (user == null) throw new IllegalArgumentException("user не должен быть null")` |
| ✅ Коротко и читаемо | ❌ Многословно |
| ✅ Стандартный стиль Spring | ❌ Свой стиль |

## Когда использовать Assert

| Сценарий | Пример |
|----------|--------|
| ✅ В **сервисах** для проверки аргументов | `Assert.notNull(user, ...)` |
| ✅ В **конструкторах** для проверки зависимостей | `Assert.notNull(repo, ...)` |
| ✅ В **методах** для проверки состояния | `Assert.state(isReady, ...)` |
| ✅ В **валидации входных данных** | `Assert.hasText(email, ...)` |
| ❌ В **бизнес-логике** для валидации данных пользователя | Используйте **Bean Validation** (`@NotNull`, `@Valid`) |
| ❌ Для **обработки ошибок** в runtime | Используйте **try-catch** |

## Пример: валидация в сервисе

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        Assert.notNull(userRepository, "UserRepository не должен быть null");
        this.userRepository = userRepository;
    }

    public User createUser(String username, String email) {
        Assert.hasText(username, "Username не должен быть пустым");
        Assert.hasText(email, "Email не должен быть пустым");
        Assert.isTrue(email.contains("@"), "Email должен содержать @");

        return userRepository.save(new User(username, email));
    }
}
```
