---
aliases:
  - Spring
  - Spring Cloud Config
---
## 📘 Spring Cloud Config: Полное руководство

**Spring Cloud Config** — это решение для **централизованного управления конфигурациями** в распределённых системах (микросервисах). Оно позволяет выносить настройки (`application.yml`, `.properties`) из кода приложений во внешний репозиторий и управлять ими динамически.

---

## 🎯 Зачем он нужен?

| Проблема без Config | Решение с Spring Cloud Config |
|---------------------|-------------------------------|
| 🔴 Конфиги размазаны по 20+ сервисам | 🟢 Единый источник истины (Git/ SVN/ Vault) |
| 🔴 Изменение конфига → пересборка → redeploy | 🟢 Hot-reload через `/refresh` или Spring Cloud Bus |
| 🔴 Секреты в коде или env-переменных | 🟢 Шифрование значений (`{cipher}...`) + интеграция с Vault |
| 🔴 Разные конфиги для dev/stage/prod вручную | 🟢 Профили + labels + версионирование в Git |
| 🔴 Невозможно откатить конфиг быстро | 🟢 `git revert` → мгновенный откат для всех сервисов |

> 💡 **Идея:** Конфигурация — это код. Храните её в Git, версионируйте, ревьювьте, тестируйте.

---

## 🏗 Архитектура

```
┌─────────────────┐
│   Config Server │  ← Читает конфиги из Git / SVN / Vault / JDBC
│   (Spring Boot) │     Экспонирует REST API: /{app}/{profile}/{label}
└────────┬────────┘
         │ HTTP/HTTPS
         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Service A      │     │  Service B      │     │  Service C      │
│  (Config Client)│     │  (Config Client)│     │  (Config Client)│
│  @RefreshScope  │     │  @RefreshScope  │     │  @RefreshScope  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │                       │                       │
         └───────────────────────┴───────────────────────┘
                                 │
                    ┌─────────────────────────┐
                    │  Spring Cloud Bus       │
                    │  (RabbitMQ / Kafka)     │
                    │  → broadcast /refresh   │
                    └─────────────────────────┘
```

---

## ⚙️ 1. Настройка Config Server

### 🔹 Зависимости (`pom.xml`)
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<!-- Опционально: шифрование -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-crypto</artifactId>
</dependency>
```

### 🔹 Главный класс
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer  // ✅ Включаем сервер конфигураций
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

### 🔹 `application.yml` сервера
```yaml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo.git
          username: ${GIT_USERNAME}
          password: ${GIT_PASSWORD}  # Лучше использовать token
          default-label: main        # Ветка по умолчанию
          search-paths: '{application}' # Опционально: папки по имени сервиса
          
          # Кэширование (production must-have)
          clone-on-start: true
          timeout: 5
          force-pull: true

# 🔐 Шифрование (опционально, но рекомендуется)
encrypt:
  key: ${ENCRYPT_KEY:my-secret-key}  # Или использовать keystore
```

### 🔹 Структура репозитория конфигураций
```
config-repo/
├── application.yml              # Общие настройки для всех сервисов
├── application-prod.yml         # Общие настройки для prod
├── loans-service.yml            # Конфиг для сервиса loans-service
├── loans-service-prod.yml       # Конфиг для loans-service в prod
├── notifications-service.yml
└── ...
```

---

## 🧩 2. Настройка Config Client (микросервиса)

### 🔹 Зависимости
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId> <!-- Для /refresh -->
</dependency>
```

### 🔹 `bootstrap.yml` (или `application.yml` в Spring Boot 2.4+)
> ⚠️ В Spring Boot 2.4+ `bootstrap.yml` отключён по умолчанию. Включите:
> ```xml
> <dependency>
>     <groupId>org.springframework.cloud</groupId>
>     <artifactId>spring-cloud-starter-bootstrap</artifactId>
> </dependency>
> ```

```yaml
spring:
  application:
    name: loans-service          # ✅ Имя приложения = имя файла конфига
  profiles:
    active: prod                 # ✅ Профиль = суффикс файла (loans-service-prod.yml)
  cloud:
    config:
      uri: http://config-server:8888  # Адрес Config Server
      fail-fast: true                  # ❌ Не запускаться, если конфиг не получен
      retry:
        max-attempts: 6
        initial-interval: 1000
        max-interval: 2000
        multiplier: 1.1
      # 🔐 Базовая аутентификация (если сервер защищён)
      username: ${CONFIG_USER}
      password: ${CONFIG_PASS}
```

