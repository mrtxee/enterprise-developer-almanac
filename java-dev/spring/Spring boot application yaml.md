---
aliases:
  - application yaml
  - application.yaml
  - application.yml
  - Spring Boot
---
# Полный список стандартных атрибутов application.yaml в Spring Boot

## Источники
- [Spring Boot Common Application Properties](https://docs.spring.io/spring-boot/appendix/application-properties/index.html)
- [Spring Boot 3.4.x Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html)

---

## 1. Core Properties (Ядро)

```yaml
spring:
  application:
    name: my-app                    # Имя приложения (для логирования, discovery)
  profiles:
    active: dev,prod                # Активные профили
    include: common,secure          # Дополнительные профили для включения
  config:
    import: classpath:additional.yml # Импорт дополнительных конфигураций
    location: optional:config/      # Директории поиска конфигурации
    use-legacy-processing: false    # Использовать новый механизм обработки
  
  main:
    web-application-type: servlet   # servlet/reactive/none
    banner-mode: console            # console/log/off
    allow-circular-references: false # Разрешить циклические ссылки
    lazy-initialization: false      # Ленивая инициализация бинов
    log-startup-info: true          # Логировать информацию о запуске
  
  mandatory-file-encoding: UTF-8    # Обязательная кодировка файлов
  messages:
    basename: messages              # Базовое имя для i18n
    encoding: UTF-8                 # Кодировка
    fallback-to-system-locale: true
    always-use-message-format: false
```

## 2. Web Properties (Веб)

```yaml
server:
  port: 8080                       # HTTP порт
  address: 0.0.0.0                 # Адрес для привязки
  servlet:
    context-path: /api             # Контекстный путь приложения
    session:
      timeout: 30m                 # Таймаут сессии
      cookie:
        http-only: true
        secure: false
        max-age: -1
  error:
    path: /error                   # Путь для ошибок
    include-message: always        # always/never/on-param
    include-stacktrace: never      # always/never/on-param
    include-binding-errors: never
    include-exception: false
  compression:
    enabled: true                  # Включить сжатие ответов
    mime-types: application/json,application/xml
    min-response-size: 2048
  tomcat:
    threads:
      max: 200                     # Максимум рабочих потоков
      min-spare: 10                # Минимум запасных потоков
    connection-timeout: 20000      # Таймаут соединения (мс)
    max-connections: 10000         # Максимум соединений
    accept-count: 100              # Размер очереди
    max-swallow-size: 2MB          # Максимальный размер тела запроса
  undertow:
    threads:
      io: 4                        # IO потоки
      worker: 64                   # Рабочие потоки
    buffer-size: 16384             # Размер буфера
  netty:
    connection-timeout: 5000       # Таймаут соединения

spring:
  mvc:
    view:
      prefix: /WEB-INF/views/      # Префикс для JSP
      suffix: .jsp                 # Суффикс для JSP
    static-path-pattern: /**       # Паттерн для статики
    throw-exception-if-no-handler-found: true
    pathmatch:
      matching-strategy: ant_path_matcher # ant_path_matcher / path_pattern_parser
    format:
      date: yyyy-MM-dd             # Формат даты
      date-time: yyyy-MM-dd HH:mm:ss
      time: HH:mm:ss
  web:
    resources:
      static-locations: classpath:/static,classpath:/public
      add-mappings: true
      cache:
        period: 3600               # Кэширование статики (сек)
      chain:
        enabled: true
        compressed: true
  thymeleaf:
    enabled: true
    prefix: classpath:/templates/
    suffix: .html
    mode: HTML
    encoding: UTF-8
    cache: true                    # Кэширование шаблонов
    check-template-location: true
```

## 3. Data Properties (Базы данных)

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/db
    username: postgres
    password: password
    driver-class-name: org.postgresql.Driver
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      pool-name: HikariPool
      connection-test-query: SELECT 1
      register-mbeans: true
    tomcat:
      max-active: 100
      max-idle: 10
      min-idle: 5
    hikari:
      schema: public
  
  jpa:
    show-sql: false                # Показывать SQL
    open-in-view: false            # Открывать EntityManager в View
    database: POSTGRESQL           # Тип БД
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    generate-ddl: false
    hibernate:
      ddl-auto: validate           # none/validate/update/create/create-drop
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
        implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
    properties:
      hibernate:
        format_sql: true
        default_schema: public
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
    mapping-resources: META-INF/orm.xml
  
  jdbc:
    template:
      fetch-size: -1
      max-rows: -1
      query-timeout: 0
    datasource:
      auto-commit: true
  
  flyway:
    enabled: true
    baseline-on-migrate: false
    baseline-version: 0
    locations: classpath:db/migration
    encoding: UTF-8
    table: flyway_schema_history
    validate-on-migrate: true
  
  liquibase:
    enabled: true
    change-log: classpath:/db/changelog/db.changelog-master.yaml
    default-schema: public
    liquibase-schema: public
    drop-first: false
    database-change-log-table: DATABASECHANGELOG
    database-change-log-lock-table: DATABASECHANGELOGLOCK
  
  data:
    redis:
      host: localhost
      port: 6379
      password: 
      database: 0
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
    mongodb:
      uri: mongodb://localhost:27017/db
      database: test
      auto-index-creation: true
    elasticsearch:
      uris: http://localhost:9200
      username: 
      password:
      connection-timeout: 1s
```

## 4. Transaction Properties (Транзакции)

```yaml
spring:
  transaction:
    default-timeout: 30            # Таймаут транзакции (сек)
    rollback-on-commit-failure: false
```

## 5. Actuator Properties (Мониторинг)

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics  # Какие endpoints включить
        exclude: env,heapdump         # Какие исключить
      base-path: /actuator            # Базовый путь
      path-mapping:
        health: healthcheck
    jmx:
      exposure:
        include: "*"
      domain: my-app
  endpoint:
    health:
      show-details: always           # always/never/when-authorized
      show-components: always
      show-details: always
    info:
      enabled: true
    metrics:
      enabled: true
    loggers:
      enabled: true
    env:
      enabled: false
      post-processing: enabled
  info:
    env:
      enabled: true
    build:
      enabled: true
    java:
      enabled: true
    git:
      enabled: true
      mode: full
  metrics:
    export:
      prometheus:
        enabled: true
        step: 1m
    tags:
      application: ${spring.application.name}
    distribution:
      percentiles-histogram:
        http.server.requests: true
  server:
    port: 8081                     # Отдельный порт для actuator
    address: 127.0.0.1
  security:
    enabled: true                  # Включить безопасность для endpoints
```

## 6. Logging Properties (Логирование)

```yaml
logging:
  level:
    root: INFO                     # Корневой уровень
    com.example: DEBUG
    org.springframework: WARN
    org.hibernate: INFO
    org.hibernate.SQL: DEBUG
    org.hibernate.type: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
    dateformat: yyyy-MM-dd HH:mm:ss
  file:
    name: logs/my-app.log          # Имя файла лога
    path: /var/log                # Директория логов
    max-size: 10MB                # Максимальный размер
    max-history: 30               # Максимум файлов истории
    total-size-cap: 100MB
  logback:
    rollingpolicy:
      clean-history-on-start: false
      file-name-pattern: ${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz
  group:
    tomcat: org.apache.catalina,org.apache.coyote
    web: org.springframework.web,org.springframework.web.servlet
```

## 7. Security Properties (Безопасность)

```yaml
spring:
  security:
    user:
      name: user                  # Имя пользователя по умолчанию
      password: password          # Пароль по умолчанию
      roles: USER                 # Роли пользователя
    oauth2:
      client:
        registration:
          google:
            client-id: xxx
            client-secret: xxx
            scope: openid,profile,email
      resourceserver:
        jwt:
          issuer-uri: https://issuer.com
          jwk-set-uri: https://issuer.com/.well-known/jwks.json
    filter:
      dispatcher-types: async,error,request
    ignored: /css/**,/js/**       # Игнорируемые пути для безопасности
```

## 8. Kafka Properties (Сообщения)

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    client-id: my-app
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all
      retries: 3
      batch-size: 16384
      buffer-memory: 33554432
      properties:
        enable.idempotence: true
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      group-id: my-group
      auto-offset-reset: earliest
      enable-auto-commit: false
      max-poll-records: 500
    listener:
      ack-mode: batch
      concurrency: 3
      missing-topics-fatal: false
    admin:
      auto-create: true
      operation-timeout: 30s
    stream:
      application-id: my-stream-app
      properties:
        default.key.serde: org.apache.kafka.common.serialization.Serdes$StringSerde
```

## 9. Cache Properties (Кэширование)

```yaml
spring:
  cache:
    type: redis                  # simple/redis/caffeine/none
    cache-names: users,products
    redis:
      time-to-live: 60000ms      # TTL для кэша
      cache-null-values: false
      key-prefix: my-cache
      use-key-prefix: true
    caffeine:
      spec: maximumSize=500,expireAfterWrite=60s
    jcache:
      config: classpath:ehcache.xml
      provider: org.ehcache.jsr107.EhcacheCachingProvider
```

## 10. Async & Scheduling (Асинхронность)

```yaml
spring:
  task:
    execution:
      pool:
        core-size: 8
        max-size: 20
        queue-capacity: 100
        keep-alive: 60s
      thread-name-prefix: task-
      shutdown:
        await-termination: true
        await-termination-period: 30s
    scheduling:
      pool:
        size: 5
      thread-name-prefix: scheduling-
      shutdown:
        await-termination: true
        await-termination-period: 30s
```

## 11. Mail Properties (Почта)

```yaml
spring:
  mail:
    host: smtp.gmail.com
    port: 587
    username: user@gmail.com
    password: password
    protocol: smtp
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
            required: true
          connectiontimeout: 5000
          timeout: 5000
          writetimeout: 5000
    test-connection: false
    default-encoding: UTF-8
```

## 12. Quartz Scheduler

```yaml
spring:
  quartz:
    job-store-type: memory        # memory/jdbc
    jdbc:
      initialize-schema: never    # never/always/embedded
      comment-prefix: --
      schema: classpath:org/quartz/impl/jdbcjobstore/tables_@@platform@@.sql
    properties:
      org:
        quartz:
          scheduler:
            instanceName: MyScheduler
            instanceId: AUTO
          threadPool:
            threadCount: 10
            threadPriority: 5
          jobStore:
            class: org.quartz.impl.jdbcjobstore.JobStoreTX
            driverDelegateClass: org.quartz.impl.jdbcjobstore.PostgreSQLDelegate
            tablePrefix: QRTZ_
            isClustered: false
    auto-startup: true
    startup-delay: 0
    overwrite-existing-jobs: false
```

## 13. Jackson Properties (JSON)

```yaml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: UTC
    default-property-inclusion: non_null  # always/non_null/non_absent/non_default
    serialization:
      write-dates-as-timestamps: false
      indent-output: true
      fail-on-empty-beans: false
      write-enums-using-to-string: true
    deserialization:
      fail-on-unknown-properties: false
      fail-on-ignored-properties: false
      accept-single-value-as-array: true
    parser:
      allow-unquoted-control-chars: true
      allow-single-quotes: true
    generator:
      write-numbers-as-strings: false
      auto-close-target: true
    property-naming-strategy: SNAKE_CASE    # SNAKE_CASE/KEBAB_CASE/LOWER_CASE
    visibility:
      field: any
      getter: none
      setter: none
      creator: none
```

## 14. Validation Properties (Валидация)

```yaml
spring:
  validation:
    enabled: true
    validate: true
    auto-startup: true
```

---

## Примечания

1. **Версии Spring Boot**: Некоторые свойства появились в определённых версиях. Указанные свойства актуальны для Spring Boot 3.4.x
2. **Замена значений**: Используйте синтаксис `${}` для подстановки значений из переменных окружения
3. **Профили**: Разные профили могут переопределять значения:
   ```yaml
   spring:
     config:
       activate:
         on-profile: dev
   ```

4. **Ссылки на документацию**:
   - [Spring Boot 3.4 Common Application Properties](https://docs.spring.io/spring-boot/appendix/application-properties/index.html)
   - [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/html/)