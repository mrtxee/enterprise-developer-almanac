---
aliases:
  - Offline Concurrency
  - Implicit Lock
  - Coarse Grained Lock
  - Pessimistic Lock
  - Optimistic Lock
---
Offline Concurrency by [[PoEAA]]
1. Optimistic Lock
2. Pessimistic Lock
3. Coarse Grained Lock
4. Implicit Lock
---
## 🔐 Паттерны конкурентности из «[[PoEAA|Patterns of Enterprise Application Architecture]]» (Martin Fowler)

В книге **POEAA** Мартин Фаулер описывает паттерны для решения **проблемы конкурентного доступа к данным** в корпоративных приложениях. Разберём каждый паттерн подробно.

---

## 📚 Контекст: Проблема конкурентности

### Сценарий «Параллельное Чтение-Изменение-Запись»:

```
Пользователь A: Читает заказ #123 (статус = "Новый")
Пользователь B: Читает заказ #123 (статус = "Новый")
Пользователь A: Меняет статус на "В обработке" → Сохраняет
Пользователь B: Меняет статус на "Отменён" → Сохраняет ← ПЕРЕЗАПИСЫВАЕТ изменения А!
```

→ **Проблема**: Пользователь B не знал, что данные изменились после его чтения.

---

## 1️⃣ Offline Concurrency (Конкурентность вне соединения)

### 📌 Что это?

**Общий паттерн** для решения конкурентности, когда объекты:
1. Загружаются из БД
2. Работают с ними **вне соединения** (в памяти, возможно долго)
3. Сохраняются обратно в БД

> 💡 **Ключевая идея**:  
> *«Мы не держим соединение с БД открытым во время работы с объектом. Как предотвратить конфликты при сохранении?»*

---

### 🏗️ Решения для Offline Concurrency:

