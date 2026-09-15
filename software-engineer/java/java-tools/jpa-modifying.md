---
aliases:
  - "@Modifying"
  - "@PostUpdate"
  - "@PreUpdate"
  - "@Query"
  - "@QueryHint"
  - "@QueryHints"
  - "@Transactional"
  - bulk update
  - clearAutomatically
  - Entity lifecycle callbacks
  - flushAutomatically
  - JPA
  - JpaRepository
  - L2 cache
  - second-level cache
  - persistence context
  - Spring Data JPA
  - stale data
  - bulk-операция
  - Контекст персистентности
  - Кэш второго уровня
  - Устаревшие данные
---
## JPA @Modifying

**Краткий ответ**

- **`@Modifying`** — аннотация Spring Data JPA для WRITE-операций (UPDATE, DELETE) в репозиториях.
- **`clearAutomatically`** — очищает контекст персистентности после выполнения.
- **`flushAutomatically`** — сбрасывает pending изменения перед выполнением.

---

## Базовое использование

### Ошибка без @Modifying

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // Exception: InvalidDataAccessApiUsageException
    @Query("UPDATE User u SET u.active = false WHERE u.id = :id")
    void deactivateUser(@Param("id") Long id);
}
```

### Правильно с @Modifying

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.id = :id")
    void deactivateUser(@Param("id") Long id);
}
```

---

## Атрибуты @Modifying

Исходный код аннотации:

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Modifying {
    boolean flushAutomatically() default false;
    boolean clearAutomatically() default false;
}
```

---

## Атрибут flushAutomatically

**Что делает**

Сбрасывает все pending изменения из persistence context в БД перед выполнением запроса.

**Пример без flushAutomatically**

```java
@Transactional
public void updateUser() {
    User user = userRepository.findById(1L).orElseThrow();
    user.setName("New Name");
    // Изменение ещё в persistence context (не в БД)

    userRepository.bulkDeactivateOldUsers();
    // bulk-запрос НЕ увидит изменение name!
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // Не увидит pending изменения
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.updatedAt < :date")
    void bulkDeactivateOldUsers(@Param("date") LocalDateTime date);
}
```

**Пример с flushAutomatically = true**

```java
@Transactional
public void updateUser() {
    User user = userRepository.findById(1L).orElseThrow();
    user.setName("New Name");
    // Изменение в persistence context

    userRepository.bulkDeactivateOldUsers();
    // Сначала flush → изменение в БД
    // Потом bulk-запрос увидит актуальные данные
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Modifying(flushAutomatically = true)
    @Query("UPDATE User u SET u.active = false WHERE u.updatedAt < :date")
    void bulkDeactivateOldUsers(@Param("date") LocalDateTime date);
}
```

**Визуализация**

Без `flushAutomatically`:

1. `user.setName()` → изменение в Persistence Context.
2. `bulkUpdate()` → не видит изменения (ещё в PC).
3. `transaction commit` → запись в Database.

С `flushAutomatically = true`:

1. `user.setName()` → изменение в Persistence Context.
2. `bulkUpdate()` → FLUSH → Database.
3. `bulkUpdate()` → видит актуальные данные.
4. `transaction commit`.

---

## Атрибут clearAutomatically

**Что делает**

Очищает persistence context после выполнения запроса.

**Зачем нужно**

- Предотвращает stale data (устаревшие данные в кэше).
- Избегает конфликтов при последующих операциях чтения.

**Пример без clearAutomatically**

```java
@Transactional
public void processUser() {
    User user = userRepository.findById(1L).orElseThrow();
    // user.active = true (в persistence context)

    userRepository.bulkDeactivateUser(1L);
    // БД: active = false
    // Но в persistence context: active = true (STALE!)

    User sameUser = userRepository.findById(1L).orElseThrow();
    // Вернётся из persistence context с active = true!
    System.out.println(sameUser.isActive()); // true (неверно!)
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.id = :id")
    void bulkDeactivateUser(@Param("id") Long id);
}
```

**Пример с clearAutomatically = true**

```java
@Transactional
public void processUser() {
    User user = userRepository.findById(1L).orElseThrow();
    // user.active = true

    userRepository.bulkDeactivateUser(1L);
    // БД: active = false
    // Persistence context очищен!

    User sameUser = userRepository.findById(1L).orElseThrow();
    // Загрузится из БД с актуальными данными
    System.out.println(sameUser.isActive()); // false (верно!)
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Modifying(clearAutomatically = true)
    @Query("UPDATE User u SET u.active = false WHERE u.id = :id")
    void bulkDeactivateUser(@Param("id") Long id);
}
```

**Визуализация**

Без `clearAutomatically`:

1. `findById()` → Persistence Context: `User{active=true}`.
2. `bulkUpdate()` → Database: `active=false`.
3. `findById()` → из PC: `User{active=true}` (STALE!).

С `clearAutomatically = true`:

1. `findById()` → Persistence Context: `User{active=true}`.
2. `bulkUpdate()` → Database: `active=false`.
3. `clear()` → Persistence Context: EMPTY.
4. `findById()` → из БД: `User{active=false}` (FRESH!).

---

## Сравнение атрибутов

| Атрибут | Когда срабатывает | Что делает | Когда использовать |
| ------- | ----------------- | ---------- | ------------------ |
| **`flushAutomatically`** | **перед** запросом | Сбрасывает pending изменения в БД | Если есть модификации Entity перед bulk-операцией |
| **`clearAutomatically`** | **после** запроса | Очищает persistence context | Если будет чтение тех же Entity после bulk-операции |
| **оба** | перед и после | Полная синхронизация | Для максимальной консистентности |

---

## Примеры использования

**Только `clearAutomatically` (наиболее частый случай)**

```java
@Modifying(clearAutomatically = true)
@Query("UPDATE User u SET u.active = false WHERE u.id = :id")
void deactivateUser(@Param("id") Long id);
```

Рекомендуется по умолчанию — предотвращает stale data.

**Только `flushAutomatically`**

```java
@Modifying(flushAutomatically = true)
@Query("UPDATE Order o SET o.status = 'CANCELLED' WHERE o.createdAt < :date")
void cancelOldOrders(@Param("date") LocalDateTime date);
```

Подходит, когда перед этим модифицировали Entity в той же транзакции.

**Оба атрибута**

```java
@Modifying(flushAutomatically = true, clearAutomatically = true)
@Query("UPDATE Product p SET p.price = p.price * :factor WHERE p.category = :cat")
void updatePrices(@Param("cat") String category, @Param("factor") BigDecimal factor);
```

Для максимальной консистентности в сложных сценариях.

**Без атрибутов (по умолчанию)**

```java
@Modifying
@Query("DELETE FROM TempData t WHERE t.createdAt < :date")
void deleteOldData(@Param("date") LocalDateTime date);
```

Подходит, когда нет pending изменений и чтение после запроса не требуется.

---

## Важные нюансы

**Требуется @Transactional**

```java
// Ошибка: TransactionRequiredException
@Modifying
@Query("UPDATE User u SET u.active = false")
void deactivateAll();

