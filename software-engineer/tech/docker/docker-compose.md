---
aliases:
  - Compose
  - configs
  - Docker
  - Docker Compose
  - Docker network
  - docker compose
  - docker-compose
  - docker-compose.yml
  - env_file
  - named volume
  - networks
  - restart policy
  - services
  - volumes
  - Docker сети
  - Именованные тома
  - конфигурации
  - политики перезапуска
  - секреты
  - сервисы
  - сети Docker
---

# Docker Compose

**Docker Compose** — инструмент управления мультиконтейнерным приложением: сервисы, сети и тома описываются в одном файле `docker-compose.yml` и управляются одной командой.

## Команды управления приложением

**Базовые команды**

```bash
# запуск мультиконтейнерного приложения без привязки к рунтайм-логу
docker-compose up -d
# остановить мультиконтейнерное приложение без удаления томов (volumes)
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
# список запущенных контейнеров
docker-compose ps
```

## Структура docker-compose.yml

Пример файла с сервисами Kafka и kafka-ui:

```yaml
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

**Запуск**

```bash
docker-compose up -d
```

## Ключевые разделы docker-compose.yml

Описание основных top-level ключей и их структуры в файле `docker-compose.yml` для версии 3.8.

**Структура файла**

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

> ⚠️ В режиме Docker Compose (не Swarm) используются только `services`, `networks`, `volumes`. `configs` и `secrets` работают только в Swarm-режиме.

---

### services

Определяет контейнеры (сервисы), которые будут запущены.

**Общая структура сервиса**

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

**Ключевые подразделы и их значения**

| Ключ | Тип / Примеры | Описание |
|------|----------------|--------|
| `image` | строка | Образ: `nginx:alpine`, `myapp:latest` |
| `build` | строка или объект | Путь к Dockerfile или объект с `context`, `dockerfile`, `args` |
| `ports` | список | Проброс портов: `- "8080:80"`, `- "5000"` |
| `expose` | список | Открыть порт внутри сети, не на хост: `- "8080"` |
| `environment` | список или объект | Переменные окружения: `DB_HOST=db` или `- DB_HOST=db` |
| `env_file` | строка или список | Файл(ы) с переменными: `.env`, `prod.env` |
| `volumes` | список | Монтирование томов: `- ./data:/app/data`, `- myvol:/data` |
| `networks` | список или объект | Подключение к сетям: `- mynet` или `mynet: {aliases: [web]}` |
| `depends_on` | список | Зависимости запуска: `- db` (не ждёт готовности!) |
| `restart` | строка | Политика перезапуска: `no` (по умолчанию), `always`, `on-failure[:max-retries]`, `unless-stopped` |
| `command` | строка или список | Переопределить CMD: `["python", "app.py"]` |
| `entrypoint` | строка или список | Переопределить ENTRYPOINT |
| `healthcheck` | объект | Проверка здоровья: `test: ["CMD", "curl", "-f", "http://localhost"]`, `interval: 30s`, `timeout: 10s`, `retries: 3` |
| `deploy` | объект | Только для Swarm! Ресурсы, реплики и т.д. |
| `user` | строка | Пользователь в контейнере: `"1000:1000"` |
| `working_dir` | строка | Рабочая директория: `/app` |
| `stdin_open`, `tty` | boolean | `-T` и `-i` в `docker run`: `true` / `false` |
| `labels` | объект или список | Метки: `com.example.env: prod` |
| `container_name` | строка | Имя контейнера (не рекомендуется — ломает масштабирование) |
| `extra_hosts` | список | Добавить записи в `/etc/hosts`: `- "host.docker.internal:host-gateway"` |
| `logging` | объект | Драйвер логов: `driver: "json-file"`, `options: {max-size: "10m"}` |

**restart — политики перезапуска**

1. `no` (по умолчанию). Контейнер никогда не перезапускается автоматически: даже при завершении с ошибкой остаётся остановленным. Это значение по умолчанию, если `restart` не указан.

```yaml
restart: "no"
```

> 💡 Подходит для одноразовых задач или отладки.

2. `always`. Контейнер всегда перезапускается независимо от кода завершения. После ручной остановки (`docker stop`) перезапускается только после перезапуска Docker-демона.

```yaml
restart: always
```

> 💡 Используется для критически важных сервисов, которые должны быть всегда вверху.

3. `on-failure[:max-retries]`. Контейнер перезапускается только при ненулевом коде выхода (ошибке). Можно указать максимальное число попыток (например, `on-failure:3`); после исчерпания лимита перезапуски прекращаются. При успешном завершении (`exit 0`) не перезапускается.

```yaml
restart: on-failure
# или с ограничением попыток:
restart: on-failure:5
```

> 💡 Идеально для задач, которые могут временно падать, но не должны работать вечно (например, batch-процессы).

4. `unless-stopped`. Контейнер всегда перезапускается, кроме случаев, когда он был остановлен вручную (`docker stop` или `docker-compose stop`). После ручной остановки не перезапускается даже при перезапуске Docker-демона.

```yaml
restart: unless-stopped
```

> 💡 Самая популярная политика для продакшена: веб-серверы, базы данных, Kafka, UI.

**Сравнение политик перезапуска**

| Политика | Перезапуск при ошибке | Перезапуск при успехе (`exit 0`) | Перезапуск после `docker stop` | После перезапуска Docker-демона |
|--------|----------------------|-------------------------------|------------------------------|-------------------------------|
| `no` | ❌ | ❌ | ❌ | ❌ |
| `always` | ✅ | ✅ | ❌* | ✅ |
| `on-failure` | ✅ | ❌ | ❌ | ✅ (если упал) |
| `unless-stopped` | ✅ | ✅ | ❌ | ✅ (если не остановлен вручную) |

> Примечание: после `docker stop` контейнер с политикой `always` не перезапускается до перезагрузки демона, но после перезапуска демона `always` снова запустит его.

**Рекомендации по выбору политики**

- Для веб-приложений, баз данных, Kafka, UI:
  ```yaml
  restart: unless-stopped
  ```
- Для временных задач или отладки:
  ```yaml
  restart: no
  ```
- Для сервисов, которые должны перезапускаться только при падении:
  ```yaml
  restart: on-failure:3
  ```

---

### networks

Определяет пользовательские сети, которые используются сервисами.

**Пример**

```yaml
networks:
  frontend:
    driver: bridge
  backend:
    external: true
    name: my-existing-net
