---
aliases:
  - Hibernate
  - Hibernate Cache
  - Кэширование в Hibernate
---
# Кэширование в Hibernate

Hibernate использует многоуровневую систему кэширования для оптимизации производительности при работе с базой данных.

## Уровни кэширования в Hibernate

### 1. **Кэш первого уровня (First-Level Cache)**
- **Область видимости**: Сессия (Session)
- **Автоматическое управление**: Включен по умолчанию
- **Жизненный цикл**: Существует только во время сессии

```java
Session session = sessionFactory.openSession();
// Первый запрос - идет в БД
User user1 = session.get(User.class, 1L);
// Второй запрос - берется из кэша первого уровня
User user2 = session.get(User.class, 1L);
session.close();
```

### 2. **Кэш второго уровня (Second-Level Cache)**
- **Область видимости**: SessionFactory (приложение)
- **Требует настройки**: Не включен по умолчанию
- **Поставщики**: EhCache, Infinispan, Hazelcast

```xml
<!-- configuration -->
<property name="hibernate.cache.use_second_level_cache">true</property>
<property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>
```

```java
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    // ...
}
```

### 3. **Кэш запросов (Query Cache)**
- **Хранит**: Результаты запросов с параметрами
- **Требует**: Включения отдельно

```xml
<property name="hibernate.cache.use_query_cache">true</property>
```

```java
Query query = session.createQuery("FROM User u WHERE u.active = :active");
query.setParameter("active", true);
query.setCacheable(true);
List<User> users = query.list();
```

## Стратегии кэширования

### **READ_ONLY**
- Только для чтения
- Неизменяемые данные
- Максимальная производительность

### **READ_WRITE**
- Чтение и запись
- Использует мягкую блокировку
- Подходит для частого чтения, редкой записи

### **NONSTRICT_READ_WRITE**
- Без строгой согласованности
- Возможны временные несоответствия
- Высокая производительность

### **TRANSACTIONAL**
- Полная транзакционная поддержка
- Для кластерных сред
- Наивысшая согласованность

## Конфигурация

### **XML конфигурация**
```xml
<property name="hibernate.cache.use_second_level_cache">true</property>
<property name="hibernate.cache.use_query_cache">true</property>
<property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>
<property name="hibernate.generate_statistics">true</property>
```

### **Annotation-based**
```java
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    @Id
    private Long id;
    
    @OneToMany(mappedBy = "product")
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    private Set<Order> orders;
}
```

## Практические примеры

### **Кэширование сущности**
```java
@Entity
@Table(name = "users")
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
@NaturalIdCache
public class User {
    @Id
    private Long id;
    
    @NaturalId
    private String email;
}
```

### **Кэширование коллекций**
```java
@Entity
public class Department {
    @Id
    private Long id;
    
    @OneToMany(mappedBy = "department")
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    private Set<Employee> employees;
}
```

## Статистика и мониторинг

```java
Statistics stats = sessionFactory.getStatistics();
stats.setStatisticsEnabled(true);

// Получение статистики
long secondLevelCacheHitCount = stats.getSecondLevelCacheHitCount();
long secondLevelCacheMissCount = stats.getSecondLevelCacheMissCount();
```

## Лучшие практики

1. **Кэшируйте часто читаемые, редко изменяемые данные**
2. **Избегайте кэширования часто изменяемых данных**
3. **Используйте подходящую стратегию кэширования**
4. **Настройте размер кэша и политику вытеснения**
5. **Мониторьте эффективность кэширования**

## Распространенные проблемы

- **Устаревшие данные**: При неправильной стратегии инвалидации
- **Memory leaks**: При кэшировании больших объемов данных
- **Согласованность**: В кластерных средах

Кэширование в Hibernate значительно повышает производительность, но требует тщательной настройки и тестирования.