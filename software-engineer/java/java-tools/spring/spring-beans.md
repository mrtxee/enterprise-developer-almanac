---
aliases:
  - Application Scope
  - Bean
  - Bean Scope
  - Bean Scopes
  - Beans
  - Circular dependencies
  - Component
  - ComponentScan
  - ConditionalOn
  - Configuration
  - Constructor Injection
  - Controller
  - Dependency Injection
  - FactoryBean
  - Field Injection
  - Injection
  - Lazy
  - Method Injection
  - PostConstruct
  - PreDestroy
  - Primary
  - Profile
  - Prototype
  - Qualifier
  - Repository
  - Request
  - RequiredArgsConstructor
  - Scope
  - ScopedProxyMode
  - Service
  - Session
  - Setter Injection
  - Singleton
  - Spring
  - Spring Bean
  - Spring Beans
  - WebSocket
  - XML Configuration
  - XML-конфигурация
  - Бин
  - Бин-scope
  - Внедрение зависимостей
  - Инъекция через метод
  - Инъекция через поле
  - Инъекция через сеттер
  - Конструкторная инъекция
  - Область видимости бина
  - Скоуп
  - Циклические зависимости
---

## Область видимости бина в Spring

**Суть**

`@Scope` в Spring управляет областью видимости бина: сколько экземпляров создаётся и как долго они живут. Область видимости (scope) — это жизненный цикл бина:

- Сколько раз создаётся?
- Когда уничтожается?
- Кто его использует?

---

### Стандартные scopes в Spring

| Scope | Описание | Когда использовать |
|-------|----------|--------------------|
| `singleton` | По умолчанию. Один экземпляр на всё приложение | Всегда, если не нужен другой |
| `prototype` | Новый экземпляр при каждом `getBean()` или инъекции | Когда каждый вызов должен получать новый объект |
| `request` | Один экземпляр на HTTP-запрос | В веб-приложениях |
| `session` | Один экземпляр на HTTP-сессию | Для хранения данных пользователя |
| `application` | Один экземпляр на `ServletContext` | Для хранения данных приложения |
| `websocket` | Один экземпляр на WebSocket-соединение | В WebSocket-приложениях |

---

### singleton

По умолчанию создаётся один экземпляр на всё приложение.

**Пример на Java**

```java
@Component
public class MyService {
    // по умолчанию: @Scope("singleton")
    public void doSomething() {
        System.out.println("Вызов singleton-бина");
    }
}
```

**Особенности**

- ✅ Один экземпляр на всё приложение
- ✅ Создаётся при старте (если не `@Lazy`)

---

### prototype

При каждом запросе создаётся новый экземпляр.

**Пример на Java**

```java
@Component
@Scope("prototype")
public class UniqueService {
    private final String id = UUID.randomUUID().toString();

    public String getId() {
        return id;
    }
}
```

```java
@Service
public class MainService {

    @Autowired
    private UniqueService service1; // разные ID

    public void test() {
        System.out.println(service1.getId());

        // Вручную получить новый экземпляр:
        UniqueService anotherService = applicationContext.getBean(UniqueService.class);
        System.out.println(anotherService.getId());
    }
}
```

**Особенности**

- ✅ Новый экземпляр при каждом вызове
- ❌ Spring не управляет жизненным циклом: `@PreDestroy` не вызывается

---

### request

Для веб-приложений: один экземпляр на HTTP-запрос.

**Пример на Java**

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedService {

    private String requestId = UUID.randomUUID().toString();

    public String getRequestId() {
        return requestId;
    }

    public void setRequestData(String data) {
        // данные запроса
    }
}
```

`proxyMode = ScopedProxyMode.TARGET_CLASS` обязательно, если бин инжектируется в `singleton` (а он почти всегда инжектируется в контроллер, который singleton).

```java
@RestController
public class MyController {

    @Autowired
    private RequestScopedService requestService; // новый экземпляр для каждого запроса

    @GetMapping("/request-id")
    public String getRequestId() {
        return requestService.getRequestId();
    }
}
```

**Особенности**

- ✅ Один экземпляр на HTTP-запрос
- ✅ Уничтожается после завершения запроса

---

### session

Для веб-приложений: один экземпляр на HTTP-сессию.

**Пример на Java**

```java
@Component
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class SessionScopedService {

    private String sessionId = UUID.randomUUID().toString();
    private String username;

    public void setUsername(String username) {
        this.username = username;
    }

    public String getUsername() {
        return username;
    }
}
```

```java
@RestController
public class LoginController {

    @Autowired
    private SessionScopedService sessionService;

