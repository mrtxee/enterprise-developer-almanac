---
aliases:
  - Automatic Module
  - Classpath
  - exports
  - exports ... to
  - Java Platform Module System
  - jlink
  - JLink
  - JPMS
  - JVM-level encapsulation
  - module descriptor
  - Module Path
  - module-info
  - module-info.java
  - Named Module
  - opens
  - opens ... to
  - Project Jigsaw
  - provides
  - requires
  - requires static
  - requires transitive
  - Service Provider Interface
  - SPI
  - System Module
  - Unnamed Module
  - uses
  - Автоматический модуль
  - Безымянный модуль
  - дескриптор модуля
  - Именованный модуль
  - инкапсуляция на уровне JVM
  - Системный модуль
---

## Java Platform Module System (JPMS)

**Краткий ответ**
> **JPMS (Project Jigsaw)** — система модулей в Java, которая добавляет инкапсуляцию на уровне пакетов и явные зависимости между частями приложения.
> **`module-info.java`** — дескриптор модуля, который объявляет его имя, зависимости и экспортируемые пакеты.

---

## Назначение JPMS

| Проблема до Java 9 | Решение в Java 9+ |
|--------------------|-------------------|
| Classpath hell — конфликты версий | ✅ Module Path — явные зависимости |
| Нет инкапсуляции — `public` виден везде | ✅ Exports — пакеты видны только явно |
| Большой runtime — весь JDK всегда | ✅ JLink — кастомный runtime |
| Нет мета-информации | ✅ module-info.java — декларативные зависимости |

---

## Сравнение Classpath и Module Path

| Аспект | Classpath (до Java 9) | Module Path (Java 9+) |
|--------|----------------------|-----------------------|
| Видимость | Все JAR в classpath, любой класс виден | Видны только экспортированные пакеты |
| Проверки | Компилятор не проверяет, что реально используется | Компилятор и JVM проверяют зависимости |
| Риски | JAR hell — конфликты версий, скрытые зависимости | Нет скрытых зависимостей |
| Инкапсуляция | Отсутствует | На уровне модулей |

---

## Структура модуля

Модуль — это группа пакетов + `module-info.java`, которая явно объявляет имя, зависимости и экспортируемые пакеты.

Ключевые свойства модуля:

| Свойство | Описание |
|----------|----------|
| Именованный | У модуля есть уникальное имя (`com.example.mymodule`) |
| С инкапсуляцией | Только экспортированные пакеты видны снаружи |
| С зависимостями | Явно объявляет, от каких модулей зависит |
| С границами | JVM проверяет доступ на уровне модулей, а не только классов |

Пример структуры модуля:

- `module-info.java` — дескриптор модуля (обязательно)
- `com/example/api/` — экспортируемый пакет (публичный API)
- `com/example/internal/` — внутренний пакет (не экспортируется)
- `com/example/Main.java` — точка входа
- `src/` — исходный код, `bin/` — результат сборки

---

## Синтаксис module-info.java

```java
module com.example.mymodule {
    requires java.sql;
    requires transitive java.logging;
    requires static com.fasterxml.jackson.databind;

    exports com.example.mymodule.api;
    exports com.example.mymodule.service to com.example.othermodule;

    opens com.example.mymodule.internal;
    opens com.example.mymodule.model to com.fasterxml.jackson.databind;

    provides com.example.myservice.MyService with com.example.mymodule.MyServiceImpl;
    uses com.example.myservice.MyService;
}
```

---

## Ключевые директивы

### requires

`requires` — явное объявление зависимости модуля. Без него классы из других модулей недоступны, даже если JAR есть в module-path.

| Директива | Описание | Пример |
|-----------|----------|--------|
| `requires` | Обязательная зависимость (compile + runtime) | `requires java.sql;` |
| `requires transitive` | Зависимость передаётся клиентам модуля | `requires transitive java.logging;` |
| `requires static` | Опциональная, только при компиляции | `requires static lombok;` |

