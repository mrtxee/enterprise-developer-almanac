---
aliases:
  - module-info
  - module-info.java
---
## 🧩 Java Platform Module System (JPMS) — Java 9+

### 📌 Краткий ответ

> **JPMS (Project Jigsaw)** — система модулей в Java, которая добавляет **инкапсуляцию на уровне пакетов** и **явные зависимости** между частями приложения.  
> **`module-info.java`** — дескриптор модуля, который объявляет его имя, зависимости и экспортируемые пакеты.

---

## 🎯 Зачем нужен JPMS?

| Проблема до Java 9 | Решение в Java 9+ |
|--------------------|-------------------|
| **Classpath hell** — конфликты версий | ✅ **Module Path** — явные зависимости |
| **Нет инкапсуляции** — `public` виден везде | ✅ **Exports** — пакеты видны только явно |
| **Большой runtime** — весь JDK всегда | ✅ **JLink** — кастомный runtime |
| **Нет мета-информации** | ✅ **module-info.java** — декларативные зависимости |

---

## 📁 Структура модуля

```
my-module/
├── src/
│   └── com.example.mymodule/
│       ├── module-info.java    ← Дескриптор модуля
│       └── MyClass.java
└── bin/
```

---

## 📄 module-info.java — Синтаксис

```java
module com.example.mymodule {
    // 1. Зависимости от других модулей
    requires java.sql;
    requires transitive java.logging;
    requires static com.fasterxml.jackson.databind;
    
    // 2. Экспорт пакетов (публичный API)
    exports com.example.mymodule.api;
    exports com.example.mymodule.service to com.example.othermodule;
    
    // 3. Открытые пакеты (для рефлексии)
    opens com.example.mymodule.internal;
    opens com.example.mymodule.model to com.fasterxml.jackson.databind;
    
    // 4. Предоставление услуг (Service Provider)
    provides com.example.myservice.MyService 
        with com.example.mymodule.MyServiceImpl;
    
    // 5. Использование услуг (Service Consumer)
    uses com.example.myservice.MyService;
}
```

---

## 🔑 Ключевые директивы

### 1. **`requires`** — Зависимость от модуля

| Модификатор | Описание | Пример |
|-------------|----------|--------|
| `requires` | Обязательная зависимость | `requires java.sql;` |
| `requires transitive` | Зависимость передаётся дальше | `requires transitive java.logging;` |
| `requires static` | Опциональная (compile-time) | `requires static lombok;` |

```java
// ✅ Базовая зависимость
requires java.sql;

// ✅ Транзитивная (клиенты этого модуля тоже получат зависимость)
requires transitive java.logging;

// ✅ Опциональная (нужна только при компиляции)
requires static lombok;
```

---

### 2. **`exports`** — Экспорт пакетов

| Модификатор | Описание | Пример |
|-------------|----------|--------|
| `exports` | Экспорт всем модулям | `exports com.example.api;` |
| `exports ... to` | Экспорт конкретным модулям | `exports com.example.internal to com.example.other;` |

```java
// ✅ Публичный API — виден всем
exports com.example.mymodule.api;

// ✅ Внутренний пакет — виден только указанному модулю
exports com.example.mymodule.internal to com.example.trusted;

// ❌ Пакет не экспортирован — недоступен из других модулей
// com.example.mymodule.private (нет exports)
```

---

### 3. **`opens`** — Открытие для рефлексии

| Директива | Для чего | Пример |
|-----------|----------|--------|
| `opens` | Рефлексия (все модули) | `opens com.example.model;` |
| `opens ... to` | Рефлексия (конкретные модули) | `opens com.example.model to com.fasterxml.jackson;` |

```java
// ✅ Для фреймворков (Hibernate, Jackson, Spring)
opens com.example.mymodule.model to com.fasterxml.jackson.databind;
opens com.example.mymodule.entity to org.hibernate.orm.core;

// ⚠️ Важно: exports ≠ opens
// exports — компиляция + runtime доступ
// opens — только runtime рефлексия
```

---

### 4. **`provides` / `uses`** — Service Provider Interface (SPI)

