---
aliases:
  - Apache Iceberg
  - Data Lake
  - Data Lakehouse
  - Data Warehouse
  - Databricks
  - Delta Lake
  - Snowflake
  - Trino
  - Даталейкхаус
---

## Data Lakehouse: объединение Data Lake и Data Warehouse

**Data Lakehouse (Даталейкхаус)** — это **современная архитектура**, которая **объединяет лучшее из [[data-lake|Data Lake]] и [[data-warehouse|Data Warehouse]]**.

> **Data Lakehouse** — это **платформа, которая позволяет хранить данные в сыром виде (как Data Lake), но с возможностью быстрой аналитики (как Data Warehouse)**.

**Простыми словами:**

> Это **«бассейн данных» + «завод по производству отчётов»** — всё в одном месте.

---

## Зачем нужен Data Lakehouse

| Цель | Объяснение |
|------|------------|
| **Гибкость** | Можно хранить любые данные (JSON, CSV, видео) |
| **Производительность** | Быстрые запросы без [[ELT|ELT]] |
| **Стоимость** | Дешевле, чем традиционный [[data-warehouse|DW]] |
| **Масштабируемость** | До PB/TB данных |
| **Упрощение архитектуры** | Нет необходимости в отдельном DW |

---

## Архитектура Data Lakehouse

```mermaid
---
title: Data Lakehouse: поток данных и потребители
---
flowchart LR
    A[Источники данных<br>eg. OLTP, API, IoT, логи] --> B[Data Lakehouse<br>eg. Databricks, Snowflake, Delta Lake]
    B --> C[BI Tools<br>eg. Power BI, Tableau]
    B --> D[ML/DS<br>eg. Python, Spark]
    B --> E[Real-time Analytics<br>eg. Flink, Kafka]

    style A fill:#6c757d,stroke:#fff,color:#fff
    style B fill:#1e50b7,stroke:#fff,color:#fff
    style C fill:#d4edda,stroke:#155724,color:#000
    style D fill:#ffc107,stroke:#333,color:#000
    style E fill:#e9ecef,stroke:#6c757d,color:#000
```

### Как работает Data Lakehouse

1. **Данные приходят** в сыром виде (JSON, CSV, [[parquet|Parquet]]).
2. **Хранятся в формате таблиц** (например, [[delta-lake|Delta Lake]]).
3. **Можно выполнять SQL-запросы** прямо на них.
4. **Пользователи** могут:
   - строить дашборды (Power BI);
   - делать ML (Python);
   - анализировать в реальном времени (Flink).
→ **Без необходимости в [[ETL|ETL]] и отдельном DW**

### Основные компоненты

| Компонент | Роль |
|-----------|------|
| **Источники данных** | [[OLTP|OLTP]], [[kafka|Kafka]], файлы, IoT-устройства |
| **Data Lakehouse** | Databricks, Snowflake, [[delta-lake|Delta Lake]], Iceberg |
| **BI-инструменты** | Power BI, Tableau — для дашбордов |
| **ML/DS** | Python, PySpark, TensorFlow — для анализа |
| **Реальная аналитика** | Flink, Kafka — для потоков |

### Технологии Data Lakehouse

| Платформа | Особенности |
|-----------|-------------|
| **Databricks** | Использует Delta Lake, поддерживает Spark, ML, BI |
| **Snowflake** | Cloud-native, поддерживает SQL, BI, ML |
| **Delta Lake** | Открытое хранилище на основе Parquet + ACID |
| **Apache Iceberg** | Открытый стандарт для таблиц в [[data-lake|Data lake]] |
| **Presto/Trino** | Для быстрых запросов к данным |

### Преимущества и недостатки Data Lakehouse

| Плюс/Минус | Объяснение |
|------------|-----------|
| ✅ Единая платформа | Нет необходимости в двух системах |
| ✅ Быстрая аналитика | SQL-запросы на сырых данных |
| ✅ ACID-гарантии | Поддержка транзакций (в Delta Lake, Iceberg) |
| ✅ Масштабируемость | До PB/TB данных |
| ✅ Гибкость | Поддержка всех типов данных |
| ✅ Дешевизна | Хранение на S3/HDFS |
| ❌ Сложность | Требует знаний о Spark, SQL, ML |
| ❌ Безопасность | Требуется управление доступом (IAM, ACL) |
| ❌ Качество данных | Нет автоматической проверки данных |
| ❌ Обучение команды | Нужны навыки работы с новой архитектурой |

---

**Когда использовать Data Lakehouse**

| Сценарий | Рекомендация |
|----------|--------------|
| Нужно хранить все данные (логи, события, фото) | ➤ **Data Lakehouse** |
| Нужно делать быстрые отчёты | ➤ **Data Lakehouse** |
| Нужно делать ML/DS | ➤ **Data Lakehouse** |
| Нужно анализировать в реальном времени | ➤ **Data Lakehouse** |
| Нужны сложные отчёты с историей | ➤ **Data Warehouse** (если уже есть) |

---

## Data Lake vs Data Warehouse vs Data Lakehouse

| Критерий | **Data Lake** | **Data Warehouse** | **Data Lakehouse** |
|----------|---------------|--------------------|--------------------|
| **Формат данных** | Любые (сырые) | Структурированные | Структурированные + сырые ⭐ |
| **Цель** | Хранение и исследование | Аналитика и отчёты | Все: хранение, аналитика, ML ⭐ |
| **Скорость запросов** | Медленная | Быстрая ⭐ | Быстрая ⭐ |
| **Стоимость** | Низкая | Высокая ⭐ | Средняя |
| **Масштабируемость** | Очень высокая ⭐ | Высокая | Очень высокая ⭐ |
| **Примеры** | S3, HDFS | ClickHouse, Redshift | Databricks, Snowflake, Delta Lake |

> ✅ **Data Lakehouse — это будущее аналитики**.
> Он **объединяет гибкость Data Lake и производительность Data Warehouse** — в одной платформе.
