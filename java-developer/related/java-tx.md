---
aliases:
  - java-tx
  - JDBC
  - EntityTransaction
  - EntityManager
  - JpaTransactionManager
  - PlatformTransactionManager
  - "@Transactional"
  - Transactional
  - "@EnableTransactionManagement"
  - EnableTransactionManagement
  - jOOQ
  - TransactionalRunnable
  - DSLContext
  - EntityManagerFactory
  - SessionFactory
  - Hibernate
---
## Cпособы создания транзакций в Java-приложении

---

### 1. Нативный способ (JDBC)

Самый низкоуровневый подход — управление транзакциями через `Connection`.

```java
Connection conn = null;
try {
    conn = dataSource.getConnection();
    conn.setAutoCommit(false);  // отключаем автокоммит
    
    // Выполняем операции
    PreparedStatement stmt1 = conn.prepareStatement("UPDATE accounts SET balance = balance - 100 WHERE id = 1");
    stmt1.executeUpdate();
    
    PreparedStatement stmt2 = conn.prepareStatement("UPDATE accounts SET balance = balance + 100 WHERE id = 2");
    stmt2.executeUpdate();
    
    conn.commit();  // фиксируем транзакцию
} catch (SQLException e) {
    if (conn != null) {
        conn.rollback();  // откат
    }
    throw new RuntimeException(e);
} finally {
    if (conn != null) {
        conn.setAutoCommit(true);
        conn.close();
    }
}
```

**Особенности:**
- Полный контроль над транзакцией.
- Требуется явное управление `commit()` / `rollback()`.
- Сложность в обработке исключений и освобождении ресурсов.

---

### 2. JPA (Java Persistence API)

Управление транзакциями через `EntityTransaction`.

```java
EntityManager em = entityManagerFactory.createEntityManager();
EntityTransaction tx = em.getTransaction();
try {
    tx.begin();
    
    User user = em.find(User.class, 1L);
    user.setBalance(user.getBalance() - 100);
    em.persist(user);
    
    Order order = new Order();
    order.setUserId(1L);
    order.setAmount(100);
    em.persist(order);
    
    tx.commit();
} catch (Exception e) {
    if (tx.isActive()) {
        tx.rollback();
    }
    throw e;
} finally {
    em.close();
}
```

**Особенности:**
- Работает через `EntityManager`.
- Требует явного `begin()` / `commit()` / `rollback()`.
- Персистентность автоматически отслеживается JPA-провайдером.

---

### 3. Hibernate (родной API)

Через `Session` и `Transaction`.

```java
Session session = sessionFactory.openSession();
Transaction tx = null;
try {
    tx = session.beginTransaction();
    
    User user = session.get(User.class, 1L);
    user.setBalance(user.getBalance() - 100);
    session.update(user);
    
    Order order = new Order();
    order.setUserId(1L);
    order.setAmount(100);
    session.save(order);
    
    tx.commit();
} catch (Exception e) {
    if (tx != null) {
        tx.rollback();
    }
    throw e;
} finally {
    session.close();
}
```

**Особенности:**
- Родной Hibernate API.
- Более низкий уровень контроля.
- Использует `SessionFactory` вместо `EntityManagerFactory`.

---

### 4. jOOQ

Через `DSLContext` и `TransactionalRunnable`.

```java
DSLContext dsl = DSL.using(dataSource, SQLDialect.POSTGRES);
try {
    dsl.transaction(configuration -> {
        DSLContext ctx = DSL.using(configuration);
        
        ctx.update(ACCOUNTS)
            .set(ACCOUNTS.BALANCE, ACCOUNTS.BALANCE.subtract(100))
            .where(ACCOUNTS.ID.eq(1))
            .execute();
        
        ctx.update(ACCOUNTS)
            .set(ACCOUNTS.BALANCE, ACCOUNTS.BALANCE.add(100))
            .where(ACCOUNTS.ID.eq(2))
            .execute();
    });
} catch (Exception e) {
    // rollback автоматический при исключении
}
```

**Особенности:**
- Автоматический `commit` / `rollback` внутри лямбды.
- Типобезопасные SQL-запросы.
- Удобная обработка ошибок.

---

### 5. Spring MVC + @Transactional (декларативный)

Самый популярный способ в Spring-приложениях.

#### 🔧 Настройка

**1. Включение управления транзакциями:**

```java
@Configuration
@EnableTransactionManagement  // Включает обработку @Transactional
public class AppConfig {
    
    @Bean
    public DataSource dataSource() {
        // настройка DataSource
    }
    
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
    
    @Bean
    public JpaTransactionManager transactionManager(EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
}
```

**2. Использование @Transactional:**

```java
@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private OrderRepository orderRepository;
    
    public void transferMoney(Long userId, BigDecimal amount) {
        // Обе операции в одной транзакции
        User user = userRepository.findById(userId).orElseThrow();
        user.setBalance(user.getBalance().subtract(amount));
        userRepository.save(user);
        
        Order order = new Order();
        order.setUserId(userId);
        order.setAmount(amount);
        orderRepository.save(order);
    }
}
```