```java
// ✅ Модуль предоставляет реализацию
provides com.example.myservice.PaymentService 
    with com.example.mymodule.StripePaymentService;

// ✅ Модуль использует сервис
uses com.example.myservice.PaymentService;
```

---

## 📊 Типы модулей

| Тип | Описание | Пример |
|-----|----------|--------|
| **Named Module** | Есть `module-info.java` | `module com.example.app { }` |
| **Unnamed Module** | Нет `module-info.java` (классpath) | Legacy код |
| **Automatic Module** | JAR без `module-info`, имя из манифеста | `requires com.fasterxml.jackson.databind;` |
| **System Module** | Модули JDK | `java.base`, `java.sql` |

---

## 🧪 Практический пример

### Модуль 1: `com.example.api`

```java
// module-info.java
module com.example.api {
    exports com.example.api;
}

// com/example/api/UserService.java
package com.example.api;

public interface UserService {
    User findById(Long id);
}
```

### Модуль 2: `com.example.service`

```java
// module-info.java
module com.example.service {
    requires transitive com.example.api;
    exports com.example.service;
}

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

### Модуль 3: `com.example.app`

```java
// module-info.java
module com.example.app {
    requires com.example.api;
    requires com.example.service;
}

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

## 🛠️ Компиляция и запуск

### Компиляция

```bash
# Скомпилировать модуль
javac -d out/module1 src/module1/module-info.java src/module1/**/*.java
javac -d out/module2 --module-path out --module com.example.service src/module2/**/*.java

# Или через Maven/Gradle (автоматически)
mvn clean install
```

### Запуск

```bash
# Запуск модуля
java --module-path out --module com.example.app/com.example.app.Main

# С указанием главного класса в module-info
java --module-path out --module com.example.app
```

---

## 📦 Maven поддержка

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

---

## ⚠️ Распространённые ошибки

### 1. **Package belongs to another module**

```
Error: package com.example.api is declared in module com.example.api, 
       but module com.example.service does not read it
```

✅ **Решение:** Добавить `requires com.example.api;`

### 2. **Class is not public in exported package**

```
Error: class com.example.api.Internal is not public
```

✅ **Решение:** Сделать класс `public` или не экспортировать пакет

### 3. **Illegal reflective access**

```
Warning: Illegal reflective access by com.fasterxml.jackson
```

✅ **Решение:** Добавить `opens com.example.model to com.fasterxml.jackson.databind;`

### 4. **Split package**

```
Error: package com.example.api defined in multiple modules
```

✅ **Решение:** Один пакет = один модуль (нельзя дублировать)

---

## 🔄 Миграция с Classpath на Module Path

| Шаг | Действие |
|-----|----------|
| 1 | Добавить `module-info.java` в каждый модуль |
| 2 | Объявить зависимости через `requires` |
| 3 | Экспортировать пакеты через `exports` |
| 4 | Открыть пакеты для рефлексии через `opens` |
| 5 | Перейти с `--classpath` на `--module-path` |
| 6 | Протестировать с `--illegal-access=deny` |

---

## 📋 Чек-лист: использовать ли JPMS?

```
□ Приложение большое (100K+ строк)? → ✅ Да
□ Нужна строгая инкапсуляция? → ✅ Да
□ Кастомный JRE (jlink)? → ✅ Да
□ Много legacy зависимостей? → ⚠️ Сложно
□ Маленький проект (< 10K строк)? → ❌ Не обязательно
□ Команда не знакома с модулями? → ⚠️ Обучение нужно
```

---

## 📌 Памятка

