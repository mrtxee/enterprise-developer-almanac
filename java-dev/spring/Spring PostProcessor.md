---
aliases:
  - BeanFactoryPostProcessor
  - BeanPostProcessor
  - Spring PostProcessor
---
**BeanFactoryPostProcessor** срабатывает **раньше всех** - еще до создания любых бинов. Это самый ранний этап в жизненном цикле Spring приложения.

## Полная временная шкала с BeanFactoryPostProcessor:

```
ПОРЯДОК ИНИЦИАЛИЗАЦИИ SPRING CONTEXT:
═══════════════════════════════════════════════
1. ApplicationContext создается
   │
2. BeanFactoryPostProcessor.postProcessBeanFactory()
   │    ↑
   │    └── Модификация BeanDefinition-ов
   │        (можно изменять метаданные бинов)
   │
3. BeanPostProcessor-ы регистрируются
   │    (но их методы еще не вызываются!)
   │
4. Создание singleton бинов (ленивых и не-ленивых)
   │    ├── Constructor
   │    ├── BeanPostProcessor.postProcessBeforeInitialization()
   │    ├── @PostConstruct
   │    ├── InitializingBean.afterPropertiesSet()
   │    ├── init-method
   │    └── BeanPostProcessor.postProcessAfterInitialization()
   │
5. ContextRefreshedEvent публикуется
   │
6. ApplicationRunner / CommandLineRunner выполняются
   │
7. Приложение работает
   │
8. ContextClosedEvent (при shutdown)
```

## Визуализация:

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}

@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        System.out.println("1. BeanFactoryPostProcessor выполняется ПЕРВЫМ!");
        // Здесь еще НЕТ созданных бинов
        // Но ЕСТЬ BeanDefinition'ы (метаданные будущих бинов)
    }
}

@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("3. BeanPostProcessor (before init)");
        return bean;
    }
}

@Component
public class MyService {
    
    public MyService() {
        System.out.println("2. Конструктор MyService");
    }
    
    @PostConstruct
    public void init() {
        System.out.println("4. @PostConstruct MyService");
    }
}
```

## Что происходит в `postProcessBeanFactory`:

```java
@Component
public class CustomBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) 
            throws BeansException {
        
        // 1. Можно получить ВСЕ определения бинов
        String[] beanNames = beanFactory.getBeanDefinitionNames();
        System.out.println("Всего BeanDefinition'ов: " + beanNames.length);
        
        // 2. Модифицировать существующие BeanDefinition'ы
        BeanDefinition myServiceDef = beanFactory.getBeanDefinition("myService");
        myServiceDef.setLazyInit(true); // Изменить на ленивую инициализацию
        myServiceDef.setScope("prototype"); // Изменить scope
        
        // 3. Добавить новые BeanDefinition'ы
        BeanDefinitionBuilder builder = BeanDefinitionBuilder
            .genericBeanDefinition(AdditionalService.class)
            .addPropertyValue("url", "http://example.com");
        
        beanFactory.registerBeanDefinition("additionalService", builder.getBeanDefinition());
        
        // 4. Удалить BeanDefinition'ы
        beanFactory.removeBeanDefinition("unwantedService");
        
        // 5. Изменить зависимости
        BeanDefinition beanDef = beanFactory.getBeanDefinition("someBean");
        beanDef.setDependsOn("firstBean"); // Установить порядок инициализации
        
        // 6. Работать с property placeholder'ами
        // (PropertySourcesPlaceholderConfigurer делает это)
    }
}
```

## Порядок выполнения нескольких BeanFactoryPostProcessor:

Если есть несколько BeanFactoryPostProcessor, они выполняются по приоритету:

```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE) // или implements PriorityOrdered
public class FirstBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    // Выполнится ПЕРВЫМ
}