### 🔹 Использование в коде
```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.*;

@RestController
@RefreshScope  // ✅ Бин будет пересоздан при POST /refresh
public class ConfigController {

    @Value("${app.feature.enabled:false}")
    private boolean featureEnabled;

    @Value("${app.database.timeout:30000}")
    private int dbTimeout;

    @GetMapping("/config")
    public Map<String, Object> getConfig() {
        return Map.of(
            "featureEnabled", featureEnabled,
            "dbTimeout", dbTimeout
        );
    }
}
```

---

## 🔄 3. Динамическое обновление конфигурации (`@RefreshScope`)

### 🔹 Как работает:
1. Вы меняете конфиг в Git → пушите изменения
2. Отправляете запрос: `POST http://service:8080/actuator/refresh`
3. Spring помечает все `@RefreshScope` бины как "устаревшие"
4. При следующем обращении бин пересоздаётся с новыми значениями из Config Server

### 🔹 Альтернатива: Spring Cloud Bus (broadcast на все сервисы)
```yaml
# В config-server и всех клиентах:
spring:
  cloud:
    bus:
      enabled: true
      refresh:
        enabled: true
  rabbitmq:  # или kafka
    host: localhost
    port: 5672
```

→ Отправляете **один** запрос: `POST /actuator/busrefresh` → все сервисы обновляются.

### 🔹 Webhook для авто-обновления (GitHub/GitLab)
```java
@RestController
public class ConfigWebhookController {

    private final ContextRefresher refresher;
    
    @PostMapping("/webhook/config-change")
    public ResponseEntity<?> onConfigChange(
            @RequestBody Map<String, Object> payload,
            @RequestHeader("X-Gitlab-Event") String event) {
        
        if ("Push Hook".equals(event)) {
            // ✅ Принудительно обновляем конфиги
            Set<String> keys = refresher.refresh();
            return ResponseEntity.ok(Map.of("refreshed", keys));
        }
        return ResponseEntity.badRequest().build();
    }
}
```

---

## 🔐 4. Шифрование чувствительных данных

### 🔹 Генерация зашифрованного значения:
```bash
# Через REST API Config Server
curl -X POST http://config-server:8888/encrypt \
  -d 'my-secret-password' \
  -H 'Content-Type: text/plain'

# Ответ: a3d8f7b2c1e9... (зашифрованное значение)
```

### 🔹 Использование в конфиге:
```yaml
# loans-service-prod.yml
spring:
  datasource:
    password: '{cipher}a3d8f7b2c1e9...'  # Config Server расшифрует на лету
```

### 🔹 Настройка ключа шифрования:
```yaml
# Вариант 1: симметричный ключ
encrypt:
  key: my-strong-secret-key-min-32-chars

# Вариант 2: keystore (production)
encrypt:
  key-store:
    location: classpath:config-server.jks
    password: ${KEYSTORE_PASSWORD}
    alias: configkey
    secret: ${KEY_SECRET}
```

> ⚠️ **Никогда не храните `encrypt.key` в Git!** Используйте env-переменные или Vault.

---

## 🛡️ 5. Безопасность Config Server

| Угроза | Решение |
|--------|---------|
| 🔓 Несанкционированный доступ к конфигам | Basic Auth / OAuth2 + HTTPS |
| 🔑 Утечка ключа шифрования | Хранить в Vault / Kubernetes Secrets, ротация ключей |
| 🕵️ Чтение конфигов других сервисов | Изоляция репозиториев, RBAC в Git |
| 🧨 DoS через частые запросы | Rate limiting, кэширование (`spring.cloud.config.server.git.timeout`) |
| 📦 Уязвимости в зависимостях | Регулярный `mvn dependency-check`, SCA-сканирование |