```
┌─────────────────────────────────────────────────────────────┐
│  JPMS (Java Platform Module System)                         │
│                                                             │
│  requires           → Зависимость от модуля                 │
│  requires transitive→ Зависимость передаётся дальше        │
│  requires static    → Опциональная зависимость              │
│                                                             │
│  exports            → Пакет виден всем                      │
│  exports ... to     → Пакет виден указанным модулям         │
│                                                             │
│  opens              → Рефлексия (все модули)                │
│  opens ... to       → Рефлексия (конкретные модули)         │
│                                                             │
│  provides / uses    → Service Provider Interface            │
│                                                             │
│  ✅ Инкапсуляция, явные зависимости, кастомный JRE          │
│  ⚠️ Сложность миграции, split package проблема             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Итог

| Вопрос                          | Ответ                                         |
| ------------------------------- | --------------------------------------------- |
| **Зачем нужен JPMS?**           | Инкапсуляция, явные зависимости, модульность  |
| **Что такое module-info.java?** | Дескриптор модуля с зависимостями и экспортом |
| **Когда использовать?**         | Большие проекты, строгая архитектура          |
| **Можно ли без него?**          | ✅ Да (unnamed module)                         |
| **Совместим с legacy?**         | ✅ Да (automatic modules)                      |

> 💡 **Совет:** Для новых больших проектов используйте **JPMS**. Для маленьких или legacy — можно отложить, но изучите для понимания экосистемы Java 9+.

## 1️⃣ Что является модулем в контексте JPMS?

### 📌 Определение

> **Модуль** — это **группа пакетов + `module-info.java`**, которая явно объявляет:
> - Своё имя
> - От каких модулей зависит
> - Какие пакеты экспортирует наружу

### 📁 Структура модуля

```
com.example.mymodule/
├── module-info.java          ← Дескриптор модуля (обязательно)
├── com/
│   └── example/
│       ├── api/              ← Экспортируемый пакет (публичный API)
│       │   └── UserService.java
│       ├── internal/         ← Внутренний пакет (не экспортируется)
│       │   └── Helper.java
│       └── Main.java
```

### 📄 module-info.java

```java
module com.example.mymodule {
    // Зависимости
    requires java.sql;
    
    // Экспорт (публичный API)
    exports com.example.api;
    
    // Внутренний пакет НЕ экспортируется → инкапсуляция
    // com.example.internal доступен только внутри модуля
}
```

### ✅ Ключевые свойства модуля

| Свойство | Описание |
|----------|----------|
| **Именованный** | У модуля есть уникальное имя (`com.example.mymodule`) |
| **С инкапсуляцией** | Только экспортированные пакеты видны снаружи |
| **С зависимостями** | Явно объявляет, от каких модулей зависит |
| **С границами** | JVM проверяет доступ на уровне модулей (не только классов) |

---

## 2️⃣ Что такое инкапсуляция на уровне JVM?

### 📌 Определение

> **Инкапсуляция на уровне JVM** — это механизм, при котором **виртуальная машина блокирует доступ** к классам/пакетам, которые не экспортированы модулем, **даже если они `public`**.

### 🔄 Сравнение: До Java 9 vs После Java 9

```
┌─────────────────────────────────────────────────────────────┐
│  ДО Java 9 (Classpath)                                      │
│                                                             │
│  public class Internal { }  ← виден ВСЕМ                    │
│  (даже если в "внутреннем" пакете)                          │
│                                                             │
│  ❌ Нет защиты на уровне JVM                                │
│  ❌ Любой код может сделать new Internal()                  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  ПОСЛЕ Java 9 (Module Path)                                 │
│                                                             │
│  public class Internal { }  ← виден ТОЛЬКО внутри модуля    │
│  (если пакет не экспортирован)                              │
│                                                             │
│  ✅ JVM блокирует доступ на runtime                         │
│  ❌ new Internal() → IllegalAccessError                     │
└─────────────────────────────────────────────────────────────┘
```

### 🧪 Пример инкапсуляции

#### Модуль A: `com.example.core`

```java
// module-info.java
module com.example.core {
    exports com.example.core.api;  // ✅ Публичный API
    // ❌ com.example.core.internal НЕ экспортируется
}

// com/example/core/api/UserService.java (публичный)
package com.example.core.api;
public class UserService { }

// com/example/core/internal/Helper.java (внутренний)
package com.example.core.internal;
public class Helper { }  // ← public, но пакет не экспортирован!
```

#### Модуль B: `com.example.app`

```java
// module-info.java
module com.example.app {
    requires com.example.core;
}

// com/example/app/Main.java
package com.example.app;

