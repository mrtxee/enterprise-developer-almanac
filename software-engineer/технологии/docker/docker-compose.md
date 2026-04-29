# docker-compose bash

Управление **Мультиконтейнерным приложением Docker Compose** 
* проектом Docker Compose

```bash
# запуск мультиконтейнерное приложение без привязки к в рунтайм логу
docker-compose up -d
# остановить мультиконтейнерное приложение без удаление томов (volumes)
docker-compose down
# удалить мультиконтейнерное приложение с удалением томов (volumes)
docker-compose down -v
# смотреть лог контейнера kafka
docker-compose logs kafka -f
# запуск одного сервиса
docker-compose up -d <service_name>
# остановка одного сервиса
docker-compose stop <service_name>
# перезапуск одного сервиса
docker-compose restart <service_name>
# удаление одного сервиса (остановка + удаление контейнера) без удаления томов
docker-compose rm -f <service_name>
# cписок запущенных контейнеров
docker-compose ps
```

* more https://devops.org.ru/dockercompose-summary#q
# docker-compose structure

docker-compose.yml
```yml
version: '3.8'  
  
services:  
  kafka:  
    image: apache/kafka:3.7.0  
    container_name: kafka  
    ports:  
      - "9092:9092"  
    environment:  
      KAFKA_PROCESS_ROLES: broker,controller  
      KAFKA_NODE_ID: 1  
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka:9093"  
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093  
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092  
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER  
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1  
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1  
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1  
      KAFKA_LOG_DIRS: /tmp/kafka-logs  
    volumes:  
      - kafka-data:/tmp/kafka-logs  
    networks:  
      - kafka-net  
    restart: unless-stopped  
  
  kafka-ui:  
    image: provectuslabs/kafka-ui:latest  
    container_name: kafka-ui  
    depends_on:  
      - kafka  
    ports:  
      - "8093:8080"  
    environment:  
      - KAFKA_CLUSTERS_0_NAME=local-kraft  
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka:9092  
    restart: unless-stopped  
    networks:  
      - kafka-net  
  
volumes:  
  kafka-data:  
  
networks:  
  kafka-net:  
    driver: bridge
```

запуск
```bash
docker-compose up -d
```

# docker-compose guide
Вот полное описание **основных разделов (top-level keys)** и их структуры в файле `docker-compose.yml` для **версии 3.8**, с указанием **допустимых значений** и кратких пояснений.

---

## 📌 структура `docker-compose.yml`

на примере вресии v3.8

```yaml
version: "3.8"

services:
  # Определения контейнеров

networks:
  # Пользовательские сети

volumes:
  # Именованные тома

configs:
  # Конфигурации (для Swarm)

secrets:
  # Секреты (для Swarm)
```

> ⚠️ **Важно**: В режиме **Docker Compose (не Swarm)** используются только `services`, `networks`, `volumes`. `configs` и `secrets` работают **только в Swarm-режиме**.

---

## `services`
Определяет контейнеры (сервисы), которые будут запущены.

Общая структура сервиса:
```yaml
services:
  <service_name>:
    image: ...
    build: ...
    ports: ...
    environment: ...
    volumes: ...
    networks: ...
    restart: ...
    # и т.д.
```

**🔹 Ключевые подразделы и их значения:**

