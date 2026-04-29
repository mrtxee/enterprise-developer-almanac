**`SpringValidatorAdapter`** — это класс-адаптер в **Spring Framework**, который позволяет использовать **Bean Validation (JSR-303/JSR-380)** валидаторы в инфраструктуре валидации Spring.

---

## ✅ Зачем он нужен?

В Spring есть **две системы валидации**:

| Система | Описание |
|---------|----------|
| ✅ **Spring Validator** | Интерфейс `org.springframework.validation.Validator` (старая система Spring) |
| ✅ **Bean Validation** | Аннотации `javax.validation` / `jakarta.validation` (например, `@NotNull`, `@Size`) |

`SpringValidatorAdapter` **преобразует Bean Validation-валидатор** в **Spring Validator**, чтобы их можно было использовать вместе.

---

## ✅ Как это работает?

```
Bean Validation Validator (javax.validation.Validator)
           ↓
   SpringValidatorAdapter
           ↓
Spring Validator (org.springframework.validation.Validator)
```

---

## ✅ Пример использования

### 🔹 1. Без адаптера (только Bean Validation)

```java
@RestController
public class UserController {

    @Autowired
    private javax.validation.Validator validator; // Bean Validation

    @PostMapping("/users")
    public ResponseEntity<?> createUser(@RequestBody User user) {
        Set<ConstraintViolation<User>> violations = validator.validate(user);
        if (!violations.isEmpty()) {
            // Обработка ошибок
        }
        return ResponseEntity.ok().build();
    }
}
```

### 🔹 2. С адаптером (Spring + Bean Validation)

```java
@Configuration
public class ValidationConfig {

    @Autowired
    private javax.validation.Validator beanValidator;

    @Bean
    public org.springframework.validation.Validator springValidator() {
        return new SpringValidatorAdapter(beanValidator);
    }
}
```

Теперь вы можете использовать **Spring-валидацию** с аннотациями Bean Validation:

```java
@RestController
public class UserController {

    @Autowired
    private org.springframework.validation.Validator validator; // Spring Validator

    @PostMapping("/users")
    public ResponseEntity<?> createUser(@RequestBody User user) {
        Errors errors = new BeanPropertyBindingResult(user, "user");
        validator.validate(user, errors);
        if (errors.hasErrors()) {
            // Обработка ошибок
        }
        return ResponseEntity.ok().build();
    }
}
```

---

## ✅ Когда используется автоматически?

В **Spring Boot** `SpringValidatorAdapter` используется **автоматически** при наличии зависимости:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

→ Spring Boot **автоматически создаёт** `LocalValidatorFactoryBean`, который **внутри использует `SpringValidatorAdapter`**.

---

## ✅ Основные сценарии использования

| Сценарий | Описание |
|----------|----------|
| ✅ **`@Valid` в контроллерах** | Spring использует адаптер для Bean Validation |
| ✅ **`MethodValidationPostProcessor`** | Для валидации аргументов методов |
| ✅ **Ручная валидация** | Когда вы вызываете `validator.validate()` вручную |
| ✅ **Интеграция со старым кодом** | Если у вас есть код, использующий `org.springframework.validation.Validator` |

---

## ✅ Пример: Валидация с `@Valid`

```java
@RestController
public class UserController {

    @PostMapping("/users")
    public ResponseEntity<?> createUser(@Valid @RequestBody User user) {
        // Spring автоматически валидирует через SpringValidatorAdapter
        return ResponseEntity.ok().build();
    }
}

public class User {
    @NotNull
    private String username;

    @Email
    private String email;

    // getters/setters
}
```

→ Spring **автоматически** использует `SpringValidatorAdapter` для валидации.

---

## ✅ Сравнение: Spring Validator vs Bean Validation

| Характеристика | Spring Validator | Bean Validation |
|----------------|------------------|-----------------|
| ✅ Пакет | `org.springframework.validation` | `javax.validation` / `jakarta.validation` |
| ✅ Метод валидации | `validate(Object, Errors)` | `validate(Object)` → `Set<ConstraintViolation>` |
| ✅ Аннотации | Свои (редко используются) | `@NotNull`, `@Size`, `@Email` и т.д. |
| ✅ Адаптер | `SpringValidatorAdapter` | — |

---

## ✅ Когда НЕ нужен `SpringValidatorAdapter`?

| Сценарий | Почему |
|----------|--------|
| ✅ Вы используете **только `@Valid`** | Spring Boot настроит всё автоматически |
| ✅ Вы используете **только Bean Validation** | Не нужно преобразовывать в Spring Validator |
| ✅ Вы используете **Spring Boot starter-validation** | Адаптер уже настроен |

---

## ✅ Когда НУЖЕН `SpringValidatorAdapter`?

| Сценарий | Почему |
|----------|--------|
| ✅ Вы используете **старый код** с `org.springframework.validation.Validator` | Нужно адаптировать Bean Validation |
| ✅ Вы хотите **ручную валидацию** через Spring Validator | `validator.validate(obj, errors)` |
| ✅ Вы пишете **кастомный валидатор** | Нужно интегрировать с Bean Validation |
| ✅ Вы используете **MethodValidationPostProcessor** | Требуется адаптер для валидации методов |

---

## ✅ Пример: Ручная валидация с адаптером

```java
@Service
public class UserService {

    @Autowired
    private org.springframework.validation.Validator validator;

    public void createUser(User user) {
        Errors errors = new BeanPropertyBindingResult(user, "user");
        validator.validate(user, errors);
        
        if (errors.hasErrors()) {
            throw new ValidationException(errors);
        }
        // Сохранение
    }
}
```

---

## ✅ Резюме:

| Вопрос | Ответ |
|--------|-------|
| ✅ Что такое `SpringValidatorAdapter`? | Адаптер между **Bean Validation** и **Spring Validator** |
| ✅ Когда используется? | При интеграции **двух систем валидации** |
| ✅ Нужен ли в Spring Boot? | ❌ **Нет** — настраивается автоматически |
| ✅ Когда нужен вручную? | При **ручной валидации** или **старом коде** |
| ✅ Какие аннотации поддерживает? | Все **Bean Validation** (`@NotNull`, `@Size`, `@Email` и т.д.) |

---

✅ **Готово!**  
Теперь вы знаете, **для чего нужен `SpringValidatorAdapter`** и **когда его использовать** в Spring-приложениях.