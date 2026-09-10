---
aliases:
  - ISO 5962
  - ISO 5962 SPDX
  - ISO/IEC 5962:2021
  - Software Package Data Exchange
  - SPDX
---

**SPDX** (Software Package Data Exchange) — это открытый стандарт для документирования и обмена данными о лицензиях, авторских правах и происхождении программного обеспечения. Развивается под эгидой Linux Foundation и признан как международный стандарт (ISO/IEC 5962:2021).

## Что он решает

В первую очередь SPDX нужен, чтобы снять неопределённость в вопросах:

* **Какие лицензии используются** в проекте и его зависимостях.
* **Как эти лицензии сочетаются** между собой (например, «MIT + GPL‑3.0 with classpath exception»).
* **Откуда взялся каждый компонент** (пакет, файл, фрагмент кода).
* **Есть ли у компонента известные уязвимости** и какие условия использования нужно соблюдать.

Это критично для compliance, аудита и безопасности цепочки поставок ПО (supply chain security).

---

## Ключевые части стандарта

1. **Формат документа SPDX.** Позволяет описать весь проект и его зависимости в одном файле. Поддерживаются форматы: SPDX‑Tag, JSON, YAML, RDF/XML, Turtle.
2. **SPDX License List.** Это канонический список лицензий с короткими уникальными идентификаторами (SPDX IDs): `MIT`, `Apache-2.0`, `GPL-3.0`, `AGPL-3.0-only` и т. д. Вместо того чтобы копировать полный текст лицензии, в метаданных указывают её ID.
3. **Правила идентификации лицензий.** Включают рекомендации, как сопоставлять текст лицензии с ID, а также выражения для сложных случаев (например, `MIT OR Apache-2.0`).
4. **Модель данных.** Описывает сущности: пакеты (packages), файлы (files), отношения (отношения зависимости, производные работы и т. п.), лицензии, уведомления (notices).

---

## Как это выглядит на практике

### SPDX ID в коде

Самый простой и распространённый способ — добавить в начало файла комментарий:

```java
// SPDX-License-Identifier: MIT
```

или для нескольких лицензий:

```java
// SPDX-License-Identifier: GPL-3.0-or-later AND MIT
```

Это сразу даёт юридическую ясность: какой лицензией покрывается именно этот файл.

### SPDX‑документ (SBOM)

Для всего проекта создают файл (например, `spdx.json` или `spdx.rdf`), где перечисляют:

* пакеты и их версии;
* лицензии каждого пакета;
* отношения между компонентами;
* ссылки на исходники, контрольные суммы и т. п.

Такой документ — это по сути **SBOM** (Software Bill of Materials) в формате SPDX.

---

## Связь с твоими Java‑задачами

Учитывая твои предыдущие вопросы (детектирование лицензий, работа с ZIP, генерация SBOM, валидация ID):

* **Валидация SPDX ID.** Ты можешь брать строку (например, результат детектора) и проверять через `SpdxId.isValid()` из `java-spdx-library`. Это гарантирует, что ID действительно существует в списке SPDX.
* **Нормализация.** `SpdxId.toCanonical()` приводит ID к единому виду (убирает лишние пробелы, нормализует регистр).
* **Получение эталонных текстов.** Через `LicenseListVersion.getLatest()` ты получаешь все известные лицензии и их тексты — это база для твоего детектора (сравнение по схожести, расстоянию Левенштейна и т. п.).
* **Генерация SPDX‑документа.** После того как ты определил лицензии для компонентов (в том числе из ZIP‑архивов), можно собрать полноценный SPDX‑SBOM (JSON/YAML) с помощью `spdx-jackson-store` или `spdx-v3jsonld-store`.
* **Интеграция с инструментами.** FOSSology, ScanCode и другие сканеры умеют выдавать результаты именно в терминах SPDX ID, что удобно для дальнейшей обработки в Java.

---

## Пример: как это связано с библиотеками, которые ты рассматривал

| Библиотека | Роль в работе со SPDX |
| --- | --- |
| `java-spdx-library` | Доступ к списку лицензий, валидация ID, утилиты для сравнения. |
| `spdx-java-core` | Модели данных (классы для пакетов, файлов, лицензий, отношений). |
| `spdx-jackson-store` | Сериализация/десериализация SPDX‑документов в JSON. |
| `spdx-v3jsonld-store` | Работа с SPDX v3 в формате JSON‑LD. |

## Лицензии

### 📜 SPDX ID в контексте пакетов и лицензий

#### 📌 Краткий ответ

> **SPDX ID** (Software Package Data Exchange Identifier) — это **стандартизированный короткий идентификатор лицензии**, определённый спецификацией **SPDX** (ISO/IEC 5962).
> **Цель:** унифицировать названия лицензий в экосистеме ПО, чтобы инструменты могли **автоматически распознавать** и **сравнивать** лицензии.

