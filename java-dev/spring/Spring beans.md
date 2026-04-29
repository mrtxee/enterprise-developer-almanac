

# spring bean scopes

Отлично!  
`@Scope` в Spring позволяет **управлять "областью видимости" бина**, то есть **сколько экземпляров** создаётся и **как долго они живут**.

---

## ✅ `@Scope` — что это?

**Область видимости (scope)** — это **жизненный цикл бина**:

- Сколько раз создаётся?
- Когда уничтожается?
- Кто его использует?

---

## ✅ Стандартные scopes в Spring

| Scope | Описание | Когда использовать |
|-------|----------|--------------------|
| `singleton` | **По умолчанию**. Один экземпляр на всё приложение | ✅ Всегда, если не нужен другой |
| `prototype` | Новый экземпляр при каждом `getBean()` или инъекции | ✅ Когда каждый вызов должен получать новый объект |
| `request` | Один экземпляр на HTTP-запрос | ✅ В веб-приложениях |
| `session` | Один экземпляр на HTTP-сессию | ✅ Для хранения данных пользователя |
| `application` | Один экземпляр на `ServletContext` | ✅ Для хранения данных приложения |
| `websocket` | Один экземпляр на WebSocket-соединение | ✅ В WebSocket-приложениях |

---

## ✅ 1. `singleton` (по умолчанию)

```java
@Component
public class MyService {
    // по умолчанию: @Scope("singleton")
    public void doSomething() {
        System.out.println("Вызов singleton-бина");
    }
}
```

> ✅ **Один экземпляр** на всё приложение  
> ✅ **Создаётся при старте** (если не `@Lazy`)

---

## ✅ 2. `prototype`

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
    private UniqueService service1; // ← разный ID

    public void test() {
        System.out.println(service1.getId()); // например: 123e4567-e89b-12d3-a456-426614174000

        // Вручную получить новый экземпляр:
        UniqueService anotherService = applicationContext.getBean(UniqueService.class);
        System.out.println(anotherService.getId()); // например: 123e4567-e89b-12d3-a456-426614174001
    }
}
```

> ✅ **Новый экземпляр при каждом вызове**  
> ❌ **Spring не управляет жизненным циклом** — `@PreDestroy` **не вызывается**!

---

## ✅ 3. `request` (для веб-приложений)

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

> ⚠️ **`proxyMode = ScopedProxyMode.TARGET_CLASS`** — **обязательно**, если бин инжектируется в `singleton` (а он почти всегда инжектируется в контроллер, который singleton).

```java
@RestController
public class MyController {

    @Autowired
    private RequestScopedService requestService; // ← будет новый экземпляр для каждого запроса

    @GetMapping("/request-id")
    public String getRequestId() {
        return requestService.getRequestId();
    }
}
```

> ✅ **Один экземпляр на HTTP-запрос**  
> ✅ **Уничтожается после завершения запроса**

---

## ✅ 4. `session` (для веб-приложений)

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

> ✅ **Один экземпляр на HTTP-сессию**  
> ✅ **Сохраняется между запросами одного пользователя**

---

## ✅ 5. `application` (в веб-приложениях)

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

> ✅ **Один экземпляр на `ServletContext`**  
> ✅ **Общий для всего веб-приложения**

---

## ✅ 6. `websocket` (в WebSocket-приложениях)

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

> ✅ **Один экземпляр на WebSocket-соединение**  
> ✅ **Живёт до разрыва соединения**

---

## ✅ Пример: `@Bean` с `@Scope`

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

## ✅ Важно: `proxyMode = ScopedProxyMode.TARGET_CLASS`

Если вы создаёте бин с `request`, `session`, `application`, `websocket` — и **инжектируете его в `singleton`** — **Spring не сможет это сделать напрямую**, потому что:

- `singleton` создаётся **один раз** при старте.
- `request` — **для каждого HTTP-запроса**.

→ Spring **создаёт прокси-объект**, который **делегирует вызовы нужному экземпляру**.

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestBean {
    // ...
}

@Service // ← singleton
public class MyService {

    @Autowired
    private RequestBean requestBean; // ← это будет прокси!
}
```

---

## ✅ Когда использовать каждый `@Scope`?

| Scope | Когда использовать |
|-------|--------------------|
| ✅ `singleton` | ✅ **По умолчанию**, если нет особых требований |
| ✅ `prototype` | ✅ Когда каждый вызов должен создавать **новый экземпляр** (например, DTO-билдер, счётчик) |
| ✅ `request` | ✅ Для хранения **данных одного HTTP-запроса** (например, `RequestContext`, `UserContext`) |
| ✅ `session` | ✅ Для хранения **данных пользователя между запросами** (например, `Cart`, `UserProfile`) |
| ✅ `application` | ✅ Для хранения **данных, общих для всего приложения** (например, `AppConfig`, `CacheManager`) |
| ✅ `websocket` | ✅ Для данных, специфичных для **одного WebSocket-соединения** |