### 🔹 Пример защиты Basic Auth:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }

    @Bean
    public UserDetailsService users() {
        UserDetails user = User.builder()
            .username("config-client")
            .password("{noop}client-secret")  // В prod используйте {bcrypt}...
            .roles("CLIENT")
            .build();
        return new InMemoryUserDetailsManager(user);
    }
}
```

---

## 🚀 6. Production Best Practices

| Практика | Зачем |
|----------|-------|
| ✅ **Кэширование Git-репозитория** | `clone-on-start: true`, `cache-ttl` — ускоряет старт, снижает нагрузку на Git |
| ✅ **Fail-fast + retry** | Не запускать сервис без конфига, но давать время на восстановление сети |
| ✅ **Health check для Config Server** | `/actuator/health/config` → мониторинг доступности |
| ✅ **Версионирование через Git tags** | `label: v1.2.3` в `bootstrap.yml` → детерминированные деплои |
| ✅ **Тестирование конфигов в CI** | Запускать `config-server` в тестовом профиле, валидировать синтаксис YAML |
| ✅ **Аудит изменений** | Git history + PR review + `git log -p config-repo/` |
| ✅ **Резервное копирование репозитория** | Git remote mirror, automated backup |

### 🔹 Пример production `bootstrap.yml`:
```yaml
spring:
  application:
    name: loans-service
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:prod}
  cloud:
    config:
      uri: https://config.internal:8888
      fail-fast: true
      retry:
        max-attempts: 10
        initial-interval: 2000
        max-interval: 5000
      encryption:
        enabled: true
      # TLS
      tls:
        enabled: true
        verify-hostname: true
```

---

## 🆚 Альтернативы Spring Cloud Config

| Решение | Плюсы | Минусы | Когда выбирать |
|---------|-------|--------|----------------|
| **Spring Cloud Config** | Git-native, шифрование, refresh, экосистема Spring | Требует отдельный сервер, сложнее в настройке | Микросервисы на Spring, нужен Git как source of truth |
| **Kubernetes ConfigMaps/Secrets** | Нативно в K8s, не требует доп. инфраструктуры | Нет версионирования, hot-reload сложнее | Всё уже в K8s, небольшие проекты |
| **HashiCorp Vault** | Секреты + динамические креды, аудит, ротация | Сложнее, не для обычных конфигов | Высокие требования к безопасности, compliance |
| **AWS AppConfig / Parameter Store** | Managed, интеграция с AWS | Vendor lock, меньше фич | Полностью на AWS, serverless |
| **Consul/KV** | Service discovery + конфиги в одном месте | Меньше фич, чем у Config Server | Уже используете Consul для discovery |

---

## ✅ Чек-лист внедрения

- [ ] Config Server развёрнут с HTTPS + аутентификацией
- [ ] Git-репозиторий с конфигами настроен, доступ ограничен
- [ ] Ключ шифрования хранится в secure storage (не в Git!)
- [ ] Клиенты используют `bootstrap.yml` + `fail-fast: true`
- [ ] `@RefreshScope` применён только к бинам, которым это нужно
- [ ] Actuator endpoints (`/refresh`, `/health`) защищены
- [ ] Настроен мониторинг: доступность сервера, latency, cache hit rate
- [ ] Документирован процесс: как добавить конфиг, как зашифровать, как откатить

---

## 💡 Итог

| Вопрос | Ответ |
|--------|-------|
| ✅ Что делает Spring Cloud Config? | Централизует управление конфигурациями микросервисов через внешний репозиторий (Git) |
| ✅ Когда использовать? | 5+ микросервисов, частые изменения конфигов, требования к аудиту/безопасности |
| ✅ Как обновлять конфиги без перезапуска? | `@RefreshScope` + `POST /actuator/refresh` или Spring Cloud Bus |
| ✅ Как хранить секреты? | Шифрование `{cipher}...` + Vault / Kubernetes Secrets |
| ✅ Что делать, если Config Server упал? | `fail-fast: true` + retry, кэширование на клиенте, fallback values |

Spring Cloud Config — это **стандарт де-факто** для управления конфигурациями в Spring-микросервисах. При правильной настройке он даёт:

- 🔹 **Безопасность**: секреты зашифрованы, доступ контролируется
- 🔹 **Гибкость**: профили, ветки, динамический reload
- 🔹 **Надёжность**: fail-fast, retry, кэширование
- 🔹 **Аудируемость**: Git history = полный changelog конфигов

Нужен готовый `docker-compose.yml` для Config Server + Git + Vault или пример интеграции с **Kubernetes ConfigMaps**? Напишите — подготовлю production-ready шаблон.
