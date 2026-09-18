---
aliases:
  - CAP-теорема
  - CQRS
  - Distributed information systems
  - Event Sourcing
  - Eventual Consistency
  - Last write wins
  - LWW
  - saga
  - Strong Consistency
  - two-phase commit
  - двухфазный коммит
  - микросервисы
  - распределенные информационные базы
  - распределенные информационные системы
  - распределённые транзакции
  - согласованность в конечном итоге
  - строгая согласованность
---

## Распределённые информационные системы

### Целостность данных

```mermaid
---
title: Целостность данных в распределённых системах
---
graph LR

    %% ========== ОСНОВНЫЕ БЛОКИ ==========
    A[Atomicity]:::atomic
    B[Consistency]:::consistency
    C[Isolation]:::isolation
    D[Durability]:::durability

    %% ========== ПОДБЛОКИ ==========
    subgraph "Техники обеспечения целостности"
        direction TB
        E[Двухфазный коммит, паттерн Saga, распределённые транзакции]:::tech
        F[Согласованность в конечном итоге]:::tech
        G[Репликация данных, журнал транзакций, Event Sourcing]:::tech
        H[Контроль версий данных, регулярные аудиты целостности, End-to-end шифрование, логирование и мониторинг, тестирование на отказоустойчивость, проверка на идемпотентность]:::tech
    end

    %% ========== СВЯЗИ ==========
    A --> E
    B --> F
    C --> H
    D --> G

    E --> H
    F --> H
    G --> H

    %% ========== СТИЛИ ==========
    classDef atomic fill:#ff69b4,stroke:#fff,color:#fff
    classDef consistency fill:#00a080,stroke:#fff,color:#fff
    classDef isolation fill:#1e50b7,stroke:#fff,color:#fff
    classDef durability fill:#f0b34d,stroke:#fff,color:#fff
    classDef tech fill:#ffffff,stroke:#333

    class A atomic
    class B consistency
    class C isolation
    class D durability
    class E,F,G,H tech
```

### CAP-теорема

Главная проблема распределённых информационных систем — [[cap-theorem|CAP theorem]].

При проектировании распределённой информационной системы приходится выбирать компромисс. Главная дилемма состоит в том, что для улучшения доступности системы приходится снижать требования к целостности и надёжности.

В частности, приходится принимать:

- модель **Eventual Consistency** вместо **Strong Consistency**;
- механизмы, допускающие потерю данных:
    - например, LWW (Last Write Wins) как способ разрешения конфликтов.

**Eventual Consistency vs Strong Consistency**

```mermaid
---
title: Строгая согласованность vs согласованность в конечном итоге
---
graph TB
    %% ========== ЛЕВЫЙ БЛОК: СТРОГАЯ СОГЛАСОВАННОСТЬ ==========
    subgraph "Строгая согласованность"
        direction TB

        %% Шаги
        step1[1]:::step
        step2[2]:::step
        step3[3]:::step
        step4[4]:::step

        %% Ноды
        node1[Нода 1]:::node
        node2[Нода 2]:::node

        %% Описание
        desc1["Система всегда возвращает факт или событие последней записи; Восстановление данных гарантировано"]:::desc

        %% Связи
        step1 -->|Запись от клиента| node1
        node1 -->|Запись распространилась по кластеру| step2
        step2 --> node2
        node2 -->|Внутреннее подтверждение| step3
        step3 --> node1
        node1 -->|Подтверждение отправлено клиенту| step4
        step4 -->|Подтверждение отправлено клиенту| client1[Клиент]

        style step1 fill:#000,stroke:#fff,color:#fff
        style step2 fill:#000,stroke:#fff,color:#fff
        style step3 fill:#000,stroke:#fff,color:#fff
        style step4 fill:#000,stroke:#fff,color:#fff
    end

    %% ========== ПРАВЫЙ БЛОК: СОГЛАСОВАННОСТЬ В КОНЕЧНОМ ИТОГЕ ==========
    subgraph "Согласованность в конечном итоге"
        direction TB

        %% Шаги
        step5[1]:::step
        step6[2]:::step
        step7[3]:::step

        %% Ноды
        node3[Нода 1]:::node
        node4[Нода 2]:::node

        %% Описание
        desc2["В конечном итоге система возвращает факт или событие последней записи; Если нода падает, возможна потеря данных"]:::desc

        %% Связи
        step5 -->|Запись от клиента| node3
        node3 -->|Подтверждение отправлено клиенту| step6
        step6 -->|Подтверждение отправлено клиенту| client2[Клиент]
        node3 -->|Итоговое распространение записи| step7
        step7 --> node4

        style step5 fill:#000,stroke:#fff,color:#fff
        style step6 fill:#000,stroke:#fff,color:#fff
        style step7 fill:#000,stroke:#fff,color:#fff
    end

    %% ========== СТИЛИ ==========
    classDef title_blue fill:#1e50b7,stroke:#fff,color:#fff
    classDef title_pink fill:#ff69b4,stroke:#fff,color:#fff
    classDef step fill:#ffffff,stroke:#000
    classDef node fill:#1e50b7,stroke:#fff,color:#fff

    class title1 title_blue
    class title2 title_pink
    class node1,node2,node3,node4 node
```

### Распределённые транзакции

Когда операции охватывают несколько сервисов, обеспечение согласованности и целостности данных требует управления распределёнными транзакциями. Распределённые транзакции сложны из-за необходимости поддерживать свойства [[acid-consistency|acid-consistency]] (Atomicity, Consistency, Isolation, Durability) в различных сервисах.

Распределённые транзакции координируют выполнение операций в нескольких сервисах, чтобы гарантировать, что все части транзакции либо зафиксируются, либо откатятся вместе. Для решения этих задач существуют разные подходы.

**Паттерны распределённых транзакций**
- Способы обеспечить атомарность распределённых транзакций:
    - [[saga|Saga]]
    - [[two-phase-commit|Two-phase commit]]
- Способы компенсировать отсутствие атомарности распределённых транзакций:
    - [[event-sourcing|Event sourcing]]
    - [[CQRS|CQRS]]

### Принцип нулевой потери данных

**Принцип нулевой потери данных** предполагает, что данные ни при каких обстоятельствах не должны теряться или повреждаться. В контексте требований [[acid-consistency|acid-consistency]] эту концепцию можно связать с устойчивостью (Durability).