    @PostMapping("/login")
    public String login(@RequestParam String username) {
        sessionService.setUsername(username);
        return "Logged in as: " + username;
    }

    @GetMapping("/whoami")
    public String whoami() {
        return "User: " + sessionService.getUsername();
    }
}
```

**Особенности**

- ✅ Один экземпляр на HTTP-сессию
- ✅ Сохраняется между запросами одного пользователя

---

### application

В веб-приложениях: один экземпляр на `ServletContext`.

**Пример на Java**

```java
@Component
@Scope(value = "application", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ApplicationScopedService {

    private final long startTime = System.currentTimeMillis();

    public long getUptime() {
        return System.currentTimeMillis() - startTime;
    }
}
```

```java
@RestController
public class StatusController {

    @Autowired
    private ApplicationScopedService appService;

    @GetMapping("/uptime")
    public String uptime() {
        return "Uptime: " + appService.getUptime() + " ms";
    }
}
```

**Особенности**

- ✅ Один экземпляр на `ServletContext`
- ✅ Общий для всего веб-приложения

---

### websocket

В WebSocket-приложениях: один экземпляр на WebSocket-соединение.

**Пример на Java**

```java
@Component
@Scope(value = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class WebSocketSessionService {

    private final String connectionId = UUID.randomUUID().toString();

    public String getConnectionId() {
        return connectionId;
    }
}
```

**Особенности**

- ✅ Один экземпляр на WebSocket-соединение
- ✅ Живёт до разрыва соединения

---

### Декларация бина с `@Scope` через `@Bean`

**Пример на Java**

```java
@Configuration
public class BeanConfig {

    @Bean
    @Scope("prototype")
    public MyPrototypeBean myPrototypeBean() {
        return new MyPrototypeBean();
    }

    @Bean
    @Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
    public MyRequestBean myRequestBean() {
        return new MyRequestBean();
    }
}
```

---

### proxyMode = ScopedProxyMode.TARGET_CLASS

Если бин с `request`, `session`, `application`, `websocket` инжектируется в `singleton`, Spring не может сделать это напрямую:

- `singleton` создаётся один раз при старте.
- `request` создаётся для каждого HTTP-запроса.

Spring создаёт прокси-объект, который делегирует вызовы нужному экземпляру.

**Пример на Java**

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestBean {
    // ...
}

@Service // singleton
public class MyService {

    @Autowired
    private RequestBean requestBean; // это будет прокси
}
```

---

### Выбор области видимости

| Scope | Когда использовать |
|-------|--------------------|
| `singleton` | По умолчанию, если нет особых требований |
| `prototype` | Когда каждый вызов должен создавать новый экземпляр (например, DTO-билдер, счётчик) |
| `request` | Для хранения данных одного HTTP-запроса (например, `RequestContext`, `UserContext`) |
| `session` | Для хранения данных пользователя между запросами (например, `Cart`, `UserProfile`) |
| `application` | Для хранения данных, общих для всего приложения (например, `AppConfig`, `CacheManager`) |
| `websocket` | Для данных, специфичных для одного WebSocket-соединения |

---

### Плюсы и минусы `@Scope`

| Плюсы | Минусы |
|-------|--------|
| ✅ Гибкость | ❌ Сложнее тестировать |
| ✅ Правильное управление памятью | ❌ Сложнее отлаживать |
| ✅ Поддержка веб-сценариев (request, session) | ❌ `prototype` — `@PreDestroy` не работает |
| ✅ `proxyMode` позволяет инжектировать в singleton | ❌ `proxyMode` может замедлить вызовы |

---

## Способы создания бина в Spring

**Суть**

В Spring существует много способов создать бин, у каждого — своя область применения, плюсы, минусы и лучшие практики.

### Создание через стереотипные аннотации

`@Component`, `@Service`, `@Repository`, `@Controller` — самый популярный способ.

**Пример на Java**

```java
@Component
public class MyService {
    // ...
}

@Service
public class UserService {
    // ...
}

@Repository
public class UserRepository {
    // ...
}

@Controller
public class UserController {
    // ...
}
```

### `@Bean` в `@Configuration` классе

Очень популярный способ.

**Пример на Java**

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

### `@Bean` в `@Component` классе

Редко используется.

**Пример на Java**

```java
@Component
public class MyComponentConfig {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

### `@Configuration` + `@Bean` с параметрами

`@Bean`-методы принимают другие бины как параметры. Очень популярный способ.

**Пример на Java**

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService(DataSource dataSource) {
        return new MyService(dataSource);
    }
}
```

### `@Import` — импорт конфигураций

Импортирует другие `@Configuration`-классы, а также классы, не помеченные `@Configuration`.

**Пример на Java**

```java
@Configuration
@Import(DataSourceConfig.class)
public class AppConfig {
    // ...
}
```

```java
@Configuration
@Import(MyService.class) // если MyService помечен @Component
public class AppConfig {
    // ...
}
```

### `@ComponentScan` — автоматическое сканирование

**Пример на Java**

```java
@SpringBootApplication
@ComponentScan(basePackages = "com.example")
public class Application {
    // ...
}
```

### XML-конфигурация

Устаревший способ.

**Пример XML**

```xml
<beans>
    <bean id="myService" class="com.example.MyService"/>
</beans>
```

### `FactoryBean` — сложные сценарии

Редко, для сложных сценариев.

**Пример на Java**

```java
@Component
public class MyFactoryBean implements FactoryBean<MyService> {

    @Override
    public MyService getObject() throws Exception {
        return new MyService();
    }

    @Override
    public Class<?> getObjectType() {
        return MyService.class;
    }
}
```

### Условное создание бинов: `@ConditionalOn...` и `@Profile`

`@ConditionalOn...` создаёт бин по условию, `@Profile` — по профилю.

**Пример на Java**

```java
@Configuration
public class ConditionalConfig {

    @Bean
    @ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
    public MyService myService() {
        return new MyService();
    }
}
```

```java
@Configuration
public class AppConfig {

    @Bean
    @Profile("dev")
    public MyService devService() {
        return new DevService();
    }

    @Bean
    @Profile("prod")
    public MyService prodService() {
        return new ProdService();
    }
}
```

### `@Scope` — область видимости

**Пример на Java**

```java
@Component
@Scope("prototype") // или "request", "session", "singleton" (по умолчанию)
public class MyService {
    // ...
}
```

### `@Primary` и `@Qualifier` — выбор бина из нескольких

**Пример на Java**

```java
@Component
@Primary
public class DefaultService implements MyService {
    // ...
}

@Component
public class SpecialService implements MyService {
    // ...
}
```

### `@Lazy` — ленивая инициализация

**Пример на Java**

```java
@Component
@Lazy
public class LazyService {
    // ...
}
```

### `@PostConstruct` и `@PreDestroy` — инициализация и уничтожение

**Пример на Java**

```java
@Component
public class MyService {

    @PostConstruct
    public void init() {
        // ...
    }

    @PreDestroy
    public void destroy() {
        // ...
    }
}
```

### Сравнение способов создания

| Способ | Когда использовать | Плюсы | Минусы |
|--------|--------------------|-------|--------|
| `@Component` / `@Service` / `@Repository` | Для обычных классов | ✅ Просто, автоматически сканируется | ❌ Менее гибкий |
| `@Bean` в `@Configuration` | Когда нужно настроить бин программно | ✅ Гибко, можно передавать зависимости | ❌ Более многословно |
| `@Import` | Когда нужно импортировать другие конфиги | ✅ Чистая структура | ❌ Сложно отследить зависимости |
| `@Profile` | Для разных окружений | ✅ Гибкость | ❌ Усложняет тестирование |
| `@ConditionalOn...` | Условное создание бинов | ✅ Умная автоконфигурация | ❌ Сложно отлаживать |
| `FactoryBean` | Для сложной логики создания бина | ✅ Максимальная гибкость | ❌ Сложно |
| XML-конфигурация | Legacy-проекты | ❌ Устарел | ❌ Не читаемо, не типобезопасно |
| `@Scope` | Когда нужна область не singleton | ✅ Гибкость | ❌ Сложнее тестировать |
| `@Lazy` | Для ускорения старта приложения | ✅ Ускоряет старт | ❌ Ошибка при обращении к бину до инициализации |

### Выбор способа создания

| Ситуация | Рекомендуемый способ |
|----------|----------------------|
| Обычный сервис, репозиторий, контроллер | `@Service`, `@Repository`, `@Controller` |
| Создание бина с параметрами или логикой | `@Bean` в `@Configuration` |
| Условное создание бина (по свойству, профилю) | `@ConditionalOn...`, `@Profile` |
| Импорт других конфигураций | `@Import` |
| Сложная логика создания объекта | `FactoryBean` |
| Управление областью видимости | `@Scope` |
| Ленивая инициализация | `@Lazy` |

**Рекомендации**

- ✅ Используйте стереотипные аннотации для обычных классов.
- ✅ Используйте `@Bean` в `@Configuration` для бинов, требующих настройки.
- ✅ Используйте `@Profile` и `@ConditionalOn...` для условных бинов.
- ❌ Избегайте XML-конфигурации, если не работаете с legacy.
- ✅ Используйте `@RequiredArgsConstructor` от Lombok для инъекции зависимостей через конструктор.

---

## Способы инъекции бина в Spring

**Суть**

В Spring существует несколько способов инжектировать один бин в другой, у каждого — свои особенности, плюсы, минусы и лучшие практики.

### Constructor Injection

Рекомендуемый способ.

**Пример на Java**

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) { // через конструктор
        this.userRepository = userRepository;
    }
}
```

Или с `@RequiredArgsConstructor` от Lombok:

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository; // Lombok генерирует конструктор
}
```