---

## ✅ Плюсы и минусы `@Scope`

| Плюсы | Минусы |
|-------|--------|
| ✅ Гибкость | ❌ Сложнее тестировать |
| ✅ Правильное управление памятью | ❌ Сложнее отлаживать |
| ✅ Поддержка веб-сценариев (request, session) | ❌ `prototype` — `@PreDestroy` не работает |
| ✅ `proxyMode` — позволяет инжектировать в singleton | ❌ `proxyMode` — может замедлить вызовы |

---

## ✅ Резюме: `@Scope`

| Scope | Когда использовать |
|-------|--------------------|
| `singleton` | ✅ **По умолчанию**, почти всегда |
| `prototype` | ✅ Когда нужен **новый экземпляр каждый раз** |
| `request` | ✅ В веб-приложениях, **для данных запроса** |
| `session` | ✅ В веб-приложениях, **для данных сессии** |
| `application` | ✅ В веб-приложениях, **для данных приложения** |
| `websocket` | ✅ В WebSocket-приложениях |

---

✅ **Вывод**:
> ✅ **`@Scope` — мощный инструмент**, позволяющий **управлять жизненным циклом бинов**.  
> ✅ Используйте `singleton` по умолчанию, `prototype` — если нужен новый экземпляр, и `request/session` — в веб-приложениях.  
> ⚠️ Не забывайте про `proxyMode`, если инжектируете scoped-бин в singleton.

Удачи! 🚀
# все способы создать spring bean

Отличный вопрос!  
В Spring существует **много способов создать бин**, и у каждого — **своя область применения**, **плюсы**, **минусы**, и **лучшие практики**.

---

## ✅ Все способы создать Spring-бин

### 1. **`@Component` / `@Service` / `@Repository` / `@Controller`** ✅ (самый популярный)

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

---

### 2. **`@Bean` в `@Configuration` классе** ✅ (очень популярный)

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

---

### 3. **`@Bean` в `@Component` классе** ⚠️ (редко используется)

```java
@Component
public class MyComponentConfig {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

---

### 4. **`@Configuration` + `@Bean` с параметрами (инъекция других бинов)** ✅ (очень популярный)

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService(DataSource dataSource) {
        return new MyService(dataSource);
    }
}
```

---

### 5. **`@Import`** ✅ (для импорта других `@Configuration`)

```java
@Configuration
@Import(DataSourceConfig.class)
public class AppConfig {
    // ...
}
```

---

### 6. **`@Import` с классами, не помеченными `@Configuration`**

```java
@Configuration
@Import(MyService.class) // ← если MyService помечен @Component
public class AppConfig {
    // ...
}
```

---

### 7. **`@ComponentScan`** ✅ (автоматическое сканирование)

```java
@SpringBootApplication
@ComponentScan(basePackages = "com.example")
public class Application {
    // ...
}
```

---

### 8. **XML-конфигурация** ❌ (устаревший способ)

```xml
<beans>
    <bean id="myService" class="com.example.MyService"/>
</beans>
```

---

### 9. **`FactoryBean`** ⚠️ (редко, для сложных сценариев)

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

---

### 10. **`@ConditionalOn...` и другие условия** ✅ (для условного создания бинов)

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

---

### 11. **`@Profile`** ✅ (для условного создания бинов по профилю)

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

---

### 12. **`@Scope`** ✅ (для указания области видимости)

```java
@Component
@Scope("prototype") // или "request", "session", "singleton" (по умолчанию)
public class MyService {
    // ...
}
```

---

### 13. **`@Primary` и `@Qualifier`** ✅ (для выбора бина из нескольких)

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

---

### 14. **`@Lazy`** ✅ (ленивая инициализация бина)

```java
@Component
@Lazy
public class LazyService {
    // ...
}
```

---

### 15. **`@PostConstruct` и `@PreDestroy`** ✅ (для инициализации/уничтожения)

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

---

## ✅ Сравнение способов

| Способ | Когда использовать | Плюсы | Минусы |
|--------|--------------------|-------|--------|
| `@Component` / `@Service` / `@Repository` | Для обычных классов | ✅ Просто, автоматически сканируется | ❌ Менее гибкий |
| `@Bean` в `@Configuration` | Когда нужно настроить бин программно | ✅ Гибко, можно передавать зависимости | ❌ Более многословно |
| `@Import` | Когда нужно импортировать другие конфиги | ✅ Чистая структура | ❌ Может быть сложно отследить зависимости |
| `@Profile` | Для разных окружений | ✅ Гибкость | ❌ Усложняет тестирование |
| `@ConditionalOn...` | Условное создание бинов | ✅ Умная автоконфигурация | ❌ Сложно отлаживать |
| `FactoryBean` | Для сложной логики создания бина | ✅ Максимальная гибкость | ❌ Сложно |
| XML-конфигурация | Legacy-проекты | ❌ Устарело | ❌ Не читаемо, не типобезопасно |
| `@Scope` | Когда нужна не singleton область | ✅ Гибкость | ❌ Сложнее тестировать |
| `@Lazy` | Для ускорения старта приложения | ✅ Ускоряет старт | ❌ Ошибка при обращении к бину до инициализации |

