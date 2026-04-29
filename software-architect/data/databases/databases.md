---
aliases:
  - выбрать базу данных
  - выбрать бд
  - какую базу выбрать
---

гайд по выбору базы
```mermaid
graph LR
    A[Какую базу данных выбрать?] --> B{Данные структурированы?}
    B -- да --> C{Данных много?}
    B -- нет --> D{Используется стек MEAN?}

    C -- да --> E{Нужен технологически независимый стек?}
    C -- нет --> F{Нагрузка смешанная?}

    E -- да --> G{Нужна быстрая разработка?}
    E -- нет --> H{Технологии Microsoft?}

    G -- да --> I[PostgreSQL]
    G -- нет --> J{Есть сложные операции с данными?}

    H -- да --> K[MS SQL]
    H -- нет --> L{Используется PHP?}

    L -- да --> M[MySQL]
    L -- нет --> N[PostgreSQL]

    F -- да --> O{Используются бессерверные вычисления?}
    F -- нет --> P[MySQL]

    O -- да --> Q[Yandex Database]
    O -- нет --> R[MySQL]

    J -- да --> S{Есть администратор БД?}
    J -- нет --> T[ClickHouse]

    S -- да --> U{Нужно изменять данные?}
    S -- нет --> V[Redis]

    U -- да --> W[Elasticsearch]
    U -- нет --> X[ClickHouse]

    D -- да --> Y{Основная цель — аналитические запросы?}
    D -- нет --> Z{Данных много?}

    Y -- да --> AA{Нужен полнотекстовый поиск?}
    Y -- нет --> AB[ClickHouse]

    AA -- да --> AC[Elasticsearch]
    AA -- нет --> AD[Redis]

    Z -- да --> AE[MongoDB]
    Z -- нет --> AF[MySQL]

    %% Стили узлов по категориям
    classDef sql fill:#4A90E2,stroke:#333,color:white;
    classDef nosql fill:#7ED321,stroke:#333,color:black;
    classDef analytics fill:#F5A623,stroke:#333,color:black;
    classDef search fill:#D0021B,stroke:#333,color:white;
    classDef cache fill:#9013FE,stroke:#333,color:white;

    class I,M,K,N,P,R,AF sql
    class AE nosql
    class T,X,AB,Q analytics
    class W,AC search
    class V,AD cache

    %% Стили вопросов (светлый фон)
    classDef question fill:#f9f9f9,stroke:#555,color:#333;
    class B,C,D,E,F,G,H,J,L,O,S,U,Y,Z,AA question
```


duarbility в распределенных базах

| База данных | Транзакционная поддержка (ACID) | Журналирование (WAL/Binary Logs) | Контрольные суммы       | Репликация             | Идемпотентность        | Eventual Consistency |
| ----------- | ------------------------------- | -------------------------------- | ----------------------- | ---------------------- | ---------------------- | -------------------- |
| PostgreSQL  | Да                              | Да (WAL)                         | Да                      | Синхронная/асинхронная | Нет                    | Нет                  |
| Cassandra   | Lightweight Transactions        | Нет                              | Да (на уровне хранения) | Асинхронная            | Да (через конфликты)   | Да                   |
| MongoDB     | На уровне документа             | Нет                              | Нет                     | Асинхронная            | Да (через поля версий) | Да                   |
| Oracle      | Да                              | Да                               | Да                      | Синхронная             | Да                     | Нет                  |
| MySQL       | Да (InnoDB)                     | Да (Binary Logs)                 | Нет                     | Асинхронная            | Нет                    | Нет                  |
| ClickHouse  | Нет                             | Нет                              | Да                      | Асинхронная            | Да                     | Да                   |