### Field Injection

Не рекомендуется.

**Пример на Java**

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository; // через поле
}
```

### Setter Injection

Редко используется.

**Пример на Java**

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) { // через сеттер
        this.userRepository = userRepository;
    }
}
```

### Method Injection

Редко используется.

**Пример на Java**

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void configure(UserRepository userRepository) { // через метод
        this.userRepository = userRepository;
    }
}
```

### Сравнение способов инъекции

| Способ | Безопасность | Тестируемость | Читаемость | Сложность | Циклические зависимости | Рекомендуется |
|--------|--------------|---------------|------------|-----------|--------------------------|---------------|
| Constructor Injection | ✅ Высокая (`final`, `null` невозможен) | ✅ Высокая (легко мокать) | ✅ Высокая (ясно, что нужно) | ✅ Простой | ❌ Нет (если не настроить иначе) | ✅ Да |
| Field Injection | ❌ Низкая (может быть `null`) | ❌ Сложно (Spring нужен для тестов) | ❌ Низкая (неочевидно, что нужно) | ✅ Простой | ✅ Да (Spring делает magic) | ❌ Нет |
| Setter Injection | ⚠️ Средняя (может быть `null`) | ⚠️ Средняя | ⚠️ Средняя | ✅ Простой | ✅ Да | ⚠️ Только если нужна опциональная зависимость |
| Method Injection | ⚠️ Средняя | ⚠️ Средняя | ⚠️ Низкая | ❌ Сложный | ✅ Да | ⚠️ Только в особых случаях |

### Выбор способа инъекции

**Constructor Injection — всегда**, так как даёт иммутабельность (`final`-поля), явно указывает зависимости и легко тестируется.

**Field Injection — никогда (или почти)**, кроме legacy-кода и тестов:

- ❌ Невозможно создать объект без Spring (в тестах).
- ❌ Сложно мокать зависимости.
- ❌ Поля могут быть `null`, если Spring не инжектирует.
- ❌ Сложно отследить зависимости (не видно в конструкторе).

**Setter Injection — только если зависимость опциональна** или её нужно динамически менять.

**Method Injection — очень редко**, для внедрения нескольких зависимостей за раз или конфигурации.

**Примеры на Java**

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository; // обязательная зависимость
    private final EmailService emailService;    // обязательная зависимость
}
```

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setup(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
    }
}
```

### Циклические зависимости

Если `UserService` зависит от `EmailService`, а `EmailService` — от `UserService`, при конструкторной инъекции Spring выбросит ошибку:

```java
@Service
public class UserService {
    public UserService(EmailService emailService) { ... }
}

