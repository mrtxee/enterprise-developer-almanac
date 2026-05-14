---
aliases:
  - OLTP
  - OLAP
  - Online Transaction Processing
  - Online Analytical Processing
---
# OLTP vs OLAP транзакции

**OLTP (Online Transaction Processing)** и **OLAP (Online Analytical Processing)** — это **две разные архитектуры баз данных**, каждая для своей задачи.
> ✅ **OLTP — для "делать"**  
> ✅ **OLAP — для "понимать"**
> 💬 _“OLTP is for the now. OLAP is for the past.”_

| Критерий                     | 🔹OLTP 🔹                        | 🔸OLAP🔸                                   |
| ---------------------------- | -------------------------------- | ------------------------------------------ |
| **Цель**                     | Добавить/обновить/удалить данные | Понять, что происходит, и принять решение  |
| **Запросы**                  | Короткие, частые, CRUD           | Длинные, редкие, сложные (аналитика)       |
| **Объём данных**             | Маленькие транзакции             | Большой (TB)                               |
| **Скорость**                 | Высокая (миллисекунды)           | Медленная (секунды–минуты)                 |
| **Нормализация**             | Высокая (3NF)                    | Низкая (денормализованная, Star Schema)    |
| **Количество пользователей** | Тысячи одновременно              | Несколько (аналитики, бизнес-аналитики)    |
| **Пример**                   | Создать заказ                    | "Какие товары продали больше всего за Q3?" |
| **БД**                       | PostgreSQL, MySQL, Oracle        | ClickHouse, Redshift, BigQuery             |
| **Используется**             | Веб-приложения, CRM, ERP         | BI, Data Science, Reporting                |
| **Масштабируемость**         | Горизонтальная (по серверам)     | Вертикальная (по мощности)                 |
схематично
```mermaid
---
config:
  layout: elk
title: OLTP vs OLAP transactions
---
flowchart TB
 subgraph OLAP["OLAP"]
    direction TB
        OLAPClient["Клиент OLAP"]
        OLAPOperationsWrite["Запросы к БД"]
        OLAPOperationsWrite2["Запросы к БД"]
        OLAPOperationsWrite3["Запросы к БД"]
        OLAPOperationsRead["<br><br>Операции чтения<br><br><br>"]
        OLAPOperationsRead2["<br><br>Операции чтения<br><br><br>"]
        OLAPDatabase["База данных"]
  end
 subgraph OLTP["OLTP"]
    direction TB
        OLTPClient["Клиент OLTP"]
        OLTPOperationsWrite["Операции записи"]
        OLTPOperationsWrite2["Операции записи"]
        OLTPOperationsWrite3["Операции записи"]
        OLTPOperationsWrite4["Операции записи"]
        OLTPOperationsWrite5["Операции записи"]
        OLTPOperationsRead["Операции чтения"]
        OLTPOperationsRead2["Операции чтения"]
        OLTPOperationsRead3["Операции чтения"]
        OLTPOperationsRead4["Операции чтения"]
        OLTPOperationsRead5["Операции чтения"]
        OLTPDatabase["База данных"]
  end
    OLTPOperationsWrite --- OLTPOperationsWrite2
    OLTPOperationsWrite2 --- OLTPOperationsWrite3
    OLTPOperationsWrite3 --- OLTPOperationsWrite4
    OLTPOperationsWrite4 --- OLTPOperationsWrite5
    OLTPOperationsWrite5 --> OLTPDatabase
    OLTPOperationsRead --- OLTPOperationsRead2
    OLTPOperationsRead2 --- OLTPOperationsRead3
    OLTPOperationsRead3 --- OLTPOperationsRead4
    OLTPOperationsRead4 --- OLTPOperationsRead5
    OLTPOperationsRead5 --- OLTPDatabase
    OLTPClient --- OLTPOperationsRead & OLTPOperationsWrite
    OLAPOperationsWrite --- OLAPOperationsWrite2
    OLAPOperationsWrite2 --- OLAPOperationsWrite3
    OLAPOperationsWrite3 --> OLAPDatabase
    OLAPOperationsRead --- OLAPOperationsRead2
    OLAPOperationsRead2 --- OLAPDatabase
    OLAPClient --- OLAPOperationsRead & OLAPOperationsWrite

    OLAPClient@{ shape: rounded}
    OLAPDatabase@{ shape: db}
    OLTPClient@{ shape: rounded}
    OLTPDatabase@{ shape: db}
     OLAPClient:::Rose
     OLAPOperationsWrite:::Rose
     OLAPOperationsWrite2:::Rose
     OLAPOperationsWrite3:::Rose
     OLAPOperationsRead:::Rose
     OLAPOperationsRead2:::Rose
     OLAPDatabase:::Ash
     OLTPClient:::Sky
     OLTPOperationsWrite:::Sky
     OLTPOperationsWrite2:::Sky
     OLTPOperationsWrite3:::Sky
     OLTPOperationsWrite4:::Sky
     OLTPOperationsWrite5:::Sky
     OLTPOperationsRead:::Sky
     OLTPOperationsRead2:::Sky
     OLTPOperationsRead3:::Sky
     OLTPOperationsRead4:::Sky
     OLTPOperationsRead5:::Sky
     OLTPDatabase:::Ash
    classDef Ash stroke-width:1px, stroke-dasharray:none, stroke:#999999, fill:#EEEEEE, color:#000000,stroke-width:1px, stroke-dasharray:none, stroke:#999999, fill:#EEEEEE, color:#000000
    classDef Sky stroke-width:1px, stroke-dasharray:none, stroke:#374D7C, fill:#E2EBFF, color:#374D7C
    classDef Rose stroke-width:1px, stroke-dasharray:none, stroke:#FF5978, fill:#FFDFE5, color:#8E2236
```

**Вывод: реализация OLTP и OLAP процессинга над единым хранилищем данных является антипаттерном** т.к. процессы нацелены на взимоисключающие цели. Приводит к конфликту приоритетов: система не может одновременно оптимизировать и скорость мелких транзакций, и обработку массивных аналитических запросов

**Архитектура: Как они работают вместе?**

```mermaid
graph TB
    A[OLTP Database<br>eg. PostgreSQL] -->|ETL/Streaming| B[Data Warehouse<br>eg. ClickHouse]
    B --> C[BI Tools<br>eg. Power BI, Tableau]
    C --> D[Отчёты и дашборды]

    style A fill:#1e50b7,stroke:#fff,color:#fff
    style B fill:#d4edda,stroke:#155724,color:#000
    style C fill:#ffc107,stroke:#333,color:#000
    style D fill:#6c757d,stroke:#fff,color:#fff
```

→ **OLTP** — для операций  
→ **OLAP** — для аналитики
→ **[[ELT]]/Streaming** — для передачи данных из OLTP в OLAP

---

## ✅ Что такое OLTP?

> **OLTP** — **онлайн-обработка транзакций**.  
> Система, которая обрабатывает **множество коротких, быстрых операций** (INSERT, UPDATE, DELETE).

💬 Примеры:
- Банковские транзакции
- Заказы в интернет-магазине
- Регистрация пользователей

---

## ✅ Что такое OLAP?

> **OLAP** — **онлайн-аналитическая обработка**.  
> Система для **сложных аналитических запросов** (SELECT с GROUP BY, JOIN, SUM, AVG).

💬 Примеры:
- Отчёты по продажам за месяц
- Анализ поведения пользователей
- Прогнозирование спроса

Данные для OLAP процессинга накапливаются в специальных хранилищах [[Data Warehouse]] (DW) путем ETL стриминга из OLTP хранилищ, eg. MS SQL.

---