---

### 🧩 Что такое SPDX?

| Характеристика | Описание |
|----------------|----------|
| **Полное название** | Software Package Data Exchange |
| **Стандарт** | ISO/IEC 5962:2021 |
| **Организация** | Linux Foundation |
| **Версия списка** | SPDX License List v3.22+ (обновляется ежеквартально) |
| **Количество лицензий** | 600+ в официальном списке |
| **Формат** | JSON, XML, YAML, RDF, Tag:Value |

---

### 🎯 Зачем нужен SPDX ID?

#### Проблема без SPDX

```
┌─────────────────────────────────────────────────────────────┐
│  ❌ Одна лицензия — много названий                          │
│                                                             │
│  "Apache 2.0"                                               │
│  "Apache License 2.0"                                       │
│  "Apache-2.0"                                               │
│  "Apache License, Version 2.0"                              │
│  "ASL 2.0"                                                  │
│  "Apache Software License 2.0"                              │
│                                                             │
│  → Инструменты не могут понять, что это ОДНА лицензия      │
└─────────────────────────────────────────────────────────────┘
```

#### Решение с SPDX

```
┌─────────────────────────────────────────────────────────────┐
│  ✅ SPDX ID — единый идентификатор                          │
│                                                             │
│  Все варианты выше → SPDX ID: "Apache-2.0"                  │
│                                                             │
│  → Инструменты автоматически распознают лицензию            │
│  → Можно проверить совместимость                            │
│  → Можно генерировать отчёты                                │
└─────────────────────────────────────────────────────────────┘
```

---

### 📋 Примеры популярных SPDX ID

| SPDX ID | Полное название | Тип |
|---------|-----------------|-----|
| **`MIT`** | MIT License | Permissive |
| **`Apache-2.0`** | Apache License 2.0 | Permissive |
| **`GPL-3.0-only`** | GNU General Public License v3.0 only | Copyleft |
| **`GPL-3.0-or-later`** | GNU GPL v3.0 or later | Copyleft |
| **`LGPL-2.1-only`** | GNU Lesser GPL v2.1 only | Weak Copyleft |
| **`BSD-2-Clause`** | BSD 2-Clause "Simplified" | Permissive |
| **`BSD-3-Clause`** | BSD 3-Clause "New" or "Revised" | Permissive |
| **`0BSD`** | BSD Zero Clause License | Permissive |
| **`ISC`** | ISC License | Permissive |
| **`MPL-2.0`** | Mozilla Public License 2.0 | Weak Copyleft |
| **`AGPL-3.0-only`** | GNU Affero GPL v3.0 | Strong Copyleft |
| **`Unlicense`** | The Unlicense | Public Domain |
| **`CC-BY-4.0`** | Creative Commons Attribution 4.0 | Creative Commons |
| **`EPL-2.0`** | Eclipse Public License 2.0 | Weak Copyleft |

---

### 🔗 Где используется SPDX ID?

#### 1. **В файлах манифестов пакетов**

##### npm (`package.json`)

```json
{
  "name": "my-package",
  "version": "1.0.0",
  "license": "MIT"
}
```

##### Maven (`pom.xml`)

```xml
<licenses>
    <license>
        <name>Apache License, Version 2.0</name>
        <url>https://www.apache.org/licenses/LICENSE-2.0</url>
        <distribution>repo</distribution>
    </license>
</licenses>
```

##### Cargo (`Cargo.toml`)

```toml
[package]
name = "my-crate"
version = "1.0.0"
license = "MIT OR Apache-2.0"  # ← SPDX expression
```

##### Go (`go.mod`)

```go
// LICENSE файл + аннотации
// SPDX-License-Identifier: MIT
```

##### Python (`pyproject.toml`)

```toml
[project]
name = "my-package"
license = {text = "MIT"}
```

---

#### 2. **В заголовках исходных файлов**

```java
/*
 * SPDX-License-Identifier: Apache-2.0
 * Copyright 2024 Example Corp
 */
public class MyClass { }
```

```python
# SPDX-License-Identifier: MIT
# Copyright (c) 2024 Example Corp

def main():
    pass
```

---

#### 3. **В SBOM (Software Bill of Materials)**

```json
{
  "spdxVersion": "SPDX-2.3",
  "name": "my-application-sbom",
  "packages": [
    {
      "name": "spring-boot",
      "versionInfo": "3.2.0",
      "licenseConcluded": "Apache-2.0",
      "licenseDeclared": "Apache-2.0",
      "copyrightText": "Copyright 2024 VMware Inc."
    },
    {
      "name": "jackson-databind",
      "versionInfo": "2.15.2",
      "licenseConcluded": "Apache-2.0",
      "licenseDeclared": "Apache-2.0"
    }
  ]
}
```