@Service
public class EmailService {
    public UserService(UserService userService) { ... }
}
```

```
The dependencies of some of the beans in the application context form a cycle
```

**Решение**

- ❌ Переделать архитектуру (цикл — признак плохого дизайна).
- ✅ Использовать `@Lazy` в конструкторе:

```java
@Service
public class UserService {
    public UserService(@Lazy EmailService emailService) { ... }
}
```

Field Injection решает цикл через proxy «магически», но скрывает проблему архитектуры и усложняет тестирование:

```java
@Service
public class UserService {
    @Autowired
    private EmailService emailService;
}

@Service
public class EmailService {
    @Autowired
    private UserService userService;
}
```

### Итоговая рекомендация

| Ситуация | Рекомендуемый способ |
|----------|----------------------|
| Обычный бин с обязательными зависимостями | Constructor Injection (`final`-поля + `@RequiredArgsConstructor`) |
| Бин с опциональной зависимостью | Setter Injection (редко) |
| В тестах (иногда) | Field Injection (`@MockBean`, `@Autowired` на полях теста) |
| В legacy-коде | Постепенно мигрировать на Constructor Injection |
| Циклические зависимости | Переделать архитектуру или использовать `@Lazy` в конструкторе |

**Рекомендации**

- ✅ Всегда используйте Constructor Injection: это безопасно, тестируемо, понятно, явно и соответствует принципам DI.
- ✅ Используйте `@RequiredArgsConstructor` от Lombok, чтобы не писать конструктор вручную.