| Паттерн                                           | Когда использовать                           | Сложность |
| ------------------------------------------------- | -------------------------------------------- | --------- |
| **[Optimistic Lock](#2-optimistic-lock)**         | Редкие конфликты, высокая производительность | Низкая    |
| **[Pessimistic Lock](#3-pessimistic-lock)**       | Частые конфликты, критичные данные           | Высокая   |
| **[Coarse Grained Lock](#4-coarse-grained-lock)** | Работа с агрегатами объектов                 | Средняя   |

---

### 💻 Пример проблемы:

```java
// Пользователь A
Order orderA = orderRepository.findById(123L);
orderA.setStatus("PROCESSING");
// ... пользователь думает 5 минут ...
orderRepository.save(orderA); // Сохраняет успешно

// Пользователь B (параллельно)
Order orderB = orderRepository.findById(123L);
orderB.setStatus("CANCELLED");
// ... пользователь думает 3 минуты ...
orderRepository.save(orderB); // ПЕРЕЗАПИСЫВАЕТ изменения А!
```

---

## 2️⃣ Optimistic Lock (Оптимистичная блокировка)

### 📌 Что это?

Проверка конфликтов **только при сохранении**. Предполагаем, что конфликты редки, и проверяем их в момент записи.

> 💡 **Аналогия**:  
> *«Я редактирую документ в Google Docs. Если кто-то изменил его до меня — система предупредит при сохранении.»*

---

### 🏗️ Как работает:

```mermaid
sequenceDiagram
    participant UserA as Пользователь A
    participant UserB as Пользователь B
    participant DB as База данных
    
    UserA->>DB: SELECT * FROM orders WHERE id=123<br/>version=1
    UserB->>DB: SELECT * FROM orders WHERE id=123<br/>version=1
    
    UserA->>UserA: Изменяет данные
    UserA->>DB: UPDATE orders SET status='PROCESSING',<br/>version=2 WHERE id=123 AND version=1
    Note over DB: ✅ Успешно (версия совпала)
    
    UserB->>UserB: Изменяет данные
    UserB->>DB: UPDATE orders SET status='CANCELLED',<br/>version=2 WHERE id=123 AND version=1
    Note over DB: ❌ Ошибка! (версия теперь = 2)
    DB-->>UserB: OptimisticLockException
```

---

### 💻 Реализация на Java (JPA/Hibernate):

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private Long id;
    
    private String status;
    
    @Version // ← Ключевая аннотация!
    private Integer version;
    
    // getters/setters...
}

// Использование:
try {
    Order order = entityManager.find(Order.class, 123L);
    order.setStatus("PROCESSING");
    entityManager.merge(order); // Автоматически проверит версию
} catch (OptimisticLockException e) {
    // Обработка конфликта
    System.out.println("Данные были изменены другим пользователем!");
}
```

---

### 💻 Реализация на SQL:

```sql
-- Таблица с версией
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    status VARCHAR(50),
    version INT NOT NULL DEFAULT 0
);

-- Чтение
SELECT id, status, version FROM orders WHERE id = 123;
-- Результат: id=123, status='NEW', version=1

-- Обновление с проверкой версии
UPDATE orders 
SET status = 'PROCESSING', version = version + 1 
WHERE id = 123 AND version = 1; -- ← Ключевое условие!

-- Если версия изменилась, вернёт 0 обновлённых строк
SELECT ROW_COUNT(); -- 0 = конфликт!
```

---

### ✅ Преимущества Optimistic Lock:

| Преимущество | Объяснение |
|--------------|------------|
| **Высокая производительность** | Нет блокировок при чтении |
| **Масштабируемость** | Подходит для высоконагруженных систем |
| **Простота** | Легко реализовать (аннотация `@Version`) |
| **Отсутствие взаимоблокировок** | Нет риска deadlock |

---

### ❌ Недостатки Optimistic Lock:

| Недостаток | Объяснение |
|------------|------------|
| **Конфликты обнаруживаются поздно** | Только при сохранении |
| **Требует обработки ошибок** | Нужно реализовать логику разрешения конфликтов |
| **Не подходит для частых конфликтов** | Постоянные ошибки раздражают пользователей |

---

### 🏥 Когда использовать:

| Сценарий | Рекомендация |
|----------|--------------|
| **Веб-приложения** | ✅ Идеально (короткие сессии) |
| **Редактирование документов** | ✅ Подходит (редкие конфликты) |
| **Финансовые операции** | ❌ Лучше Pessimistic |
| **Высокая конкуренция** | ❌ Частые конфликты = плохой UX |

---

## 3️⃣ Pessimistic Lock (Пессимистичная блокировка)

### 📌 Что это?

**Блокировка данных при чтении**. Другие процессы не могут читать/писать эти данные до тех пор, пока блокировка не будет снята.

> 💡 **Аналогия**:  
> *«Я беру книгу в библиотеке и кладу на неё табличку "Занято". Никто другой не может её взять, пока я не верну.»*

---

### 🏗️ Как работает:

```mermaid
sequenceDiagram
    participant UserA as Пользователь A
    participant UserB as Пользователь B
    participant DB as База данных
    
    UserA->>DB: SELECT * FROM orders WHERE id=123<br/>FOR UPDATE
    Note over DB: 🔒 Блокирует строку
    
    UserB->>DB: SELECT * FROM orders WHERE id=123<br/>FOR UPDATE
    Note over DB: ⏳ Ждёт освобождения блокировки...
    
    UserA->>UserA: Работает с данными (10 минут)
    UserA->>DB: COMMIT
    Note over DB: 🔓 Освобождает блокировку
    
    Note over UserB: Теперь может продолжить
    UserB->>DB: Получает данные
```

---

### 💻 Реализация на Java (JPA):

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private Long id;
    
    private String status;
    
    // getters/setters...
}

// Использование:
Order order = entityManager.find(
    Order.class, 
    123L, 
    LockModeType.PESSIMISTIC_WRITE // ← Блокируем при чтении
);

order.setStatus("PROCESSING");
entityManager.merge(order);
entityManager.getTransaction().commit(); // Освобождает блокировку
```

---

### 💻 Реализация на SQL:

```sql
-- Блокировка при чтении
SELECT * FROM orders WHERE id = 123 FOR UPDATE;

-- Теперь другие транзакции НЕ МОГУТ:
-- - Читать эту строку с FOR UPDATE
-- - Обновлять эту строку
-- - Удалять эту строку

-- Работаем с данными...
UPDATE orders SET status = 'PROCESSING' WHERE id = 123;

-- COMMIT освобождает блокировку
COMMIT;
```

---

### ✅ Преимущества Pessimistic Lock:

| Преимущество | Объяснение |
|--------------|------------|
| **Гарантированная согласованность** | Никаких конфликтов при сохранении |
| **Предсказуемость** | Всегда знаешь, что данные не изменятся |
| **Простота логики** | Не нужно обрабатывать конфликты |

---

### ❌ Недостатки Pessimistic Lock:

| Недостаток | Объяснение |
|------------|------------|
| **Низкая производительность** | Блокировки замедляют систему |
| **Риск взаимоблокировок (deadlock)** | Две транзакции блокируют друг друга |
| **Плохая масштабируемость** | Не подходит для высоконагруженных систем |
| **Долгие блокировки** | Если пользователь "ушёл" — данные заблокированы |

---

### 🏥 Когда использовать:

| Сценарий | Рекомендация |
|----------|--------------|
| **Финансовые операции** | ✅ Критично для согласованности |
| **Резервирование мест** | ✅ Нельзя продать одно место дважды |
| **Высокая конкуренция** | ✅ Частые конфликты |
| **Веб-приложения** | ❌ Пользователь может "зависнуть" |
| **Долгие операции** | ❌ Блокировка на 10 минут = проблемы |

---

## 4️⃣ Coarse Grained Lock (Крупнозернистая блокировка)

### 📌 Что это?

Блокировка **целого агрегата** (группы связанных объектов), а не отдельных объектов. Упрощает управление конкурентностью.

> 💡 **Аналогия**:  
> *«Вместо того чтобы запирать каждую комнату в доме, мы запираем весь дом одним ключом.»*

---

### 🏗️ Проблема, которую решает:

```java
// Без Coarse Grained Lock:
Order order = orderRepository.findById(123L);
order.getItems().get(0).setQuantity(5); // Нужно блокировать каждый товар
order.getItems().get(1).setQuantity(3);
orderRepository.save(order);

// Проблема: что если другой процесс изменил товар #2 между чтением и записью?
```

---

### 🏗️ Решение через агрегат:

```mermaid
erDiagram
    ORDER ||--o{ ORDER_ITEM : contains
    ORDER {
        Long id PK
        String status
        Integer version
    }
    ORDER_ITEM {
        Long id PK
        Long order_id FK
        String product_name
        Integer quantity
    }
```

---

### 💻 Реализация на Java:

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private Long id;
    
    @Version // ← Блокировка на уровне заказа
    private Integer version;
    
    @OneToMany(cascade = CascadeType.ALL, mappedBy = "order")
    private List<OrderItem> items;
    
    // Метод для изменения агрегата
    public void updateItemQuantity(Long itemId, Integer quantity) {
        OrderItem item = items.stream()
            .filter(i -> i.getId().equals(itemId))
            .findFirst()
            .orElseThrow();
        
        item.setQuantity(quantity);
        // Версия заказа увеличится при сохранении
    }
}

