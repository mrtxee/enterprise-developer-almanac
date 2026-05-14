---
aliases:
  - XMLHttpRequest
  - XHR
---
## 🌐 XMLHttpRequest (XHR) — Полное объяснение

### 📌 Краткий ответ

> **XMLHttpRequest (XHR)** — это встроенный в браузер API для выполнения **HTTP-запросов из JavaScript**.  
> **Использовался до появления Fetch API (2015), но всё ещё поддерживается.**

---

## 🧩 Что такое XHR?

| Характеристика | Описание |
|----------------|----------|
| **Появился** | 2002 (Internet Explorer 5) |
| **Стандартизирован** | W3C (2006) |
| **Название** | Историческое (поддерживал только XML) |
| **Поддержка** | ✅ Все браузеры (включая IE) |
| **Современная альтернатива** | Fetch API, Axios |

---

## 📊 Как работает XHR

```
┌─────────────────────────────────────────────────────────────┐
│  JavaScript                                                 │
│  const xhr = new XMLHttpRequest();                          │
└────────────────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│  XMLHttpRequest Object                                      │
│  • open() — настройка запроса                               │
│  • send() — отправка                                        │
│  • onreadystatechange — обработчик событий                  │
└────────────────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│  Browser Network Stack                                      │
│  • DNS lookup                                               │
│  • TCP/TLS handshake                                        │
│  • HTTP request/response                                    │
└────────────────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│  Callback (onreadystatechange / onload)                     │
│  • xhr.response                                             │
│  • xhr.status                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Базовое использование

### 1. **GET запрос**

```javascript
const xhr = new XMLHttpRequest();

// 1. Инициализация
xhr.open('GET', 'https://api.example.com/users', true);

// 2. Обработчик изменения состояния
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {  // DONE
        if (xhr.status === 200) {
            console.log('Success:', xhr.responseText);
        } else {
            console.error('Error:', xhr.status);
        }
    }
};

// 3. Отправка
xhr.send();
```

### 2. **GET с JSON**

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/users', true);

// Указываем, что ожидаем JSON
xhr.responseType = 'json';

xhr.onload = function() {
    if (xhr.status === 200) {
        console.log('Users:', xhr.response); // Уже распарсенный JSON
    }
};

xhr.onerror = function() {
    console.error('Network error');
};

xhr.send();
```

### 3. **POST запрос**

```javascript
const xhr = new XMLHttpRequest();
xhr.open('POST', 'https://api.example.com/users', true);

// Заголовки
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.setRequestHeader('Authorization', 'Bearer token123');

xhr.onload = function() {
    if (xhr.status === 201) {
        console.log('Created:', xhr.response);
    }
};

xhr.onerror = function() {
    console.error('Error');
};

// Отправка данных
xhr.send(JSON.stringify({ name: 'John', email: 'john@example.com' }));
```

### 4. **Прогресс загрузки**

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/large-file', true);

// Прогресс загрузки
xhr.onprogress = function(event) {
    if (event.lengthComputable) {
        const percent = (event.loaded / event.total) * 100;
        console.log(`Загрузка: ${percent.toFixed(2)}%`);
    }
};

// Прогресс отправки (для POST/PUT)
xhr.upload.onprogress = function(event) {
    console.log(`Отправлено: ${event.loaded} байт`);
};

xhr.onload = function() {
    console.log('Загрузка завершена');
};

