---
aliases:
  - Открытые форматы таблиц
  - Transactional Table Formats for Big Data
  - Data Lake Table Formats
  - Open Table Formats
  - Форматы хранения табличных данных для больших данных
---
> Open Table Formats
> Data Lake Table Formats
> Transactional Table Formats for Big Data
> Открытые форматы таблиц
> Форматы хранения табличных данных для больших данных

Распространенные форматы: Delta Lake, Apache Hudi, Apache [[Iceberg]]
Конечно! **Delta Lake**, **Apache Hudi** и **Apache Iceberg** — это три ведущих **open-table format** (формата таблиц), превращающих объектные хранилища (S3, ADLS, GCS) в надёжные **lakehouse**. Все они решают схожие задачи, но с разными подходами.

Ниже — **детальное сравнение** по ключевым критериям.

---

## 📊 Сравнительная таблица

| Критерий                      | **Delta Lake**                                                                       | **Apache Hudi**                                        | **Apache Iceberg**                                           |
| ----------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| **Инициатор**                 | Databricks                                                                           | Uber                                                   | Netflix                                                      |
| **Лицензия**                  | Delta Lake: Apache 2.0 + proprietary features<br>**OSS Delta**: полностью Apache 2.0 | Apache 2.0                                             | Apache 2.0                                                   |
| **Основной движок**           | Spark                                                                                | Spark, Flink                                           | Spark, Trino, Flink, Hive                                    |
| **Поддержка SQL-движков**     | ✅ Spark<br>⚠️ Trino (ограниченно)<br>❌ PrestoDB                                      | ⚠️ Trino/Presto (ограниченно)<br>✅ Spark/Flink         | ✅ **Trino, Spark, Flink, Hive, Dremio**                      |
| **ACID-транзакции**           | ✅                                                                                    | ✅                                                      | ✅                                                            |
| **Time Travel**               | ✅                                                                                    | ✅                                                      | ✅                                                            |
| **Эволюция схемы**            | ✅                                                                                    | ✅                                                      | ✅                                                            |
| **Типы операций**             | - Append<br>- Upsert (merge)<br>- Delete                                             | - **COW** (Copy-On-Write)<br>- **MOR** (Merge-On-Read) | - Append<br>- Upsert<br>- Delete<br>- **Positional deletes** |
| **Производительность записи** | Хорошая                                                                              | 🔥 **Лучшая при частых обновлениях** (MOR)             | Хорошая                                                      |
| **Производительность чтения** | Хорошая                                                                              | COW: быстрая<br>MOR: медленнее                         | 🔥 **Лучшая при большом числе файлов** (manifests)           |
| **Каталог метаданных**        | Hive Metastore, Unity Catalog                                                        | Hive Metastore, AWS Glue                               | Hive Metastore, **Nessie**, JDBC, Glue                       |
| **Branching / Tagging**       | ❌                                                                                    | ❌                                                      | ✅ (**с Nessie**)                                             |
| **Streaming**                 | ✅ (Spark Structured Streaming)                                                       | ✅ (**лучше всего** для потоков)                        | ✅ (Flink, Spark)                                             |
| **Язык запросов**             | Spark SQL                                                                            | Spark SQL, [[HiveQL]]                                  | **SQL (любой движок)**                                       |
| **Сообщество**                | Большое (Databricks)                                                                 | Растущее                                               | 🔥 **Самое открытое и нейтральное**                          |

---

## 🔍 Подробный разбор ключевых аспектов

### 1. **Открытость и экосистема**
- **Iceberg**: полностью open-source, поддерживается **Netflix, Apple, AWS, Google, Dremio**.  
  → **Независим от вендора**, лучшая совместимость.
- **Hudi**: open-source, изначально от Uber, активно развивается **AWS, Alibaba**.
- **Delta Lake**: OSS-версия есть, но **лучшие фичи — только в Databricks** (Unity Catalog, serverless).

> 💡 Если вы **не в Databricks** — Iceberg или Hudi предпочтительнее.

---

### 2. **Поддержка SQL-движков**
- **Iceberg**:  
  ✅ **Trino, Presto, Spark, Flink, Hive, Dremio, Snowflake (preview)**  
  → Идеален для **мультидвижковых сред**.