// Использование:
Order order = orderRepository.findById(123L);
order.updateItemQuantity(456L, 10); // Изменяем один товар
orderRepository.save(order); // ← Проверит версию заказа целиком
```

---

### ✅ Преимущества Coarse Grained Lock:

| Преимущество | Объяснение |
|--------------|------------|
| **Простота управления** | Одна блокировка вместо многих |
| **Согласованность агрегата** | Все связанные объекты изменяются вместе |
| **Меньше конфликтов** | Реже случаются частичные обновления |

---

### ❌ Недостатки Coarse Grained Lock:

| Недостаток | Объяснение |
|------------|------------|
| **Избыточная блокировка** | Блокируем весь агрегат, даже если нужно изменить один объект |
| **Снижение параллелизма** | Два пользователя не могут одновременно менять разные товары в одном заказе |

---

### 🏥 Когда использовать:

| Сценарий | Рекомендация |
|----------|--------------|
| **Агрегаты DDD** | ✅ Идеально подходит |
| **Заказ + товары** | ✅ Всегда изменяем вместе |
| **Независимые объекты** | ❌ Лучше блокировать по отдельности |

---

## 5️⃣ Implicit Lock (Неявная блокировка)

### 📌 Что это?

**Автоматическая блокировка**, управляемая фреймворком или слоем доступа к данным, **без явного кода в приложении**.

> 💡 **Аналогия**:  
> *«Вы не думаете о блокировках, когда едете на машине. ABS и стабилизация работают автоматически.»*

---

### 🏗️ Как работает:

```java
// Приложение НЕ знает о блокировках:
@Service
public class OrderService {
    
    @Transactional // ← Фреймворк сам управляет блокировками
    public void processOrder(Long orderId) {
        Order order = orderRepository.findById(orderId);
        order.setStatus("PROCESSING");
        // ... бизнес-логика ...
    }
}

// Фреймворк (Spring/Hibernate) автоматически:
// 1. Открывает транзакцию
// 2. Применяет настройки блокировок (из @Transactional)
// 3. Фиксирует или откатывает
// 4. Освобождает блокировки
```

---

### 💻 Конфигурация на Spring:

```java
@Service
public class OrderService {
    
