---
aliases:
  - Coordinated Universal Time
  - Date
  - DateTime
  - GMT
  - Greenwich Mean Time
  - Instant
  - java.sql.Date
  - java.sql.Timestamp
  - java.time
  - java.util.Date
  - LocalDate
  - LocalDateTime
  - LocalTime
  - Timestamp
  - UTC
  - ZonedDateTime
  - ZoneId
  - Всемирное координированное время
---
## Типы дат и времени в Java
> `java.time.Instant`

**Сравнение типов дат в Java:**

- `java.util.Date` (с Java 1.0) — устаревший класс, представляет конкретный момент времени (миллисекунды с 1970-01-01)
- `java.sql.Date` (с Java 1.1) — подкласс `java.util.Date`, только дата без времени
- `java.time.LocalDateTime` (с Java 8) — дата и время без временной зоны
- `java.time.Instant` (с Java 8) — момент времени на временной шкале (UTC)

### java.util.Date (с Java 1.0)

**Что это:** устаревший класс, представляет конкретный момент времени в миллисекундах с 1970-01-01.

```java
Date date = new Date(); // Текущее время
```

**Когда использовать:** избегать, только для легаси-кода.

### java.sql.Date (с Java 1.1)

**Что это:** подкласс `java.util.Date` для работы с SQL DATE (только дата, без времени).

```java
java.sql.Date sqlDate = java.sql.Date.valueOf("2023-12-25");
```

**Когда использовать:** только для JDBC при работе с полями SQL DATE.

### java.time.LocalDateTime (с Java 8)

**Что это:** дата и время без временной зоны.

```java
LocalDateTime ldt = LocalDateTime.now();
LocalDateTime specific = LocalDateTime.of(2023, 12, 25, 10, 30);
```

**Когда использовать:**

- Локальные события (встречи, праздники)
- Когда временная зона не важна
- Работа с датами без привязки к часовому поясу

### java.time.Instant (с Java 8)

**Что это:** момент времени на временной шкале (UTC).

```java
Instant now = Instant.now();
Instant specific = Instant.parse("2023-12-25T10:30:00Z");
```

**Когда использовать:**

- Таймстампы событий
- Логирование
- Работа с distributed systems
- Когда нужна точная точка во времени

---

**Сравнительная таблица**

| Тип | Временная зона | Точность | Использование |
|-----|----------------|----------|---------------|
| `java.util.Date` | ❌ Устаревший | Милисекунды | Избегать |
| `java.sql.Date` | ❌ Только дата | Дни | JDBC (SQL DATE) |
| `LocalDateTime` | ❌ Без зоны | Наносекунды | Локальные события |
| `Instant` | ✅ UTC | Наносекунды | Точные моменты времени |

---

**Рекомендации по использованию**

**Используйте java.time (Java 8+)**

```java
// Для локальных дат/времени
LocalDate date = LocalDate.now();
LocalTime time = LocalTime.now();
LocalDateTime dateTime = LocalDateTime.now();

// Для моментов времени
Instant instant = Instant.now();
ZonedDateTime zoned = ZonedDateTime.now();

// Для работы с БД
java.sql.Timestamp timestamp = java.sql.Timestamp.from(instant);
java.sql.Date sqlDate = java.sql.Date.valueOf(date);
```

**Избегайте java.util.Date**

```java
// ПЛОХО
Date oldDate = new Date();

// ХОРОШО
Instant instant = Instant.now();
```

---

**Примеры преобразований**

```java
// Instant ↔ LocalDateTime
Instant instant = Instant.now();
LocalDateTime ldt = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());

// LocalDateTime → java.sql.Timestamp
LocalDateTime ldt2 = LocalDateTime.now();
java.sql.Timestamp timestamp = java.sql.Timestamp.valueOf(ldt2);

// java.util.Date → Instant
Date utilDate = new Date();
Instant instant2 = utilDate.toInstant();
```

---

**Ключевые выводы**