```

**Возможные параметры**

| Ключ | Значения | Описание |
|------|--------|--------|
| `driver` | `bridge` (по умолчанию), `overlay`, `host`, `none` | Драйвер сети |
| `driver_opts` | объект | Опции драйвера |
| `external` | `true` / `false` | Использовать существующую сеть |
| `name` | строка | Имя внешней сети |
| `internal` | `true` / `false` | Запретить исходящий трафик (`true`) |
| `attachable` | `true` / `false` | Разрешить подключение внешних контейнеров (для `overlay`) |

**external**

Указывает, что сеть уже существует в Docker и не должна создаваться Compose.

```yaml
networks:
  my-existing-net:
    external: true
    name: production-network
```

- `external: true` — Compose не создаёт сеть, а подключается к существующей.
- `name` — фактическое имя сети в Docker (если не указано, используется имя из ключа: `my-existing-net`).

> 💡 Используется, когда сеть создана вручную:
> ```bash
> docker network create production-network
> ```

Если `external: false` (или не указано):

- Compose создаёт сеть автоматически при `docker-compose up`.
- Имя сети: `<project_name>_<network_name>` (например, `myapp_default`).

**internal**

Запрещает контейнерам в этой сети выходить в интернет (изолирует от внешнего мира).

```yaml
networks:
  isolated-net:
    internal: true
```

- Контейнеры могут общаться друг с другом, но не делать исходящие запросы (например, `curl` к внешним хостам не сработает).
- Полезно для безопасности: базы данных, внутренние микросервисы.

> ⚠️ Даже DNS-запросы к внешним хостам будут заблокированы.

**driver**

Указывает драйвер сети, который Docker использует для создания сети.

| Драйвер | Описание |
|--------|--------|
| `bridge` (по умолчанию) | Стандартная сеть на одном хосте. Используется в большинстве случаев. |
| `overlay` | Сеть между несколькими хостами (требуется Swarm или Docker в режиме swarm). |
| `host` | Контейнер использует сетевой стек хоста напрямую (редко в Compose). |
| `none` | Отключает сетевой стек (контейнер полностью изолирован). |
| `macvlan` / `ipvlan` | Присваивает контейнеру MAC/IP-адрес в физической сети. |

**Пример**

```yaml
networks:
  backend:
    driver: bridge
  swarm-net:
    driver: overlay