xhr.send();
```

---

## 📋 Ready States (readyState)

| Значение | Константа | Описание |
|----------|-----------|----------|
| **0** | `UNSENT` | Объект создан, `open()` не вызван |
| **1** | `OPENED` | `open()` вызван |
| **2** | `HEADERS_RECEIVED` | `send()` вызван, заголовки получены |
| **3** | `LOADING` | Загрузка тела ответа (download) |
| **4** | `DONE` | Запрос завершён |

```javascript
xhr.onreadystatechange = function() {
    console.log(`ReadyState: ${xhr.readyState}`);
    
    switch(xhr.readyState) {
        case 0: console.log('UNSENT'); break;
        case 1: console.log('OPENED'); break;
        case 2: console.log('HEADERS_RECEIVED'); break;
        case 3: console.log('LOADING'); break;
        case 4: console.log('DONE'); break;
    }
};
```

---

## 🎯 События XHR

| Событие | Когда срабатывает |
|---------|-------------------|
| `onloadstart` | Запрос начался |
| `onprogress` | Получение данных (загрузка) |
| `onload` | Успешное завершение |
| `onerror` | Ошибка сети |
| `onabort` | Запрос отменён |
| `ontimeout` | Превышено время ожидания |
| `onloadend` | Запрос завершён (успех или ошибка) |
| `onreadystatechange` | Изменение readyState (устаревает) |

### Современный подход (события вместо onreadystatechange):

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data', true);

xhr.onload = () => {
    if (xhr.status >= 200 && xhr.status < 300) {
        console.log('Success:', xhr.response);
    } else {
        console.error('HTTP Error:', xhr.status);
    }
};

xhr.onerror = () => {
    console.error('Network Error');
};

xhr.ontimeout = () => {
    console.error('Timeout');
};

xhr.onabort = () => {
    console.log('Aborted');
};

xhr.timeout = 5000; // 5 секунд
xhr.send();
```

---

## ⚙️ Продвинутые возможности

### 1. **Таймаут**

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data', true);
xhr.timeout = 5000; // 5 секунд

xhr.ontimeout = function() {
    console.error('Request timed out');
};

xhr.send();
```

### 2. **Отмена запроса**

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data', true);

xhr.onload = function() {
    console.log('Success');
};

// Отмена через 2 секунды
setTimeout(() => {
    xhr.abort();
    console.log('Request aborted');
}, 2000);

xhr.send();
```

### 3. **Заголовки**

```javascript
const xhr = new XMLHttpRequest();
xhr.open('POST', 'https://api.example.com/data', true);

// Установка заголовков
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.setRequestHeader('Authorization', 'Bearer token123');
xhr.setRequestHeader('X-Custom-Header', 'value');

// Получение заголовков ответа
xhr.onload = function() {
    const contentType = xhr.getResponseHeader('Content-Type');
    const allHeaders = xhr.getAllResponseHeaders();
    console.log('Content-Type:', contentType);
};

xhr.send();
```

### 4. **Cookie и Credentials**

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data', true);

// Отправлять cookie с запросом
xhr.withCredentials = true;

xhr.send();
```

### 5. **Синхронный запрос (⚠️ Не рекомендуется)**

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data', false); // false = синхронно

xhr.send(); // ❌ Блокирует UI поток

console.log(xhr.responseText); // Доступно сразу
```

> ⚠️ **Запрещено в Web Workers и может блокировать браузер**

---

## 🔄 XHR vs Fetch API

| Критерий | **XHR** | **Fetch** |
|----------|---------|-----------|
| **Синтаксис** | Callback-based | Promise-based |
| **Читаемость** | ⚠️ Callback hell | ✅ Чистый код |
| **JSON парсинг** | ❌ Вручную (`JSON.parse`) | ✅ `response.json()` |
| **Прогресс** | ✅ Встроенный | ❌ Через ReadableStream |
| **Отмена** | ✅ `xhr.abort()` | ✅ `AbortController` |
| **Таймаут** | ✅ `xhr.timeout` | ❌ Через AbortController |
| **IE поддержка** | ✅ IE 5+ | ❌ IE 11+ (частично) |
| **CORS** | ✅ Автоматически | ✅ Автоматически |
| **Cookies** | ✅ `withCredentials` | ✅ `credentials` |
| **Размер** | Встроен в браузер | Встроен (кроме IE) |

---

## 🧪 Пример: XHR vs Fetch

### XHR

```javascript
function fetchDataXHR() {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open('GET', 'https://api.example.com/data', true);
        
        xhr.onload = () => {
            if (xhr.status >= 200 && xhr.status < 300) {
                resolve(JSON.parse(xhr.responseText));
            } else {
                reject(new Error(`HTTP ${xhr.status}`));
            }
        };
        
        xhr.onerror = () => reject(new Error('Network error'));
        xhr.send();
    });
}
```

