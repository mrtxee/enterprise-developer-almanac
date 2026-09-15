---
aliases:
  - Java record
  - java.lang.Record
  - record
  - Record
  - record class
  - record type
  - запись
  - класс-запись
---

## Record type в Java

Тип добавлен в Java 16 и расширяет абстрактный класс `java.lang.Record`.

Тип `record` сокращает объём кода при описании класса, так как реализует базовый функционал класса:

- Все поля объекта типа record — `private final`
- Сам класс record — `final`
