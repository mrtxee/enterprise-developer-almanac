---
aliases:
  - Load Balancing
  - Location
  - Nginx
  - NGINX
  - nginx-conf
  - nginx.conf
  - Reverse Proxy
  - Server Block
  - Upstream
  - Virtual Host
  - Балансировка нагрузки
  - Виртуальный хост
  - Директивы nginx
  - Конфигурация nginx
  - Обратный прокси
---

## Синтаксис конфигураций Nginx: основные элементы и пояснения

Конфигурационные файлы [[nginx|Nginx]] имеют древовидную структуру из **контекстов** (блоков) и **директив** (инструкций). Главный файл — `/etc/nginx/nginx.conf`.

### Общие правила синтаксиса

- Каждая директива заканчивается **точкой с запятой** (`;`).
- Блоки открываются `{` и закрываются `}`.
- Комментарии начинаются с `#`.
- Пробелы и табуляции используются как разделители.

### Основные контексты (блоки)

**Контекст `main`**

- **Где:** вне всех блоков.
- **Для чего:** общие настройки сервера.
- **Примеры директив:**

```nginx
user www-data;
worker_processes auto;
pid /var/run/nginx.pid;
```

**Контекст `events`**

- **Где:** в корне конфигурации.
- **Для чего:** настройка обработки сетевых соединений.
- **Пример:**

```nginx
events {
    worker_connections 1024;
}
```

**Контекст `http`**

- **Где:** в корне конфигурации.
- **Для чего:** настройки [[http-server|HTTP-сервера]] (виртуальные хосты, прокси, кеширование).
- **Примеры директив:**

```nginx
http {
    include mime.types;
    default_type application/octet-stream;
    server { ... }
}
```

**Контекст `server`**

- **Где:** внутри блока `http`.
- **Для чего:** конфигурация виртуального хоста (сайта).
- **Ключевые директивы:**

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com;
    index index.html;
    location / { ... }
}
```

**Контекст `location`**

- **Где:** внутри блока `server`.
- **Для чего:** обработка запросов по URI (путям).
- **Примеры:**

```nginx
location / {
    try_files $uri $uri/ =404;
}
location /images/ {
    root /data;
}
```

**Контекст `upstream`**

- **Где:** внутри блока `http`.
- **Для чего:** балансировка нагрузки между серверами.
- **Пример:**

```nginx
upstream backend {
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}
```

### Ключевые директивы

| Директива | Назначение | Пример |
| --------- | ---------- | ------ |
| `user` | Пользователь, от имени которого работают процессы Nginx | `user www-data;` |
| `worker_processes` | Число рабочих процессов (обычно `auto`) | `worker_processes auto;` |
| `listen` | Порт и IP для прослушивания (например, `80` для HTTP) | `listen 80;` |
| `server_name` | Доменные имена, обслуживаемые этим блоком `server` | `server_name example.com www.example.com;` |
| `root` | Корневая директория для файлов сайта | `root /var/www/example.com;` |
| `index` | Файлы, которые считаются индексными (порядок важен) | `index index.html index.htm;` |
| `access_log`, `error_log` | Пути к логам | `access_log /var/log/nginx/access.log;`<br>`error_log /var/log/nginx/error.log;` |
| `include` | Подключение других конфигурационных файлов | `include /etc/nginx/sites-enabled/*;` |
| `proxy_pass` | Адрес бэкенд-сервера для проксирования | `proxy_pass http://127.0.0.1:8080;` |
| `try_files` | Проверка существования файлов по порядку | `try_files $uri $uri/ =404;` |
| `return` | Перенаправление или возврат кода ответа | `return 301 https://$host$request_uri;` |
| `ssl_certificate`, `ssl_certificate_key` | Пути к SSL-сертификату и ключу | `ssl_certificate /etc/ssl/certs/example.com.crt;`<br>`ssl_certificate_key /etc/ssl/private/example.com.key;` |

### Пример минимальной конфигурации

```nginx
user www-data;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include mime.types;
    default_type application/octet-stream;

    server {
        listen 80;
        server_name example.com;

        root /var/www/example.com;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}
```

### Важные замечания

1. **Порядок директив** имеет значение: Nginx обрабатывает их последовательно.
2. **Вложенность блоков** строго определена: например, `location` только внутри `server`, а `server` — внутри `http`.
3. **Проверка конфигурации** перед перезагрузкой: `sudo nginx -t`.
4. **Перезагрузка Nginx** после изменений: `sudo systemctl reload nginx`.
