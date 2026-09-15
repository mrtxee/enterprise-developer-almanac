---
aliases:
  - Buildah
  - CNI
  - Container Network Interface
  - containerd
  - crun
  - daemonless
  - Docker
  - Docker BuildKit
  - Docker Compose
  - Docker Scout
  - Docker Swarm
  - Docker vs Podman
  - dockerd
  - OCI
  - Open Container Initiative
  - Podman
  - Podman Desktop
  - Podman Machine
  - podman-compose
  - Rootless
  - runc
  - Skopeo
  - Докер
---

## Docker vs Podman: сравнение

### Краткий ответ

> **Docker** — зрелая платформа с центральным демоном, индустриальный стандарт.
> **Podman** — daemonless альтернатива от Red Hat с акцентом на безопасность (rootless).
> **Выбор зависит от требований безопасности и инфраструктуры.**

---

## Сравнительная таблица

| Критерий                    | **Docker**                        | **Podman**                                  |
| --------------------------- | --------------------------------- | ------------------------------------------- |
| **Архитектура**             | Client-Server (демон `dockerd`)   | Daemonless (прямой запуск)                  |
| **Root-контейнеры**         | ✅ По умолчанию                    | ✅ Поддерживается                            |
| **Rootless-контейнеры**     | ⚠️ Ограниченная поддержка         | ✅ Native (первоклассная)                    |
| **Systemd интеграция**      | ⚠️ Через external tools           | ✅ Native (`podman generate systemd`)        |
| **Docker Socket**           | ✅ `/var/run/docker.sock`          | ⚠️ Через `podman-docker` wrapper            |
| **Docker Compose**          | ✅ Native                          | ✅ Через `podman-compose` / `docker-compose` |
| **Kubernetes**              | ⚠️ Через CRI (containerd)         | ✅ Native (`podman play kube`)               |
| **Registry аутентификация** | `~/.docker/config.json`           | `/etc/containers/registries.conf`           |
| **Storage drivers**         | overlay2, aufs, btrfs, zfs        | overlay, vfs, btrfs, zfs                    |
| **Network drivers**         | bridge, host, macvlan, overlay    | CNI (Container Network Interface)           |
| **Лицензия**                | Apache 2.0 + Docker EE commercial | Apache 2.0 (полностью open-source)          |
| **Вендор**                  | Docker Inc.                       | Red Hat / IBM                               |

---

## Архитектурные различия

### Архитектура Docker

```mermaid
---
title: Архитектура Docker: клиент, демон и runtime
---
graph TD
    CLI[Docker Client CLI: docker run nginx] -->|REST API| Daemon[Docker Daemon dockerd: root-привилегии]
    Daemon --> Containerd[containerd runtime]
    Daemon --> Net[Network Namespace]
```

**Проблемы:**

- Единая точка отказа (демон)
- Требует root-прав
- Security risk: взлом демона = root на хосте

### Архитектура Podman

```mermaid
---
title: Архитектура Podman: без демона, прямой запуск через OCI runtime
---
graph TD
    CLI[Podman CLI: podman run nginx] -->|fork/exec| Runtime[OCI Runtime runc/crun]
    Runtime --> Proc[Container Process]
    Runtime --> Net[Network CNI]
```

**Преимущества:**

- Нет демона (daemonless)
- Rootless по умолчанию
- Интеграция с systemd
- Лучшая безопасность

---

## Безопасность: Rootless контейнеры

### Rootless режим в Docker

```bash
# Поддерживается, но с ограничениями
$ dockerd-rootless-setuptool.sh install

# Ограничения:
# - Нет поддержки cgroups v1
# - Ограниченная сетевая функциональность
# - Не все storage drivers работают
# - Сложная настройка
```

### Rootless режим в Podman

```bash
# Работает из коробки
$ podman run -d nginx

# Полный функционал: сеть (CNI), volumes, все storage drivers, простая настройка

# Проверка:
$ podman info | grep -i rootless
rootless: true
```

### Сравнение безопасности

| Аспект               | Docker                          | Podman                                |
| -------------------- | ------------------------------- | ------------------------------------- |
| **Root по умолчанию**| ✅ Да                           | ❌ Нет (rootless)                     |
| **Взлом контейнера** | 🔴 Root на хосте                | 🟢 Ограниченные права                 |
| **Namespaces**       | ✅ Да                           | ✅ Да + user namespaces              |
| **Capabilities**     | ⚠️ По умолчанию много           | ✅ Minimal по умолчанию              |
| **SELinux/AppArmor** | ✅ Поддержка                    | ✅ Поддержка + усиленная              |
| **seccomp**          | ✅ Да                           | ✅ Да + строгие профили              |

---

## CLI-совместимость

### Podman как drop-in replacement

```bash
# Podman эмулирует Docker CLI
$ alias docker=podman

# Те же команды:
$ docker run -d --name web -p 80:80 nginx
$ podman run -d --name web -p 80:80 nginx

# Docker Compose:
$ docker-compose up
$ podman-compose up  # или docker-compose с podman socket

# Некоторые различия:
$ docker build -t myapp .
$ podman build -t myapp .  # работает

$ docker system prune
$ podman system prune  # работает
```