```java
requires java.sql;
requires transitive java.logging;
requires static lombok;
```

Что даёт `requires`:

- ✅ Явные зависимости — видно, от чего зависит модуль
- ✅ Проверка на компиляцию — компилятор ошибётся, если забыли `requires`
- ✅ Проверка на runtime — JVM упадёт fail-fast, если модуль не найден
- ✅ Инкапсуляция — модуль видит только то, что явно заявил
- ✅ Надёжная сборка — нет скрытых зависимостей через classpath

`requires transitive` применяют, когда модуль является «обёрткой» над другим: клиенты автоматически «читают» зависимость без явного `requires`. `requires static` — для аннотационных процессоров (Lombok, AutoValue), которые нужны только при компиляции.

Когда использовать:

| Сценарий | Директива |
|----------|-----------|
| Обычная зависимость | `requires` |
| Публичный API модуля | `requires transitive` |
| Аннотационный процессор | `requires static` |
| Тестовая зависимость | `requires static` |

### exports

`exports` — экспорт пакетов наружу, публичный API модуля.

| Модификатор | Описание | Пример |
|-------------|----------|--------|
| `exports` | Экспорт всем модулям | `exports com.example.api;` |
| `exports ... to` | Экспорт конкретным модулям | `exports com.example.internal to com.example.other;` |

```java
exports com.example.mymodule.api;
exports com.example.mymodule.internal to com.example.trusted;
```

Пакет без `exports` недоступен из других модулей.

### opens

`opens` — открытие пакетов для рефлексии в рантайме.

| Директива | Для чего | Пример |
|-----------|----------|--------|
| `opens` | Рефлексия (все модули) | `opens com.example.model;` |
| `opens ... to` | Рефлексия (конкретные модули) | `opens com.example.model to com.fasterxml.jackson;` |

```java
opens com.example.mymodule.model to com.fasterxml.jackson.databind;
opens com.example.mymodule.entity to org.hibernate.orm.core;
```

Важно: `exports` ≠ `opens`.

- `exports` — компиляция + runtime доступ
- `opens` — только runtime рефлексия

### provides / uses — Service Provider Interface (SPI)

```java
provides com.example.myservice.PaymentService with com.example.mymodule.StripePaymentService;
uses com.example.myservice.PaymentService;
```

- `provides` — модуль предоставляет реализацию сервиса
- `uses` — модуль использует сервис

---

## Типы модулей

| Тип | Описание | Пример |
|-----|----------|--------|
| Named Module | Есть `module-info.java` | `module com.example.app { }` |
| Unnamed Module | Нет `module-info.java` (classpath) | Legacy код |
| Automatic Module | JAR без `module-info`, имя из манифеста | `requires com.fasterxml.jackson.databind;` |
| System Module | Модули JDK | `java.base`, `java.sql` |

---

## Инкапсуляция на уровне JVM

**Суть**
Инкапсуляция на уровне JVM — механизм, при котором виртуальная машина блокирует доступ к классам и пакетам, не экспортированным модулем, даже если они `public`. До Java 9 класс `public class Internal` в любом пакете был виден всем, и любой код мог выполнить `new Internal()`. После Java 9 JVM блокирует доступ на runtime: попытка доступа к неэкспортированному пакету приводит к `IllegalAccessError`, а через рефлексию — к `InaccessibleObjectException`.

Что проверяет JVM:

| Уровень | До Java 9 | После Java 9 (JPMS) |
|---------|-----------|---------------------|
| Класс | `public` — виден везде | `public` + пакет экспортирован |
| Пакет | Нет проверки | Должен быть в `exports` |
| Модуль | Нет понятия | Должен быть в `requires` |

Пример:

```java
// module-info.java модуля com.example.core
module com.example.core {
    exports com.example.core.api;
}
```

```java
// com/example/core/api/UserService.java (публичный)
package com.example.core.api;
public class UserService { }
```