    // Изоляция по умолчанию: READ_COMMITTED
    @Transactional
    public void updateOrder(Long id, String status) {
        Order order = orderRepository.findById(id);
        order.setStatus(status);
    }
    
    // Явное указание уровня изоляции
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void criticalUpdate(Long id) {
        // SERIALIZABLE = самая строгая изоляция (как Pessimistic)
    }
    
    // Только для чтения = нет блокировок записи
    @Transactional(readOnly = true)
    public Order getOrder(Long id) {
        return orderRepository.findById(id);
    }
}
```

---

### 🏗️ Уровни изоляции транзакций (SQL):

| Уровень | Описание | Блокировки |
|---------|----------|------------|
| **READ UNCOMMITTED** | Может читать незафиксированные данные | Минимальные |
| **READ COMMITTED** | Читает только зафиксированные данные | Строковые при записи |
| **REPEATABLE READ** | Гарантирует повторяемость чтения | Строковые + диапазонные |
| **SERIALIZABLE** | Полная изоляция (как последовательное выполнение) | Максимальные |

---

### ✅ Преимущества Implicit Lock:

| Преимущество | Объяснение |
|--------------|------------|
| **Простота кода** | Не нужно думать о блокировках в бизнес-логике |
| **Единообразие** | Все транзакции управляются одинаково |
| **Меньше ошибок** | Не забудешь освободить блокировку |

---

### ❌ Недостатки Implicit Lock:

| Недостаток | Объяснение |
|------------|------------|
| **Меньше контроля** | Сложно настроить тонкие сценарии |
| **Производительность** | Фреймворк может использовать избыточные блокировки |
| **Отладка** | Сложнее понять, где возникла блокировка |

---

## 🆚 Сравнение всех паттернов

| Паттерн | Когда конфликт обнаруживается | Блокировка | Производительность | Сложность |
|---------|-------------------------------|------------|-------------------|-----------|
| **Optimistic Lock** | При сохранении | Нет (до сохранения) | ⭐⭐⭐⭐⭐ | Низкая |
| **Pessimistic Lock** | При чтении | Да (до коммита) | ⭐⭐ | Высокая |
| **Coarse Grained Lock** | При сохранении агрегата | Нет / Да (зависит от реализации) | ⭐⭐⭐⭐ | Средняя |
| **Implicit Lock** | Зависит от уровня изоляции | Автоматическая | ⭐⭐⭐ | Низкая (для разработчика) |

---

## 📊 Матрица выбора паттерна

```mermaid
flowchart TD
    A[Начало] --> B{Частые конфликты?}
    B -->|Да| C{Критична согласованность?}
    B -->|Нет| D[Optimistic Lock]
    C -->|Да| E[Pessimistic Lock]
    C -->|Нет| F{Работаем с агрегатами?}
    F -->|Да| G[Coarse Grained Lock]
    F -->|Нет| D
    D --> H[Готово]
    E --> H
    G --> H
```

---

## 💡 Рекомендации по выбору

### ✅ Используйте **Optimistic Lock**, если:
- Конфликты редки (< 5% операций)
- Производительность важнее, чем абсолютная согласованность
- Веб-приложение с короткими сессиями

### ✅ Используйте **Pessimistic Lock**, если:
- Конфликты часты (> 30% операций)
- Данные критичны (финансы, резервирование)
- Можно держать соединение открытым недолго

### ✅ Используйте **Coarse Grained Lock**, если:
- Работаете с агрегатами (заказ + товары)
- Нужна согласованность всей группы объектов
- Объекты всегда изменяются вместе

### ✅ Используйте **Implicit Lock**, если:
- Хотите упростить код
- Доверяете фреймворку
- Стандартные уровни изоляции подходят

---

## 🏥 Реальные примеры

### 🛒 Электронная коммерция:

```java
// Корзина покупок - редкие конфликты
@Version
private Integer version; // ← Optimistic Lock

// Резервирование товара на складе - частые конфликты
@Transactional(isolation = Isolation.SERIALIZABLE)
public void reserveStock(Long productId, Integer quantity) {
    // ← Pessimistic Lock через уровень изоляции
}
```

---

### 🏦 Банковское приложение:

```java
// Перевод денег - критичная операция
@Transactional(isolation = Isolation.SERIALIZABLE)
public void transferMoney(Account from, Account to, BigDecimal amount) {
    // ← Pessimistic Lock гарантирует согласованность
}
```

---

### 📝 Система управления документами:

```java
// Редактирование документа - редкие конфликты
@Version
private Integer version; // ← Optimistic Lock