- **Delta Lake**:  
  ✅ Spark  
  ⚠️ Trino (требует коннектора, не все фичи)  
  ❌ PrestoDB
- **Hudi**:  
  ✅ Spark, Flink  
  ⚠️ Trino (ограниченная поддержка)

> 🎯 **Iceberg — лидер по совместимости**.

---

### 3. **Модель обновлений**
- **Delta Lake**:  
  Использует **transaction log** (журнал транзакций).  
  Upsert через `MERGE INTO`.
- **Hudi**:  
  - **COW**: перезаписывает весь файл при обновлении → быстрое чтение, медленная запись.  
  - **MOR**: хранит дельты отдельно → быстрая запись, медленное чтение (требует компакции).
- **Iceberg**:  
  Использует **snapshot-based модель**.  
  Поддерживает **positional deletes** — удаление по позиции в файле (без перезаписи).

> 💡 **Hudi MOR** — лучший выбор для **частых обновлений** (например, CDC).  
> **Iceberg** — лучше для **аналитики с редкими обновлениями**.

---

### 4. **Производительность**
- **Чтение**:  
  Iceberg выигрывает за счёт **manifest-файлов** с min/max статистикой → эффективный **partition pruning**.
- **Запись**:  
  Hudi (MOR) быстрее при высокой частоте обновлений.  
  Delta и Iceberg требуют больше ресурсов на запись.

---

### 5. **Дополнительные возможности**
| Возможность | Delta | Hudi | Iceberg |
|------------|-------|------|---------|
| **Ветвление (branching)** | ❌ | ❌ | ✅ (с Nessie) |
| **Row-level deletes** | ✅ | ✅ | ✅ (positional deletes) |
| **Schema evolution** | ✅ | ✅ | ✅ |
| **Hidden partitioning** | ✅ | ✅ | ✅ |
| **Time travel** | ✅ | ✅ | ✅ |

> 💡 **Iceberg + Nessie** = Git для данных (ветки, теги, CI/CD).

---

## 🎯 Когда что выбрать?

| Сценарий | Рекомендация |
|---------|--------------|
| **Вы полностью в Databricks** | ✅ **Delta Lake** (лучшая интеграция) |
| **Частые обновления (CDC, IoT)** | ✅ **Apache Hudi** (MOR) |
| **Мультидвижковая аналитика (Trino + Spark + Flink)** | ✅ **Apache Iceberg** |
| **Нужны ветки и теги (CI/CD для данных)** | ✅ **Iceberg + Nessie** |
| **Простая аналитика без обновлений** | Любой, но **Iceberg** — самый открытый |

---

## 🧩 Архитектурные различия

### Delta Lake
```
[Data Files] + [Transaction Log (_delta_log)]
```
→ Журнал транзакций управляет версиями.

### Apache Hudi
```
[Base Files] + [Delta Logs (для MOR)]
```
→ Два режима: COW и MOR.

### Apache Iceberg
```
[Data Files] + [Manifest Lists] + [Metadata Files]
```
→ Snapshot-based модель, никаких "логов" в пути.

---

## ⚠️ Важные нюансы

1. **Delta Lake вне Databricks** — возможен, но теряются преимущества (Unity Catalog, оптимизации).
2. **Hudi требует компакции** — фоновые задачи для MOR → сложность оперирования.
3. **Iceberg требует каталога метаданных** — Hive Metastore или Nessie.

---

## 💡 Заключение

> - **Delta Lake** — выбор для **экосистемы Databricks**.  
> - **Apache Hudi** — выбор для **потоковой обработки и частых обновлений**.  
> - **Apache Iceberg** — выбор для **открытой, мультидвижковой lakehouse-архитектуры**.

Все три формата **зрелые и production-ready**. Выбор зависит от:
- Вашей **экосистемы**
- **Требований к обновлениям**
- **Желания избегать vendor lock-in**

---

Хочешь пример **миграции с Delta на Iceberg** или **настройки Hudi с Flink**? Напиши — подготовлю! 😊
