---
aliases:
  - MF
  - Webpack
---
# Module Federation

**Module Federation** — это **революционная функция Webpack 5**, которая превращает **микросервисную архитектуру** в **реальность для фронтенда**. Она позволяет **динамически подключать код из одного приложения в другое на этапе выполнения** — без сборки всего в один бандл.

---

## ✅ Что такое **Module Federation**?

> **Module Federation (MF)** — это **механизм в Webpack 5**, позволяющий **одному приложению динамически загружать и использовать компоненты, экспортированные другим приложением** — **в реальном времени, в браузере**.

> 💡 Это **не сборка в один бандл** — это **реальное "подключение" модулей на лету**, как если бы они были библиотеками, но **созданными независимыми командами**.

### 🎯 Цель:
> **Создавать независимые фронтенд-приложения (микрофронтенды), которые могут совместно использовать компоненты, стили и логику — без монолитной сборки.**

---
## ✅ Как устроен Module Federation? (Архитектура)

### 🧩 Основные понятия:

| Понятие          | Описание                                                                                                   |
| ---------------- | ---------------------------------------------------------------------------------------------------------- |
| **Host**         | Приложение, которое **использует** компоненты из других приложений                                         |
| **Remote**       | Приложение, которое **экспортирует** компоненты для других                                                 |
| **Remote Entry** | Файл (например, `remoteEntry.js`), который содержит **метаданные** о том, какие модули экспортирует Remote |
| **Shared**       | Зависимости, которые **разделяются** между Host и Remote (например, React, Redux)                          |

### 🔗 Принцип работы:

```mermaid
graph LR
    A[Host App] -->|Запрашивает| B[Remote App]
    B -->|Отдаёт| C[remoteEntry.js]
    C -->|Описывает| D[Exposed Module: Button]
    D -->|Загружает| E[Button.js]
    E -->|Рендерит| A
    F[Shared: React] --> A
    F --> B
```

1. **Host** (например, главный портал) хочет использовать кнопку из **Remote** (например, каталога товаров).
2. Host **договаривается** с Remote: *"Я хочу использовать компонент `Button` из вашего приложения."*
3. Host **загружает `remoteEntry.js`** — файл, который **описывает, какие модули доступны**.
4. Host **загружает нужный модуль** (`Button.js`) через динамический импорт.
5. Оба приложения **используют один экземпляр React** (из `shared`), чтобы не было дублирования.

> ✅ **Всё происходит в браузере — без сборки!**


**Module Federation Build** — это результат процесса сборки отдельного микрофронтенда с использованием **Webpack Module Federation Plugin**.
* **Webpack** — **сборщик модулей JavaScript** с открытым исходным кодом. Написан на JavaScript, но может преобразовывать и внешние ресурсы, такие как HTML, CSS и изображения, если включены соответствующие загрузчики.
	* **Официальный сайт**: webpack.js.org.

По сути, это собранный артефакт (набор JS, CSS файлов), который:
- **Предоставляет ("exposes")** какие-то свои модули (компоненты, утилиты, страницы) для использования другими приложениями.
- **Использует ("remotes")** модули из других Federated Builds.

**Webpack Module Federation Host** (или просто **Host**) — это центральное приложение, которое на этапе своей **сборки** интегрирует в себя один или несколько **Module Federation Builds** от других приложений (которые в этом контексте называются **Remote**).

Здесь и заключается суть **Build-Time композиции**: Host-приложение во время своей сборки "видит" Remote-приложения (их Federation Builds) и включает их код в свой финальный бандл.

---


## ✅ Зачем нужен Module Federation?

### ❌ Проблемы без MF:

| Проблема                          | Объяснение                                                                                                           |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Монолитный фронтенд**           | Все компоненты собраны в один бандл → медленная сборка, сложная разработка, все команды работают в одном репозитории |
| **Независимый деплой невозможен** | Чтобы обновить кнопку — нужно пересобрать и перезалить всё приложение                                                |
| **Дублирование зависимостей**     | Два микрофронтенда используют React — и в итоге в бандле два React                                                   |
| **Команды зависят друг от друга** | Команда A не может выпустить новую версию, пока команда B не готова                                                  |