| Ключ | Тип / Примеры | Описание |
|------|----------------|--------|
| `image` | строка | Образ: `nginx:alpine`, `myapp:latest` |
| `build` | строка или объект | Путь к Dockerfile или объект с `context`, `dockerfile`, `args` |
| `ports` | список | Проброс портов: `- "8080:80"`, `- "5000"` |
| `expose` | список | Открыть порт **внутри сети**, не на хост: `- "8080"` |
| `environment` | список или объект | Переменные окружения: `DB_HOST=db` или `- DB_HOST=db` |
| `env_file` | строка или список | Файл(ы) с переменными: `.env`, `prod.env` |
| `volumes` | список | Монтирование томов: `- ./data:/app/data`, `- myvol:/data` |
| `networks` | список или объект | Подключение к сетям: `- mynet` или `mynet: {aliases: [web]}` |
| `depends_on` | список | Зависимости запуска: `- db` (не ждёт готовности!) |
| `restart` | строка | Политика перезапуска:<br>`no` (по умолчанию),<br>`always`,<br>`on-failure[:max-retries]`,<br>`unless-stopped` |
| `command` | строка или список | Переопределить CMD: `["python", "app.py"]` |
| `entrypoint` | строка или список | Переопределить ENTRYPOINT |
| `healthcheck` | объект | Проверка здоровья:<br>`test: ["CMD", "curl", "-f", "http://localhost"]`<br>`interval: 30s`, `timeout: 10s`, `retries: 3` |
| `deploy` | объект | **Только для Swarm!** Ресурсы, реплики и т.д. |
| `user` | строка | Пользователь в контейнере: `"1000:1000"` |
| `working_dir` | строка | Рабочая директория: `/app` |
| `stdin_open`, `tty` | boolean | `-T` и `-i` в `docker run`: `true` / `false` |
| `labels` | объект или список | Метки: `com.example.env: prod` |
| `container_name` | строка | Имя контейнера (не рекомендуется — ломает масштабирование) |
| `extra_hosts` | список | Добавить записи в `/etc/hosts`: `- "host.docker.internal:host-gateway"` |
| `logging` | объект | Драйвер логов: `driver: "json-file"`, `options: {max-size: "10m"}` |

### restart – политики перезапуска

1. `no` (по умолчанию)
```yaml
restart: "no"
```
- **Контейнер никогда не перезапускается автоматически**.
- Даже если он завершился с ошибкой — остаётся остановленным.
- Это значение по умолчанию, если `restart` не указан.

> 💡 Подходит для **одноразовых задач** или отладки.

---

2. `always`
```yaml
restart: always
```
- **Контейнер всегда перезапускается**, **независимо от кода завершения**.
- Перезапускается даже после ручной остановки (`docker stop`) — **если Docker-демон перезапущен**.
- После `docker stop` контейнер **не перезапускается до перезагрузки демона**.

> 💡 Используется для **критически важных сервисов**, которые должны быть всегда "вверху".

---

3. `on-failure[:max-retries]`
```yaml
restart: on-failure
# или с ограничением попыток:
restart: on-failure:5
```
- Контейнер перезапускается **только если завершился с ненулевым кодом выхода** (ошибка).
- Опционально можно указать **максимальное число попыток** (например, `on-failure:3`).
- Если указан лимит — после его исчерпания перезапуски прекращаются.
- При успешном завершении (`exit 0`) — **не перезапускается**.

> 💡 Идеально для **задач, которые могут временно падать**, но не должны работать вечно (например, batch-процессы).

---

4. `unless-stopped`
```yaml
restart: unless-stopped
```
- Контейнер **всегда перезапускается**, **кроме случаев, когда он был остановлен вручную** (`docker stop` или `docker-compose stop`).
- После ручной остановки — **не перезапускается даже при перезапуске Docker-демона**.
- Это **самая популярная политика** для сервисов в продакшене.

> 💡 Рекомендуется для **веб-серверов, баз данных, Kafka, UI и т.д.**

---

📊 Сравнительная таблица

| Политика | Перезапуск при ошибке | Перезапуск при успехе (`exit 0`) | Перезапуск после `docker stop` | После перезапуска Docker-демона |
|--------|----------------------|-------------------------------|------------------------------|-------------------------------|
| `no` | ❌ | ❌ | ❌ | ❌ |
| `always` | ✅ | ✅ | ❌* | ✅ |
| `on-failure` | ✅ | ❌ | ❌ | ✅ (если упал) |
| `unless-stopped` | ✅ | ✅ | ❌ | ✅ (если не был остановлен вручную) |

> \* — после `docker stop` контейнер не перезапускается **до перезагрузки демона**, но если демон перезапущен — `always` снова запустит его.

---

💡 Рекомендации