---

## ✅ Когда какой способ использовать?

| Ситуация | Рекомендуемый способ |
|----------|----------------------|
| ✅ Обычный сервис, репозиторий, контроллер | ✅ `@Service`, `@Repository`, `@Controller` |
| ✅ Создание бина с параметрами или логикой | ✅ `@Bean` в `@Configuration` |
| ✅ Условное создание бина (по свойству, профилю) | ✅ `@ConditionalOn...`, `@Profile` |
| ✅ Импорт других конфигураций | ✅ `@Import` |
| ✅ Сложная логика создания объекта | ✅ `FactoryBean` |
| ✅ Управление областью видимости | ✅ `@Scope` |
| ✅ Ленивая инициализация | ✅ `@Lazy` |
| ✅ Автоконфигурация (Spring Boot) | ✅ `@Configuration` + `@ConditionalOn...` |

---

## ✅ Профессиональные советы

- ✅ **Используйте `@Component` / `@Service` / `@Repository`** — для обычных классов.
- ✅ **Используйте `@Bean` в `@Configuration`** — для бинов, требующих настройки.
- ✅ **Используйте `@Profile` и `@ConditionalOn...`** — для условных бинов.
- ❌ **Избегайте XML-конфигурации**, если не работаете с legacy.
- ✅ **Используйте `@RequiredArgsConstructor` от Lombok** — для инъекции зависимостей через конструктор.

---

## ✅ Итог

| Способ | Когда использовать |
|--------|--------------------|
| ✅ `@Component`, `@Service`, `@Repository`, `@Controller` | ✅ **Всегда**, когда бин — это бизнес-логика |
| ✅ `@Bean` в `@Configuration` | ✅ Когда нужно **настроить бин программно** |
| ✅ `@ConditionalOn...`, `@Profile` | ✅ Когда бин **зависит от окружения** |
| ✅ `@Import` | ✅ Для **структурирования конфигураций** |
| ✅ `FactoryBean` | ✅ Когда нужно **контролировать создание бина** |
| ❌ XML-конфигурация | ❌ **Только в legacy-коде** |
| ✅ `@Scope`, `@Lazy` | ✅ Когда нужно **управлять жизненным циклом бина** |

---

✅ **Вывод**:
> ✅ **`@Component` / `@Service` / `@Repository` / `@Controller`** — **по умолчанию**  
> ✅ **`@Bean` в `@Configuration`** — **когда нужна гибкость**  
> ✅ **`@Profile`, `@ConditionalOn...`** — **для условной логики**  
> ✅ **`FactoryBean`** — **только если нужно полное управление**  
> ❌ **XML-конфигурация** — **не использовать** в новых проектах

Удачи! 🚀
# все способы инжектить spring bean

Отличный вопрос!  
В Spring существует **несколько способов инжектировать один бин в другой**, и у каждого — **свои особенности**, **плюсы**, **минусы** и **лучшие практики**.

---

## ✅ Все способы инжектировать бин в бин

### 1. **Constructor Injection (через конструктор)** ✅ (рекомендуемый)

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) { // ← через конструктор
        this.userRepository = userRepository;
    }
}
```

#### ✅ Или с `@RequiredArgsConstructor` от Lombok:

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository; // ← Lombok генерирует конструктор
}
```

---

### 2. **Field Injection (через поле)** ❌ (не рекомендуется)

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository; // ← через поле
}
```

---

### 3. **Setter Injection (через сеттер)** ⚠️ (редко используется)

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) { // ← через сеттер
        this.userRepository = userRepository;
    }
}
```

---