---

### 🧮 SPDX License Expressions

> SPDX поддерживает **составные выражения** для двойного лицензирования и исключений.

#### Операторы

| Оператор | Значение | Пример |
|----------|----------|--------|
| **`AND`** | Обе лицензии применяются | `Apache-2.0 AND MIT` |
| **`OR`** | Выбор из лицензий | `MIT OR Apache-2.0` |
| **`WITH`** | Лицензия с исключением | `GPL-2.0-only WITH Classpath-exception-2.0` |

#### Примеры выражений

```
# Двойное лицензирование (выбор пользователя)
MIT OR Apache-2.0

# Обе лицензии одновременно
Apache-2.0 AND BSD-3-Clause

# GPL с исключением для classpath
GPL-2.0-only WITH Classpath-exception-2.0

# Сложное выражение
(MIT OR Apache-2.0) AND BSD-3-Clause
```

#### Где это используется

| Экосистема | Пример |
|------------|--------|
| **Rust (Cargo)** | `license = "MIT OR Apache-2.0"` |
| **Linux Kernel** | `GPL-2.0-only WITH Linux-syscall-note` |
| **OpenJDK** | `GPL-2.0-only WITH Classpath-exception-2.0` |
| **Eclipse** | `EPL-2.0 OR GPL-2.0-only WITH Classpath-exception-2.0` |

---

### 📊 Классификация лицензий по SPDX

```mermaid
graph TB
    A["SPDX Licenses"] --> B["Permissive<br/>Разрешительные"]
    A --> C["Copyleft<br/>Свободные"]
    A --> D["Public Domain<br/>Общественное достояние"]
    A --> E["Proprietary<br/>Проприетарные"]
    
    B --> B1["MIT"]
    B --> B2["Apache-2.0"]
    B --> B3["BSD-2-Clause"]
    B --> B4["BSD-3-Clause"]
    B --> B5["ISC"]
    B --> B6["0BSD"]
    
    C --> C1["Strong Copyleft"]
    C --> C2["Weak Copyleft"]
    
    C1 --> C1a["GPL-3.0-only"]
    C1 --> C1b["AGPL-3.0-only"]
    
    C2 --> C2a["LGPL-2.1-only"]
    C2 --> C2b["MPL-2.0"]
    C2 --> C2c["EPL-2.0"]
    
    D --> D1["Unlicense"]
    D --> D2["CC0-1.0"]
    
    E --> E1["LicenseRef-*"]
    
    style A fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    style B fill:#c8e6c9,stroke:#388e3c
    style C fill:#fff9c4,stroke:#fbc02d
    style D fill:#e1bee7,stroke:#7b1fa2
    style E fill:#ffcdd2,stroke:#c62828
```

---

### 🏢 LicenseRef: кастомные лицензии

> Если лицензия **не входит** в официальный SPDX список, используется префикс **`LicenseRef-`**.

```
LicenseRef-SberTech-property
LicenseRef-Company-Internal
LicenseRef-Commercial-EULA
```

#### Пример в вашем контексте

```json
{
  "spdxId": "LicenseRef-SberTech-property",
  "nameLikeIn": [
    "SberTech property",
    "Sbertech property",
    "sbt-tasktracker-licence"
  ],
  "type": "ALLOWED"
}
```

> ⚠️ **`LicenseRef-*`** — это **не стандартная** лицензия SPDX, а **внутренняя/кастомная**.

---

### 🛠️ Инструменты, работающие с SPDX

| Инструмент | Назначение | Команда/Формат |
|------------|------------|----------------|
| **SPDX Tools** | Валидация и конвертация | `spdx-tools validate file.spdx` |
| **Syft** | Генерация SBOM | `syft packages dir:/app -o spdx-json` |
| **Trivy** | Сканирование лицензий | `trivy fs --scanners license .` |
| **FOSSA** | Compliance анализ | SaaS |
| **Black Duck** | Enterprise compliance | SaaS |
| **license-maven-plugin** | Maven отчёт | `mvn license:check` |
| **licensee** | Определение лицензии | `licensee detect ./repo` |
| **npm license-checker** | Проверка npm-зависимостей | `npx license-checker` |

---

### 🧪 Пример: генерация SBOM с SPDX

#### Syft

```bash
# Генерация SBOM в формате SPDX
syft packages dir:/app -o spdx-json > sbom.spdx.json

# Результат:
{
  "spdxVersion": "SPDX-2.3",
  "packages": [
    {
      "name": "spring-core",
      "licenseConcluded": "Apache-2.0"
    }
  ]
}
```