@Component
@Order(Ordered.LOWEST_PRECEDENCE)
public class LastBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    // Выполнится ПОСЛЕДНИМ
}
```

## Разница между BeanFactoryPostProcessor и BeanPostProcessor:

| Аспект | BeanFactoryPostProcessor | BeanPostProcessor |
|--------|--------------------------|-------------------|
| **Когда срабатывает** | До создания любых бинов | До/после инициализации каждого бина |
| **Что получает** | ConfigurableListableBeanFactory (метаданные) | Сам бин-экземпляр |
| **Что можно делать** | Модифицировать BeanDefinition'ы | Модифицировать/заменить экземпляры бинов |
| **Количество вызовов** | Один раз на все приложение | Для КАЖДОГО создаваемого бина |

## Практические примеры использования:

### 1. **Динамическая регистрация бинов:**
```java
@Component
public class DynamicBeanRegistrar implements BeanFactoryPostProcessor {
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // Читаем конфигурацию и регистрируем бины
        List<String> serviceNames = readConfig();
        
        for (String serviceName : serviceNames) {
            BeanDefinition definition = BeanDefinitionBuilder
                .genericBeanDefinition(RemoteService.class)
                .addPropertyValue("endpoint", "http://api/" + serviceName)
                .setScope(BeanDefinition.SCOPE_SINGLETON)
                .getBeanDefinition();
            
            beanFactory.registerBeanDefinition(serviceName, definition);
        }
    }
}
```

### 2. **Модификация свойств бинов:**
```java
@Component
public class PropertyOverrideProcessor implements BeanFactoryPostProcessor {
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // Заменяем значения свойств из property placeholder'ов
        BeanDefinition bd = beanFactory.getBeanDefinition("dataSource");
        MutablePropertyValues pvs = bd.getPropertyValues();
        
        // Заменяем ${db.url} на реальное значение
        pvs.addPropertyValue("url", "jdbc:mysql://localhost:3306/mydb");
        pvs.addPropertyValue("username", "root");
        pvs.addPropertyValue("password", "secret");
    }
}
```

### 3. **Проверка конфигурации:**
```java
@Component
public class ConfigurationValidator implements BeanFactoryPostProcessor {
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // Проверяем, что обязательные бины сконфигурированы
        String[] requiredBeans = {"dataSource", "transactionManager", "entityManagerFactory"};
        
        for (String beanName : requiredBeans) {
            if (!beanFactory.containsBeanDefinition(beanName)) {
                throw new IllegalStateException("Отсутствует обязательный бин: " + beanName);
            }
        }
    }
}
```

### 4. **Профилирование bean definition'ов:**
```java
@Component
public class BeanDefinitionProfiler implements BeanFactoryPostProcessor {
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        String[] beanNames = beanFactory.getBeanDefinitionNames();
        
        System.out.println("=== Bean Definition Report ===");
        System.out.println("Total: " + beanNames.length);
        
        Map<String, Integer> scopes = new HashMap<>();
        Map<String, Integer> types = new HashMap<>();
        
        for (String beanName : beanNames) {
            BeanDefinition bd = beanFactory.getBeanDefinition(beanName);
            
            // Статистика по scope
            scopes.merge(bd.getScope(), 1, Integer::sum);
            
            // Статистика по классам
            String className = bd.getBeanClassName();
            if (className != null) {
                types.merge(className, 1, Integer::sum);
            }
        }
        
        System.out.println("Scopes: " + scopes);
        System.out.println("Most common types: " + 
            types.entrySet().stream()
                .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
                .limit(5)
                .collect(Collectors.toList()));
    }
}
```

### 5. **PropertySourcesPlaceholderConfigurer (встроенный в Spring):**
```java
@Configuration
public class PropertyConfig {
    