import com.example.core.api.UserService;      // ✅ OK
import com.example.core.internal.Helper;      // ❌ COMPILE ERROR!

public class Main {
    public static void main(String[] args) {
        UserService service = new UserService();  // ✅ OK
        Helper helper = new Helper();             // ❌ COMPILE ERROR!
    }
}
```

### 🔒 Что проверяет JVM?

| Уровень | До Java 9 | После Java 9 (JPMS) |
|---------|-----------|---------------------|
| **Класс** | `public` = виден везде | `public` + пакет экспортирован |
| **Пакет** | Нет проверки | Должен быть в `exports` |
| **Модуль** | Нет понятия | Должен быть в `requires` |

---

## 3️⃣ Зачем управлять видимостью пакетов внутри JAR?

### 🎯 Причины

| Причина | Описание | Пример |
|---------|----------|--------|
| **1. Скрытие внутренней реализации** | Можно менять код без breaking changes | Рефакторинг `internal` пакета |
| **2. Защита от неправильного использования** | Пользователи не зависят от внутренних классов | `Helper`, `Utils`, `Cache` |
| **3. Безопасность** | Ограничение доступа к чувствительному коду | Криптография, аутентификация |
| **4. Явный контракт (API)** | Чёткая граница между публичным и приватным | Документация + компилятор |
| **5. Стабильность** | Внутренние изменения не ломают клиентов | Версионирование API |

---

## 🧪 Практический пример: Библиотека с инкапсуляцией

### Структура проекта

```
mylib/
├── module-info.java
└── com/
    └── example/
        ├── api/              ← Экспортируется (публичный API)
        │   ├── HttpClient.java
        │   └── HttpResponse.java
        ├── internal/         ← НЕ экспортируется (внутренняя реализация)
        │   ├── ConnectionPool.java
        │   ├── RetryLogic.java
        │   └── SSLHelper.java
        └── util/             ← НЕ экспортируется (внутренние утилиты)
            └── StringUtils.java
```

### module-info.java

```java
module com.example.mylib {
    // ✅ Экспортируем только публичный API
    exports com.example.api;
    
    // ❌ com.example.internal и com.example.util НЕ экспортируются
    
    // Зависимости
    requires java.net.http;
}
```

### Публичный API (виден клиентам)

```java
// com/example/api/HttpClient.java
package com.example.api;

import com.example.internal.ConnectionPool;  // ✅ Внутреннее использование OK

public class HttpClient {
    private final ConnectionPool pool = new ConnectionPool();  // ✅
    
    public HttpResponse get(String url) {
        // Клиенты видят только этот метод
        return pool.execute(url);
    }
}
```

### Внутренняя реализация (скрыта от клиентов)

```java
// com/example/internal/ConnectionPool.java
package com.example.internal;

public class ConnectionPool {
    // ❌ Клиенты НЕ могут создать new ConnectionPool()
    // ❌ Клиенты НЕ могут вызвать методы этого класса
    
    public HttpResponse execute(String url) {
        // Внутренняя логика
    }
}
```

### Код клиента (что видят пользователи библиотеки)

```java
// client-module/module-info.java
module com.example.client {
    requires com.example.mylib;
}

// client-module/Main.java
package com.example.client;

import com.example.api.HttpClient;        // ✅ OK
import com.example.api.HttpResponse;      // ✅ OK
import com.example.internal.ConnectionPool; // ❌ COMPILE ERROR!

public class Main {
    public static void main(String[] args) {
        HttpClient client = new HttpClient();  // ✅ OK
        client.get("https://api.example.com"); // ✅ OK
        
        // ❌ Нельзя получить доступ к внутренностям:
        // ConnectionPool pool = new ConnectionPool(); // COMPILE ERROR!
    }
}
```

---

## 📊 Преимущества такой архитектуры

| Преимущество | Описание |
|--------------|----------|
| **Рефакторинг без breaking changes** | Можно менять `internal` пакет как угодно — клиенты не пострадают |
| **Меньше зависимостей** | Клиенты не зависят от внутренних классов |
| **Чёткий контракт** | `exports` = документация публичного API |
| **Безопасность** | Злоумышленник не получит доступ к внутренним методам |
| **Производительность** | JVM может оптимизировать код лучше (знает границы) |

---

## ⚠️ Что будет при нарушении инкапсуляции?

### Ошибка компиляции

```java
// Client.java
import com.example.internal.Helper; // ❌ Error: package is not visible

