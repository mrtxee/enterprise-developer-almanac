---
aliases:
  - spring-retry
  - spring retry
---
Для автоматических повторов при исключениях в Spring используется библиотека **`spring-retry`**. Ниже приведена полная пошаговая инструкция с production-готовым примером.

---

## 📦 1. Добавьте зависимости

Вам нужны **два** артефакта: сам `spring-retry` и `spring-boot-starter-aop` (без него `@EnableRetry` не создаст прокси).

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

> 💡 В Spring Boot версии управляются автоматически через `spring-boot-starter-parent`.

---

## 🔛 2. Включите механизм повторов

Добавьте `@EnableRetry` в главный класс или конфигурацию:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.retry.annotation.EnableRetry;

@SpringBootApplication
@EnableRetry // ✅ Обязательно
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

## 🔄 3. Используйте `@Retryable` на методе

```java
import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Recover;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Service;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;
import java.time.LocalDateTime;

@Slf4j
@Service
public class ExternalClientService {

    // ✅ 3 попытки, экспоненциальная задержка, только для IOException
    @Retryable(
        retryFor = {IOException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2.0, maxDelay = 10000, random = true)
    )
    public String fetchData(String url) throws IOException {
        log.info("🌐 Запрос к {} в {}", url, LocalDateTime.now());
        
        // Имитация нестабильного внешнего сервиса
        if (Math.random() < 0.7) {
            throw new IOException("Connection timeout");
        }
        
        return "Data from " + url;
    }

    // ✅ Вызовется, если все 3 попытки упали
    @Recover
    public String recoverFetchData(IOException e, String url) {
        log.error("❌ Все повторы исчерпаны для {}: {}", url, e.getMessage());
        return "FALLBACK_DATA";
    }
}
```

---

## 🔍 Разбор параметров `@Retryable`

| Параметр | Описание | Пример |
|----------|----------|--------|
| `retryFor` | Какие исключения повторять | `{IOException.class, TimeoutException.class}` |
| `noRetryFor` | Какие исключения **не** повторять (даже если подходят под `retryFor`) | `{IllegalArgumentException.class}` |
| `maxAttempts` | Максимум попыток (включая первую) | `3` |
| `backoff` | Стратегия задержки между попытками | `@Backoff(delay = 1000, multiplier = 2.0)` |
| `exceptionExpression` | SpEL-выражение для условного повтора | `@Retryable(exceptionExpression = "#{message.contains('timeout')}")` |

### ⏱️ Параметры `@Backoff`:
| Параметр | Поведение |
|----------|-----------|
| `delay` | Начальная задержка в мс |
| `multiplier` | Коэффициент умножения (`1000 → 2000 → 4000`) |
| `maxDelay` | Максимальная задержка (ограничивает рост) |
| `random` | Добавляет случайный джиттер (защищает от thundering herd) |

---

## ⚠️ Критически важные правила `@Recover`

Метод с `@Recover` **должен строго соответствовать** сигнатуре `@Retryable`:

1. **Первый параметр** → тип исключения, которое вызвало повтор (`retryFor`)
2. **Остальные параметры** → параметры исходного метода **в том же порядке**
3. **Тип возврата** → должен совпадать с `@Retryable` методом

```java
// ❌ ОШИБКА: не совпадает порядок параметров
@Recover
public String recover(IOException e, int count, String url) { ... }

// ✅ ПРАВИЛЬНО
@Recover
public String recover(IOException e, String url) { ... }
```

---

## 🚫 Частые ошибки и как их избежать

| Ошибка | Причина | Решение |
|--------|---------|---------|
| 🔸 Повторы не работают | Забыли `spring-boot-starter-aop` | Добавьте зависимость |
| 🔸 Метод вызывается напрямую внутри класса | Self-invocation не проходит через прокси | Вызовите метод через `this` → не сработает. Вынесите в отдельный сервис или используйте `AopContext.currentProxy()` |
| 🔸 Исключение "съедается" внутри метода | `try-catch` без `throw` | Spring Retry видит только **выброшенные** исключения |
| 🔸 `@Recover` не вызывается | Сигнатура не совпадает или нет `retryFor` для этого исключения | Проверьте порядок параметров и типы |
| 🔸 Долгие задержки в тестах | `delay` слишком большой | Используйте `@TestPropertySource(properties = "retry.backoff.delay=0")` |

---

## 🛠 Альтернатива: Программный `RetryTemplate`

Если нужна сложная логика (динамические задержки, кастомные политики, метрики):