    @Bean
    public static BeanFactoryPostProcessor propertyPlaceholderConfigurer() {
        PropertySourcesPlaceholderConfigurer configurer = 
            new PropertySourcesPlaceholderConfigurer();
        configurer.setLocation(new ClassPathResource("application.properties"));
        return configurer;
    }
}
```

## Специальные BeanFactoryPostProcessor в Spring:

1. **`PropertySourcesPlaceholderConfigurer`** - заменяет `${...}` в значениях свойств
2. **`PropertyOverrideConfigurer`** - переопределяет свойства бинов
3. **`CustomScopeConfigurer`** - регистрирует кастомные scope'ы
4. **`ConfigurationClassPostProcessor`** - обрабатывает `@Configuration` классы

## Отладка порядка:

```java
@SpringBootApplication
public class DebugApp {
    public static void main(String[] args) {
        ConfigurableApplicationContext ctx = SpringApplication.run(DebugApp.class, args);
        
        // Выведет порядок BeanFactoryPostProcessor'ов
        String[] bfppNames = ctx.getBeanNamesForType(BeanFactoryPostProcessor.class);
        System.out.println("BeanFactoryPostProcessor'ы: " + Arrays.toString(bfppNames));
    }
}
```

**Ключевой вывод:** `BeanFactoryPostProcessor` работает на уровне **метаданных** бинов (BeanDefinition), а не на уровне **экземпляров**. Это позволяет менять конфигурацию приложения еще до того, как будут созданы какие-либо бины.

---
---

## **ContextRefreshedEvent**, **ApplicationRunner** и **CommandLineRunner** - это механизмы Spring для выполнения кода **после полной инициализации контекста**.

## 1. **ContextRefreshedEvent** (Событие)

```java
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;

@Component
public class StartupListener implements ApplicationListener<ContextRefreshedEvent> {
    
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        // Выполняется, когда ApplicationContext полностью инициализирован
        System.out.println("Контекст Spring обновлен/инициализирован!");
        
        ApplicationContext ctx = event.getApplicationContext();
        
        // Можно получать бины и выполнять логику
        MyService service = ctx.getBean(MyService.class);
        service.initialize();
    }
}
```

**Когда срабатывает:**
- После полной инициализации `ApplicationContext`
- При каждом обновлении контекста (refresh)
- Может срабатывать **несколько раз** в одном приложении (если контекст обновляется)

**Использовать, когда:**
- Нужно выполнить код при каждом обновлении контекста
- Для многоуровневых контекстов (parent/child)
- Для веб-приложений с `DispatcherServlet`

## 2. **ApplicationRunner** (Интерфейс)

```java
import org.springframework.boot.ApplicationRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(1) // Можно задать порядок
public class MyApplicationRunner implements ApplicationRunner {
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("ApplicationRunner выполняется!");
        
        // Аргументы командной строки
        System.out.println("Source args: " + args.getSourceArgs());
        System.out.println("Non-option args: " + args.getNonOptionArgs());
        System.out.println("Option names: " + args.getOptionNames());
        
        // Проверка наличия опции
        if (args.containsOption("debug")) {
            System.out.println("Debug mode enabled");
        }
        
        // Бизнес-логика инициализации
        initializeDatabase();
        startBackgroundJobs();
    }
    
    private void initializeDatabase() {
        // Инициализация БД
    }
}
```

## 3. **CommandLineRunner** (Интерфейс)

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class MyCommandLineRunner implements CommandLineRunner {
    
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner выполняется!");
        
        // args - массив строк аргументов командной строки
        for (int i = 0; i < args.length; i++) {
            System.out.println("arg[" + i + "]: " + args[i]);
        }
        
        // Инициализационная логика
        loadInitialData();
        validateConfiguration();
    }
}
```

## Сравнение всех трех:

| Критерий | ContextRefreshedEvent | ApplicationRunner | CommandLineRunner |
|----------|----------------------|------------------|-------------------|
| **Когда выполняется** | После инициализации контекста | После контекста, до запуска приложения | После контекста, до запуска приложения |
| **Аргументы** | Нет | `ApplicationArguments` (структурированные) | `String... args` (сырые) |
| **Порядок** | По событию | `@Order` аннотация | `@Order` аннотация |
| **Количество вызовов** | Может быть несколько (при refresh) | Один раз | Один раз |
| **Spring Boot** | Да (но из Spring Framework) | Да (только Boot) | Да (только Boot) |

## Порядок выполнения:

```
1. BeanFactoryPostProcessor
2. BeanPostProcessor регистрируются
3. Singleton бины создаются
4. @PostConstruct методы
5. ContextRefreshedEvent публикуется (много слушателей)
6. ApplicationRunner.run() (с @Order)
7. CommandLineRunner.run() (с @Order)
8. Приложение запущено
```

## Практические примеры использования:

### **Инициализация базы данных:**
```java
@Component
public class DatabaseInitializer implements ApplicationRunner {
    
    private final UserRepository userRepository;
    
    public DatabaseInitializer(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        if (userRepository.count() == 0) {
            User admin = new User("admin", "admin@example.com");
            userRepository.save(admin);
            System.out.println("Admin user created");
        }
    }
}
```

### **Запуск фоновых задач:**
```java
@Component
public class SchedulerStarter implements CommandLineRunner {
    
    private final TaskScheduler scheduler;
    
    @Override
    public void run(String... args) throws Exception {
        // Запускаем периодическую задачу
        scheduler.scheduleAtFixedRate(
            this::cleanupTempFiles,
            Duration.ofMinutes(30)
        );
    }
    
    private void cleanupTempFiles() {
        // Очистка временных файлов
    }
}
```

### **Проверка зависимостей:**
```java
@Component
public class DependencyChecker implements ApplicationListener<ContextRefreshedEvent> {
    
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        ApplicationContext ctx = event.getApplicationContext();
        
        // Проверяем, что все обязательные бины доступны
        checkBeanExists(ctx, "dataSource");
        checkBeanExists(ctx, "entityManagerFactory");
        checkBeanExists(ctx, "transactionManager");
        
        // Проверяем соединение с внешним сервисом
        checkExternalService();
    }
    
    private void checkBeanExists(ApplicationContext ctx, String beanName) {
        if (!ctx.containsBean(beanName)) {
            throw new IllegalStateException("Missing required bean: " + beanName);
        }
    }
}
```

### **Парсинг аргументов командной строки:**
```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class ArgumentParser implements ApplicationRunner {
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        // --mode=dev --port=8080 --debug
        String mode = args.getOptionValues("mode").get(0);
        String port = args.getOptionValues("port").get(0);
        boolean debug = args.containsOption("debug");
        
        System.setProperty("app.mode", mode);
        System.setProperty("server.port", port);
        
        if (debug) {
            Logger.setLevel(Level.DEBUG);
        }
    }
}
```

### **Несколько runner'ов с порядком:**
```java
@Component
@Order(1)
public class FirstRunner implements CommandLineRunner {
    @Override public void run(String... args) {
        System.out.println("First: Config validation");
    }
}

@Component  
@Order(2)
public class SecondRunner implements CommandLineRunner {
    @Override public void run(String... args) {
        System.out.println("Second: DB initialization");
    }
}

@Component
@Order(3)
public class ThirdRunner implements CommandLineRunner {
    @Override public void run(String... args) {
        System.out.println("Third: Start services");
    }
}
```

## Как отключить runner'ов:

```yaml
# application.yml
spring:
  main:
    web-application-type: none # для не-web приложений
```

Или через профили:

```java
@Component
@Profile("!test") // Не выполняется в тестах
public class ProductionRunner implements ApplicationRunner {
    // ...
}
```

## Дебаг и мониторинг:

```java
@Component
public class StartupMonitor implements ApplicationListener<ContextRefreshedEvent>,
                                         ApplicationRunner {
    
    private long startTime;
    
    @PostConstruct
    public void init() {
        startTime = System.currentTimeMillis();
    }
    
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        long contextReadyTime = System.currentTimeMillis();
        System.out.println("Context initialized in: " + 
                          (contextReadyTime - startTime) + "ms");
    }
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        long appReadyTime = System.currentTimeMillis();
        System.out.println("Application ready in: " + 
                          (appReadyTime - startTime) + "ms");
        
        // Статистика бинов
        ApplicationContext ctx = SpringApplication.run(App.class, args)
            .getApplicationContext();
        System.out.println("Total beans: " + ctx.getBeanDefinitionCount());
    }
}
```

## Резюме:

- **ContextRefreshedEvent** - когда нужно реагировать на инициализацию/обновление контекста
- **ApplicationRunner** - для структурированной работы с аргументами командной строки
- **CommandLineRunner** - для простой работы с сырыми аргументами командной строки

**Рекомендация:** Используйте `ApplicationRunner` для большинства задач инициализации в Spring Boot приложениях. Он предоставляет более удобный доступ к аргументам командной строки.