// Публикация документа - критичная операция
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void publishDocument(Document doc) {
    // ← Coarse Grained Lock на уровне документа
}
```

---

## ⚠️ Распространённые ошибки

| Ошибка | Последствие | Как исправить |
|--------|-------------|---------------|
| **Забыть @Version** | Перезапись данных | Всегда добавлять версию для изменяемых сущностей |
| **Долгая транзакция с Pessimistic** | Блокировка на минуты | Использовать Optimistic для долгих операций |
| **Смешивать подходы** | Непредсказуемое поведение | Выбрать один подход на агрегат |
| **Игнорировать конфликты** | Тихая перезапись данных | Всегда обрабатывать OptimisticLockException |

---

## 💬 Итог

> **Offline Concurrency** — общая проблема конкурентного доступа к данным вне соединения.  
> **Optimistic Lock** — проверяем конфликты при сохранении (быстро, но возможны ошибки).  
> **Pessimistic Lock** — блокируем при чтении (медленно, но надёжно).  
> **Coarse Grained Lock** — блокируем агрегат целиком (проще, но менее гибко).  
> **Implicit Lock** — фреймворк управляет блокировками автоматически (проще для разработчика).

> 💡 **Главное правило**:  
> *«Выбирайте паттерн на основе частоты конфликтов и критичности данных, а не на основе личных предпочтений.»*

---

✅ **Оптимальный подход в современных приложениях**:  
Используйте **Optimistic Lock** по умолчанию, переключайтесь на **Pessimistic** только для критичных операций, и применяйте **Coarse Grained Lock** для агрегатов.


# Implicit Lock vs Pessimistic Lock

## 🔐 Implicit Lock vs Pessimistic Lock — ключевые различия

Это **два разных уровня абстракции** управления конкурентностью, которые часто путают. Разберём детально.

---

## 📊 Сравнительная таблица

| Критерий                       | **Implicit Lock**                   | **Pessimistic Lock**                 |
| ------------------------------ | ----------------------------------- | ------------------------------------ |
| **Уровень абстракции**         | Высокий (фреймворк)                 | Низкий (приложение)                  |
| **Кто управляет блокировками** | Фреймворк / ORM                     | Разработчик явно                     |
| **Видимость в коде**           | Скрытая (аннотации, конфигурация)   | Явная (вызовы методов, SQL)          |
| **Гибкость**                   | Ограниченная (настройки фреймворка) | Полная (контроль на уровне запросов) |
| **Отладка**                    | Сложнее (не видно в коде)           | Проще (явные вызовы)                 |
| **Переносимость**              | Зависит от фреймворка               | Зависит от СУБД                      |
| **Производительность**         | Оптимизирована фреймворком          | Зависит от реализации                |

---

## 🔍 Подробное объяснение

### **Implicit Lock (Неявная блокировка)**

**Что это**: Блокировки, которые **автоматически применяет фреймворк** на основе настроек транзакций, **без явного кода** в приложении.

**Ключевая идея**: *«Я не думаю о блокировках — фреймворк делает это за меня»*

---

### 💻 Пример на Spring:

```java
@Service
public class OrderService {
    
    // Implicit Lock через уровень изоляции транзакции
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void updateOrderStatus(Long orderId, String newStatus) {
        // Фреймворк автоматически:
        // 1. Открывает транзакцию
        // 2. Применяет блокировки на основе уровня изоляции
        // 3. Фиксирует транзакцию
        // 4. Освобождает блокировки
        
        Order order = orderRepository.findById(orderId);
        order.setStatus(newStatus);
    }
    
    // Только для чтения = минимальные блокировки
    @Transactional(readOnly = true)
    public Order getOrder(Long orderId) {
        return orderRepository.findById(orderId);
    }
}
```

---

### 💻 Пример на Hibernate:

```java
@Entity
public class Order {
    @Id
    private Long id;
    
    private String status;
    
    // Implicit Optimistic Lock через @Version
    @Version
    private Integer version;
}