public class Client {
    Helper h = new Helper(); // ❌ Error: cannot access Helper
}
```

```
Error: package com.example.internal is declared in module com.example.mylib,
       which does not export it to module com.example.client
```

### Ошибка runtime (через рефлексию)

```java
// Попытка доступа через рефлексию
Class<?> clazz = Class.forName("com.example.internal.Helper");
// ❌ InaccessibleObjectException (Java 9+)
```

---

## 📋 Чек-лист: когда экспортировать пакет?

```
□ Этот класс — часть публичного API библиотеки? → ✅ exports
□ Этот класс используется только внутри модуля? → ❌ Не экспортировать
□ Нужна рефлексия (Jackson, Hibernate)? → ✅ opens (не exports)
□ Планируется рефакторинг внутренней реализации? → ❌ Не экспортировать
□ Это утилитный класс для внутреннего использования? → ❌ Не экспортировать
```

---

## 📌 Памятка

```
┌─────────────────────────────────────────────────────────────┐
│  JPMS Модуль                                                │
│                                                             │
│  Модуль = пакеты + module-info.java                         │
│                                                             │
│  Инкапсуляция на уровне JVM:                                │
│  • public + exports = виден                                 │
│  • public + нет exports = НЕ виден (JVM блокирует)          │
│                                                             │
│  Зачем управлять видимостью:                                │
│  • Скрыть внутреннюю реализацию                             │
│  • Защитить от неправильного использования                  │
│  • Рефакторинг без breaking changes                         │
│  • Чёткий публичный контракт (API)                          │
│                                                             │
│  ✅ exports  → компиляция + runtime доступ                  │
│  ✅ opens    → только runtime рефлексия                     │
│  ❌ ничего   → полностью инкапсулировано                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Итог

| Вопрос | Ответ |
|--------|-------|
| **Что такое модуль JPMS?** | Группа пакетов + `module-info.java` с явными зависимостями и экспортом |
| **Что такое инкапсуляция на уровне JVM?** | JVM блокирует доступ к неэкспортированным пакетам (даже если `public`) |
| **Зачем управлять видимостью?** | Скрыть реализацию, защитить API, позволить рефакторинг без breaking changes |
| **Что видит клиент библиотеки?** | Только экспортированные пакеты |
| **Можно ли обойти через рефлексию?** | ❌ Нет (Java 9+ блокирует `InaccessibleObjectException`) |

> 💡 **Совет:** Проектируйте модули так, чтобы **минимизировать `exports`**. Чем меньше публичный API — тем проще поддерживать и развивать библиотеку без breaking changes.


## 🎯 Зачем нужны `requires ...` в `module-info.java`?

### 📌 Краткий ответ

> **`requires`** — это **явное объявление зависимостей модуля**.  
> Оно говорит компилятору и JVM: *"Этот модуль нуждается в другом модуле для работы"*.  
> **Без `requires` вы не сможете использовать классы из других модулей**, даже если JAR-файл есть в `module-path`.

---

## 🧩 Что даёт `requires`?

| Преимущество | Описание |
|--------------|----------|
| **✅ Явные зависимости** | Видно, от чего зависит модуль (не нужно гадать) |
| **✅ Проверка на компиляцию** | Компилятор ошибётся, если забыли `requires` |
| **✅ Проверка на runtime** | JVM упадёт, если модуль не найден (fail-fast) |
| **✅ Инкапсуляция** | Модуль видит только то, что явно заявил |
| **✅ Надёжная сборка** | Нет "скрытых" зависимостей через classpath |

---

## 📊 Сравнение: Classpath (до Java 9) vs Module Path (Java 9+)

### ❌ Classpath: "Всё видно всем"

