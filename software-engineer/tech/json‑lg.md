---
aliases:
  - JavaScript Object Notation for Linked Data
  - JSON‑LD
---
## JavaScript Object Notation for Linked Data

**JSON‑LD** (JavaScript Object Notation for Linked Data) — это формат на базе JSON, который добавляет в данные **контекст и семантику**, чтобы машины могли понимать смысл полей, а не просто их структуру.

### В чём главная фишка

Обычный JSON:

```json
{
  "name": "Alice",
  "job": "engineer"
}
```

Машине непонятно, что значит `name` и `job` — это просто ключи.

JSON‑LD добавляет `@context`, который связывает ключи с понятиями из онтологий (например, schema.org, SPDX):

```json
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Alice",
  "jobTitle": "engineer"
}
```

Теперь ясно: `name` — это имя человека, `jobTitle` — должность. Данные становятся **машиночитаемыми** и пригодными для обмена между разными системами.

---

### Ключевые элементы

- **`@context`**: URL словаря/онтологии, где описано, что означают поля.
- **`@type`**: тип сущности по этому словарю (например, `Package`, `License`, `Person`).
- **Связи через URI**: вместо простых строк можно использовать URL как идентификаторы, чтобы ссылаться на другие объекты.

---

### Зачем это в SPDX

В [[ISO 5962 SPDX|SPDX]] v3 используют JSON‑LD, потому что:

- **Можно ссылаться на лицензии, пакеты, файлы по URI**, а не дублировать их описания.
- **Легко расширять**: если словарь обновится, новые поля будут понятны другим инструментам.
- **Подходит для графов знаний**: удобно строить граф зависимостей компонентов и лицензий — это как раз нужно для SBOM.

Пример SPDX в JSON‑LD (упрощённо):

```json
{
  "@context": ["https://spdx.dev/spdx-jsonld-v3-schema"],
  "@type": "spdx:Document",
  "spdx:spdxIdentifier": "SPDXRef-DOCUMENT",
  "spdx:hasPackage": [
    {
      "@type": "spdx:Package",
      "spdx:name": "my-lib",
      "spdx:licenseConcluded": "Apache-2.0"
    }
  ]
}
```

---
