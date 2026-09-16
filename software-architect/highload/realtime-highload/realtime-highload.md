---
aliases:
  - Active-Active
  - Active-Passive
  - Bulkhead
  - Cluster Autoscaler
  - EDA
  - Event Sourcing
  - Event Storming
  - Event-Driven Architecture
  - Flink
  - Geo-Redundancy
  - HPA
  - Kafka
  - Kubernetes
  - Rate Limiting
  - Snapshotting
  - Storm
  - stream processing
  - Transaction log tailing
  - Transactional outbox
  - VPA
  - высокая нагрузка
  - отказоустойчивость
  - потоковая обработка
  - реальное время
---

## Highload в realtime-среде

### Темы

- методы оценки масштабируемости систем и внедрение контейнеризации для динамического масштабирования с помощью Kubernetes и его механизмов HPA, VPA и Cluster Autoscaler;
- Apache Kafka, Flink и Storm;
- стратегии отказоустойчивости: Active-Passive, Active-Active и Geo-Redundancy;
- Rate Limiting и Bulkhead;
- проектирование front-to-back и back-to-back интеграций;
- технологии: [[REST|REST]], [[graph-ql|GraphQL]], [[web-socket|WebSockets]], [[RPC|gRPC]], [[rabbit-mq|RabbitMQ]], Apache ActiveMQ Artemis и Kafka.

### Структура

- [[performance-testing|Нагрузочное тестирование]]
- [[kubernetes-scaling|Kubernetes scaling]]
    - Vertical Pod Autoscaler — выделяем больше ресурсов поду:
        - существует три режима работы VPA: «Off», «Initial» и «Auto»;
    - Horizontal Pod Autoscaler — наращивает число подов;
    - Cluster Autoscaler — наращиваем число нодов.
- [[resilience-patterns|Fault Tolerance]] — отказоустойчивость системы
    - системы [[observability|мониторинга и алертов]];
    - [[resilience-patterns|Rate Limiting]] — ограничение скорости;
    - [[resilience-patterns|Bulkhead]] — отсек;
    - [[retry-policy|Retry policy]];
    - [[rate-limiting|Rate limiter]];
    - устойчивость микросервисов:
        - [[saga|Saga]]:
            - компенсационные транзакции, undo;
            - тайм-ауты и дедлайны;
        - [[two-phase-commit|Two-phase commit]] — двухфазные коммиты.
- [[event-driven-architecture|EDA]] — Event-driven architecture
    - [[event-storming|Event storming]];
    - [[CQRS|CQRS]];
    - [[event-sourcing|Event sourcing]]:
        - специализированные БД событий:
            - [[event-store-db|EventStoreDB]];
            - Axon Server;
            - Apache [[kafka|Kafka]];
        - паттерн [[snapshotting|Snapshotting]], чтобы сократить длину лога;
    - паттерны обеспечения консистентности событий между сервисом и event-базой:
        - [[transaction-log-tailing|Transaction log tailing]]:
            - внешний tailer напрямую читает WAL и выносит события в брокер;
        - [[transactional-outbox|Transactional outbox]]:
            - дополнительная таблица, в которую выносятся события, появляющиеся в одной транзакции;
            - отдельный демон публикует события этой таблицы в брокер;
    - событийно-ориентированная обработка ошибок.
- [[stream-processing|Stream processing]]
    - существует 4 модели потоковой обработки:
        - синхронная микропакетная обработка;
        - асинхронная микропакетная обработка;
        - streaming-модель;
        - модель непрерывного обновления;
    - в потоковой обработке используется метод оконной агрегации.
