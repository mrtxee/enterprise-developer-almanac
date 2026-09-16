---
aliases:
  - billing
  - Cloud Function
  - cost breakdown
  - cost optimization
  - DataLens
  - Fractional vCPU
  - Instance Groups
  - Instance.Groups
  - Pay as you go
  - PAYG
  - Preemptible VM
  - SKU
  - stock keeping unit
  - Yandex Cloud
  - ycloud
  - биллинг
  - виртуальная машина
  - детализация затрат
  - затраты
  - оплата по мере использования
  - оптимизация затрат
  - прерываемые ВМ
---

## Модель оплаты Pay as you go (PAYG)

«Плати, пока пользуешься». Аналогия с каршерингом: пока едешь на арендованной машине — платишь, поездка закончилась — платёж прекращается.

> 👉 Затраты в Yandex Cloud зависят от количества потреблённых ресурсов и времени их использования.

## SKU (stock keeping unit)

Соответствие ресурсов облака и позиций SKU в биллинге:

```mermaid
---
title: SKU – stock keeping unit
---
flowchart LR
  subgraph ro["ресурсы в облаке"]
    v1["Виртуальная машина 1"]
    v2["Виртуальная машина 2"]
  end
  subgraph sku["ресурсы в биллинге (sku)"]
    r1["1: Intel Cascade Lake. 100% vCPU"]
    r2["2: Intel Cascade Lake. RAM"]
    r3["3: Intel Cascade Lake. 100% vCPU"]
    r4["4: Intel Cascade Lake. RAM"]
  end
  v1 --> r1 & r2
  v2 --> r3 & r4
  v1:::Sky
  v2:::Sky
  r1:::Rose
  r2:::Rose
  r3:::Rose
  r4:::Rose
  classDef Sky stroke-width:1px, stroke-dasharray:none, stroke:#374D7C, fill:#E2EBFF, color:#374D7C
  classDef Rose stroke-width:1px, stroke-dasharray:none, stroke:#FF5978, fill:#FFDFE5, color:#8E2236
```

## Биллинг Yandex Cloud

- Yandex Cloud доступен только резидентам РФ.
- Для расчёта стоимости по SKU предусмотрен калькулятор.

Три способа контролировать затраты в Yandex Cloud:

1. Детализация затрат в консоли.
2. Использование [DataLens](https://datalens.yandex.ru/).
  - DataLens — сервис визуализации данных. Для биллинга он позволяет следить за каждым ресурсом, например за конкретной виртуальной машиной.
1. Отгрузка детализации в формате CSV.

### Оптимизация затрат

Схема выбора варианта экономии в зависимости от [[highload|нагрузки]]:

```mermaid
---
title: Выбор варианта экономии на виртуальных машинах
---
flowchart TB
  A["Небольшие вычисления? (чат-бот, навык Алисы)"] -- да --> B["Cloud Function"]
  A -- нет --> C["Кратковременные задачи? (рендеринг, тесты)"]
  C -- да --> D["Прерываемые ВМ"]
  C -- нет --> E["Нагрузка волатильная?"]
  E -- да --> F["Instance.Groups"]
  E -- нет --> G["Нужна высокая производительность?"]
  G -- да --> H["Оптимизация расходов на другие сервисы"]
  G -- нет --> I["Часть ядра"]
  A:::Sky
  C:::Sky
  E:::Sky
  G:::Sky
  classDef Sky stroke-width:1px, stroke-dasharray:none, stroke:#374D7C, fill:#E2EBFF, color:#374D7C
```
