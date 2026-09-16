---
aliases:
  - API Composition
  - CDC
  - Change Data Capture
  - Database Decomposition
  - Debezium
  - Denormalization
  - Distributed Transactions
  - Functional Decomposition
  - Horizontal Decomposition
  - OLAP
  - OLTP
  - Saga Pattern
  - Sharding
  - Vertical Decomposition
  - Вертикальная декомпозиция
  - Горизонтальная декомпозиция
  - Декомпозиция базы данных
  - Денормализация
  - Распределённые транзакции
  - Функциональная декомпозиция
  - Шардирование
---
## Database Decomposition

**Database Decomposition** (декомпозиция базы данных) — это стратегия разделения единой монолитной базы данных на несколько меньших, специализированных баз данных, каждая из которых принадлежит отдельному микросервису.

Это **самый сложный и критический этап** при переходе от монолита к [[microservice|микросервисам]].

## Зачем это нужно?

**Проблема монолитной БД:**

- единая точка отказа;
- tight coupling между сервисами;
- сложности с масштабированием;
- блокировки при высокой нагрузке;
- единая модель данных для всех сервисов.

**Решение:**

- каждый [[microservice|микросервис]] владеет своей собственной БД;
- сервисы общаются только через API;
- независимое масштабирование;
- выбор оптимальной БД для каждой задачи.

## Стратегии декомпозиции

### Vertical Decomposition (Вертикальная)

Разделение по бизнес-доменам.

### Horizontal Decomposition (Горизонтальная)

Шардирование данных по ключу.

**Пример:** разделение пользователей по географическому признаку.

### Functional Decomposition (Функциональная)

Разделение по паттернам доступа.

**Пример:**

- `users_db` — OLTP (транзакции);
- `users_analytics_db` — OLAP (аналитика).

## Проблемы и решения

### Проблема: Join across services

**Решение:**

- API composition pattern;
- кэширование данных;
- [[database-normalization|денормализация]] (копия нужных полей).

Пример API-вызовов вместо SQL JOIN:

```java
// Вместо SQL JOIN делаем API вызовы
public OrderDetails getOrderWithUser(Long orderId) {
  Order order = orderRepository.findById(orderId);
  User user = userServiceClient.getUser(order.getUserId());
  return new OrderDetails(order, user);
}
```

### Проблема: Distributed transactions

**Решение:**

- [[saga|Saga pattern]];
- event-driven architecture;
- compensating transactions.

Реализация паттерна Saga:

```java
// Saga pattern implementation
@Saga
public class OrderCreationSaga {
  @StartSaga
  @SagaEventHandler(associationProperty = "orderId")
  public void handle(OrderCreatedEvent event) {
    // 1. Reserve products
    sagaManager.send(new ReserveProductsCommand(event.getOrderId()));
  }

  @SagaEventHandler(associationProperty = "orderId")
  public void handle(ProductsReservedEvent event) {
    // 2. Process payment
    sagaManager.send(new ProcessPaymentCommand(event.getOrderId()));
  }
}
```

### Проблема: Data consistency

**Решение:**

- [[event-sourcing|Event sourcing]];
- [[change-data-capture|change data capture]] (CDC);
- асинхронная репликация.

Пример CDC с помощью Debezium:

```sql
-- CDC с помощью Debezium
CREATE CONNECTOR user_cdc WITH (
  'connector.class' = 'io.debezium.connector.postgresql.PostgresConnector',
  'database.hostname' = 'user_db',
  'database.dbname' = 'user_service',
  'table.include.list' = 'public.users'
);
```