// Использование — блокировка происходит автоматически при сохранении
Order order = entityManager.find(Order.class, 123L);
order.setStatus("PROCESSING");
entityManager.merge(order); // ← Фреймворк автоматически проверит версию
```

---

### 🏗️ Как работает Implicit Lock:

```mermaid
sequenceDiagram
    participant App as Приложение
    participant Spring as Spring Framework
    participant Hibernate as Hibernate
    participant DB as База данных
    
    App->>Spring: Вызов @Transactional метода
    Spring->>Hibernate: Начать транзакцию
    Hibernate->>DB: BEGIN TRANSACTION<br/>SET ISOLATION LEVEL READ COMMITTED
    
    App->>Hibernate: entityManager.find(Order.class, 123)
    Hibernate->>DB: SELECT * FROM orders WHERE id = 123
    
    App->>Hibernate: order.setStatus("PROCESSING")
    App->>Hibernate: entityManager.merge(order)
    
    Hibernate->>DB: UPDATE orders SET status = 'PROCESSING',<br/>version = version + 1 WHERE id = 123 AND version = 1
    
    Hibernate->>DB: COMMIT
    Note over DB: Фреймворк автоматически<br/>освобождает блокировки
```

---

### ✅ Преимущества Implicit Lock:

| Преимущество | Объяснение |
|--------------|------------|
| **Простота кода** | Не нужно писать код блокировок |
| **Единообразие** | Все транзакции управляются одинаково |
| **Меньше ошибок** | Не забудешь освободить блокировку |
| **Интеграция с фреймворком** | Работает с остальными фичами Spring/Hibernate |

---

### ❌ Недостатки Implicit Lock:

| Недостаток | Объяснение |
|------------|------------|
| **Меньше контроля** | Нельзя точно настроить блокировки |
| **Сложнее отладка** | Не видно, где возникает блокировка |
| **Зависимость от фреймворка** | Трудно перейти на другой стек |
| **Производительность** | Фреймворк может использовать избыточные блокировки |

---

## 🔒 Pessimistic Lock (Пессимистичная блокировка)

**Что это**: **Явное захватывание блокировки** на уровне приложения или БД, чтобы **предотвратить доступ других транзакций** к данным.

**Ключевая идея**: *«Я знаю, что данные могут измениться, поэтому блокирую их сразу»*

---

### 💻 Пример на JPA:

```java
@Service
public class OrderService {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public void processOrderWithPessimisticLock(Long orderId) {
        // ЯВНАЯ блокировка при чтении
        Order order = entityManager.find(
            Order.class, 
            orderId, 
            LockModeType.PESSIMISTIC_WRITE  // ← Явная блокировка
        );
        
        // Теперь другие транзакции НЕ МОГУТ:
        // - Читать эту строку с блокировкой
        // - Обновлять эту строку
        // - Удалять эту строку
        
        order.setStatus("PROCESSING");
        
        // Работаем с данными...
        // Блокировка держится до конца транзакции
        
        // Коммит освободит блокировку
    }
}
```

---

### 💻 Пример на чистом SQL:

```sql
-- ЯВНАЯ блокировка при чтении
BEGIN TRANSACTION;

SELECT * FROM orders WHERE id = 123 FOR UPDATE;
-- ↑ Теперь другие транзакции заблокированы для этой строки

-- Работаем с данными (может занять несколько минут)
UPDATE orders SET status = 'PROCESSING' WHERE id = 123;

COMMIT;  -- ← Освобождает блокировку
```

---

### 💻 Пример с таймаутом блокировки:

```java
@Service
public class OrderService {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public void processWithTimeout(Long orderId) {
        Map<String, Object> properties = new HashMap<>();
        properties.put("javax.persistence.lock.timeout", 5000); // 5 секунд
        
        Order order = entityManager.find(
            Order.class,
            orderId,
            LockModeType.PESSIMISTIC_WRITE,
            properties  // ← Таймаут блокировки
        );
        
        // Если блокировка не получена за 5 секунд → выбросит исключение
        order.setStatus("PROCESSING");
    }
}
```

---

### 🏗️ Как работает Pessimistic Lock:

```mermaid
sequenceDiagram
    participant UserA as Пользователь A
    participant UserB as Пользователь B
    participant DB as База данных
    
    UserA->>DB: SELECT * FROM orders WHERE id=123<br/>FOR UPDATE
    Note over DB: 🔒 Блокирует строку
    
    UserB->>DB: SELECT * FROM orders WHERE id=123<br/>FOR UPDATE
    Note over DB: ⏳ Ждёт освобождения блокировки...
    
    UserA->>UserA: Работает с данными (10 минут)
    UserA->>DB: COMMIT
    Note over DB: 🔓 Освобождает блокировку
    
    Note over UserB: Теперь может продолжить
    UserB->>DB: Получает данные