### Fetch

```javascript
async function fetchDataFetch() {
    const response = await fetch('https://api.example.com/data');
    
    if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
    }
    
    return await response.json();
}
```

---

## ⚠️ CORS и XHR

### CORS применяется к XHR

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data', true);

xhr.onload = function() {
    // ✅ Если сервер вернул правильные заголовки
    console.log(xhr.response);
};

xhr.onerror = function() {
    // ❌ Если CORS не настроен
    console.error('CORS Error');
};

xhr.send();
```

### Необходимые заголовки от сервера

```
Access-Control-Allow-Origin: https://your-domain.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Content-Type, Authorization
```

---

## 📦 Библиотеки на основе XHR

### Axios (самая популярная)

```javascript
// Axios использует XHR в браузере
import axios from 'axios';

// GET
axios.get('https://api.example.com/users')
    .then(response => console.log(response.data))
    .catch(error => console.error(error));

// POST
axios.post('https://api.example.com/users', { name: 'John' })
    .then(response => console.log(response.data));

// Интерцепторы
axios.interceptors.request.use(config => {
    config.headers.Authorization = 'Bearer token';
    return config;
});
```

### Почему Axios популярен?

| Преимущество | Описание |
|--------------|----------|
| **Promise API** | Чище чем raw XHR |
| **Автоматический JSON** | Не нужно парсить вручную |
| **Интерцепторы** | Логирование, auth, ошибки |
| **Отмена запросов** | `CancelToken` / `AbortController` |
| **Прогресс** | Встроенная поддержка |
| **Timeout** | Простая настройка |
| **IE поддержка** | Работает в старых браузерах |

---

## 🛠️ Когда использовать XHR в 2026?

| Сценарий | Рекомендация |
|----------|--------------|
| **Новый проект** | ❌ Fetch или Axios |
| **Поддержка IE** | ✅ XHR или Axios |
| **Прогресс загрузки** | ✅ XHR (нативный) |
| **Legacy код** | ⚠️ Рефакторить на Fetch |
| **React/Vue/Angular** | ❌ Fetch/Axios |
| **Node.js** | ❌ XHR не доступен (используй http/https) |

---

## 📋 Шпаргалка

```javascript
// ✅ Минимальный GET
const xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.onload = () => console.log(xhr.response);
xhr.send();

// ✅ С JSON
xhr.responseType = 'json';
xhr.onload = () => console.log(xhr.response); // Уже объект

// ✅ POST
xhr.open('POST', url);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(JSON.stringify(data));

// ✅ Таймаут
xhr.timeout = 5000;
xhr.ontimeout = () => console.error('Timeout');

// ✅ Отмена
xhr.abort();

// ✅ Прогресс
xhr.onprogress = (e) => console.log(e.loaded / e.total);

// ✅ Cookie
xhr.withCredentials = true;
```

---

## 📌 Памятка

```
┌─────────────────────────────────────────────────────────────┐
│  XMLHttpRequest (XHR)                                       │
│                                                             │
│  ✅ Все браузеры (включая IE)                               │
│  ✅ Прогресс загрузки (нативный)                            │
│  ✅ Таймаут (xhr.timeout)                                   │
│  ✅ Отмена (xhr.abort())                                    │
│                                                             │
│  ❌ Callback-based (хуже чем Promise)                       │
│  ❌ JSON парсинг вручную                                    │
│  ❌ Устаревает в пользу Fetch                               │
│                                                             │
│  📦 Используйте Axios для современного API поверх XHR       │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Итог

| Вопрос | Ответ |
|--------|-------|
| **Что такое XHR?** | API для HTTP-запросов из JavaScript |
| **Актуален ли в 2026?** | ⚠️ Устаревает, но поддерживается |
| **Что использовать вместо?** | Fetch API или Axios |
| **Когда ещё нужен?** | Поддержка IE, прогресс загрузки |
| **CORS применяется?** | ✅ Да, как к любому HTTP-запросу |

---
