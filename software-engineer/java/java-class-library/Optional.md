---
aliases:
  - empty
  - ifPresent
  - isPresent
  - java.util.Optional
  - of
  - ofNullable
  - Optional
  - Optional class
  - orElse
  - orElseThrow
  - Опциональные типы
  - Опциональный тип
---
## Класс Optional
> `java.util.Optional`

**Суть**

Объявление Optional-объектов позволяет разрешить или запретить возможность записи NULL-объектов в переменную. Для этого служат статические методы `of()`, `ofNullable()`, `empty()`.

**Создание Optional-объектов**

```java
// Пустой Optional-объект
Optional<Person> optionalPerson = Optional.empty();
// Optional-объект с ненулевым значением
Optional<Person> optionalNonNull = Optional.of(somePerson);
// Optional-объект с возможностью нулевого значения
Optional<Person> optionalNullable = Optional.ofNullable(somePerson);
```

**Проверка значения: ifPresent**

`ifPresent()` проверяет на NULL и выполняет лямбда-выражение.

```java
if (person != null) {
  System.out.println(person);
}
person.ifPresent(System.out::println);
person.ifPresent(st -> System.out.println(st));
```

**Проверка значения: isPresent**

`isPresent()` возвращает булево значение как результат проверки, является ли объект NULL.

```java
if (person != null) {
  System.out.println(person);
}
if (person.isPresent()) {
  System.out.println(person.get());
}
```

**Получение значения: orElse, orElseThrow**

`orElse()`, `orElseThrow()` — методы, сокращающие код путём применения лямбда-выражений.

```java
State st = Optional.ofNullable(getStateOrNull(str)).orElse(new State());
Optional<Person> person = Optional.ofNullable(p);
Person personNew = person.orElse(new Person());
Person personNewThrow = person.orElseThrow(Exception::new);
```
