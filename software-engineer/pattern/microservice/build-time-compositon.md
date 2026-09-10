---
aliases:
  - Build-Time Composition
  - Композиция на этапе сборки
  - Module Federation
  - Nginx-based Composition
  - Композиция на основе Nginx
  - Runtime Import
  - Runtime импорт
  - Health Check
  - Проверка работоспособности
  - CORS
---
## Build-Time Композиция

**Build-Time Композиция** — это подход в микрофронтендах, при котором отдельные приложения (микрофронты) компилируются (**build**) независимо друг от друга, но затем объединяются в единое целое на этапе **сборки (build)**, а не во время выполнения (run-time) в браузере.

Есть 3 варианта реализации:

1. [[Module Federation]] Build
  - плагин JavaScript сборщика пакетов webpack
2. Nginx-based build
3. **Runtime импорт с проверкой**

### Компоновка Module Federation

### Компоновка на основе Nginx

Конфигурация nginx.conf для main-ui:

```nginx
server {
  listen 80;

  location / {
    root /usr/share/nginx/html;
    try_files $uri $uri/ /index.html;
  }

  # Прокси к deposit-ui для модулей
  location /deposit-ui/ {
    proxy_pass http://deposit-ui:3001/;
  }
}
```

### Runtime импорт с проверкой

Enhanced компонент с обработкой ошибок:

```jsx
const useRemoteModule = (scope, module) => {
  const [Component, setComponent] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    const loadComponent = async () => {
      try {
        // Динамический импорт
        const comp = await import(
          /* webpackIgnore: true */ `http://deposit-ui:3001/${module}.js`
        );
        setComponent(() => comp.default);
      } catch (err) {
        setError(err.message);
      }
    };

    loadComponent();
  }, [scope, module]);

  return { Component, error };
};
```

---

### Безопасность и оптимизация

#### CORS настройки

Конфигурация nginx для deposit-ui:

```nginx
server {
  listen 3001;

  add_header 'Access-Control-Allow-Origin' 'http://main-ui:3000';
  add_header 'Access-Control-Allow-Methods' 'GET, OPTIONS';
  add_header 'Access-Control-Allow-Headers' 'Content-Type';

  location / {
    root /usr/share/nginx/html;
    try_files $uri $uri/ =404;
  }
}
```

#### Health checks

Health check для deposit-ui:

```javascript
// Health check для deposit-ui
const checkDepositUIAvailability = async () => {
  try {
    const response = await fetch('http://deposit-ui:3001/remoteEntry.js');
    return response.ok;
  } catch {
    return false;
  }
};
```

---

### Мониторинг и логирование

#### Метрики взаимодействия

Интеграция с мониторингом:

```javascript
// Интеграция с мониторингом
const withMetrics = (WrappedComponent, moduleName) => {
  return (props) => {
    useEffect(() => {
      // Логирование загрузки модуля
      performance.mark(`${moduleName}-load-start`);
    }, []);

    return <WrappedComponent {...props} />;
  };
};
```

### Рекомендации

1. **Используйте Webpack Module Federation** — это стандарт для micro-frontends.
2. **Настройте shared dependencies** — для избежания дублирования React.
3. **Реализуйте graceful degradation** — если deposit-ui недоступен.
4. **Используйте Docker network** для внутренней коммуникации.
5. **Настройте health checks** в docker-compose.

Такая архитектура обеспечит seamless интеграцию двух UI систем в единый интерфейс для пользователя!