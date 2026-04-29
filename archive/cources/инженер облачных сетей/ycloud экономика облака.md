---
aliases:
  - ycloud
---
# Pay as you go – PAYG
«плати, пока пользуешься». Как в каршеринге: пока едешь на арендованной машине — платишь, поездка закончилась — закончили платить.

>👉 Затраты в Yandex Cloud зависят от количества потреблённых ресурсов и времени их использования.

# SKU – stock keeping unit

```mermaid
---
title: SKU – stock keeping unit
---
flowchart LR
 subgraph ro["ресурсы в облаке"]
        v1["Виртуальная машина 1"]
        v2["Виртуальная машина 2"]
  end
 subgraph sku["ресурсы в билинге (sku)"]
        r1["1: Intel Casscade Lake. 100% vCPU"]
        r2["2: Intel Casscade Lake. RAM"]
        r3["3: Intel Casscade Lake. 100% vCPU"]
        r4["4: Intel Casscade Lake. RAM"]
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
# биллинт yclod
* ycloud могут только резиденты рф
* есть калькулятор для расчета стоимости оп SKU
Вот три способа контролировать затраты в Yandex Cloud:
1. Детализация затрат в консоли.
2. Использование [DataLens](https://datalens.yandex.ru/).
	1. DataLens — это знакомый вам сервис визуализации данных. Для биллинга он удобен тем, что позволяет следить за каждым ресурсом. Например, за конкретной виртуальной машиной.
3. Отгрузка детализации в формате CSV.
## оптимизация затрат

```mermaid
---
title: Как экономить на виртуальных машинах в зависимости от задач?
---
flowchart TB
    A["Небольшие вычисления? (чат-бот, навык Алисы)"] -- да --> B["Cloud Function"]
    A -- нет --> C["Кратковременные задачи? (рендеринг, тесты)"]
    C -- да --> D["Прерываемые ВМ"]
    C -- нет --> E["Нагрузка волатильная?"]
    E -- да --> F["Instance.Groups"]
    E -- нет --> G["Нужна высокая производительность?"]
    G -- да --> H["Попробуйте оптимизировать расходы на другие сервисы"]
    G -- нет --> I["Часть ядра"]

     A:::Sky
     C:::Sky
     E:::Sky
     G:::Sky
    classDef Sky stroke-width:1px, stroke-dasharray:none, stroke:#374D7C, fill:#E2EBFF, color:#374D7C
```