### 4. **Method Injection (через метод)** ⚠️ (редко используется)

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void configure(UserRepository userRepository) { // ← через метод
        this.userRepository = userRepository;
    }
}
```

---

## ✅ Сравнение: Чем они отличаются?

| Способ | Безопасность | Тестируемость | Читаемость | Сложность | Циклические зависимости | Рекомендуется |
|--------|--------------|---------------|------------|-----------|--------------------------|---------------|
| **Constructor Injection** | ✅ Высокая (`final`, `null` невозможен) | ✅ Высокая (легко мокать) | ✅ Высокая (ясно, что нужно) | ✅ Простой | ❌ Нет (если не настроить иначе) | ✅ **Да** |
| **Field Injection** | ❌ Низкая (может быть `null`) | ❌ Сложно (Spring нужен для тестов) | ❌ Низкая (неочевидно, что нужно) | ✅ Простой | ✅ Да (Spring делает magic) | ❌ **Нет** |
| **Setter Injection** | ⚠️ Средняя (может быть `null`) | ⚠️ Средняя | ⚠️ Средняя | ✅ Простой | ✅ Да | ⚠️ **Только если нужна опциональная зависимость** |
| **Method Injection** | ⚠️ Средняя | ⚠️ Средняя | ⚠️ Низкая | ❌ Сложный | ✅ Да | ⚠️ **Только в особых случаях** |

---

## ✅ Когда какой способ использовать?

### ✅ 1. Constructor Injection (через конструктор) → **ВСЕГДА**

#### ✅ Когда использовать:
- ✅ Почти **всегда**, когда бин **обязательно нужен**.
- ✅ Если вы хотите **иммутабельность** (`final` поля).
- ✅ Если вы **тестируете** класс (легко передать моки в конструктор).
- ✅ Если вы хотите **явно указать зависимости**.

#### ✅ Пример:
```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository; // ← обязательная зависимость
    private final EmailService emailService;    // ← обязательная зависимость
}
```

---

### ❌ 2. Field Injection (через поле) → **НИКОГДА (или почти)**

#### ❌ Когда использовать:
- ❌ **Никогда**, если можно использовать конструктор.
- ❌ **Только в legacy-коде**, если переписать невозможно.
- ❌ **Только если вы используете Spring Test и `@Autowired` на полях тестов** — это исключение.

#### ❌ Почему не рекомендуется:
- ❌ Невозможно создать объект без Spring (в тестах).
- ❌ Сложно мокать зависимости.
- ❌ Поля могут быть `null`, если Spring не инжектирует.
- ❌ Сложно отследить зависимости (не видно в конструкторе).

---

### ⚠️ 3. Setter Injection (через сеттер) → **Только если зависимость опциональна**

#### ⚠️ Когда использовать:
- ⚠️ Когда зависимость **не обязательна**.
- ⚠️ Когда вы хотите **динамически изменить зависимость** (редко).

#### ⚠️ Пример:
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

---

### ⚠️ 4. Method Injection (через метод) → **Очень редко**

#### ⚠️ Когда использовать:
- ⚠️ Если нужно внедрить **несколько зависимостей за раз**.
- ⚠️ В **специфичных случаях**, например, для конфигурации.

#### ⚠️ Пример:
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

---

## ✅ Бонус: Циклические зависимости

### ❌ Constructor Injection и циклы
Если `UserService` зависит от `EmailService`, а `EmailService` от `UserService`:

```java
@Service
public class UserService {
    public UserService(EmailService emailService) { ... }
}

@Service
public class EmailService {
    public EmailService(UserService userService) { ... }
}
```

→ **Spring выбросит ошибку:**
```
The dependencies of some of the beans in the application context form a cycle
```

#### ✅ Как решить:
- ❌ Переделать архитектуру (цикл — признак плохого дизайна).
- ✅ Использовать `@Lazy`:
```java
@Service
public class UserService {
    public UserService(@Lazy EmailService emailService) { ... }
}
```

---

### ✅ Field Injection и циклы
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

→ **Spring "магически" решит цикл** (инжектирует бины через proxy)  
❌ Но это **плохо**, потому что:
- ❌ Скрывает проблему архитектуры
- ❌ Сложно тестировать

---

## ✅ Итог: Какой способ использовать?

| Ситуация | Рекомендуемый способ |
|----------|----------------------|
| ✅ Обычный бин с обязательными зависимостями | ✅ Constructor Injection (`final` поля + `@RequiredArgsConstructor`) |
| ✅ Бин с опциональной зависимостью | ⚠️ Setter Injection (редко) |
| ✅ В тестах (иногда) | ✅ Field Injection (для `@MockBean`, `@Autowired` на полях теста) |
| ✅ В legacy-коде | ✅ Постепенно мигрировать на Constructor Injection |
| ✅ Циклические зависимости | ❌ Переделать архитектуру или использовать `@Lazy` в конструкторе |

---

## ✅ Профессиональный совет

> ✅ **Всегда используйте Constructor Injection**.  
> ✅ Это **безопасно**, **тестируемо**, **понятно**, **явно** и **соответствует принципам DI**.  
> ✅ Используйте `@RequiredArgsConstructor` от Lombok, чтобы **не писать конструктор вручную**.

---

✅ **Вывод**:
- **Constructor Injection** → ✅ **Всегда**
- **Field Injection** → ❌ **Никогда**
- **Setter Injection** → ⚠️ **Только если зависимость опциональна**
- **Method Injection** → ⚠️ **Только в особых случаях**

Удачи! 🚀