```

---

### ✅ Преимущества Pessimistic Lock:

| Преимущество | Объяснение |
|--------------|------------|
| **Полный контроль** | Точно знаешь, когда и что блокируется |
| **Гарантированная согласованность** | Никаких конфликтов при сохранении |
| **Предсказуемость** | Всегда знаешь поведение системы |
| **Гибкость** | Можно настроить таймауты, типы блокировок |

---

### ❌ Недостатки Pessimistic Lock:

| Недостаток | Объяснение |
|------------|------------|
| **Низкая производительность** | Блокировки замедляют систему |
| **Риск взаимоблокировок (deadlock)** | Две транзакции блокируют друг друга |
| **Плохая масштабируемость** | Не подходит для высоконагруженных систем |
| **Сложность кода** | Нужно явно управлять блокировками |

---

## 🆚 Ключевые различия в деталях

### 1. **Когда применяется блокировка**

| Паттерн | Когда блокируется |
|---------|-------------------|
| **Implicit Lock** | Автоматически при начале транзакции (на основе уровня изоляции) |
| **Pessimistic Lock** | Явно при выполнении `SELECT ... FOR UPDATE` или `LockModeType.PESSIMISTIC_WRITE` |

---

### 2. **Контроль над блокировкой**

```java
// Implicit Lock — контроль через настройки
@Transactional(isolation = Isolation.SERIALIZABLE)
public void implicitApproach() {
    // Блокировка происходит "магически"
    Order order = orderRepository.findById(123L);
    order.setStatus("PROCESSING");
}

// Pessimistic Lock — полный контроль
public void explicitApproach() {
    Order order = orderRepository.findByIdWithPessimisticLock(123L);
    // ↑ Я ВИЖУ, что происходит блокировка
    
    order.setStatus("PROCESSING");
}
```

---

### 3. **Отладка и мониторинг**

```java
// Implicit Lock — сложно отследить
@Transactional
public void process() {
    // Где блокировка? Когда освободится?
    // Нужно смотреть логи Hibernate/Spring
}

// Pessimistic Lock — явно видно
public void process() {
    log.info("Захватываю блокировку...");
    Order order = entityManager.find(Order.class, 123L, LockModeType.PESSIMISTIC_WRITE);
    log.info("Блокировка захвачена");
    
    // ... работа ...
    
    log.info("Освобождаю блокировку при коммите");
}
```

---

### 4. **Таймауты и обработка ошибок**

```java
// Implicit Lock — ограниченные возможности
@Transactional(isolation = Isolation.SERIALIZABLE, timeout = 30)
public void implicitWithTimeout() {
    // Таймаут на ВСЮ транзакцию, не на блокировку
}

// Pessimistic Lock — гибкие настройки
public void explicitWithTimeout() {
    Map<String, Object> props = new HashMap<>();
    props.put("javax.persistence.lock.timeout", 5000); // 5 сек на блокировку
    
    try {
        Order order = entityManager.find(Order.class, 123L, 
            LockModeType.PESSIMISTIC_WRITE, props);
    } catch (PersistenceException e) {
        if (e.getCause() instanceof LockTimeoutException) {
            // Обработка таймаута блокировки
            log.warn("Не удалось захватить блокировку");
        }
    }
}
```

---

## 📊 Когда использовать какой паттерн?

### ✅ Используйте **Implicit Lock**, если:

| Сценарий | Почему |
|----------|--------|
| **Стандартные CRUD-операции** | Фреймворк отлично справляется |
| **Низкая конкуренция** | Редкие конфликты |
| **Простые транзакции** | Нет сложной бизнес-логики |
| **Хотите простой код** | Не хотите думать о блокировках |
| **Используете фреймворк** | Spring, Hibernate, JPA |

---

### ✅ Используйте **Pessimistic Lock**, если:

| Сценарий | Почему |
|----------|--------|
| **Высокая конкуренция** | Частые конфликты (> 30% операций) |
| **Критичные данные** | Финансы, резервирование, инвентарь |
| **Долгие операции** | Нужно удерживать блокировку несколько минут |
| **Требуется точный контроль** | Нужно настроить таймауты, типы блокировок |
| **Сложная бизнес-логика** | Нужно блокировать в определённые моменты |

---

## 🏥 Реальные примеры

### 🛒 Электронная коммерция:

```java
@Service
public class ShoppingCartService {
    
    // Implicit Lock — для корзины (редкие конфликты)
    @Transactional
    public void addToCart(Long userId, Long productId, Integer quantity) {
        Cart cart = cartRepository.findByUserId(userId);
        cart.addItem(productId, quantity);
        // Фреймворк сам управляет блокировками
    }
    