```java
import org.springframework.retry.support.RetryTemplate;
import org.springframework.retry.backoff.ExponentialBackOffPolicy;
import org.springframework.retry.policy.SimpleRetryPolicy;
import org.springframework.stereotype.Component;

@Component
public class CustomRetryService {

    private final RetryTemplate retryTemplate;

    public CustomRetryService() {
        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(3);
        retryPolicy.setRetryableExceptions(IOException.class);

        ExponentialBackOffPolicy backOffPolicy = new ExponentialBackOffPolicy();
        backOffPolicy.setInitialInterval(1000);
        backOffPolicy.setMultiplier(2.0);
        backOffPolicy.setMaxInterval(10000);

        this.retryTemplate = RetryTemplate.builder()
            .retryPolicy(retryPolicy)
            .backOffPolicy(backOffPolicy)
            .build();
    }

    public String executeWithRetry(String url) {
        return retryTemplate.execute(context -> {
            log.info("Попытка {}", context.getRetryCount() + 1);
            return callExternal(url);
        }, context -> {
            log.error("Фолбэк после {} попыток", context.getRetryCount());
            return "FALLBACK";
        });
    }
}
```

---

## ✅ Чек-лист production-настройки

| Параметр | Рекомендация |
|----------|--------------|
| `maxAttempts` | `3–5` (больше обычно бессмысленно) |
| `delay` | `1000–3000ms` |
| `multiplier` | `1.5–2.0` |
| `random = true` | ✅ Всегда включайте для распределённых систем |
| `retryFor` | Указывайте **конкретные** исключения, не ловите `Exception.class` |
| Логирование | Логируйте `context.getRetryCount()` для мониторинга |
| Метрики | Интегрируйте с `Micrometer` для отслеживания `retry.attempts.total` |

---

## 📊 Итог

| Задача | Решение |
|--------|---------|
| ✅ Простые повторы | `@Retryable` + `@Backoff` |
| ✅ Обработка финального отказа | `@Recover` |
| ✅ Сложная логика/метрики | `RetryTemplate` |
| ✅ Обязательно | `spring-boot-starter-aop` + `@EnableRetry` |

`spring-retry` — это **стандарт индустрии** для отказоустойчивости в Spring. При правильной настройке он покрывает 95% сценариев работы с нестабильными внешними сервисами, БД и очередями.

Нужен пример интеграции с **Prometheus-метриками** или **распределённым трейсингом (OpenTelemetry)** для повторных попыток?

Отличный вопрос, касающийся **потоковой модели** Spring Retry.


# Неблокирующий Spring retry

❓Если при вызове метода раз происходит сбой и только на 9 attempt отрабатывает успешно, то какой поток выполнения будет ожидать delay и повторно запускать выполнение?

✅ Короткий ответ

**Тот же самый поток**, который вызвал метод `fetchData()`, будет:
1. 🔄 Ловить исключение
2. 😴 Ждать задержку (`Thread.sleep()`)
3. 🔁 Повторно вызывать метод

**Никакие новые потоки не создаются.** Повторы выполняются **синхронно и блокирующе** в том же потоке.

---

## 🔍 Как это работает внутри?

### 1. Схема выполнения

```
[Вызывающий поток: http-nio-8080-exec-3]
         │
         ▼
┌─────────────────────────┐
│ @Retryable interceptor  │
└─────────────────────────┘
         │
         ▼
┌─────────────────────────┐
│ 1-я попытка: fetchData()│
│ ❌ IOException          │
└─────────────────────────┘
         │
         ▼
┌─────────────────────────┐
│ BackOffPolicy.backOff() │
│ Thread.sleep(1000) 😴   │ ← Тот же поток спит!
└─────────────────────────┘
         │
         ▼
┌─────────────────────────┐
│ 2-я попытка: fetchData()│
│ ❌ IOException          │
└─────────────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Thread.sleep(2000) 😴   │ ← multiplier = 2.0
└─────────────────────────┘
         │
         ▼
   ... (повторы 3-8) ...
         │
         ▼
┌─────────────────────────┐
│ 9-я попытка: fetchData()│
│ ✅ Успех!               │
└─────────────────────────┘
         │
         ▼
[Возврат результата вызывающему коду]
```

### 2. Что происходит с `Thread.sleep()`?

Класс `ExponentialBackOffPolicy` (используется по умолчанию с `@Backoff`) реализует задержку так:

```java
// Упрощённо из Spring Retry исходников
public void backOff(BackOffContext context) throws BackOffInterruptedException {
    try {
        long sleepTime = context.getSleepAndIncrement();
        Thread.sleep(sleepTime);  // ⚠️ Блокирует текущий поток!
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        throw new BackOffInterruptedException("Thread interrupted", e);
    }
}
```

> ⚠️ **Это ключевой момент:** `Thread.sleep()` блокирует **именно тот поток**, который выполняет метод.

---

## 📊 Влияние на приложение

| Сценарий | Последствия |
|----------|-------------|
| ✅ **Одиночный запрос** | Ничего страшного: поток просто подождёт ~1+2+4+8+... секунд |
| ⚠️ **Высокая нагрузка** | Потоки томката/веб-сервера **забиваются** ожидающими повторами → рост очереди запросов → `503 Service Unavailable` |
| ⚠️ **Долгие задержки** | При `maxDelay = 10000` и 10 попытках суммарная задержка может достигать **~30+ секунд** на один запрос |
| ✅ **Фоновые задачи** | Для `@Scheduled` или асинхронных сервисов — приемлемо, если есть запас по времени |

---

## 🔧 Как сделать асинхронные повторы?

