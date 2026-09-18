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
  - например, [[kubernetes-scaling|Kubernetes scaling]]
- Улучшай наблюдаемость системы
  - [[monitoring|Monitoring]] – что сломалось в системе?
    - **Реактивный подход** к анализу состояния системы
    - [[metrics-highload|Метрики мониторинга]], [[SLI|SLI]]
    - [[prometheus|Prometheus]] + [[grafana|Grafana]]
  - [[observability|Observability]] – почему это сломалось?
    - **Проактивный подход** к анализу состояния системы
    - 1. [[metrics-highload|Highload metrics]] – метрики системы
      - Агрегация метрик по [[metrics-highload|USE]], [[metrics-highload|RED]], [[metrics-highload|Four Golden Signals]]
      - [[reliability-metrics|метрики доступности системы]]
    - 2. [[logging|Logging]] – события
    - 3. [[tracing|Tracing]] – трассировка запросов
    - 4. [[ELK|ELK]] – стэк для работы с распределенными логами и трейсами
- Используй [[stream-processing|потоковую обработку]] в среде реального времени, где это требуется.

## highload в realtime-среде

**plan**

- Проектирование front-to-back и [[back-to-back|Back-to-back]] интеграций.
- Технологии: [[REST|REST]], GraphQL, WebSockets, [[RPC|gRPC]], [[rabbit-mq|RabbitMQ]], Apache ActiveMQ Artemis и Kafka.

**data**

- front-to-back
  - [[client-pull|Client pull]]
    - паттерн Polling
      - short polling
      - long polling
    - композиция API
  - [[server-push|Server push]]
    - [[web-socket|WebSocket]]
    - [[server-push|SSE]] (Server sent events)
    - [[server-push|GraphQL Subscriptions]]
- [[back-to-back|Back-to-back]]
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
  - [[REST|REST]] для общего случая;
  - [[graph-ql|GraphQL]] чтобы избежать избыточности.

## [[kubernetes-scaling|Kubernetes scaling]]

- **[[kubernetes-scaling|VPA]]** – Vertical Pod Autoscaler – выделяем больше ресурсов поду
  - Существует три режима работы VPA: «Off», «Initial» и «Auto»
- **[[kubernetes-scaling|HPA]]** – Horizontal Pod Autoscaler – наращивает число подов
- **CA** – Cluster Autoscaler – наращиваем число нодов

## [[event-driven-architecture|Event-driven architecture]]

- [[event-driven-architecture|EDA]] – событийно-ориентированная архитектура
  - [[event-storming|Event storming]]
  - [[CQRS|CQRS]]
  - [[event-sourcing|Event sourcing]]
    - специализированные БД событий
      - [[event-store-db|EventStoreDB]]
      - Axon Server
      - Apache Kafka
    - паттерн [[snapshotting|Snapshotting]], чтобы кропать длину лога
  - паттерны обеспечения консистентности событий между сервисом и ивент-басом
    - [[transaction-log-tailing|Transaction log tailing]]
      - Есть внешний tailer, который напрямую читает WAL и выносит из него события в брокер
    - [[transactional-outbox|Transactional outbox]]
      - дополнительная таблица, в которую выносятся события, появляющиеся в 1 транзакции
      - отдельный демон публикует события этой таблицы в брокер
  - событийно-ориентированная обработка ошибок

## [[stream-processing|Stream processing]]

- Существует 4 модели потоковой обработки:
  - синхронная микропакетная обработка
  - асинхронная микропакетная обработка
  - streaming-модель
  - модель непрерывного обновления
- В потоковой обработке используется метод оконной агрегации
- Apache Kafka, [[realtime-highload|Flink]] и [[realtime-highload|Storm]]