```
┌─────────────────────────────────────────────────────────────┐
│  Приложение                                                 │
│  • Все JAR в classpath                                      │
│  • Любой класс из любого JAR виден                          │
│  • Компилятор не проверяет, что реально используется        │
│  • Риск: случайное использование внутреннего API            │
│  • Риск: "JAR hell" — конфликты версий                      │
└─────────────────────────────────────────────────────────────┘
```

### ✅ Module Path: "Только то, что явно объявлено"

```
┌─────────────────────────────────────────────────────────────┐
│  Модуль com.example.app                                     │
│  module-info.java:                                          │
│  requires com.example.api;  ← ЯВНАЯ ЗАВИСИМОСТЬ            │
│                                                             │
│  • Компилятор: проверит, что com.example.api доступен       │
│  • JVM: проверит, что модуль есть в module-path            │
│  • Инкапсуляция: видны только экспортированные пакеты       │
│  • Надёжность: нет скрытых зависимостей                     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧪 Практический пример

### Модуль `com.example.api`

```java
// module-info.java
module com.example.api {
    exports com.example.api;
}

// com/example/api/UserService.java
package com.example.api;
public class UserService {
    public String findUser(Long id) { return "User-" + id; }
}
```

### Модуль `com.example.app` — **БЕЗ `requires`** (ошибка)

```java
// module-info.java
module com.example.app {
    // ❌ Нет requires com.example.api!
}

// com/example/app/Main.java
package com.example.app;
import com.example.api.UserService;  // ❌ COMPILE ERROR!

public class Main {
    public static void main(String[] args) {
        UserService service = new UserService();  // ❌ Не скомпилируется
    }
}
```

```
Error: module com.example.app does not read module com.example.api
```

### Модуль `com.example.app` — **С `requires`** (правильно)

```java
// module-info.java
module com.example.app {
    requires com.example.api;  // ✅ Явная зависимость
}

// com/example/app/Main.java
package com.example.app;
import com.example.api.UserService;  // ✅ OK

public class Main {
    public static void main(String[] args) {
        UserService service = new UserService();  // ✅ Работает
        System.out.println(service.findUser(1L));
    }
}
```

---

## 🔑 Варианты `requires`

| Директива | Описание | Пример использования |
|-----------|----------|---------------------|
| **`requires`** | Обязательная зависимость (compile + runtime) | `requires java.sql;` |
| **`requires transitive`** | Зависимость передаётся клиентам модуля | `requires transitive com.example.api;` |
| **`requires static`** | Опциональная зависимость (только компиляция) | `requires static lombok;` |

---

### 1. **`requires` (базовый)**

```java
module com.example.app {
    requires java.sql;  // ✅ Нужен для компиляции и запуска
}
```

> Модуль `com.example.app` зависит от `java.sql`.  
> Без `java.sql` в module-path → **ошибка компиляции или runtime**.

---

### 2. **`requires transitive` (транзитивная зависимость)**

```java
// Модуль com.example.service
module com.example.service {
    requires transitive com.example.api;  // ✅ Передаётся дальше
    
    exports com.example.service;
}
```

```java
// Модуль com.example.app
module com.example.app {
    requires com.example.service;
    // ✅ Автоматически "читает" com.example.api (транзитивно)
}