- Для **веб-приложений, баз данных, Kafka, UI**:  
  ```yaml
  restart: unless-stopped
  ```
- Для **временных задач или отладки**:  
  ```yaml
  restart: no
  ```
- Для **сервисов, которые должны перезапускаться только при падении**:  
  ```yaml
  restart: on-failure:3
  ```


---

## 2. `networks` — пользовательские сети

Определяет сети, которые могут использоваться сервисами.

### Пример:
```yaml
networks:
  frontend:
    driver: bridge
  backend:
    external: true
    name: my-existing-net
```

### Возможные параметры:
| Ключ | Значения | Описание |
|------|--------|--------|
| `driver` | `bridge` (по умолчанию), `overlay`, `host`, `none` | Драйвер сети |
| `driver_opts` | объект | Опции драйвера |
| `external` | `true` / `false` | Использовать существующую сеть |
| `name` | строка | Имя внешней сети |
| `internal` | `true` / `false` | Запретить исходящий трафик (`true`) |
| `attachable` | `true` / `false` | Разрешить подключение "внешних" контейнеров (для `overlay`) |

---

## 3. `volumes` — именованные тома

Управление постоянным хранилищем.

### Пример:
```yaml
volumes:
  db-data:
    driver: local
  logs:
    external: true
    name: my-logs-vol
```

### Параметры:
| Ключ | Значения | Описание |
|------|--------|--------|
| `driver` | `local` (по умолчанию), `nfs`, `ceph`, и др. | Драйвер тома |
| `driver_opts` | объект | Опции драйвера (например, `type: "nfs"`) |
| `external` | `true` / `false` | Использовать существующий том |
| `name` | строка | Имя внешнего тома |
| `labels` | объект | Метки для тома |

---

## 4. `configs` — конфигурации (Swarm-only)

> ❗ Работает **только в Docker Swarm**.

```yaml
configs:
  my-config:
    file: ./config.yml
  nginx-conf:
    external: true
    name: prod-nginx-conf
```

Параметры: `file`, `external`, `name`, `labels`.

---

## 5. `secrets` — секреты (Swarm-only)

> ❗ Работает **только в Docker Swarm**.

```yaml
secrets:
  db-password:
    file: ./db-pass.txt
  tls-cert:
    external: true
    name: prod-cert
```

Параметры: `file`, `external`, `name`, `labels`.

---

## 💡 Полезные замечания

- **Версия 3.8** — последняя в ветке v3 (актуальна для Compose и Swarm).
- Для **локальной разработки** чаще всего используются только `services`, `volumes`, `networks`.
- `deploy` игнорируется в `docker-compose up`, но работает в `docker stack deploy`.
- Все пути (в `build`, `volumes`, `env_file`) — относительно **директории с `docker-compose.yml`**.

## networks
В контексте секции `networks` в файле `docker-compose.yml` параметры **`external`**, **`internal`** и **`driver`** управляют поведением и свойствами Docker-сетей. Вот подробное объяснение каждого:

---

### 1. `external`

**Назначение**: Указывает, что сеть **уже существует** в Docker и **не должна создаваться** Compose.

#### Пример:
```yaml
networks:
  my-existing-net:
    external: true
    name: production-network
```

- `external: true` — Compose **не создаёт** сеть, а **подключается к существующей**.
- `name` — фактическое имя сети в Docker (если не указано, используется имя из ключа: `my-existing-net`).

> 💡 Используется, когда сеть создана вручную:  
> ```bash
> docker network create production-network
> ```

#### Если `external: false` (или не указано):
- Compose **создаёт сеть автоматически** при `docker-compose up`.
- Имя сети: `<project_name>_<network_name>` (например, `myapp_default`).

---

### 2. `internal`

**Назначение**: Запрещает контейнерам в этой сети **выходить в интернет** (изолирует от внешнего мира).

#### Пример:
```yaml
networks:
  isolated-net:
    internal: true
```

- Контейнеры могут общаться **друг с другом**, но **не могут делать исходящие запросы** (например, `curl https://google.com` не сработает).
- Полезно для **безопасности**: базы данных, внутренние микросервисы.

