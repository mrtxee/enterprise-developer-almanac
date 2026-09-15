---
aliases:
  - high-load
  - highload
  - load metrics
  - высоконагруженные системы
  - нагрузка
---

## Повышение отказоустойчивости системы

- Повышай отказоустойчивость системы [[resilience-patterns|Fault Tolerance]]
  - Используй [[resilience-patterns|паттерны устойчивости]] при проектировании системы
- Используй событийно-ориентированную архитектуру [[event-driven-architecture|EDA]] для декуплинга распределенных систем
- Используй [[software-architect/highload/caching/caching|кэширование]] для кратного снижения нагрузки
- Проводи [[performance-testing|нагрузочное тестирование]]
- Автоматизируй управление масштабированием
  - например, [[kubernetes-scaling]]
- Улучшай наблюдаемость системы
  - [[monitoring]] – что сломалось в системе?
    - **Реактивный подход** к анализу состояния системы
    - [[metrics-highload|Метрики мониторинга]], [[SLI|SLI]]
    - [[Prometheus]] + [[grafana]]
  - [[observability]] – почему это сломалось?
    - **Проактивный подход** к анализу состояния системы
    - 1. [[metrics-highload]] – метрики системы
      - Агрегация метрик по [[metrics-highload|USE]], [[metrics-highload|RED]], [[metrics-highload|4-Golden-Signals]]
      - [[reliability-metrics|метрики доступности системы]]
    - 2. [[logging]] – события
    - 3. [[tracing]] – трассировка запросов
    - 4. [[ELK]] – стэк для работы с распределенными логами и трейсами
- Используй [[stream-processing|потоковую обработку]] в среде реального времени, где это требуется.

## highload в realtime-среде

**plan**

- Проектирование front-to-back и back-to-back интеграций.
- Технологии: REST, GraphQL, WebSockets, gRPC, RabbitMQ, Apache ActiveMQ Artemis и Kafka.

**data**

- front-to-back
  - [[client-pull]]
    - паттерн Polling
      - short polling
      - long polling
    - композиция API
  - [[server-push]]
    - [[web-socket|webSocket]]
    - [[server-push|SSE]] (Server sent events)
    - [[server-push|GraphQL Subscriptions]]
- [[back-to-back|back-to-back]]
  - Характер взаимодействия
    - p2p – точка-точка
    - [[publish-subscribe|Publish/Subscribe]] – публикация-подписка
  - Способ взаимодействия
    - синхронный
    - асинхронный

> pull – тянуть
> polling – опрос

### Композиция API

- С реализацией через:
  - [[REST]] для общего случая;
  - [[GraphQL]] чтобы избежать избыточности.

## [[kubernetes-scaling]]

- **VPA** – Vertical Pod Autoscaler – выделяем больше ресурсов поду
  - Существует три режима работы VPA: «Off», «Initial» и «Auto»
- **HPA** – Horizontal Pod Autoscaler – наращивает число подов
- **CA** – Cluster Autoscaler – наращиваем число нодов

## [[event-driven-architecture]]

- [[event-driven-architecture|EDA]] – событийно-ориентированная архитектура
  - [[event-storming]]
  - [[CQRS]]
  - [[event-sourcing]]
    - специализированные БД событий
      - [[event-store-db]]
      - Axon Server
      - Apache Kafka
    - паттерн [[snapshotting]], чтобы кропать длину лога
  - паттерны обеспечения консистентности событий между сервисом и ивент-басом
    - [[transaction-log-tailing]]
      - Есть внешний tailer, который напрямую читает WAL и выносит из него события в брокер
    - [[transactional-outbox]]
      - дополнительная таблица, в которую выносятся события, появляющиеся в 1 транзакции
      - отдельный демон публикует события этой таблицы в брокер
  - событийно-ориентированная обработка ошибок

## [[stream-processing]]

- Существует 4 модели потоковой обработки:
  - синхронная микропакетная обработка
  - асинхронная микропакетная обработка
  - streaming-модель
  - модель непрерывного обновления
- В потоковой обработке используется метод оконной агрегации
- Apache Kafka, Flink и Storm