```java
// com/example/core/internal/Helper.java (внутренний, но public)
package com.example.core.internal;
public class Helper { }
```

```java
// module-info.java модуля com.example.app
module com.example.app {
    requires com.example.core;
}
```

```java
// com/example/app/Main.java
package com.example.app;

import com.example.core.api.UserService;
import com.example.core.internal.Helper; // ❌ COMPILE ERROR!

public class Main {
    public static void main(String[] args) {
        UserService service = new UserService(); // ✅ OK
        Helper helper = new Helper();           // ❌ COMPILE ERROR!
    }
}
```

Зачем управлять видимостью пакетов:

| Причина | Описание | Пример |
|---------|----------|--------|
| Скрытие внутренней реализации | Можно менять код без breaking changes | Рефакторинг `internal` пакета |
| Защита от неправильного использования | Пользователи не зависят от внутренних классов | `Helper`, `Utils`, `Cache` |
| Безопасность | Ограничение доступа к чувствительному коду | Криптография, аутентификация |
| Явный контракт (API) | Чёткая граница между публичным и приватным | Документация + компилятор |
| Стабильность | Внутренние изменения не ломают клиентов | Версионирование API |

Чек-лист экспорта пакета:

- класс — часть публичного API библиотеки → ✅ `exports`
- класс используется только внутри модуля → ❌ не экспортировать
- нужна рефлексия (Jackson, Hibernate) → ✅ `opens` (не `exports`)
- планируется рефакторинг внутренней реализации → ❌ не экспортировать
- утилитный класс для внутреннего использования → ❌ не экспортировать

---

## Практический пример: библиотека с инкапсуляцией

Структура проекта:

- `module-info.java` — декларация модуля
- `com/example/api/` — экспортируется (публичный API): `HttpClient.java`, `HttpResponse.java`
- `com/example/internal/` — не экспортируется (внутренняя реализация): `ConnectionPool.java`, `RetryLogic.java`, `SSLHelper.java`
- `com/example/util/` — не экспортируется (внутренние утилиты): `StringUtils.java`

```java
// module-info.java
module com.example.mylib {
    exports com.example.api;
    requires java.net.http;
}
```

```java
// com/example/api/HttpClient.java (публичный API)
package com.example.api;

import com.example.internal.ConnectionPool; // ✅ Внутреннее использование OK

public class HttpClient {
    private final ConnectionPool pool = new ConnectionPool();

    public HttpResponse get(String url) {
        return pool.execute(url);
    }
}
```

```java
// com/example/internal/ConnectionPool.java (внутренняя реализация)
package com.example.internal;

public class ConnectionPool {
    public HttpResponse execute(String url) {
        // Внутренняя логика
    }
}
```

```java
// client-module/module-info.java
module com.example.client {
    requires com.example.mylib;
}
```

```java
// client-module/Main.java
package com.example.client;

import com.example.api.HttpClient;        // ✅ OK
import com.example.api.HttpResponse;      // ✅ OK
import com.example.internal.ConnectionPool; // ❌ COMPILE ERROR!

public class Main {
    public static void main(String[] args) {
        HttpClient client = new HttpClient(); // ✅ OK
        client.get("https://api.example.com"); // ✅ OK
    }
}
```

Преимущества такой архитектуры:

| Преимущество | Описание |
|--------------|----------|
| Рефакторинг без breaking changes | Можно менять `internal` пакет как угодно — клиенты не пострадают |
| Меньше зависимостей | Клиенты не зависят от внутренних классов |
| Чёткий контракт | `exports` — документация публичного API |
| Безопасность | Злоумышленник не получит доступ к внутренним методам |
| Производительность | JVM знает границы и лучше оптимизирует код |

---

## Практический пример: три модуля

### Модуль com.example.api

```java
// module-info.java
module com.example.api {
    exports com.example.api;
}
```