```

> 💡 Для локальной разработки почти всегда используется `bridge`.

**Сводка**

| Параметр | По умолчанию | Назначение |
|---------|-------------|-----------|
| `external` | `false` | Использовать существующую сеть, а не создавать новую |
| `internal` | `false` | Блокировать исходящий трафик из сети |
| `driver` | `bridge` | Тип сетевого драйвера |

**Пример комплексного использования**

```yaml
networks:
  db-net:
    driver: bridge
    internal: true          # БД не выходит в интернет
  public-net:
    external: true          # Используем уже созданную сеть
    name: traefik-public
```

Позволяет гибко управлять сетевой изоляцией и интеграцией с внешними компонентами (например, Traefik, внешние балансировщики).

---

### volumes

Управление постоянным хранилищем (именованные тома).

**Пример**

```yaml
volumes:
  db-data:
    driver: local
  logs:
    external: true
    name: my-logs-vol
```

**Параметры**

| Ключ | Значения | Описание |
|------|--------|--------|
| `driver` | `local` (по умолчанию), `nfs`, `ceph` и др. | Драйвер тома |
| `driver_opts` | объект | Опции драйвера (например, `type: "nfs"`) |
| `external` | `true` / `false` | Использовать существующий том |
| `name` | строка | Имя внешнего тома |
| `labels` | объект | Метки для тома |

---

### configs — конфигурации (Swarm-only)

Работает только в Docker Swarm.

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

### secrets — секреты (Swarm-only)

Работает только в Docker Swarm.

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

**Полезные замечания**

- Версия 3.8 — последняя в ветке v3 (актуальна для Compose и Swarm).
- Для локальной разработки чаще всего используются только `services`, `volumes`, `networks`.
- `deploy` игнорируется в `docker-compose up`, но работает в `docker stack deploy`.
- Все пути (в `build`, `volumes`, `env_file`) — относительно директории с `docker-compose.yml`.

## env_file

Пример использования `env_file` в `docker-compose.yml`.

**Создание файла с переменными окружения**

Файл `.env.local` (имя может быть любым):

```env
# .env.local
DB_HOST=db
DB_PORT=5432
DB_NAME=myapp
DB_USER=admin
DB_PASSWORD=secret123
LOG_LEVEL=INFO
```

> ⚠️ Формат: ключ=значение, без кавычек (если значение не содержит пробелов). Комментарии — через `#`.

**Подключение файла в docker-compose.yml**

```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    env_file:
      - .env.local
    ports:
      - "8080:8080"
    depends_on:
      - db

  db:
    image: postgres:15
    env_file:
      - .env.local          # один файл можно использовать для нескольких сервисов
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pg:/var/lib/postgresql/data

volumes:
  pg:
```

**Как это работает**

- Docker Compose загружает переменные из `.env.local` и делает их доступными внутри контейнера как переменные окружения.
- Переменные можно использовать в самом `docker-compose.yml` через `${VAR_NAME}` — как в примере с `POSTGRES_DB`.

> 💡 Важно:
> - Файл `.env.local` должен находиться в той же директории, что и `docker-compose.yml` (или указывайте полный путь).
> - Переменные из `env_file` не переопределяют переменные из секции `environment` — наоборот, `environment` имеет более высокий приоритет.

**Проверка**

Запустите контейнер и проверьте переменные:

```bash
docker-compose run --rm app env | grep DB_
```

Вывод:

```text
DB_HOST=db
DB_PORT=5432
DB_NAME=myapp
DB_USER=admin
DB_PASSWORD=secret123
```

**Дополнительно**

- Можно указать несколько файлов; значения из `.env.local` переопределят значения из `.env.defaults`:
  ```yaml
  env_file:
    - .env.defaults
    - .env.local
  ```
- Файл `.env` (без суффикса) автоматически загружается Compose для подстановки в `docker-compose.yml`, но не передаётся в контейнеры, если не указан в `env_file`.
