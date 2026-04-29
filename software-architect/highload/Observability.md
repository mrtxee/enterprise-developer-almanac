## Что такое Observability?

**Observability** — это способность понимать внутреннее состояние системы на основе анализа ее внешних выходных данных. 

>В отличие от традиционного мониторинга, который отвечает на вопрос "*Что сломалось*?", 
>observability отвечает на вопрос "*Почему это сломалось?*" и "*Что происходит внутри системы?*"

Концепция Observability включает 3 основных компонента
1. [[Metrics]]
2. [[Logging]]
3. [[Tracing]]

* Observability extensions
	4. **Профилирование (Profiling)**
		- Анализ потребления ресурсов на уровне кода
		- **Примеры**: CPU profiling, memory profiling
	 5. **Synthetic Monitoring**
		- Имитация пользовательских сценариев
		- **Пример**: Регулярные проверки критических путей
	6. **Real User Monitoring (RUM)**
		- Сбор данных от реальных пользователей
		- **Метрики**: Core Web Vitals, user journey

### 1. **Метрики ([[Metrics]])**
- **Цифровые измерения** за определенный период времени
- **Примеры**: CPU utilization, memory usage, request rate, error rate
- **Характеристики**:
  - Агрегированные данные
  - Регулярный сбор
  - Подходят для алертинга

```python
# Пример метрик
requests_per_second = 1500
error_rate = 0.05  # 5%
response_time_p95 = 245  # ms
```

### 2. **Логи (Logs)**
- **Структурированные записи** о событиях в системе
- **Содержат**: timestamp, severity, message, context
- **Использование**: Отладка, аудит, анализ инцидентов

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "payment-service",
  "trace_id": "abc-123",
  "message": "Payment processing failed",
  "user_id": "user-789",
  "error": "Insufficient funds"
}
```

### 3. **Трейсы (Traces)**
- **Распределенная трассировка** запросов через микросервисы
- **Показывают**: latency, dependencies, bottlenecks
- **Ключевые понятия**:
  - Span - отдельная операция
  - Trace - цепочка spans
  - Context propagation

```
Trace: User Checkout Request
├─ Span: API Gateway (15ms)
├─ Span: Auth Service (8ms)
├─ Span: Cart Service (45ms)
│  ├─ Span: Database Query (12ms)
│  └─ Span: Inventory Check (25ms)
└─ Span: Payment Service (120ms)
   └─ Span: External Payment Gateway (105ms)
```

## Архитектура Observability

### Компоненты системы

```
┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   Источники     │    │  Сбор и          │    │  Анализ и        │
│   данных        │───▶│  обработка       │───▶│  визуализация    │
│                 │    │                  │    │                  │
├─────────────────┤    ├──────────────────┤    ├──────────────────┤
│ • Приложения    │    │ • Agent          │    │ • Grafana        │
│ • Инфраструктура│    │ • Collector      │    │ • Kibana         │
│ • Базы данных   │    │ • Message Broker │    │ • Data Explorer  │
│ • Сети          │    │ • ETL Pipeline   │    │ • Alert Manager  │
└─────────────────┘    └──────────────────┘    └──────────────────┘
```

## Ключевые метрики для Observability

### **Метрики [[Metrics|RED]]**
- **Rate** - количество запросов в секунду
- **Errors** - процент ошибок
- **Duration** - время обработки запросов

### **Метрики [[Metrics|USE]]**
- **Utilization** - использование ресурсов
- **Saturation** - насыщение (очереди, задержки)
- **Errors** - ошибки ресурсов


## Реализация в микросервисной архитектуре

### **Стек технологий**
```yaml
metrics:
  collector: Prometheus
  visualization: Grafana
  alerting: Alertmanager

logs:
  collection: Fluentd/Fluent Bit
  storage: Elasticsearch
  visualization: Kibana

tracing:
  collection: Jaeger/Zipkin
  instrumentation: OpenTelemetry

infrastructure:
  kubernetes: kube-state-metrics
  containers: cAdvisor
```

### **Паттерны реализации**

1. **Distributed Tracing**
2. **Structured Logging**
3. **Context Propagation**
4. **Correlation IDs**

## Пример сценария использования

**Проблема**: Увеличилась задержка в checkout процессе

**Анализ через Observability**:

1. **Метрики**: P95 latency вырос с 200ms до 800ms
2. **Трейсы**: Payment service показывает увеличение с 50ms до 600ms
3. **Логи**: В payment service появились WARN-логи о таймаутах к внешнему API
4. **Вывод**: Проблема во внешнем платежном шлюзе

## Лучшие практики

### **Инструментирование**
- Внедряйте observability на этапе разработки
- Используйте автоматическое инструментирование
- Стандартизируйте форматы логов и метрик

### **Культура**
- Обучение команды принципам observability
- Создание дашбордов для разных ролей
- Регулярные обзоры и улучшения

### **Безопасность**
- Маскировка чувствительных данных в логах
- Контроль доступа к observability данным
- Аудит использования

## Observability vs [[Monitoring]]

| **Мониторинг**     | **Observability**       |
| ------------------ | ----------------------- |
| Реактивный подход  | Проактивный подход      |
| Известные проблемы | Неизвестные проблемы    |
| Метрики и алерты   | Метрики + логи + трейсы |
| "Что сломалось?"   | "Почему сломалось?"     |
|                    |                         |

Observability превращает данные о работе системы в понимание и действия, позволяя строить более надежные и предсказуемые информационные системы.