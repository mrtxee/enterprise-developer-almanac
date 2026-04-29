**Apache Airflow** и **Camunda** — оба используются для **управления рабочими процессами**, но они **совершенно разные по философии, архитектуре и назначению**.

---

## ✅ Краткое сравнение

| Характеристика              | **Apache Airflow**                  | **Camunda**                               |
| --------------------------- | ----------------------------------- | ----------------------------------------- |
| **Основное назначение**     | Оркестрация ETL, ML, Data Pipelines | Управление бизнес-процессами (BPM)        |
| **Язык описания процессов** | Python (DAGs)                       | BPMN 2.0 (в визуальном редакторе)         |
| **Целевая аудитория**       | Data Engineers, DevOps              | Business Analysts, Process Owners         |
| **Где используется**        | Обработка данных, ML, скрипты       | Согласование заказов, onboarding, кредиты |
| **Типичная нагрузка**       | Задачи по расписанию                | Человеко-ориентированные процессы         |
| **UI / Dashboard**          | Для мониторинга DAG'ов              | Для управления процессами + история       |
| **Интеграции**              | Spark, BigQuery, Kafka, AWS         | REST, JMS, Email, LDAP, DMN               |
| **Моделирование**           | Ручное (Python код)                 | Визуальное (BPMN)                         |
| **Работа с людьми**         | ❌ Почти нет                         | ✅ Да — задачи на пользователя             |
| **События (Event-driven)**  | ⚠️ Через ExternalTaskSensor         | ✅ Да — события, таймеры, сообщения        |
| **Формы**                   | ❌ Нет                               | ✅ Да (embedded form, Camunda Forms)       |
| **Подходит для**            | Инженеры, data pipelines            | Бизнес-процессы, workflow                 |

> 🔑 **Airflow = для данных**  
> **Camunda = для людей и бизнес-логики**

---

## ✅ 1. Apache Airflow — для инженеров

> **[[Airflow]]** — это **платформа для программной оркестрации сложных пайплайнов**:  
> - [[ELT]]
> - [[ML|Машинное обучение]]
> - Аналитика
> - Скрипты администрирования

### 💡 Пример: DAG в Airflow

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

def extract_data():
    print("Extract from PostgreSQL")

def transform_data():
    print("Transform with Pandas")

def load_data():
    print("Load to BigQuery")

dag = DAG('etl_pipeline', schedule_interval='@daily')

t1 = PythonOperator(task_id='extract', python_callable=extract_data, dag=dag)
t2 = PythonOperator(task_id='transform', python_callable=transform_data, dag=dag)
t3 = PythonOperator(task_id='load', python_callable=load_data, dag=dag)

t1 >> t2 >> t3
```

→ Это **программный подход**: всё в коде → версируется → CI/CD

---

### ✅ Преимущества Airflow:
| Плюс | Объяснение |
|------|------------|
| ✅ **Code as Workflow** | DAG’и в Git → контроль версий |
| ✅ **Огромное комьюнити** | 1000+ операторов (AWS, GCP, Spark) |
| ✅ **Идеален для data pipeline** | ETL, ML training, reporting |
| ✅ **Масштабируется** | KubernetesExecutor, Celery |
| ✅ **Интеграция с DevOps** | CI/CD, тестирование, monitoring |

---

### ❌ Недостатки:
| Минус | Объяснение |
|-------|------------|
| ❌ Не подходит для человеко-задач | Нет форм, нет пользовательских задач |
| ❌ Сложно для бизнес-аналитиков | Только инженеры могут писать DAG’и |
| ❌ Нет BPMN | Все в коде, не визуально |
| ❌ Нет встроенных форм | Нужно делать отдельно |

---

## ✅ 2. Camunda — для бизнес-процессов

> **Camunda** — это **движок BPM (Business Process Management)**, основанный на стандарте **[[BPMN]] 2.0**  
> Используется для:
> - Онбординга сотрудников
> - Подтверждения заказов
> - Выдачи кредита
> - Утверждения расходов

---

### 🔄 Архитектура:

```mermaid
graph LR
    A[Бизнес-аналитик] --> B[Model in Camunda Modeler]
    B --> C[BPMN Diagram]
    C --> D[Camunda Engine]
    D --> E[User Task: Approve Order]
    E --> F[Service Task: Send Email]
    F --> G[End Event]

    style D fill:#e9ecef,stroke:#6c757d
    style E fill:#ffc107,stroke:#856404
```

---

### ✅ Преимущества Camunda:
| Плюс | Объяснение |
|------|------------|
| ✅ **Визуальное моделирование** | BPMN — понятно аналитикам |
| ✅ **Человеческие задачи (User Tasks)** | Кто должен что сделать? |
| ✅ **Формы** | Встроенные формы для согласования |
| ✅ **DMN (Decision Model Notation)** | Автоматические решения (например, «выдать кредит?») |
| ✅ **CMMN** | Случайные процессы (Case Management) |
| ✅ **Аудит и история** | Кто, когда и что делал? |
| ✅ **Zero-code workflow design** | Без написания кода |

---

### ❌ Недостатки:
| Минус | Объяснение |
|-------|------------|
| ❌ Сложнее масштабировать | Требует Java EE / Spring Boot |
| ❌ Меньше интеграций out-of-the-box | Нужно писать Java-делегаты |
| ❌ Медленнее для простых скриптов | Если просто `bash` → избыточно |
| ❌ Лицензия | Community vs Enterprise |

---

## 🆚 Когда что использовать?

| Сценарий | Рекомендация |
|----------|--------------|
| ✅ ETL, Data Warehouse | ➤ **Apache Airflow** |
| ✅ ML Training Pipeline | ➤ **Airflow** |
| ✅ Онлайн-онбординг клиента | ➤ **Camunda** |
| ✅ Утверждение заявки | ➤ **Camunda** |
| ✅ Нужны формы | ➤ **Camunda** |
| ✅ Вы хотите видеть процесс как карту | ➤ **Camunda (BPMN)** |
| ✅ Вы — Data Engineer | ➤ **Airflow** |
| ✅ Вы — бизнес-аналитик | ➤ **Camunda** |

---

## ✅ Финальный вывод

| | **Apache Airflow** | **Camunda** |
|--|-------------------|------------|
| **Цель** | Автоматизация **технических пайплайнов** | Управление **бизнес-процессами** |
| **Кто использует** | Инженеры, DevOps | Бизнес-аналитики, менеджеры |
| **Язык** | Python | BPMN, DMN |
| **Задачи** | Сервисы, API, бэкенды | Люди, формы, утверждения |
| **UI** | Для DevOps | Для бизнеса |
| **Где живёт процесс** | В коде (`DAG`) | В графическом редакторе |
| **Лучше для** | Данных, автоматизации | Процессов, compliance |

> 💬 _“Airflow asks: ‘What needs to run?’_  
> _Camunda asks: ‘Who needs to approve it?’”_

---

## 📚 Где учиться дальше?

- [Apache Airflow Docs](https://airflow.apache.org/)
- [Camunda Docs](https://docs.camunda.org/)
- YouTube: *“Airflow vs Camunda”* — TechWorld with Nana
- Book: *“Orchestrating Success with Airflow”*

---

✅ **Airflow — ваш выбор, если вы строите data platform**  
✅ **Camunda — если вы оптимизируете бизнес-процессы**

📌 Сохраните эту таблицу — она поможет выбрать правильный инструмент.

> 🔁 **Data + People = полная картина**