    // Pessimistic Lock — для резервирования товара
    @Transactional
    public void reserveProduct(Long productId, Integer quantity) {
        Product product = entityManager.find(
            Product.class, 
            productId, 
            LockModeType.PESSIMISTIC_WRITE  // ← Явная блокировка
        );
        
        if (product.getStock() >= quantity) {
            product.setStock(product.getStock() - quantity);
        } else {
            throw new InsufficientStockException();
        }
    }
}
```

---

### 🏦 Банковское приложение:

```java
@Service
public class BankingService {
    
    // Implicit Lock — для чтения баланса
    @Transactional(readOnly = true)
    public BigDecimal getBalance(Long accountId) {
        Account account = accountRepository.findById(accountId);
        return account.getBalance();
    }
    
    // Pessimistic Lock — для перевода денег (критично!)
    @Transactional
    public void transferMoney(Long fromId, Long toId, BigDecimal amount) {
        // Явно блокируем оба счёта
        Account from = entityManager.find(
            Account.class, fromId, LockModeType.PESSIMISTIC_WRITE);
        
        Account to = entityManager.find(
            Account.class, toId, LockModeType.PESSIMISTIC_WRITE);
        
        if (from.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException();
        }
        
        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));
    }
}
```

---

## ⚠️ Распространённые ошибки

### ❌ Ошибка 1: Смешивание подходов без понимания

```java
// ПЛОХО: Неясно, какая блокировка применяется
@Transactional(isolation = Isolation.SERIALIZABLE)
public void confusingMethod(Long id) {
    Order order = entityManager.find(
        Order.class, id, LockModeType.PESSIMISTIC_WRITE);
    // SERIALIZABLE + PESSIMISTIC_WRITE = избыточно!
}
```

---

### ❌ Ошибка 2: Долгая блокировка в Pessimistic

```java
// ПЛОХО: Блокировка на 10 минут!
@Transactional
public void longRunningOperation(Long id) {
    Order order = entityManager.find(
        Order.class, id, LockModeType.PESSIMISTIC_WRITE);
    
    Thread.sleep(600000); // 10 минут!
    // ← Другие пользователи ждут 10 минут!
}
```

---

### ✅ Правильно: Использовать Optimistic для долгих операций

```java
// ХОРОШО: Проверка при сохранении
@Transactional
public void longOperationWithOptimistic(Long id) {
    Order order = entityManager.find(Order.class, id);
    
    // Долгая операция без блокировки
    Thread.sleep(600000);
    
    // Проверка версии при сохранении
    order.setStatus("COMPLETED");
    // ← Если кто-то изменил данные — будет исключение
}
```

---

## 💡 Рекомендации

### 📌 Общее правило:

> **По умолчанию используйте Implicit Lock** через настройки транзакций.  
> **Переключайтесь на Pessimistic Lock** только для критичных операций с высокой конкуренцией.

---

### 📌 Матрица выбора:

| Ситуация | Рекомендация |
|----------|--------------|
| **Простые CRUD** | Implicit Lock (READ_COMMITTED) |
| **Чтение данных** | Implicit Lock (readOnly = true) |
| **Редкие конфликты** | Implicit Lock + Optimistic (@Version) |
| **Частые конфликты** | Pessimistic Lock |
| **Критичные данные** | Pessimistic Lock |
| **Долгие операции** | Optimistic Lock (не держим блокировку) |
| **Высокая нагрузка** | Optimistic Lock |

---

## 📊 Производительность: сравнение

```java
// Тест: 1000 параллельных запросов

// Implicit Lock (READ_COMMITTED)
// Среднее время: 50 мс
// Конфликты: 2% (обрабатываются через повторные попытки)

// Pessimistic Lock
// Среднее время: 200 мс (блокировки!)
// Конфликты: 0% (блокировка предотвращает конфликты)

// Optimistic Lock
// Среднее время: 45 мс
// Конфликты: 5% (требуют повторных попыток)
```

---

## 💬 Итог

> **Implicit Lock** — это **автоматическая коробка передач**:  
> - Просто в использовании  
> - Фреймворк решает, когда и как блокировать  
> - Подходит для большинства сценариев  

> **Pessimistic Lock** — это **ручная коробка передач**:  
> - Полный контроль  
> - Нужно явно управлять блокировками  
> - Подходит для критичных и конкурентных сценариев  

> 💡 **Главное правило**:  
> *«Используйте простое решение (Implicit), пока не докажете, что нужно сложное (Pessimistic).»*

---

✅ **Implicit Lock** — для повседневных задач, когда важна простота и производительность.  
✅ **Pessimistic Lock** — для критичных операций, когда важна абсолютная согласованность.