Если вы не хотите блокировать основной поток, есть два пути:

### 🔹 Вариант 1: `@Retryable` + `@Async` (простой)

```java
@Service
public class ExternalClientService {

    @Async  // ✅ Выполнится в отдельном потоке из пула
    @Retryable(
        retryFor = {IOException.class},
        maxAttempts = 10,
        backoff = @Backoff(delay = 1000, multiplier = 2.0)
    )
    public CompletableFuture<String> fetchDataAsync(String url) {
        return CompletableFuture.completedFuture(fetchData(url));
    }

    // Синхронная версия для внутренних вызовов
    public String fetchData(String url) throws IOException {
        // ... логика ...
    }
}
```

```java
// Включение асинхронности
@SpringBootApplication
@EnableRetry
@EnableAsync  // ✅ Обязательно
public class Application { ... }
```

```yaml
# Настройка пула потоков (application.yml)
spring:
  task:
    execution:
      pool:
        core-size: 10
        max-size: 50
        queue-capacity: 100
```

> ✅ Теперь повторы будут выполняться в пуле `@Async`, не блокируя веб-потоки.

---

### 🔹 Вариант 2: `RetryTemplate` + `TaskExecutor` (продвинутый)

```java
import org.springframework.retry.support.RetryTemplate;
import org.springframework.retry.backoff.ExponentialBackOffPolicy;
import org.springframework.retry.policy.SimpleRetryPolicy;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import java.util.concurrent.CompletableFuture;

@Service
public class AsyncRetryService {

    private final RetryTemplate retryTemplate;
    private final ThreadPoolTaskExecutor taskExecutor;

    public AsyncRetryService() {
        // Настройка политики повторов
        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(10);
        
        ExponentialBackOffPolicy backOffPolicy = new ExponentialBackOffPolicy();
        backOffPolicy.setInitialInterval(1000);
        backOffPolicy.setMultiplier(2.0);
        backOffPolicy.setMaxInterval(10000);
        
        this.retryTemplate = RetryTemplate.builder()
            .retryPolicy(retryPolicy)
            .backOffPolicy(backOffPolicy)
            .build();
            
        // Настройка исполнителя
        this.taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(10);
        taskExecutor.setMaxPoolSize(50);
        taskExecutor.setQueueCapacity(100);
        taskExecutor.setThreadNamePrefix("async-retry-");
        taskExecutor.initialize();
    }

    public CompletableFuture<String> fetchDataWithAsyncRetry(String url) {
        return CompletableFuture.supplyAsync(() -> 
            retryTemplate.execute(context -> {
                log.info("Попытка {}", context.getRetryCount() + 1);
                return callExternal(url); // Ваша логика
            }), 
            taskExecutor  // ✅ Повторы + задержки в отдельном пуле
        );
    }
}
```

---

## 📋 Сравнение подходов

| Подход | Блокирует вызывающий поток? | Сложность | Когда использовать |
|--------|----------------------------|-----------|-------------------|
| ✅ `@Retryable` (по умолчанию) | **Да** | 🟢 Низкая | Веб-запросы с низким RPS, фоновые задачи |
| ✅ `@Retryable` + `@Async` | **Нет** | 🟡 Средняя | Веб-запросы, где важно не блокировать томкат |
| ✅ `RetryTemplate` + `TaskExecutor` | **Нет** | 🔴 Высокая | Сложные сценарии с кастомной логикой, метриками |
| ✅ `Resilience4j` / `Sentinel` | **Нет** (опционально) | 🟡 Средняя | Микросервисы с circuit breaker, метриками |

---

## ⚠️ Практические рекомендации

| Совет | Почему |
|-------|--------|
| 🔸 Не ставьте `maxAttempts > 5` для HTTP-запросов | После 5 неудач вероятность успеха резко падает |
| 🔸 Всегда используйте `random = true` | Защищает от "thundering herd" при восстановлении сервиса |
| 🔸 Логируйте `RetryContext.getRetryCount()` | Для мониторинга и алертинга |
| 🔸 Для веб-контроллеров: выносите `@Retryable` в сервис + `@Async` | Чтобы не блокировать потоки DispatcherServlet |
| 🔸 Тестируйте сценарий "все повторы упали" | Убедитесь, что `@Recover` или фолбэк-логика работает |

---

## ✅ Итог

| Вопрос | Ответ |
|--------|-------|
| ✅ Какой поток ждёт delay? | **Тот же**, что вызвал метод |
| ✅ Блокируется ли поток? | **Да**, через `Thread.sleep()` |
| ✅ Можно ли сделать асинхронно? | **Да**, через `@Async` или `RetryTemplate + TaskExecutor` |
| ✅ Влияет ли на производительность? | **Да**, при высокой нагрузке — критично |

> 💡 **Правило большого пальца:**  
> Если метод вызывается из **веб-контроллера** — выносите повторы в `@Async`-сервис.  
> Если метод — **фоновая задача** (`@Scheduled`, очередь) — синхронные повторы ок.

Нужен пример с **метриками для мониторинга повторов** или интеграцией с **Resilience4j**?