// Правильно
@Transactional
@Modifying
@Query("UPDATE User u SET u.active = false")
void deactivateAll();
```

**Возвращаемое значение**

```java
// Возвращает количество затронутых строк
@Modifying(clearAutomatically = true)
@Query("UPDATE User u SET u.active = false WHERE u.id = :id")
int deactivateUser(@Param("id") Long id);

// Использование:
int count = userRepository.deactivateUser(1L);
log.info("Deactivated {} users", count);
```

**Не работает с @Entity lifecycle callbacks**

```java
// @PreUpdate, @PostUpdate НЕ вызываются!
@Entity
public class User {
    @PreUpdate
    void beforeUpdate() {
        // Не сработает при bulk UPDATE
    }
}

// Решение: загружать и обновлять через Entity
@Transactional
public void deactivateUser(Long id) {
    User user = userRepository.findById(id).orElseThrow();
    user.setActive(false);
    userRepository.save(user); // Callbacks сработают
}
```

**Кэширование второго уровня**

```java
// L2 cache НЕ инвалидируется автоматически!
@Modifying(clearAutomatically = true)
@Query("UPDATE User u SET u.active = false")
void deactivateAll();

// Решение: явно инвалидировать кэш
@Modifying(clearAutomatically = true)
@QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "false"))
@Query("UPDATE User u SET u.active = false")
void deactivateAll();
```

---

## Чек-лист выбора

- Pending изменения Entity перед bulk-операцией → **`flushAutomatically = true`**
- Чтение тех же Entity после bulk-операции → **`clearAutomatically = true`**
- Максимальная консистентность → **оба атрибута = true**
- Простой DELETE без последующего чтения → **без атрибутов (по умолчанию)**
- Важны `@PreUpdate`/`@PostUpdate` callbacks → **не использовать bulk-операции, загружать Entity**

---

## Альтернатива: обновление через Entity

Bulk-операция выполняется быстро, но без lifecycle callbacks:

```java
// Bulk UPDATE через @Modifying (быстро, но без callbacks)
@Modifying(clearAutomatically = true)
@Query("UPDATE User u SET u.active = false WHERE u.id = :id")
void deactivateUser(Long id);

// Обновление через Entity (медленнее, но есть callbacks)
@Transactional
public void deactivateUser(Long id) {
    User user = userRepository.findById(id).orElseThrow();
    user.setActive(false);
    userRepository.save(user); // @PreUpdate сработает
}
```

---

## Итог

| Вопрос | Ответ |
| ------ | ----- |
| Зачем `@Modifying`? | Для UPDATE/DELETE запросов в репозитории |
| Когда `flushAutomatically`? | Если есть pending изменения Entity |
| Когда `clearAutomatically`? | Если будет чтение тех же Entity после |
| Рекомендация по умолчанию? | `clearAutomatically = true` |
| Нужен ли `@Transactional`? | Обязательно |
