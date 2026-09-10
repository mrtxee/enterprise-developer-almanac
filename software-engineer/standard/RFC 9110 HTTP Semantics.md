---
aliases:
  - header
  - headers
  - http headers
  - HTTP Semantics
  - RFC 9110
  - RFC 9110 HTTP Semantics
---
## 📋 Стандарт HTTP Headers: RFC 9110

### 📌 Краткое описание

**RFC 9110 "HTTP Semantics"** — это **основной стандарт IETF**, определяющий структуру, синтаксис и семантику HTTP-заголовков (вместе с RFC 9112 для HTTP/1.1 и RFC 9113 для HTTP/2).

---

## 🔑 Ключевые моменты RFC 9110

| Аспект | Описание |
|--------|----------|
| **Название** | HTTP Semantics |
| **Номер** | RFC 9110 (июнь 2022) |
| **Статус** | Internet Standards Track |
| **Заменяет** | RFC 7230, RFC 7231, RFC 7232, RFC 7233, RFC 7234, RFC 7235 |
| **Организация** | IETF (Internet Engineering Task Force) |

---

## 📐 Структура HTTP Header (по RFC 9110)

```
Header-Name: Header-Value
```

### Формат

```
field-name ":" OWS field-value OWS

Где:
- field-name  = token (регистронезависимое имя)
- field-value = содержимое (может содержать несколько значений через запятую)
- OWS         = optional whitespace (опциональные пробелы)
```

### Примеры

```http
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
X-Custom-Header: value1, value2, value3
Cache-Control: no-cache, no-store, must-revalidate
```

---

## 📊 Категории заголовков (RFC 9110)

| Категория | Назначение | Примеры |
|-----------|------------|---------|
| **Request Headers** | Информация о запросе | `User-Agent`, `Accept`, `Authorization` |
| **Response Headers** | Информация об ответе | `Server`, `Content-Type`, `Set-Cookie` |
| **Representation Headers** | О представлении данных | `Content-Encoding`, `Content-Language` |
| **Content Headers** | О теле сообщения | `Content-Length`, `Content-Type` |

---

## 🔗 Связанные RFC документы

| RFC | Название | Описание |
|-----|----------|----------|
| **RFC 9110** | HTTP Semantics | Семантика заголовков, методов, статусов |
| **RFC 9112** | HTTP/1.1 | Синтаксис HTTP/1.1 |
| **RFC 9113** | HTTP/2 | Бинарный фреймворк HTTP/2 |
| **RFC 9114** | HTTP/3 | HTTP поверх QUIC |
| **RFC 7230-7235** | HTTP/1.1 (старые) | Устарели, заменены RFC 9110-9113 |

---

## 📌 Памятка

```
┌─────────────────────────────────────────────────────────────┐
│  HTTP Headers Standard                                      │
│                                                             │
│  📄 Основной стандарт: RFC 9110 (HTTP Semantics)            │
│  📄 Синтаксис HTTP/1.1: RFC 9112                            │
│  📄 HTTP/2: RFC 9113                                        │
│  📄 HTTP/3: RFC 9114                                        │
│                                                             │
│  ✅ Регистронезависимые имена заголовков                    │
│  ✅ Формат: "Name: Value"                                   │
│  ✅ Множественные значения через запятую                    │
│  ✅ Кастомные заголовки: X-Prefix (устарело) или без префикса│
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Итог

| Вопрос | Ответ |
|--------|-------|
| **Основной стандарт?** | RFC 9110 "HTTP Semantics" |
| **Для HTTP/1.1?** | RFC 9112 |
| **Для HTTP/2?** | RFC 9113 |
| **Организация?** | IETF |
| **Статус?** | Internet Standard |

> 💡 **Примечание:** RFC 9110 — это **объединённый стандарт**, заменивший серию RFC 723x (2014 год). Если видите ссылки на RFC 7230-7235 — они устарели.