1. Для новых проектов используйте только `java.time.*`.
2. Для точных моментов времени — `Instant`.
3. Для локальных дат — `LocalDate`, `LocalTime`, `LocalDateTime`.
4. Для JDBC — `java.sql.Date`, `Timestamp` только для взаимодействия с БД.
5. `java.util.Date` полностью устарел.

## Стандарт времени UTC (Coordinated Universal Time)

**Определение UTC**

UTC — это основной стандарт времени, по которому мир регулирует часы и время. Это преемник GMT (Greenwich Mean Time), но более точный.

**Ключевые характеристики:**

- ⏰ Базовый стандарт: основа для всех часовых поясов мира
- 🌍 Нулевой меридиан: отсчёт от Гринвичского меридиана (0° долготы)
- ⚡ Высокая точность: регулируется атомными часами с коррекцией високосных секунд
- 🚫 Без летнего времени: не подвержен сезонным изменениям

---

**Применение UTC**

**Единый стандарт для распределённых систем**

```java
// Все серверы в разных часовых поясах используют один стандарт
Instant server1Time = Instant.now(); // UTC
Instant server2Time = Instant.now(); // UTC
// Сравнение всегда корректно
```

**Избежание проблем с часовыми поясами**

```java
// ПРОБЛЕМА: разное время в разных зонах
LocalDateTime meetingTime = LocalDateTime.of(2023, 12, 25, 14, 0);
// В Нью-Йорке: 09:00 EST, в Лондоне: 14:00 GMT, в Москве: 17:00 MSK

// РЕШЕНИЕ: использовать UTC
Instant meetingTimeUTC = Instant.parse("2023-12-25T14:00:00Z");
// Все участники конвертируют в свой часовой пояс
```

**Корректное хранение в базах данных**

```java
// Все timestamp'ы в БД хранятся в UTC
@Entity
public class Event {
    @Column
    private Instant createdAt; // UTC — правильно

    // private LocalDateTime createdAt; // Может вызвать проблемы
}
```

**Логирование и аудит**

```java
// Логи с временными метками в UTC всегда согласованы
logger.info("Event occurred at: {}", Instant.now());
// Не важно, где расположен сервер, — время всегда понятно
```

---

**Практические примеры**

**Правильный подход (UTC) + конвертация**

```java
public class EventService {
    // Храним в UTC
    public void createEvent(String name) {
        Event event = new Event();
        event.setName(name);
        event.setCreatedAt(Instant.now()); // UTC

        // Для отображения конвертируем в локальную зону
        ZonedDateTime localTime = event.getCreatedAt()
            .atZone(ZoneId.of("Europe/Moscow"));
        System.out.println("Создано в: " + localTime);
    }
}
```

**Проблемный подход (только локальное время)**

```java
public class ProblematicEventService {
    public void createEvent(String name) {
        Event event = new Event();
        event.setName(name);
        event.setCreatedAt(LocalDateTime.now()); // Чья зона? Сервера? Пользователя?

        // При переносе сервера в другую зону все времена «съедут»
    }
}
```

---

**Преимущества UTC**

| Аспект | Преимущество |
|--------|--------------|
| Согласованность | Единое время для всех систем |
| Надёжность | Нет зависимости от локации сервера |
| Сравнение | Легко сравнивать времена из разных источников |
| Миграция | Перенос серверов между зонами без проблем |
| Отладка | Упрощает анализ логов и ошибок |

---

**Использование UTC**

1. Хранение в БД: все timestamp'ы.
2. API: обмен данными между системами.
3. Логирование: временные метки в логах.
4. Распределённые системы: микросервисы, очереди.
5. Международные приложения: пользователи в разных зонах.

---

**Использование локального времени**

1. UI: отображение пользователю.
2. Напоминания: локальные события.
3. Расписания: рабочие часы, встречи.
4. Отчёты: в контексте локации пользователя.

---

**Золотое правило**

Хранить в UTC, отображать в локальном времени.

```java
// Храним
Instant storedTime = Instant.now();

// Отображаем пользователю
ZonedDateTime userTime = storedTime.atZone(userTimeZone);
```

UTC — это фундамент для построения надёжных временных систем в программировании. 🕐
