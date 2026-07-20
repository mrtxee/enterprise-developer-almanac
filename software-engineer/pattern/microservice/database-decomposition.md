## Database Decomposition

**Database Decomposition** (декомпозиция базы данных) — это стратегия разделения единой монолитной базы данных на несколько меньших, специализированных баз данных, каждая из которых принадлежит отдельному микросервису.

Это **самый сложный и критический этап** при переходе от монолита к микросервисам.

## 🎯 Зачем это нужно?

**Проблема монолитной БД:**

- Единая точка отказа
- Tight coupling между сервисами
- Сложности с масштабированием
- Блокировки при высокой нагрузке
- Единая модель данных для всех сервисов
    

**Решение:**

- Каждый микросервис владеет своей собственной БД
- Сервисы общаются только через API
- Независимое масштабирование
- Выбор оптимальной БД для каждой задачи

## 🗺️ Стратегии декомпозиции

### 1. **Vertical Decomposition (Вертикальная)**

Разделение по бизнес-доменам.

### 2. **Horizontal Decomposition (Горизонтальная)**

Шардирование данных по ключу.

**Пример:** Разделение пользователей по географическому признаку

### 3. **Functional Decomposition (Функциональная)**

Разделение по patterns доступа.

**Пример:**

- `users_db` — OLTP (транзакции)
    
- `users_analytics_db` — OLAP (аналитика)

## ⚠️ Проблемы и решения

### **Problem 1: Join across services**

**Solution:**

- API composition pattern
    
- Кэширование данных
    
- Денормализация (копия нужных полей)
    

java

// Вместо SQL JOIN делаем API вызовы
public OrderDetails getOrderWithUser(Long orderId) {
    Order order = orderRepository.findById(orderId);
    User user = userServiceClient.getUser(order.getUserId());
    
    return new OrderDetails(order, user);
}

### **Problem 2: Distributed transactions**

**Solution:**

- Saga pattern
    
- Event-driven architecture
    
- Compensating transactions
    

java

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

### **Problem 3: Data consistency**

**Solution:**

- Event sourcing
    
- Change data capture (CDC)
    
- Асинхронная репликация
    

sql

-- CDC с помощью Debezium
CREATE CONNECTOR user_cdc WITH (
    'connector.class' = 'io.debezium.connector.postgresql.PostgresConnector',
    'database.hostname' = 'user_db',
    'database.dbname' = 'user_service',
    'table.include.list' = 'public.users'
);