> ⚠️ Даже DNS-запросы к внешним хостам будут заблокированы.

---

### 3. `driver`

**Назначение**: Указывает **драйвер сети**, который Docker использует для создания сети.

#### Возможные значения:
| Драйвер | Описание |
|--------|--------|
| `bridge` (по умолчанию) | Стандартная сеть на одном хосте. Используется в большинстве случаев. |
| `overlay` | Сеть между несколькими хостами (требуется Swarm или Docker в режиме swarm). |
| `host` | Контейнер использует сетевой стек хоста напрямую (редко в Compose). |
| `none` | Отключает сетевой стек (контейнер полностью изолирован). |
| `macvlan` / `ipvlan` | Присваивает контейнеру MAC/IP-адрес в физической сети. |

#### Пример:
```yaml
networks:
  backend:
    driver: bridge
  swarm-net:
    driver: overlay
```

> 💡 Для локальной разработки почти всегда используется `bridge`.

---

### 📌 Сводка

| Параметр | По умолчанию | Назначение |
|---------|-------------|-----------|
| `external` | `false` | Использовать существующую сеть, а не создавать новую |
| `internal` | `false` | Блокировать исходящий трафик из сети |
| `driver` | `bridge` | Тип сетевого драйвера |

---

### ✅ Пример комплексного использования

```yaml
networks:
  db-net:
    driver: bridge
    internal: true          # БД не выходит в интернет
  public-net:
    external: true          # Используем уже созданную сеть
    name: traefik-public
```

Это позволяет гибко управлять сетевой изоляцией и интеграцией с внешними компонентами (например, Traefik, внешние балансировщики).

## env_file
Вот простой и понятный пример использования `env_file` в `docker-compose.yml`.

---

### 📁 1. Создайте файл с переменными окружения

Например, файл `.env.local` (имя может быть любым):

```env
# .env.local
DB_HOST=db
DB_PORT=5432
DB_NAME=myapp
DB_USER=admin
DB_PASSWORD=secret123
LOG_LEVEL=INFO
```

> ⚠️ Формат: **ключ=значение**, без кавычек (если значение не содержит пробелов).  
> Комментарии — через `#`.

---

### 📄 2. Подключите его в `docker-compose.yml`

```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    env_file:
      - .env.local          # ← подключение файла
    ports:
      - "8080:8080"
    depends_on:
      - db

  db:
    image: postgres:15
    env_file:
      - .env.local          # ← можно использовать один файл для нескольких сервисов
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pg/var/lib/postgresql/data

volumes:
  pg
```

---

### 🔍 Как это работает?

- Docker Compose **загружает переменные из `.env.local`** и делает их доступными **внутри контейнера** как переменные окружения.
- Также эти переменные можно **использовать в самом `docker-compose.yml`** через `${VAR_NAME}` — как в примере с `POSTGRES_DB`.

> 💡 Важно:  
> - Файл `.env.local` должен находиться **в той же директории**, что и `docker-compose.yml` (или указывайте полный путь).  
> - Переменные из `env_file` **не переопределяют** переменные из секции `environment` — наоборот, `environment` имеет **более высокий приоритет**.

---

### 🧪 Проверка

Запустите контейнер и проверьте переменные:

```bash
docker-compose run --rm app env | grep DB_
```

Вы увидите:
```
DB_HOST=db
DB_PORT=5432
DB_NAME=myapp
DB_USER=admin
DB_PASSWORD=secret123
```

---

### 📌 Дополнительно

- Можно указать **несколько файлов**:
  ```yaml
  env_file:
    - .env.defaults
    - .env.local
  ```
  → значения из `.env.local` **переопределят** значения из `.env.defaults`.

- Файл `.env` (без суффикса) **автоматически загружается** Compose для подстановки в `docker-compose.yml`, но **не передаётся в контейнеры**, если не указан в `env_file`.

---

Готово! Теперь вы можете гибко управлять конфигурацией через внешние файлы.