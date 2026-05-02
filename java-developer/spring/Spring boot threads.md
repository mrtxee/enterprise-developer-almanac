---
aliases:
  - Spring boot threads
  - threads
  - Spring threads
---
## [[Thread|Потоки в приложении]]

> Чем соответствует поток spring boot приложения в операционной системе и в архитектуре ЭВМ?


## Основные источники потоков в Spring Boot

> Разберу подробно, сколько потоков доступно при запуске Spring Boot‑приложения — с разбивкой по компонентам и примерами настройки.

**1. Веб‑сервер (Tomcat/Undertow/Jetty)**

По умолчанию Spring Boot использует Tomcat. Поток запросов обрабатывается его пулом потоков.

* **По умолчанию:** 200 рабочих потоков (`server.tomcat.threads.max = 200`).
* **Минимальный запас:** 10 потоков всегда готовы к работе (`server.tomcat.min-spare-threads = 10`).

**Настройка в `application.yml`:**
```yaml
server:
  tomcat:
    max-threads: 300          # макс. число рабочих потоков
    min-spare-threads: 20   # мин. число свободных потоков
    accept-count: 100       # макс. длина очереди запросов
```

**2. Пул соединений с БД (HikariCP)**

Управляет соединениями к базе данных.

* **По умолчанию:** `maximum-pool-size = 10`.
* **Пример настройки:**
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
```

**3. Пул асинхронных задач (`TaskExecutor`)**
Используется для методов с `@Async`.

* **Автонастройка:** `ThreadPoolTaskExecutor` с 8 основными потоками.
* **Настройка:**
```yaml
spring:
  task:
    execution:
      pool:
        max-size: 16
        queue-capacity: 100
        keep-alive: 10s
```

**4. Планировщик задач (`TaskScheduler`)**
Для методов с `@Scheduled`.

* **По умолчанию:** 1 поток.
* **Настройка:**
```yaml
spring:
  task:
    scheduling:
      pool:
        size: 4
      thread-name-prefix: "scheduler-"
```

**5. Quartz Scheduler**
Если подключён модуль `spring-boot-starter-quartz`.

* **По умолчанию:** задаётся в `org.quartz.threadPool.threadCount` (обычно 10).
* **Настройка в `application.yml`:**
```yaml
org:
  quartz:
    threadPool:
      threadCount: 15
```

**6. Другие компоненты**
* **Kafka/RabbitMQ:** пулы потоков для слушателей.
* **WebClient/RestTemplate:** пулы для HTTP‑клиентов.
* **Actuator:** служебные потоки для метрик и health checks.

---

## Как посчитать общее количество потоков


**Формула приблизительного расчёта:**
$$
N_{\text{total}} = N_{\text{Tomcat}} + N_{\text{DB}} + N_{\text{Async}} + N_{\text{Scheduler}} + N_{\text{Quartz}} + N_{\text{Other}}
$$

**Пример для стандартных настроек:**
* Tomcat: 200;
* HikariCP: 10;
* Async: 8;
* Scheduler: 1;
* Quartz: 10 (если подключён);
* Прочие: 5–10.

**Итого:** ~230–240 потоков при старте.

---

## Как проверить реальное количество потоков

**Способ 1. Через Actuator**
Добавьте зависимость:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
Включите эндпоинт:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: metrics,threads
```
Откройте `/actuator/metrics/jvm.threads.live` — покажет текущее число потоков.

**Способ 2. Программно**
```java
@RestController
public class ThreadInfoController {

    @GetMapping("/threads")
    public Map<String, Integer> getThreadInfo() {
        int total = Thread.activeCount();
        return Map.of("totalThreads", total);
    }
}
```

**Способ 3. Через JConsole/VisualVM**
1. Запустите приложение.
2. Откройте JConsole (`jconsole` в командной строке).
3. Подключитесь к процессу.
4. Перейдите на вкладку «Threads» — увидите детализацию по группам потоков.

---

## Важные нюансы

* **Виртуальные потоки (Java 21+):** если включена опция `spring.threads.virtual.enabled = true`, Spring использует `VirtualThreadTaskExecutor`, что резко снижает нагрузку на ОС.
* **Контекстные переключения:** слишком большое число потоков (>500) может снизить производительность из‑за накладных расходов на переключение контекста.
* **Память:** каждый поток потребляет память под стек (обычно 1 MB на поток).
* **Блокировки:** потоки могут простаивать в ожидании ресурсов (БД, файлы, сеть).

---

## Рекомендации по настройке

1. **Не увеличивайте `max-threads` без анализа.** Сначала оптимизируйте время обработки запросов.
2. **Синхронизируйте пулы:** если у вас 200 потоков Tomcat, но только 10 соединений БД, 190 потоков будут простаивать.
3. **Используйте реактивные стеки** (WebFlux + Netty) для высокой параллельности при малом числе потоков.
4. **Мониторинг:** отслеживайте метрики:
   * `jvm.threads.live` — общее число потоков;
   * `tomcat.threads.busy` — занятые потоки Tomcat;
   * `hikari.connections.active` — активные соединения с БД.

Хотите, я помогу адаптировать настройки под ваш сценарий нагрузки или уточню какой‑то аспект подробнее?