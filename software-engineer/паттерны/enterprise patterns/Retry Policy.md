---
aliases:
  - Retry
  - Exponential Backoff
  - Jitter
  - Linear Backoff
  - No Retry
  - Conditional Retry
---
**Retry Policy (политика повторных попыток)** — это **ключевой элемент надёжности в распределённых системах**, особенно при работе с сетью, внешними API, базами данных и очередями.

---

## ✅ Что такое **Retry Policy**?

> **Retry Policy** — это **правило**, определяющее:
> - Когда и сколько раз **повторять операцию** после сбоя
> - С какими задержками между попытками
> - При каких ошибках стоит пробовать снова
> - Как избежать перегрузки системы

> 💡 Без retry-логики ваша система будет **ломаться при временных сбоях** (например, сетевая задержка, кратковременный недоступность сервиса).

---

## ✅ Основные типы Retry Policies

| Тип                                 | Описание                                                                                                                                                         | Когда использовать                                                  |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **1. Fixed Delay**                  | Повтор через **фиксированную задержку** (например, каждые 2 секунды)                                                                                             | Простые случаи, когда вы уверены, что сервис быстро восстановится   |
| **2. Exponential Backoff**          | Задержка растёт экспоненциально: `1s → 2s → 4s → 8s → ...`                                                                                                       | Рекомендуется для большинства случаев — снижает нагрузку на сервер  |
| **3. Exponential Backoff + Jitter** | То же, что выше, но **случайное отклонение** (`jitter`) — например, вместо 4s → 3.7s или 5.2s                                                                    | Чтобы избежать "thundering herd" — все клиенты не бьют одновременно |
| **4. Linear Backoff**               | Задержка увеличивается линейно: `1s → 2s → 3s → 4s → ...`                                                                                                        | Устаревший, почти не используется — хуже экспоненциального          |
| **5. No Retry (Fail Fast)**         | Никаких попыток — сразу ошибка                                                                                                                                   | Для HFT, критичных транзакций, где задержка = провал                |
| **6. [[Circuit Breaker]] + Retry**  | После N неудач — **перестать пытаться** на время                                                                                                                 | Если сервис упал — не нужно его забивать запросами                  |
| **7. Conditional Retry**            | Повтор только при **определённых ошибках**: <br> • `503 Service Unavailable`<br> • `Timeout`<br> • `Connection refused`<br> • Не повторять при `400 Bad Request` | Умная политика — не повторяйте, если ошибка клиента                 |

---

## 🔝 Наиболее часто используемые типы

### 🥇 **1. Exponential Backoff + Jitter**
> **Самый популярный и рекомендуемый подход**

#### 🔧 Пример:
```plaintext
Попытка 1: немедленно
Попытка 2: через 1s
Попытка 3: через 2s
Попытка 4: через 4s
Попытка 5: через 8s
```

#### 🔍 Случайный jitter:
Вместо 4s → 3.5–4.8s (рандом)

#### ✅ Преимущества:
- Предотвращает **обвал сервера** под грузом повторных запросов
- Эффективно при **временных сбоях** (сетевые флуктуации, краткие перезагрузки)
- Используется в **Google Cloud**, **AWS**, **Kafka**, **RabbitMQ**, **gRPC**

> 💬 *«Exponential backoff is the single most effective thing you can do to make your system resilient.»* — Google SRE Handbook

---

### 🥈 **2. Exponential Backoff без Jitter**
- Проще реализовать
- Но может вызвать **синхронизированные повторы** (все клиенты ждут 2s → бьют одновременно)

> ⚠️ **Используйте только если нет jitter** — а лучше всегда добавлять jitter.

---

### 🥉 **3. Conditional Retry**
> Повтор только при **временных ошибках**, а не при ошибках клиента

#### ❌ Не повторяйте:
| Код | Причина |
|-----|---------|
| `400 Bad Request` | Ошибка клиента — передавайте правильно |
| `401 Unauthorized` | Неправильный токен — повтор не поможет |
| `404 Not Found` | Ресурс не существует |
| `409 Conflict` | Конфликт — нужна бизнес-логика |

#### ✅ Повторяйте:
| Код / Ошибка | Причина |
|---------------|---------|
| `503 Service Unavailable` | Сервис перегружен — можно попробовать позже |
| `504 Gateway Timeout` | Сеть/шлюз не ответил — возможно, временно |
| `429 Too Many Requests` | Можно повторить с задержкой (retry-after) |
| `Network timeout`, `Connection reset` | Временная сетевая проблема |
| `500 Internal Server Error` | Иногда — если известно, что это временное состояние |

---

## ✅ Примеры реальных систем

### 1. **Google Cloud APIs**
```json
{
  "retry_policy": {
    "max_retries": 5,
    "initial_backoff": "1s",
    "backoff_multiplier": 2,
    "max_backoff": "60s"
  }
}
```
→ Exponential backoff: 1s → 2s → 4s → 8s → 16s → stop

---

### 2. **AWS SDK (Java)**
```java
import software.amazon.awssdk.core.retry.RetryPolicy;

// Использует exponential backoff + jitter по умолчанию
SdkClientConfiguration config = SdkClientConfiguration.builder()
    .retryPolicy(RetryPolicy.defaultRetryPolicy())
    .build();
```

> ✅ AWS SDK **автоматически применяет retry** для `ThrottlingException`, `InternalError`, `Timeout`

---

### 3. **Spring Retry (Java)**
```java
@Retryable(
    value = {RestClientException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000, multiplier = 2, maxDelay = 10000)
)
public String callExternalApi() {
    return restTemplate.getForObject("https://api.example.com/data", String.class);
}
```

→ delay: 1s → 2s → 4s → stop