#### Trivy

```bash
# Сканирование лицензий
trivy fs --scanners license --severity HIGH,CRITICAL .

# Результат:
# LICENSE: Apache-2.0 (ALLOWED)
# LICENSE: GPL-3.0-only (BLOCKED)
```

---

### 📋 Совместимость лицензий

```mermaid
graph LR
    A["MIT"] -->|Совместим с| B["Apache-2.0"]
    A -->|Совместим с| C["GPL-3.0"]
    B -->|Совместим с| C
    
    D["GPL-3.0"] -->|❌ Не совместим| E["Proprietary"]
    F["AGPL-3.0"] -->|❌ Требует| G["Open Source<br/>при сетевом использовании"]
    
    style A fill:#c8e6c9,stroke:#388e3c
    style B fill:#c8e6c9,stroke:#388e3c
    style C fill:#fff9c4,stroke:#fbc02d
    style D fill:#fff9c4,stroke:#fbc02d
    style E fill:#ffcdd2,stroke:#c62828
    style F fill:#ffcdd2,stroke:#c62828
    style G fill:#ffcdd2,stroke:#c62828
```

| Лицензия | Можно использовать в коммерческом ПО? | Обязательства |
|----------|---------------------------------------|---------------|
| **MIT** | ✅ Да | Сохранить copyright notice |
| **Apache-2.0** | ✅ Да | Сохранить notice + указать изменения |
| **BSD-3-Clause** | ✅ Да | Сохранить copyright notice |
| **GPL-3.0** | ⚠️ Только если открыть код | Производные работы → GPL |
| **AGPL-3.0** | ⚠️ Только если открыть код | Сетевое использование → open source |
| **Proprietary** | ⚠️ По условиям EULA | Зависит от лицензии |

---

### 🔍 Как определить SPDX ID пакета?

#### 1. **Через npm**

```bash
# Посмотреть лицензию пакета
npm view express license
# → MIT

# Все лицензии зависимостей
npx license-checker --summary
```

#### 2. **Через Maven**

```bash
# Лицензии зависимостей
mvn license:third-party-full-report

# В pom.xml
<licenses>
    <license>
        <name>Apache-2.0</name>
    </license>
</licenses>
```

#### 3. **Через Cargo**

```bash
cargo license
# → MIT OR Apache-2.0
```

#### 4. **Через Syft (SBOM)**

```bash
syft packages dir:/app -o spdx-json | jq '.packages[].licenseConcluded'
```

---

### 📌 Памятка

```
┌─────────────────────────────────────────────────────────────┐
│  SPDX ID                                                    │
│                                                             │
│  📌 Что: стандартизированный идентификатор лицензии         │
│  📌 Стандарт: ISO/IEC 5962 (Linux Foundation)               │
│  📌 Примеры: MIT, Apache-2.0, GPL-3.0-only                 │
│  📌 Кастомные: LicenseRef-<name>                            │
│                                                             │
│  🎯 Зачем:                                                  │
│  • Унификация названий лицензий                             │
│  • Автоматическое распознавание инструментами               │
│  • Проверка совместимости                                   │
│  • Генерация SBOM и compliance отчётов                      │
│                                                             │
│  📦 Где используется:                                       │
│  • package.json, pom.xml, Cargo.toml, go.mod               │
│  • Заголовки файлов (SPDX-License-Identifier)               │
│  • SBOM (SPDX-2.3 формат)                                   │
│  • CI/CD проверки лицензий                                  │
│                                                             │
│  🔧 Инструменты: Syft, Trivy, FOSSA, Black Duck            │
└─────────────────────────────────────────────────────────────┘
```

---

### 🚀 Итог

| Вопрос | Ответ |
|--------|-------|
| **Что такое SPDX?** | Стандарт обмена данными о пакетах ПО (ISO/IEC 5962) |
| **Что такое spdxId?** | Короткий идентификатор лицензии (например, `Apache-2.0`) |
| **Зачем нужен?** | Унификация названий лицензий для автоматизации |
| **Где используется?** | Манифесты пакетов, SBOM, заголовки файлов |
| **Что если лицензия кастомная?** | Префикс `LicenseRef-` (например, `LicenseRef-SberTech-property`) |
| **Какие инструменты?** | Syft, Trivy, FOSSA, Black Duck, license-maven-plugin |
| **Что такое SPDX expression?** | Составные лицензии: `MIT OR Apache-2.0`, `GPL-2.0 WITH exception` |

> 💡 **Совет:** В вашем `LicenseResolver` поле `spdxId` — это **нормализованный идентификатор**, к которому приводятся все варианты названий из `nameLikeIn`. Это позволяет сравнивать лицензии из разных источников (npm, Maven, PyPI) по единому ключу.