### ✅ Решение: Module Federation

| Преимущество             | Объяснение                                                                                    |
| ------------------------ | --------------------------------------------------------------------------------------------- |
| ✅ **Независимые релизы** | Команда A может обновить свой микрофронтенд — без пересборки приложения B                     |
| ✅ **Общие зависимости**  | React, Lodash, CSS-библиотеки — загружаются **один раз** и **разделяются** между приложениями |
| ✅ **Разные технологии**  | Одно приложение — React, другое — Vue, третье — Angular — все могут работать вместе           |
| ✅ **Масштабируемость**   | Можно добавлять новые микрофронтенды как плагины                                              |
| ✅ **Гибкость**           | Можно "выключить" или "заместить" компонент на лету                                           |

> ✅ **Module Federation — это "Microservices для фронтенда".**

---
## ✅ Пример: Как создать Module Federation?

Представим:  
- **Host App** — главный портал (`main-app`)  
- **Remote App** — каталог товаров (`catalog-app`)

### ✅ Шаг 1: Настройка **Remote App** (`catalog-app`)

**webpack.config.js** (Remote):

```js
const { ModuleFederationPlugin } = require('@webpack-cli/webpack');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'catalog', // Имя Remote — будет использоваться в Host
      filename: 'remoteEntry.js', // Файл, который будет загружать Host
      exposes: {
        './Button': './src/Button', // Экспортируем компонент Button
        './ProductList': './src/ProductList',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

> ✅ `exposes` — что **можно использовать извне**.  
> ✅ `shared` — зависимости, которые **разделяются** (не дублируются).

### ✅ Шаг 2: Настройка **Host App** (`main-app`)

**webpack.config.js** (Host):

```js
const { ModuleFederationPlugin } = require('@webpack-cli/webpack');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'main',
      remotes: {
        catalog: 'catalog@http://localhost:3001/remoteEntry.js', // URL к remoteEntry.js
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

> ✅ `remotes` — где искать Remote-приложения.  
> ✅ `shared` — **должны совпадать** с Remote!

### ✅ Шаг 3: Использование компонента в Host

**main-app/src/App.jsx**

```jsx
import React, { Suspense, lazy } from 'react';

const Button = lazy(() => import('catalog/Button'));
const ProductList = lazy(() => import('catalog/ProductList'));

function App() {
  return (
    <div>
      <h1>Главный портал</h1>
      <Suspense fallback="Загрузка...">
        <Button /> {/* Используем компонент из Remote */}
        <ProductList />
      </Suspense>
    </div>
  );
}

export default App;
```

> ✅ `import('catalog/Button')` — **не обычный импорт**, а **динамический импорт через Module Federation**.

---

## ✅ Как это работает в браузере?

1. **Host** загружает `index.html`
2. Загружается `main.bundle.js`
3. `main.bundle.js` встречает `import('catalog/Button')`
4. Он **запрашивает `http://localhost:3001/remoteEntry.js`** (через JSONP или Fetch)
5. Получает **метаданные**:  
   ```js
   {
     "./Button": () => import("http://localhost:3001/static/js/Button.js")
   }
   ```
6. Загружает `Button.js` по ссылке
7. **Запускает** компонент — и он **работает** как будто он из того же приложения!

> ✅ **Все компоненты работают в одном React-экземпляре** — потому что `react` объявлен в `shared`.

---

## ✅ Как пользоваться Module Federation? (Практические советы)

### ✅ 1. **Всегда используйте `shared` для React и React DOM**

```js
shared: {
  react: { singleton: true, requiredVersion: '^18.0.0' },
  'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
}
```

> ❌ Без этого — **два React** → **ошибки**, **потеря состояния**, **дублирование хуков**.

### ✅ 2. **Используйте `singleton: true`**
- Это гарантирует, что **только один экземпляр** библиотеки загружается.
- Иначе — если два Remote используют разные версии React — будет конфликт.

### ✅ 3. **Используйте `import()` с динамическим путём**

```js
// ✅ Правильно:
const Button = lazy(() => import('catalog/Button'));

// ❌ НЕ ПРАВИЛЬНО:
import Button from 'catalog/Button'; // Не работает — Webpack не знает remote на этапе сборки
```

### ✅ 4. **Используйте `Suspense`**
Всегда оборачивайте динамические импорты в `<Suspense>` — потому что модуль загружается асинхронно.

```jsx
<Suspense fallback={<div>Загрузка...</div>}>
  <Button />
</Suspense>
```

### ✅ 5. **Настройте CORS**
Remote-приложение должно разрешать запросы с Host-домена:

**Пример для Node.js/Express:**
```js
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', 'http://localhost:3000');
  res.header('Access-Control-Allow-Headers', '*');
  next();
});
```

### ✅ 6. **Используйте `@module-federation/nextjs` для Next.js**
Если используете Next.js — есть официальный плагин:
```bash
npm install @module-federation/nextjs
```

---

## ✅ Когда использовать Module Federation?

| Сценарий | Подходит? | Почему |
|----------|----------|--------|
| ✅ Микрофронтенды в корпоративном портале | ✅ **Да!** | Разные команды, разные релизы |
| ✅ Вы хотите постепенно мигрировать с Angular на React | ✅ **Да!** | Можно запускать старые и новые компоненты вместе |
| ✅ Вы делаете SaaS-платформу с плагинами | ✅ **Да!** | Плагины — это Remote-приложения |
| ✅ Вы делаете MVP с 2 компонентами | ❌ **Нет!** | Это перебор — сложность не оправдана |
| ✅ Вам нужна максимальная скорость загрузки | ⚠️ **Осторожно!** | Первый запрос `remoteEntry.js` — задержка. Но потом кэшируется. |
| ✅ Вы работаете в монорепозитории (Nx, Turborepo) | ✅ **Да!** | Module Federation — идеальный способ разделять код без сборки всего |

---

## ✅ Module Federation vs Build-time MF

| Критерий                        | **Module Federation (runtime)** | **Build-time MF**                    |
| ------------------------------- | ------------------------------- | ------------------------------------ |
| **Когда компоненты собираются** | На клиенте, при запуске         | На CI/CD, при сборке                 |
| **Загрузка компонентов**        | Динамическая (через `import()`) | Через `<script>` в `index.html`      |
| **Версии**                      | Можно менять без пересборки     | Нужно пересобрать главное приложение |
| **Скорость загрузки**           | Медленнее (1+ HTTP-запрос)      | Быстрее (один HTML)                  |
| **Сложность**                   | Высокая (CORS, версии, shared)  | Низкая                               |
| **Независимость релизов**       | ✅ Высокая                       | ❌ Низкая                             |
| **Поддержка старых браузеров**  | ❌ Плохо (требуется Fetch, ES6)  | ✅ Хорошо                             |
| **Использование в продакшене**  | ✅ Да (Spotify, Microsoft, AWS)  | ✅ Да (Netflix, Airbnb)               |

> ✅ **Module Federation — это "жизнеспособный" подход для больших систем.**  
> **Build-time — для простых, статичных систем.**

---

## ✅ Примеры реальных компаний

| Компания | Использование |
|---------|---------------|
| **Microsoft** | Office Online — 10+ микрофронтендов (Word, Excel, Teams) |
| **Spotify** | Веб-приложение — 30+ микросервисов, все через Module Federation |
| **AWS** | AWS Console — разные команды, разные релизы, единый интерфейс |
| **TikTok** | Веб-версия — гибкая архитектура с плагинами |
| **Zalando** | Электронная коммерция — независимые команды по категориям товаров |

> 💬 *«Мы перешли на Module Federation — и теперь можем выпускать новые функции за 1 день, а не за 2 недели.»* — инженер Spotify

---

## ✅ Как отладить Module Federation?

### 🔧 Инструменты:

| Инструмент | Зачем |
|----------|-------|
| **Webpack Bundle Analyzer** | Проверить, что React не дублируется |
| **DevTools → Network** | Убедиться, что `remoteEntry.js` загружается |
| **Console** | Искать ошибки: `Uncaught Error: Module not found` — значит, `remoteEntry.js` не доступен |
| **React DevTools** | Проверить, что компоненты работают в **одном React-экземпляре** |

> ✅ Если React DevTools показывает **два экземпляра React** — вы **не настроили `shared` правильно**.

---

## ✅ Лучшие практики

| Правило | Объяснение |
|--------|-----------|
| ✅ **Всегда используйте `shared` для React и React DOM** | Без этого — сломается всё |
| ✅ **Используйте `singleton: true`** | Гарантирует один экземпляр |
| ✅ **Используйте `requiredVersion`** | Чтобы не было конфликта версий |
| ✅ **Не экспортируйте `App` — экспортируйте компоненты** | `Button`, `Card`, `Modal` — не целое приложение |
| ✅ **Публикуйте `remoteEntry.js` на CDN** | Используйте S3 + CloudFront, Vercel, Netlify |
| ✅ **Используйте CI/CD для деплоя Remote** | Каждый Remote — отдельный pipeline |
| ✅ **Пишите документацию** | Какие компоненты есть, какие версии, какие зависимости |

---

## ✅ Что дальше? (Можно улучшить!)

| Технология | Зачем |
|------------|------|
| **Module Federation + React Router** | Чтобы маршруты работали между приложениями |
| **Module Federation + Micro Frontend Framework** | [Single SPA](https://single-spa.js.org/) — управляет загрузкой нескольких MF |
| **Module Federation + TypeScript** | Используйте `@module-federation/typescript` — чтобы типы работали |
| **Module Federation + Webpack 6** | Будет ещё лучше — меньше багов, больше фич |

---

## ✅ Итог: Module Federation — это **"microservices для фронтенда"**

| Вопрос | Ответ |
|--------|-------|
| **Что такое Module Federation?** | Механизм Webpack 5, позволяющий **динамически подключать компоненты из других приложений** |
| **Для чего нужен?** | Чтобы **независимые команды** могли работать **в одном интерфейсе**, **без монолита** |
| **Как устроен?** | Host загружает `remoteEntry.js` → узнаёт, какие модули есть → загружает нужный JS-файл → использует его |
| **Как пользоваться?** | 1. Настройте `ModuleFederationPlugin` в Webpack<br>2. Экспортируйте компоненты (`exposes`)<br>3. Разделите зависимости (`shared`)<br>4. Используйте `import('remote/Component')` |
| **Когда использовать?** | Когда у вас **большая команда**, **много команд**, **нужны независимые релизы** |
| **Когда не использовать?** | Если у вас **одна команда** и **один проект** — это перебор |

---

## ✅ Финальная мысль

> 🔹 **Module Federation — это не просто фича Webpack. Это новый способ строить фронтенд.**  
> 🔹 **Это как "подключить плагин" в браузере — как в VS Code.**  
> 🔹 **Это делает фронтенд так же гибким, как бэкенд.**

> 💬 *«Раньше мы строили монолиты. Потом — микросервисы. Теперь — мы строим фронтенд как набор плагинов.»*

---

## 📚 Где учиться дальше?

- **Официальная документация**: https://webpack.js.org/concepts/module-federation/
- **Книга**: *“Micro Frontends in Action”* — Manning
- **YouTube**: *“Module Federation Explained” — Maxence Poutord* (15 мин)
- **GitHub Example**: https://github.com/module-federation/module-federation-examples

---

## 🎯 Почему нет CORS ошибок при Module Federation?

### 📌 Краткий ответ

> **Module Federation загружает удалённые модули через `<script>` теги, а не через [[XHR]]/Fetch.**  
> **`<script src="...">` не подпадает под CORS ограничения браузера.**

---

## 🧩 Как работает загрузка модулей

### ❌ CORS применяется к:

| Ресурс | CORS? | Почему |
|--------|-------|--------|
| `fetch()` / `XMLHttpRequest` | ✅ Да | Запрос через JavaScript |
| `WebSocket` | ✅ Да | Сетевое соединение |
| `@font-face` | ✅ Да | Шрифты через CSS |
| `<img>` + canvas | ✅ Да | Чтение пикселей |
| **`<script src="...">`** | ❌ **Нет** | Исторически разрешено |
| **`<link rel="stylesheet">`** | ❌ **Нет** | Исторически разрешено |

---

### ✅ Module Federation использует `<script>` теги

```javascript
// webpack runtime создаёт что-то вроде:
const script = document.createElement('script');
script.src = 'https://remote-app.com/remoteEntry.js';
document.head.appendChild(script);

// ✅ Это НЕ触发рует CORS проверку
```

---

## 📊 Визуализация процесса

```
┌─────────────────────────────────────────────────────────────┐
│  Host App (localhost:3000)                                  │
│                                                             │
│  1. Загружает remoteEntry.js через <script>                 │
│     └─> https://remote-app.com/remoteEntry.js ✅ No CORS   │
│                                                             │
│  2. webpack runtime запрашивает модуль                      │
│     └─> https://remote-app.com/vendors.js ✅ No CORS       │
│                                                             │
│  3. Код выполняется в контексте Host App                    │
│     └─> API вызовы → ✅ CORS применяется!                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 Детали реализации webpack

### 1. **Загрузка remoteEntry.js**

```javascript
// Упрощённый код из webpack runtime
function loadScript(url) {
    return new Promise((resolve, reject) => {
        const script = document.createElement('script');
        script.src = url;
        script.onload = resolve;
        script.onerror = reject;
        document.head.appendChild(script);
    });
}

// Вызов:
loadScript('https://remote-app.com/remoteEntry.js');
// ✅ Никаких CORS заголовков не требуется
```

### 2. **Почему `<script>` без CORS?**

| Причина | Объяснение |
|---------|------------|
| **Историческая совместимость** | Веб всегда позволял загрузку скриптов с CDN |
| **CDN работают так** | jQuery, React, analytics — всё через `<script>` |
| **Изоляция выполнения** | Скрипт выполняется в вашем origin, не имеет доступа к другим origin |
| **Риск XSS** | ⚠️ Это обратная сторона — если remote скомпрометирован, это XSS |

---

## ⚠️ Когда CORS ВСЁ-ТАКИ применяется

### 1. **API вызовы из микрофронтенда**

```javascript
// remote-app.com/src/Component.js
export function fetchData() {
    return fetch('https://api.backend.com/data');
    // ✅ CORS применяется! Нужны заголовки от api.backend.com
}
```

### 2. **Загрузка ассетов через JavaScript**

```javascript
// ❌ CORS применяется
const img = new Image();
img.crossOrigin = 'anonymous';
img.src = 'https://cdn.example.com/image.png';

// ✅ CORS не применяется (прямой HTML)
<img src="https://cdn.example.com/image.png" />
```

### 3. **Fetch для загрузки модулей (если кастомизировано)**

```javascript
// ❌ Если вы переопределили загрузчик:
fetch('https://remote-app.com/remoteEntry.js')
    .then(r => r.text());
// ✅ CORS применяется! Нужны заголовки
```

---

## 🔐 Безопасность: о чём нужно знать

### Риски Module Federation

| Риск | Описание | Митигация |
|------|----------|-----------|
| **XSS через remote** | Если remote скомпрометирован — выполняется в вашем origin | Доверяйте только контролируемым remote |
| **Нет Subresource Integrity** | webpack не поддерживает SRI для Module Federation | Используйте HTTPS + доверяйте источнику |
| **Утечка данных** | Remote код имеет доступ к вашему DOM, localStorage, cookie | Изолируйте через sandbox/iframe если нужно |
| **Supply chain attack** | Зависимости remote могут быть скомпрометированы | Lock версии, используйте private registry |

### Best Practices

```javascript
// ✅ Настройте trusted remotes в webpack.config.js
new ModuleFederationPlugin({
    remotes: {
        remoteApp: 'remoteApp@https://trusted-domain.com/remoteEntry.js'
    },
    // ✅ Фиксируйте версии shared зависимостей
    shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' }
    }
});

// ✅ Используйте HTTPS для всех remote
// ❌ Избегайте HTTP (риск MITM атаки)

// ✅ Мониторьте remote на изменения
// ✅ Имейте план отката если remote сломался
```

---

## 🧪 Пример: когда CORS появится

### Сценарий 1: Module Federation (без CORS)

```javascript
// host-app.com
import { RemoteComponent } from 'remoteApp/Component';
// ✅ Загружается через <script> — нет CORS
```

### Сценарий 2: API вызов из remote (CORS применяется)

```javascript
// remote-app.com/Component.js
export function RemoteComponent() {
    useEffect(() => {
        fetch('https://api.host-app.com/data')
            // ❌ CORS! api.host-app.com должен вернуть:
            // Access-Control-Allow-Origin: https://remote-app.com
    }, []);
}
```

### Сценарий 3: Кастомная загрузка (CORS применяется)

```javascript
// ❌ Если вы сами загружаете модули через fetch:
const module = await fetch('https://remote.com/module.js');
// ❌ Нужны CORS заголовки от remote.com
```

---

## 📋 Чек-лист для production

```
□ Все remote на HTTPS ✅
□ Remote домены под вашим контролем ✅
□ API вызовы имеют правильные CORS заголовки ✅
□ Shared зависимости зафиксированы (singleton) ✅
□ Есть мониторинг доступности remote ✅
□ План отката при проблемах с remote ✅
□ Content Security Policy настроен ✅
□ Subresource Integrity (если возможно) ⚠️
```

---

## 📌 Памятка

```
┌─────────────────────────────────────────────────────────────┐
│  Module Federation → <script> теги → ❌ Нет CORS            │
│                                                             │
│  API вызовы → fetch/XHR → ✅ CORS применяется               │
│                                                             │
│  ⚠️ Безопасность: доверяйте только контролируемым remote   │
│  ⚠️ Remote код выполняется в ВАШЕМ origin (XSS риск)       │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Итог

| Вопрос | Ответ |
|--------|-------|
| **Почему нет CORS для модулей?** | Загрузка через `<script>` теги |
| **Применяется ли CORS вообще?** | ✅ Да, для API вызовов из кода |
| **Безопасно ли это?** | ⚠️ Только если доверяете remote |
| **Можно ли загрузить любой remote?** | Технически да, но это XSS риск |
| **Нужны ли CORS заголовки от remote?** | ❌ Нет для модулей, ✅ Да для API |

---

## 💡 Бонус: Настройка CORS для API

Если ваши микрофронтенды делают API вызовы:

```javascript
// Node.js (Express)
app.use((req, res, next) => {
    res.header('Access-Control-Allow-Origin', 'https://host-app.com');
    res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    next();
});

// Spring Boot
@Configuration
public class CorsConfig {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("https://host-app.com", "https://remote-app.com");
            }
        };
    }
}
```

---

Если хочешь — могу показать:

- Как настроить **Content Security Policy** для Module Federation
- Как изолировать remote через **iframe + postMessage**
- Как мониторить **доступность remote apps**