---

### 4. **Resilience4j (Java)**
```java
RetryConfig config = RetryConfig.custom()
    .maxAttempts(3)
    .waitDuration(Duration.ofMillis(1000))
    .exponentialBackoffMultiplier(2.0)
    .ignoreExceptions(BadRequestException.class)
    .build();

Retry retry = Retry.of("external-api", config);
```

> ✅ Поддерживает всё: exponential backoff, jitter, ignore, circuit breaker

---

### 5. **gRPC**
```yaml
# gRPC service config
{
  "methodConfig": [
    {
      "name": [{"service": "example.PaymentService"}],
      "retryPolicy": {
        "maxAttempts": 4,
        "initialBackoff": "1s",
        "maxBackoff": "10s",
        "backoffMultiplier": 2,
        "retryableStatusCodes": ["UNAVAILABLE", "DEADLINE_EXCEEDED"]
      }
    }
  ]
}
```

→ Только для **временных ошибок**, не для `INVALID_ARGUMENT`

---

## ✅ Best Practices: Как правильно делать retry?

| Правило | Объяснение |
|--------|------------|
| ✅ **Используйте exponential backoff** | Лучше, чем fixed delay |
| ✅ **Добавьте jitter** | Случайное отклонение → избегает "пиков" |
| ✅ **Ограничьте количество попыток** | Например, 3–5 раз — не бесконечно |
| ✅ **Не повторяйте при ошибках клиента** | `400`, `401`, `404` — не помогут |
| ✅ **Используйте circuit breaker** | Если 5 попыток подряд неудачны — **не пытайтесь больше** |
| ✅ **Логируйте повторы** | Чтобы видеть, где проблемы |
| ✅ **Мониторьте retry-rate** | Высокий % retry — признак проблем |
| ✅ **Учитывайте idempotency** | Повтор должен быть безопасным для бизнеса |

---

## ✅ Idempotency — почему важна?

> **Idempotent operation** — это операция, которую можно **повторять много раз без побочных эффектов**.

#### ❌ Не idempotent:
```http
POST /api/v1/payments → charge $100
```
→ Если запрос повторится — **снимет ещё $100** → плохо!

#### ✅ Idempotent:
```http
PUT /payments/123 → charge $100
```
или
```http
POST /payments?request_id=abc123 → charge $100
```
→ Сервер проверяет `request_id` → если уже был — **не снимает деньги второй раз**

> ✅ **Retry возможен только для idempotent операций!**

---

## ✅ Пример: Политика в коде (Python)

```python
import time
import random
from functools import wraps

def retry_with_exponential_backoff(max_retries=5, base_delay=1, max_delay=60):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            delay = base_delay
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except (ConnectionError, TimeoutError, HTTPError) as e:
                    if attempt == max_retries - 1:
                        raise e
                    
                    # Exponential backoff + jitter
                    sleep_time = min(delay * (2 ** attempt), max_delay)
                    jitter = random.uniform(0.5, 1.5)
                    actual_sleep = sleep_time * jitter
                    
                    print(f"Ошибка: {e}. Повтор через {actual_sleep:.2f} сек")
                    time.sleep(actual_sleep)
            return None
        return wrapper
    return decorator

@retry_with_exponential_backoff(max_retries=3, base_delay=1)
def fetch_data():
    # Ваш HTTP-вызов
    pass
```

---

## ✅ Когда **не использовать** retry?

| Сценарий | Почему |
|----------|--------|
| ✅ **HFT / Финансовые транзакции** | Задержка > 1 мс = упущенная прибыль |
| ✅ **Некоторые команды CLI** | Человек сам решает, повторить ли |
| ✅ **Если операция не idempotent** | Может привести к дублям |
| ✅ **Критичные системы реального времени** | Где каждый цикл важен |

---

## ✅ Итог: Какие retry policy используются чаще всего?

| Тип | Частота | Рекомендация |
|------|--------|--------------|
| **Exponential Backoff + Jitter** | ✅✅✅✅✅ | **Используйте всегда**, если возможно |
| **Conditional Retry** | ✅✅✅✅ | Добавляйте фильтры по ошибкам |
| **Circuit Breaker + Retry** | ✅✅✅✅ | Для production-систем |
| **Fixed Delay** | ✅✅ | Только для простых случаев |
| **No Retry** | ✅ | Для HFT, идемпотентных систем |

---

## 💬 Цитата от Google:

> _“The best way to handle transient errors is to retry with exponential backoff and jitter.”_  
> — **Site Reliability Engineering (SRE) Book**

---

## ✅ Финальный вывод

| Политика | Когда использовать |
|--------|------------------|
| **Exponential Backoff + Jitter** | ✅ Почти всегда — стандарт де-факто |
| **Conditional Retry** | ✅ Да — исключите `4xx` |
| **Circuit Breaker** | ✅ Да — если сервис часто падает |
| **Fixed Delay** | ⚠️ Только если вы уверены, что сервис быстро восстановится |
| **No Retry** | ✅ Для HFT, идемпотентных систем |

> ✅ **Exponential backoff + jitter — это золотой стандарт.**  
> Он **прост**, **эффективен**, **надёжен** и **используется повсюду**.

---

## 📚 Где учиться дальше?

- **Book**: *“Site Reliability Engineering”* — Google
- **Resilience4j**: https://resilience4j.readme.io/
- **Spring Retry**: https://github.com/spring-projects/spring-retry
- **gRPC Retry**: https://github.com/grpc/proposal/blob/master/A6-client-retries.md

---

✅ **Теперь вы знаете: не просто “делайте retry”, а “как делать retry правильно”.**  
Используйте **exponential backoff + jitter** — и ваши системы станут **намного надёжнее**.