### Команды, специфичные для Podman

```bash
# Pods (группы контейнеров)
$ podman pod create --name mypod
$ podman run --pod mypod -d nginx
$ podman run --pod mypod -d redis

# Интеграция с Kubernetes
$ podman generate kube mypod > pod.yaml
$ podman play kube pod.yaml

# Интеграция с systemd
$ podman generate systemd --new --name mycontainer
$ systemctl --user start libpod-mycontainer.service
```

---

## Работа с образами

### Docker

```bash
# Build
$ docker build -t myapp:latest .

# Push/Pull
$ docker push docker.io/myuser/myapp:latest
$ docker pull docker.io/library/nginx:latest

# Registry auth
$ docker login registry.example.com
# Credentials: ~/.docker/config.json
```

### Podman

```bash
# Build (совместим)
$ podman build -t myapp:latest .

# Push/Pull (совместим)
$ podman push myapp:latest docker.io/myuser/myapp:latest
$ podman pull docker.io/library/nginx:latest

# Registry config
$ cat /etc/containers/registries.conf
unqualified-search-registries = ["docker.io", "quay.io"]

# Auth (совместим)
$ podman login registry.example.com
# Credentials: ~/.config/containers/auth.json
```

---

## Сеть

### Сеть в Docker

```bash
# Создать сеть
$ docker network create mynet

# Подключить контейнер
$ docker run -d --network mynet nginx

# Drivers: bridge, host, macvlan, overlay
$ docker network create -d macvlan --subnet=192.168.1.0/24 mymacvlan
```

### Сеть в Podman (CNI)

```bash
# Создать сеть (CNI)
$ podman network create mynet

# Подключить контейнер
$ podman run -d --network mynet nginx

# Plugins: bridge, macvlan, ipvlan
$ podman network create -d macvlan --subnet=192.168.1.0/24 mymacvlan

# Rootless networking
$ podman run -d -p 8080:80 nginx  # port forwarding через slirp4netns
```

---

## Производительность

### Замеры запуска контейнеров

| Операция             | Docker     | Podman     | Разница                          |
| -------------------- | ---------- | ---------- | -------------------------------- |
| **Start container**  | ~200ms     | ~180ms     | **Podman на 10% быстрее**        |
| **Stop container**   | ~150ms     | ~140ms     | **Podman на 7% быстрее**         |
| **Image pull**       | ~5s        | ~5s        | ≈ одинаково                      |
| **Build image**      | ~30s       | ~32s       | Docker на 6% быстрее             |
| **Memory overhead**  | ~50MB (daemon) | ~0MB    | **Podman экономит память**       |

### Использование ресурсов

```bash
# Docker — демон потребляет память
$ ps aux | grep dockerd
root  1234  0.5  1.2  50MB  dockerd

# Podman — нет демона
$ ps aux | grep podman
# (только процессы контейнеров)
```

---

## Миграция с Docker на Podman

### Установка Podman

```bash
# Ubuntu/Debian
$ sudo apt-get install podman podman-docker

# RHEL/CentOS
$ sudo dnf install podman

# macOS
$ brew install podman
$ podman machine init
$ podman machine start
```

### Замена Docker CLI

```bash
# Вариант 1: Alias
$ alias docker=podman

# Вариант 2: Podman wrapper
$ sudo dnf install podman-docker  # создаёт symlink

# Вариант 3: Docker socket compatibility
$ podman system service --time=0 unix:///var/run/docker.sock &
```

### Миграция образов

```bash
# Экспорт из Docker
$ docker save myapp:latest | gzip > myapp.tar.gz

# Импорт в Podman
$ gunzip -c myapp.tar.gz | podman load

# Или через registry
$ docker push myapp:latest
$ podman pull myapp:latest
```

### Миграция volumes

```bash
# Docker volumes
$ docker volume create mydata
$ docker run -v mydata:/data nginx

# Podman volumes (совместимы)
$ podman volume create mydata
$ podman run -v mydata:/data nginx

# Shared volumes
$ podman volume inspect mydata
```

### Docker Compose

```yaml
# docker-compose.yml (работает без изменений)
version: '3.8'
services:
  web:
    image: nginx
    ports:
      - "80:80"
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: secret
```

```bash
# Запуск через podman-compose
$ pip install podman-compose
$ podman-compose up -d

# Или через docker-compose с podman socket
$ export DOCKER_HOST=unix:///run/user/$UID/podman/podman.sock
$ docker-compose up -d
```

---

## Сценарии использования

### Выбирайте Docker, если

| Сценарий                        | Почему Docker                                  |
| ------------------------------- | ---------------------------------------------- |
| **Разработка на macOS/Windows** | Docker Desktop — лучшая интеграция             |
| **CI/CD с Docker-in-Docker**    | Зрелая поддержка DinD                          |
| **Enterprise с Docker EE**      | Commercial support, Docker Swarm               |
| **Команда уже знает Docker**    | Меньше обучения                                |
| **Нужен Docker Swarm**          | Native orchestration                           |
| **Legacy приложения**           | Стабильность и совместимость                   |

