## Синтаксис конфигураций Nginx: основные элементы и пояснения


Конфигурационные файлы Nginx имеют древовидную структуру из **контекстов** (блоков) и **директив** (инструкций). Главный файл — `/etc/nginx/nginx.conf`.

### Общие правила синтаксиса
- Каждая директива заканчивается **точкой с запятой** (`;`).
- Блоки открываются `{` и закрываются `}`.
- Комментарии начинаются с `#`.
- Пробелы и табуляции используются как разделители.


### Основные контексты (блоки)

1. **`main`** (глобальный уровень)  
   *Где:* вне всех блоков.  
   *Для чего:* общие настройки сервера.  
   *Примеры директив:*  
   ```nginx
   user www-data;
   worker_processes auto;
   pid /var/run/nginx.pid;
   ```

2. **`events`**  
   *Где:* в корне конфигурации.  
   *Для чего:* настройка обработки сетевых соединений.  
   *Пример:*  
   ```nginx
   events {
       worker_connections 1024;
   }
   ```

3. **`http`**  
   *Где:* в корне конфигурации.  
   *Для чего:* настройки HTTP‑сервера (виртуальные хосты, прокси, кеширование).  
   *Примеры директив:*  
   ```nginx
   http {
       include mime.types;
       default_type application/octet-stream;
       server { ... }
   }
   ```

4. **`server`**  
   *Где:* внутри блока `http`.  
   *Для чего:* конфигурация виртуального хоста (сайта).  
   *Ключевые директивы:*  
   ```nginx
   server {
       listen 80;
       server_name example.com www.example.com;
       root /var/www/example.com;
       index index.html;
       location / { ... }
   }
   ```

5. **`location`**  
   *Где:* внутри блока `server`.  
   *Для чего:* обработка запросов по URI (путям).  
   *Примеры:*  
   ```nginx
   location / {
       try_files $uri $uri/ =404;
   }
   location /images/ {
       root /data;
   }
   ```

6. **`upstream`**  
   *Где:* внутри блока `http`.  
   *Для чего:* балансировка нагрузки между серверами.  
   *Пример:*  
   ```nginx
   upstream backend {
       server 127.0.0.1:8080;
       server 127.0.0.1:8081;
   }
   ```

### Ключевые директивы (с пояснениями)

- **`user`** — пользователь, от имени которого работают процессы Nginx.  
  ```nginx
  user www-data;
  ```

- **`worker_processes`** — число рабочих процессов (обычно `auto`).  
  ```nginx
  worker_processes auto;
  ```

- **`listen`** — порт и IP для прослушивания (например, `80` для HTTP).  
  ```nginx
  listen 80;
  ```

- **`server_name`** — доменные имена, обслуживаемые этим блоком `server`.  
  ```nginx
  server_name example.com www.example.com;
  ```

- **`root`** — корневая директория для файлов сайта.  
  ```nginx
  root /var/www/example.com;
  ```

- **`index`** — файлы, которые считаются индексными (порядок важен).  
  ```nginx
  index index.html index.htm;
  ```

- **`access_log`** и **`error_log`** — пути к логам.  
  ```nginx
  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;
  ```

- **`include`** — подключение других конфигурационных файлов.  
  ```nginx
  include /etc/nginx/sites-enabled/*;
  ```

- **`proxy_pass`** — адрес бэкенд‑сервера для проксирования.  
  ```nginx
  proxy_pass http://127.0.0.1:8080;
  ```

- **`try_files`** — проверка существования файлов по порядку.  
  ```nginx
  try_files $uri $uri/ =404;
  ```

- **`return`** — перенаправление или возврат кода ответа.  
  ```nginx
  return 301 https://$host$request_uri;
  ```

- **`ssl_certificate`** и **`ssl_certificate_key`** — пути к SSL‑сертификату и ключу.  
  ```nginx
  ssl_certificate /etc/ssl/certs/example.com.crt;
  ssl_certificate_key /etc/ssl/private/example.com.key;
  ```

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
2. **Вложенность блоков** строго определена: например, `location` только внутри `server`, а `server` — внутри `http`.
3. **Проверка конфигурации** перед перезагрузкой:  
   ```bash
   sudo nginx -t
   ```
4. **Перезагрузка Nginx** после изменений:  
   ```bash
   sudo systemctl reload nginx
   ```