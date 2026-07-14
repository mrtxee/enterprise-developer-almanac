---
aliases:
  - Kafka без Zooker
  - kafka
  - kraft
  - ISR
  - OSR
  - Segment
  - partition
  - topic
  - broker
  - Apache Kafka
  - Offset
---
## Структура кафки

Apache Kafka – это шина событий на базе неизменяемого лога. Обеспечивает слабую связанность сервисов, хранение истории и упорядоченную доставку сообщений.

Кафка стала популярной за счет возможностей масштабирования и устойчивости к потерям данных. Кафка вошла в технологический ландшафт таких компаний как [LinkedIn](https://engineering.linkedin.com/teams/data/data-infrastructure/streams/kafka), [Netflix](https://factorhouse.io/articles/netflix-kafka-architecture), [Slack](https://slack.engineering/building-self-driving-kafka-clusters-using-open-source-components/), [Uber](https://factorhouse.io/articles/uber-kafka-architecture),..

Масштабирование кафки достигается за счет системы партиционирования, а устойчивость за счет распределенных репликаций.

Как устроена Кафка?
```mermaid
---
title: Компоненты Kafka
---
flowchart RL
 subgraph Broker1_TopicA_Partition0["🧩 Partition-0"]
    direction TB
        Broker1_TopicA_Partition0_Segment0["Broker1_TopicA_Partition0_Segment0"]
        Broker1_TopicA_Partition0_Segments["💾 Segments"]
  end
 subgraph Broker1_TopicA["📋 Topic-A"]
    direction TB
        Broker1_TopicA_Partitions["🧩 Partitions"]
        Broker1_TopicA_Partition0
  end
 subgraph Broker1_TopicA_Partition0_Segment0["💾 Segment-0"]
    direction TB
        Broker1_TopicA_Partition0_Segment0_File[".log файл"]
        Broker1_TopicA_Partition0_Segment0_Messages["Messages"]
  end
 subgraph Broker1["🖥️ Broker-1"]
    direction TB
        Broker1_Topics["📋 Topics"]
        Broker1_TopicA
        Message["Message"]
  end
 subgraph Kafka_Cluster["☁️ Kafka Cluster"]
    direction RL
        Broker1
        Brokers["🖥️ Brokers"]
  end
 subgraph ConsumerGroup["👥 Consumer Group"]
    direction TB
        ConsumerGroup1_C1["🖥️ Consumer 1<br>хост-клиент"]
        ConsumerGroup1_C2["🖥️ Consumer 2<br>хост-клиент"]
  end
 subgraph Message["📨 Message / record"]
    direction LR
        MessageLogical["record: key, value, headers, timestamp."]
  end
 subgraph Legend["Легенда"]
    direction LR
        LegendPhysical["физическая сущность"]
        LegendLogical["логическая сущность"]
  end
    LegendPhysical ~~~ LegendLogical
    ConsumerGroup1_C1 ~~~ ConsumerGroup1_C2
    ConsumerGroup -. fetch .-> Broker1
    Producer["🖥️ Producer<br>хост-клиент"] -. produce .-> Broker1
    Broker1 -. metadata .-> Producer & ConsumerGroup
    Broker1_TopicA_Partition0_Segment0_Messages -.-o Message

    Broker1_TopicA_Partition0_Segment0@{ shape: stored-data}
    Broker1_TopicA_Partition0_Segments@{ shape: procs}
    Broker1_TopicA_Partitions@{ shape: procs}
    Broker1_TopicA_Partition0_Segment0_File@{ shape: text}
    Broker1_TopicA_Partition0_Segment0_Messages@{ shape: procs}
    Broker1_Topics@{ shape: procs}
    Brokers@{ shape: procs}
    MessageLogical@{ shape: text}
     Broker1_TopicA_Partition0_Segment0:::Logical
     Broker1_TopicA_Partition0_Segment0:::Physical
     Broker1_TopicA_Partition0_Segments:::Physical
     Broker1_TopicA_Partitions:::Logical
     Broker1_TopicA_Partition0:::Logical
     Broker1_TopicA_Partition0_Segment0_Messages:::DataUnit
     Broker1_Topics:::Logical
     Broker1_TopicA:::Logical
     Message:::DataUnit
     Broker1:::Physical
     Brokers:::Physical
     ConsumerGroup1_C1:::Physical
     ConsumerGroup1_C2:::Physical
     LegendPhysical:::Physical
     LegendLogical:::Logical
     ConsumerGroup:::Logical
     Producer:::Physical
     Kafka_Cluster:::Logical
     Legend:::Transparent
    classDef Physical stroke-width:2px, stroke:#2E86AB, fill:#D4EBF8, color:#1D3557
    classDef Logical stroke-width:1px, stroke:#FBB35A, fill:#FFEFDB, color:#8F632D
    classDef Hybrid stroke-width:2px, stroke:#6A4C93, fill:#E8DFF5, color:#4A2C6F
    classDef DataUnit stroke-width:1px, stroke:#06A77D, fill:#D1F4E8, color:#046149
    classDef Transparent fill:transparent, stroke-width:0px
```

`Kafka Cluster` — логическое объединение кафка брокеров.

`Broker` — сервер Kafka, который хранит партиции, может обрабатывает запросы клиентов, может участвовать в выборах лидера кластера

`Topic` — канал, куда отправляются сообщения (например, «логи», «заказы»).

`Partition` — часть топика, упорядоченный журнал сообщений.  
Предназначен для параллельной обработки сообщений. В идеальной ситуации число партиций совпадает с числом потребителей Consumer Group-ы

`Offset` — номер сообщения внутри партиции. Позволяет консюмеру помнить, где остановился.

`Producer / Consumer` –- клиент, который отправляет / потребляет сообщение топика.

`Consumer Group` — Группа консюмеров, которые совместно читают один топик для балансировки нагрузки. С гарантией того, что 2 консюмера одной группы не прочитают одно сообщение.

> [!important] Consumer может существовать вне Consumer Group
> В этом случае кластер не будет хранить оффсет такого консюмера. Эта задача ляжет на разработчиков консюмера.

`Record` - Единица данных, отправляемая в Kafka. Содержит ключ, значение, временную метку и заголовки сообщения.

### Kafka Metadata

`Kafka Metadata` — это «карта» кластера: какие топики есть, сколько у них партиций, кто лидер каждой партиции, какие реплики синхронизированы (In-Sync Replicas). Без неё клиенты не знают, куда слать сообщения или откуда читать.

Обмен данными `Metadata Request` происходит в бинарном формате. При запросе клиент может указать фильтр, по параметрам `include_cluster_info`, `topics`.

Пример десериализованного Kafka Cluster Metadata response body:
```json
{
  "brokers": [
    {"id": 1, "host": "broker1.example.com", "port": 9092},
    {"id": 2, "host": "broker2.example.com", "port": 9092},
    {"id": 3, "host": "broker3.example.com", "port": 9092}
  ],
  "topics": [
    {
      "name": "orders",
      "partitions": [
        {
          "partition": 0,
          "leader": 1,
          "replicas": [1, 2, 3],
          "isr": [1, 2]
        },
        {
          "partition": 1,
          "leader": 2,
          "replicas": [2, 3, 1],
          "isr": [2, 3]
        }
      ]
    },
    {
      "name": "users",
      "partitions": [
        {
          "partition": 0,
          "leader": 3,
          "replicas": [3, 1, 2],
          "isr": [3]
        }
      ]
    }
  ]
}
```

Список активных брокеров, число реплик и их типы могут меняться. Например, из‑за ребаланса, сбоев или масштабирования кластера. Поэтому клиенты регулярно опрашивают брокеров (Metadata Request), чтобы получать актуальную карту кластера.

> [!important] Kafka Metadata при старте клиента
> Список активных брокеров может меняться. Для того, чтобы клиент получил адреса и статус актуальный брокеров и партиций, при конфигурировании клиента задается список адресов брокеров (`bootstrap servers`). Если хотя бы 1 из них активен – клиент получит актуальную карту кластера

#### In-Sync Replica

**ISR (In‑Sync Replica)** — это «надёжные копии» данных в Kafka. Список реплик партиции, которые успевают за лидером и хранят актуальные данные.

В Kafka данные хранятся в нескольких копиях (партициях) на разных брокерах. Если какой‑то брокер сильно отстаёт по синхронизации лога, доверять ему нельзя. Лидер партиции исключает такую реплику из ISR.

Список ISR нужен для:

- **Скорости** — если продюсер требует «запиши надёжно» (`acks=all`), Kafka подтвердит запись только после того, как её сохранят все участники ISR.
    
- **Устойчивости** — в случае сбоя лидера партиции, новым лидером станет кто‑то из ISR.

### Ребаланс Kafka

`Kafka Rebalance` — процесс перераспределения партиций между потребителями внутри одной Consumer Group. Он нужен, чтобы при изменении состава группы нагрузка распределялась равномерно и все партиции оставались обработанными.

Когда происходит ребаланс:

- Изменилось число консюмеров группы — всем нужно выделить leader replica партицию.
- Изменилась топология топика: добавлены новые партиции — их нужно распределить по группе.

### Kafka Segments

Сообщения хранятся в бинарном формате в файловой системе.
* Имя папки:  `<имя топка>–<номер партиции>`

Внутри каждой папки есть **сегменты**.

Сегмент это 3 файла с одинаковым именем и разными расширениями:
- `.LOG` — содержит сообщения
- `.INDEX` — служит для навигации в LOG по номерам сообщений
- `.TIMEINDEX` — служит для навигации в INDEX по времени. Слабая консистентность данных из-за того, что timestamp могу устанавливаться вручную, либо сообщения могут приходить с задержкой.

Имя сегмента включает в себя номер сообщения с которого начинается. Стандартный размер сегмента - 1Гб.

Удалять данные вручную нельзя. Но, можно настроить удаление сегментов по TTL.

## Как работает Kafka

Для обеспечения компетентности состояния кластера требуется координатор. 

Ранее (до v.3.3) координатором выступал Zookeeper. Сегодня такая конфигурация является устаревшей. Роль координатора (`Active controller`) передается между брокерами при помощи алгоритма консенсуса КRaft.

```mermaid
---
title: Топология кластера Kafka с KRaft
---
flowchart TB
 subgraph Broker1["🖥️ Broker 1"]
    direction TB
        Broker1_TopicA_P0_Leader["📋 Topic-A<br>👑 Leader Replica<br>Partition-0"]
        Broker1_TopicB_P1_Follower["📋 Topic-B<br>🔄 Follower Replica<br>Partition-1"]
        Broker1_TopicB_P0_Follower["📋 Topic-B<br>🔄 Follower Replica<br>Partition-0"]
        Broker1Comment["Физический сервер/JVM<br>process.roles=broker,controller<br>KRaft-role: Active Controller"]
  end
 subgraph Broker3["🖥️ Broker 3"]
    direction TB
        Broker3_TopicA_P0_Follower["📋 Topic-A<br>🔄 Follower Replica<br>Partition-0"]
        Broker3_TopicB_P0_Follower["📋 Topic-B<br>🔄 Follower Replica<br>Partition-0"]
        Broker3_TopicB_P1_Follower["📋 Topic-B<br>🔄 Follower Replica<br>Partition-1"]
        Broker3Comment["Физический сервер/JVM<br>process.roles: broker<br>KRaft-role: Follower"]
  end
 subgraph Broker2["🖥️ Broker 2"]
    direction TB
        Broker2_TopicB_P1_Leader["📋 Topic-B<br>👑 Leader Replica<br>Partition-1"]
        Broker2_TopicB_P0_Leader["📋 Topic-B<br>👑 Leader Replica<br>Partition-0"]
        Broker2_TopicA_P0_Follower["📋 Topic-A<br>🔄 Follower Replica<br>Partition-0"]
        Broker2Comment["Физический сервер/JVM<br>process.roles=broker,controller<br>KRaft-role: Follower"]
  end
 subgraph Kafka_Cluster["☁️ Kafka Cluster <i>(логическое объединение брокеров)</i>"]
    direction LR
        Broker1
        Broker2
        Broker3
  end
 subgraph ConsumerGroup["👥 Consumer Group"]
    direction TB
        ConsumerGroup1_C1["🖥️ Consumer 1<br>хост-клиент"]
        ConsumerGroup1_C2["🖥️ Consumer 2<br>хост-клиент"]
        ConsumerGroupCommment["Логичская группа consumer-ов с общим <code>__consumer_offsets</code>"]
        ConsumerGroup1_C3["🖥️ Consumer 3<br>хост-клиент"]
  end
    Broker1Comment ~~~ Broker1_TopicA_P0_Leader
    Broker1_TopicA_P0_Leader ~~~ Broker1_TopicB_P1_Follower
    Broker1_TopicB_P1_Follower ~~~ Broker1_TopicB_P0_Follower
    Broker2Comment ~~~ Broker2_TopicA_P0_Follower
    Broker2_TopicA_P0_Follower ~~~ Broker2_TopicB_P1_Leader
    Broker2_TopicB_P1_Leader ~~~ Broker2_TopicB_P0_Leader
    Broker3Comment ~~~ Broker3_TopicA_P0_Follower
    Broker3_TopicA_P0_Follower ~~~ Broker3_TopicB_P1_Follower
    Broker3_TopicB_P1_Follower ~~~ Broker3_TopicB_P0_Follower
    ConsumerGroup1_C2 -. "fetch (чтение)<br>Topic-B, Partition-1" .-> Broker2
    ConsumerGroup1_C3 -. "fetch (чтение)<br>Topic-B, Partition-0" .-> Broker2
    ConsumerGroup1_C1 -. "fetch (чтение)<br>Topic-A, Partition-0" .-> Broker1
    Producer["🖥️ Producer<br>хост-клиент"] -. "produce record<br>Topic-A Partition-0" .-> Broker1

    Broker1Comment@{ shape: braces}
    Broker3Comment@{ shape: braces}
    Broker2Comment@{ shape: braces}
    ConsumerGroupCommment@{ shape: braces}
     Broker1_TopicA_P0_Leader:::Logical
     Broker1_TopicB_P1_Follower:::Logical
     Broker1_TopicB_P0_Follower:::Logical
     Broker3_TopicA_P0_Follower:::Logical
     Broker3_TopicB_P0_Follower:::Logical
     Broker3_TopicB_P1_Follower:::Logical
     Broker2_TopicB_P1_Leader:::Logical
     Broker2_TopicB_P0_Leader:::Logical
     Broker2_TopicA_P0_Follower:::Logical
     Broker1:::Physical
     Broker2:::Physical
     Broker3:::Physical
     ConsumerGroup1_C1:::Physical
     ConsumerGroup1_C2:::Physical
     ConsumerGroup1_C3:::Physical
     Producer:::Physical
     Kafka_Cluster:::Logical
     ConsumerGroup:::Logical
    classDef Physical stroke-width:2px, stroke:#2E86AB, fill:#D4EBF8, color:#1D3557
    classDef Logical stroke-width:1px, stroke:#FBB35A, fill:#FFEFDB, color:#8F632D
    classDef Hybrid stroke-width:2px, stroke:#6A4C93, fill:#E8DFF5, color:#4A2C6F
    classDef DataUnit stroke-width:1px, stroke:#06A77D, fill:#D1F4E8, color:#046149
    classDef Transparent fill:transparent
```


Чтение/запись происходит только в `Leader Replica` партиции.

Брокеры взаимодействуют друг с другом для:
- репликации записей из Leader replica партиций в Follower replica партиции
- выбора Active Controller (голосуют брокеры с ролью controller)
- синхронизации Metadata (карты кластера)

> [!important] Leader replica failure
> В случае сбоя Leader реплики, Active Controller смотрит на последний актуальный список ISR и назначает нового лидера.

Leader реплики следит за Follower-ами репликации, отслеживает их отставание по LEO (Log End Offset) и ведет локальный список In-Sync Replicas.

### Broker roles

Каждому брокеру назначается 1 из 3 ролей (параметр `process.roles`):
- **broker** — обслуживает клиентов: принимает, хранит, отдаёт сообщения.
- **controller** — управляет кластером: выбирает лидеров партиций, обрабатывает ребалансы, хранит и синхронизирует метаданные (в режиме [[KRaft]]).
- **broker, controller** — обе роли на одном узле. Типично для небольших кластеров.

> [!important] Passive broker
> При некоторых обстоятельствах может образоваться брокер, который содержит только Follower replica партиции. Чаще всего это временное, но штатное состояние системы. Таких брокеров так же называют `cold replica broker`

#### KRaft Active Controller
`Active Controller` выбирается брокерами с ролью `controller` при наличии KRaft‑кворума.

Active Controller делает:

- оркестрирует изменение метаданных (например добавление топика)
- назначает лидеров партиций
- обновляет метаданные на всех брокерах
- следит за ISR/OSR

### Управление доставкой

При отправке сообщения Producer устанавливает требование к уровню подтверждения доставки. Брокер должен выполнить требование, чтобы продюсер счёл отправку успешной.

Параметр `acks` (_acknowledgements_) может принимать 3 значения:

```mermaid
---
title: Kafka acknowledgements
---
flowchart TB
 subgraph s1["acks -1 (all)"]
    direction TB
        Producer1["🖥️ Producer"]
        C22["Broker-1<br>Topic-A Partition-0<br>👑 Leader Replica"]
        C23["Broker-2<br>Topic-A Partition-0<br>🔄 Follower<br>In-Sync Replica"]
        C24["Broker-3<br>Topic-A Partition-0<br>🔄 Follower<br>Out-of-Sync Replica"]
  end
 subgraph acks1["acks 1 (leader)"]
    direction LR
        Producer2["🖥️ Producer"]
        C12["Broker-1<br>Topic-A Partition-0<br>👑 Leader Replica"]
        C13["Broker-2<br>Topic-A Partition-0<br>🔄 Follower<br>In-Sync Replica"]
        C14["Broker-3<br>Topic-A Partition-0<br>🔄 Follower<br>Out-of-Sync Replica"]
  end
 subgraph acks0["acks 0 (none)"]
    direction LR
        Producer3["🖥️ Producer"]
        C02["Broker-1<br>Topic-A Partition-0<br>👑 Leader Replica"]
        C03["Broker-2<br>Topic-A Partition-0<br>🔄 Follower<br>In-Sync Replica"]
        C04["Broker-3<br>Topic-A Partition-0<br>🔄 Follower<br>Out-of-Sync Replica"]
  end
    Producer3 -- (1) write --> C02
    Producer2 -- (1) write --> C12
    C12 -. (2) ack .-> Producer2
    Producer1 -- (1) write --> C22
    C22 -. (4) ack .-> Producer1
    C23 -. (3) ack .-> C22
    C22 -- (2) write --> C23

     Producer1:::Physical
     C22:::Logical
     C23:::Logical
     C24:::Logical
     Producer2:::Physical
     C12:::Logical
     C13:::Logical
     C14:::Logical
     Producer3:::Physical
     C02:::Logical
     C03:::Logical
     C04:::Logical
    classDef Physical stroke-width:2px, stroke:#2E86AB, fill:#D4EBF8, color:#1D3557
    classDef Logical stroke-width:1px, stroke:#FBB35A, fill:#FFEFDB, color:#8F632D
    classDef Hybrid stroke-width:2px, stroke:#6A4C93, fill:#E8DFF5, color:#4A2C6F
    classDef DataUnit stroke-width:1px, stroke:#06A77D, fill:#D1F4E8, color:#046149
    classDef Transparent fill:transparent
```

Уровень гарантии влияет на надёжность и длительность доставки сообщения. Поэтому следует задавать значение с учетом бизнес-требований:

- `acks=0` : Без подтверждения доставки, риски потери данных, оптимально для телеметрии, логов, метрик. [Fire-and-forget паттерн](https://www.enterpriseintegrationpatterns.com/patterns/conversation/FireAndForget.html).
- `acks=1` : Подтверждение от Leader Replica, потеря данных возможна при сбое лидера, оптимально для большинства сценариев. Значение по-умолчанию.
- `acks=-1` : подтверждения доставки от всех ISR реплик. Данные сохраняться даже при сбое лидера, снижение пропускной способности, оптимально для критических данных (напр. платежи, заказы).