**3. Настройка параметров @Transactional:**

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    timeout = 30,
    rollbackFor = {SQLException.class, DataAccessException.class},
    noRollbackFor = {BusinessException.class},
    readOnly = true
)
public List<User> getUsers() {
    return userRepository.findAll();
}
```

---

### ✅ Как включить транзакции в контроллере?

**Способ 1: Аннотация на уровне метода контроллера**

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @PostMapping("/transfer")
    @Transactional  // ← транзакция открывается в контроллере
    public ResponseEntity<?> transferMoney(@RequestBody TransferRequest request) {
        userService.transferMoney(request.getUserId(), request.getAmount());
        return ResponseEntity.ok().build();
    }
}
```

**Способ 2: Через @Transactional на классе контроллера**

```java
@RestController
@RequestMapping("/api/orders")
@Transactional  // ← транзакция открывается для всех методов
public class OrderController {
    
    @PostMapping
    public Order createOrder(@RequestBody OrderRequest request) {
        // ...
    }
    
    @PutMapping("/{id}")
    public Order updateOrder(@PathVariable Long id, @RequestBody OrderRequest request) {
        // ...
    }
}
```

**Способ 3: Через AOP / Interceptor (глобально)**

```java
@Configuration
public class TransactionalConfig {
    
    @Bean
    public TransactionInterceptor transactionInterceptor(PlatformTransactionManager tm) {
        return new TransactionInterceptor(tm, new DefaultTransactionAttribute());
    }
    
    @Bean
    public Advisor transactionAdvisor(TransactionInterceptor interceptor) {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* com.example.controller.*.*(..))");
        return new DefaultPointcutAdvisor(pointcut, interceptor);
    }
}
```

---

### ⚠️ При каких условиях транзакция откатится (rollback)?

#### По умолчанию в Spring:

**Rollback происходит при:**
- Выброшено **непроверяемое исключение** (`RuntimeException` или его наследник).

**Rollback НЕ происходит при:**
- Выброшено **проверяемое исключение** (`Exception` или его наследник, кроме `RuntimeException`).
- Исключение перехвачено внутри метода (в `try-catch`).

---

#### Примеры:

```java
// ❌ Rollback НЕ сработает (checked exception)
@Transactional
public void process() throws SQLException {
    // ...
    throw new SQLException("DB error");  // checked → НЕ откатится
}

// ✅ Rollback сработает (unchecked exception)
@Transactional
public void process() {
    // ...
    throw new RuntimeException("Error");  // unchecked → откатится
}

// ❌ Rollback НЕ сработает (исключение перехвачено)
@Transactional
public void process() {
    try {
        // ...
        throw new RuntimeException("Error");
    } catch (Exception e) {
        System.out.println("Exception caught: " + e.getMessage());
        // Исключение проглочено → rollback НЕ будет
    }
}

// ✅ Rollback сработает (перехватили и перебросили)
@Transactional
public void process() {
    try {
        // ...
        throw new RuntimeException("Error");
    } catch (Exception e) {
        System.out.println("Exception caught: " + e.getMessage());
        throw e;  // пробрасываем дальше → rollback сработает
    }
}
```

---

#### Настройка rollbackFor / noRollbackFor:

```java
// Откат для любых исключений (включая checked)
@Transactional(rollbackFor = Exception.class)
public void process() throws SQLException {
    // ...
    throw new SQLException("DB error");
}

// Откат для конкретного исключения
@Transactional(rollbackFor = {SQLException.class, IOException.class})
public void process() throws SQLException, IOException {
    // ...
}

// Игнорировать rollback для указанных исключений
@Transactional(noRollbackFor = {BusinessException.class})
public void process() {
    // ...
    throw new BusinessException("Business error");  // rollback НЕ будет
}
```

---

### 🎯 Таблица: rollback в Spring

| Тип исключения | Аннотация | Rollback? |
|----------------|-----------|-----------|
| `RuntimeException` | `@Transactional` (по умолчанию) | ✅ Да |
| `Exception` (checked) | `@Transactional` (по умолчанию) | ❌ Нет |
| `RuntimeException` | `@Transactional(noRollbackFor = RuntimeException.class)` | ❌ Нет |
| `Exception` (checked) | `@Transactional(rollbackFor = Exception.class)` | ✅ Да |
| Любое исключение (перехвачено в catch) | `@Transactional` | ❌ Нет |
| Любое исключение (переброшено из catch) | `@Transactional` | ✅ Да |

---

### 📊 Сравнение способов

| Способ | Уровень | Автоматический rollback | Поддержка Spring |
|--------|---------|-------------------------|------------------|
| **JDBC (native)** | Низкий | ❌ Нет | ❌ Нет |
| **JPA EntityTransaction** | Средний | ❌ Нет | ⚠️ (через JPA) |
| **Hibernate Session** | Низкий | ❌ Нет | ⚠️ (через Hibernate) |
| **jOOQ** | Средний | ✅ Да (внутри лямбды) | ❌ Нет |
| **Spring @Transactional** | Высокий | ✅ Да (по умолчанию) | ✅ Да (родной) |

---

### 💡 Рекомендация

В **Spring-приложениях** используйте **`@Transactional` на уровне сервисов**, а не контроллеров. Это позволяет:
- Сохранять тонкую границу слоёв (контроллер → сервис → репозиторий).
- Легче тестировать бизнес-логику.
- Избегать проблем с сессиями (при использовании JPA).