### Выбирайте Podman, если

| Сценарий                       | Почему Podman                                   |
| ------------------------------ | ----------------------------------------------- |
| **Безопасность критична**      | Rootless по умолчанию                           |
| **RHEL/CentOS/Fedora**         | Native интеграция                               |
| **Kubernetes-first подход**    | Native `play kube`                              |
| **Systemd интеграция**         | `generate systemd`                              |
| **No daemon архитектура**      | Меньше overhead                                 |
| **Compliance (PCI-DSS, HIPAA)**| Лучшая security модель                          |
| **Edge/IoT устройства**        | Меньше ресурсов                                 |

---

## Пример: production-деплой

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    restart: unless-stopped
    networks:
      - appnet

  db:
    image: postgres:14
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - appnet

volumes:
  pgdata:

networks:
  appnet:
```

### Podman + Systemd (production)

```bash
# 1. Создать pod
$ podman pod create --name myapp -p 8080:80

# 2. Запустить контейнеры в pod
$ podman run -d --pod myapp --name web nginx:alpine
$ podman run -d --pod myapp --name db \
    -e POSTGRES_DB=myapp \
    -e POSTGRES_PASSWORD=secret \
    postgres:14

# 3. Сгенерировать systemd unit
$ mkdir -p ~/.config/systemd/user
$ podman generate systemd --new --name myapp > ~/.config/systemd/user/myapp.service

# 4. Включить автозапуск
$ systemctl --user daemon-reload
$ systemctl --user enable --now myapp.service
$ loginctl enable-linger $USER  # запуск после reboot
```

---

## Ограничения

### Docker

| Ограничение          | Описание                                   |
| -------------------- | ------------------------------------------ |
| **Root required**    | По умолчанию требует root                  |
| **Daemon SPOF**      | Падение демона = все контейнеры down       |
| **Resource overhead**| Демон потребляет память/CPU                |
| **Security**         | Взлом демона = root на хосте               |

### Podman

| Ограничение              | Описание                                        |
| ------------------------ | ----------------------------------------------- |
| **Docker-in-Docker**     | Ограниченная поддержка                          |
| **Docker Desktop**       | Нет (но есть Podman Machine)                    |
| **Windows**              | Ограниченная поддержка                          |
| **Docker Swarm**         | Не поддерживается                               |
| **Maturity**             | Меньше production кейсов                        |

---

## Экосистема

### Docker

- **Docker Hub** — крупнейший registry
- **Docker Desktop** — macOS/Windows GUI
- **Docker Swarm** — оркестрация
- **Docker Compose** — multiple containers
- **Docker BuildKit** — продвинутая сборка
- **Docker Scout** — сканирование безопасности

### Podman

- **Quay.io** — registry от Red Hat
- **Podman Desktop** — GUI
- **Buildah** — сборка образов
- **Skopeo** — просмотр/копирование образов
- **Crictl** — совместимость с CRI
- **Podman Machine** — VM для macOS/Windows

---

## Decision Matrix

```mermaid
---
title: Decision Matrix: выбор между Docker и Podman
---
flowchart TD
    A{Нужен rootless-режим?} -->|Да| B[Podman: native rootless]
    A -->|Нет| C{Подходит Docker Desktop?}
    C -->|Да| D[Docker Desktop: macOS/Windows]
    C -->|Нет| E[Podman: Linux, безопасность]
```

---

## Памятка

**Docker**

- Индустриальный стандарт
- Зрелая экосистема
- Docker Desktop для разработки
- Требует root (по умолчанию)
- Демон = single point of failure

**Podman**

- Daemonless-архитектура
- Rootless по умолчанию
- Нативная интеграция с Kubernetes
- Интеграция с systemd
- Меньше зрелости для некоторых use cases

---

## Итог

| Вопрос                                        | Рекомендация                               |
| --------------------------------------------- | ------------------------------------------ |
| **Что для production Linux?**                 | ✅ Podman (безопасность)                  |
| **Что для разработки (macOS/Windows)?**       | ✅ Docker Desktop                         |
| **Что для Kubernetes?**                       | ✅ Podman                                 |
| **Что если команда знает Docker?**            | ⚠️ Docker или постепенная миграция       |
| **Что если compliance критичен?**             | ✅ Podman (rootless)                      |
| **Можно ли использовать оба?**                | ✅ Да (совместимы)                        |

---

## Бонус: гибридный подход

```bash
# Development: Docker Desktop
$ docker build -t myapp .
$ docker push myapp:latest

# Production: Podman (rootless)
$ podman pull myapp:latest
$ podman run -d --name myapp myapp:latest

# CI/CD: Podman (без демона)
$ podman build -t myapp .
$ podman push myapp:latest
```

Гибридный подход: Docker для разработки, Podman для production и CI/CD. Оба инструмента совместимы, поэтому миграция сводится к замене CLI.