// com/example/app/Main.java
import com.example.api.UserService;  // ✅ OK, хотя app не требует api напрямую
```

> 🎯 **Зачем:** Когда ваш модуль — это "обёртка" над другим модулем, и вы хотите, чтобы клиенты могли использовать его API без явного `requires`.

---

### 3. **`requires static` (опциональная зависимость)**

```java
module com.example.app {
    requires static lombok;  // ✅ Нужен только при компиляции
    requires com.example.api; // ✅ Нужен всегда
}
```

> 🎯 **Зачем:** Для аннотационных процессоров (Lombok, AutoValue), которые нужны только при компиляции, но не при запуске.

---

## ⚙️ Что происходит "под капотом"?

### При компиляции (`javac`)

```
1. Компилятор читает module-info.java
2. Видит: requires com.example.api
3. Ищет модуль com.example.api в module-path
4. Проверяет: экспортирует ли он пакет com.example.api?
5. Если всё ОК → разрешает импорт классов из этого пакета
6. Если нет → ошибка компиляции
```

### При запуске (`java`)

```
1. JVM загружает модуль com.example.app
2. Видит: requires com.example.api
3. Ищет модуль com.example.api в module-path
4. Если не найден → запускается с ошибкой
5. Если найден → создаёт "read edge" между модулями
```

---

## 📋 Когда что использовать?

| Сценарий | Директива | Почему |
|----------|-----------|--------|
| **Обычная зависимость** | `requires` | Нужна и при компиляции, и при запуске |
| **Публичный API модуля** | `requires transitive` | Клиенты должны "видеть" эту зависимость |
| **Аннотационный процессор** | `requires static` | Нужен только при компиляции |
| **Тестовая зависимость** | `requires static` (в тестовом модуле) | Не нужна в продакшене |

---

## ⚠️ Распространённые ошибки

### 1. **Забыли `requires`**

```java
// module-info.java
module com.example.app {
    // ❌ Нет requires java.sql
}

// Код
import java.sql.Connection;  // ❌ Error: module does not read java.sql
```

✅ **Решение:** Добавить `requires java.sql;`

---

### 2. **Путают `requires` и `exports`**

```java
// ❌ Неправильно
module com.example.app {
    requires com.example.api;  // ✅ Зависимость
    requires com.example.app.internal;  // ❌ Внутренний пакет не модуль!
}
```

✅ **Правильно:**

```java
module com.example.app {
    requires com.example.api;  // ✅ Зависимость от модуля
    // Внутренние пакеты не требуют декларации в requires
}
```

---

### 3. **`requires` на пакет, а не модуль**

```java
// ❌ Неправильно
requires com.example.api;  // Если com.example.api — это пакет, а не модуль!

// ✅ Правильно
requires com.example.apimodule;  // Имя модуля, а не пакета
```

> 💡 **Запомните:** `requires` — для **имён модулей**, `exports` — для **имён пакетов**.

---

## 🔄 Как это связано с Maven/Gradle?

```xml
<!-- pom.xml: Maven управляет артефактами -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>api</artifactId>
    <version>1.0</version>
</dependency>
```

```java
// module-info.java: JPMS управляет модулями
module com.example.app {
    requires com.example.api;  // Имя модуля (может отличаться от artifactId!)
}
```

> ⚠️ **Важно:** Имя модуля (`com.example.api`) может **не совпадать** с `artifactId` (`api`).  
> Проверьте `module-info.java` внутри JAR или документацию библиотеки.

---

## 📌 Памятка

```
┌─────────────────────────────────────────────────────────────┐
│  requires в module-info.java                                │
│                                                             │
│  Зачем:                                                     │
│  • Явно объявить зависимости модуля                         │
│  • Включить проверку на компиляцию и runtime                │
│  • Обеспечить инкапсуляцию и надёжность                     │
│                                                             │
│  Варианты:                                                  │
│  • requires           → обязательная (compile+runtime)      │
│  • requires transitive → передаётся клиентам модуля         │
│  • requires static    → только компиляция                   │
│                                                             │
│  ❌ Без requires → классы из других модулей недоступны      │
│  ✅ С requires → компилятор и JVM проверяют доступность     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Итог

| Вопрос | Ответ |
|--------|-------|
| **Зачем нужен `requires`?** | Явно объявить зависимости модуля для компиляции и runtime |
| **Что будет без `requires`?** | ❌ Ошибка компиляции: "модуль не читает другой модуль" |
| **В чём разница `requires` vs `exports`?** | `requires` — зависимости, `exports` — публичный API |
| **Когда использовать `transitive`?** | Когда зависимость должна быть видна клиентам вашего модуля |
| **Когда использовать `static`?** | Для опциональных зависимостей (аннотационные процессоры) |

> 💡 **Совет:** Всегда явно объявляйте зависимости через `requires`. Это делает ваш код **надёжнее**, **понятнее** и **легче в поддержке**. Если компилятор ругается на неизвестный класс — первым делом проверьте `module-info.java`.