```java
// com/example/api/UserService.java
package com.example.api;

public interface UserService {
    User findById(Long id);
}
```

### Модуль com.example.service

```java
// module-info.java
module com.example.service {
    requires transitive com.example.api;
    exports com.example.service;
}
```

```java
// com/example/service/UserServiceImpl.java
package com.example.service;

import com.example.api.UserService;
import com.example.api.User;

public class UserServiceImpl implements UserService {
    public User findById(Long id) {
        return new User(id, "John");
    }
}
```

### Модуль com.example.app

```java
// module-info.java
module com.example.app {
    requires com.example.api;
    requires com.example.service;
}
```

```java
// com/example/app/Main.java
package com.example.app;

import com.example.api.UserService;
import com.example.service.UserServiceImpl;

public class Main {
    public static void main(String[] args) {
        UserService service = new UserServiceImpl();
        User user = service.findById(1L);
        System.out.println(user);
    }
}
```

---

## Компиляция и запуск

**Компиляция**

```bash
javac -d out/module1 src/module1/module-info.java src/module1/**/*.java
javac -d out/module2 --module-path out --module com.example.service src/module2/**/*.java
mvn clean install
```

**Запуск**

```bash
java --module-path out --module com.example.app/com.example.app.Main
java --module-path out --module com.example.app
```

---

## Поддержка Maven

```xml
<project>
    <properties>
        <maven.compiler.release>17</maven.compiler.release>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

Maven управляет артефактами, JPMS — модулями. Имя модуля (`com.example.api`) может не совпадать с `artifactId` (`api`); проверяйте `module-info.java` внутри JAR или документацию библиотеки.

---

## Распространённые ошибки

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `package ... declared in module ... does not read it` | Забыли `requires` | Добавить `requires com.example.api;` |
| `class ... is not public in exported package` | Класс не `public` | Сделать класс `public` или не экспортировать пакет |
| `Illegal reflective access` | Рефлексия без `opens` | Добавить `opens com.example.model to com.fasterxml.jackson.databind;` |
| `package ... defined in multiple modules` | Split package | Один пакет = один модуль |
| `module ... does not read module ...` | `requires` на пакет вместо модуля | Использовать имя модуля, а не пакета |

Запомните: `requires` — для имён модулей, `exports` — для имён пакетов.

---

## Миграция с Classpath на Module Path

| Шаг | Действие |
|-----|----------|
| 1 | Добавить `module-info.java` в каждый модуль |
| 2 | Объявить зависимости через `requires` |
| 3 | Экспортировать пакеты через `exports` |
| 4 | Открыть пакеты для рефлексии через `opens` |
| 5 | Перейти с `--classpath` на `--module-path` |
| 6 | Протестировать с `--illegal-access=deny` |

---

## Чек-лист внедрения JPMS

- большое приложение (100K+ строк) → ✅ стоит использовать
- нужна строгая инкапсуляция → ✅ стоит использовать
- кастомный JRE (jlink) → ✅ стоит использовать
- много legacy зависимостей → ⚠️ сложно
- маленький проект (< 10K строк) → ❌ не обязательно
- команда не знакома с модулями → ⚠️ нужно обучение

---

## Итог

| Вопрос | Ответ |
|--------|-------|
| Зачем нужен JPMS? | Инкапсуляция, явные зависимости, модульность |
| Что такое module-info.java? | Дескриптор модуля с зависимостями и экспортом |
| Что такое инкапсуляция на уровне JVM? | JVM блокирует доступ к неэкспортированным пакетам, даже если они `public` |
| Что видит клиент библиотеки? | Только экспортированные пакеты |
| Когда использовать? | Большие проекты, строгая архитектура |
| Можно ли без него? | ✅ Да (unnamed module) |
| Совместим с legacy? | ✅ Да (automatic modules) |

Совет: минимизируйте `exports` — чем меньше публичный API, тем проще поддерживать и развивать библиотеку без breaking changes.
