---
aliases:
  - Podman
  - Docker
---
## 🐳 Docker vs Podman — Полное сравнение

### 📌 Краткий ответ

> **Docker** — зрелая платформа с центральным демоном, индустриальный стандарт.  
> **Podman** — daemonless альтернатива от Red Hat с акцентом на безопасность (rootless).  
> **Выбор зависит от требований безопасности и инфраструктуры.**

---

## 📊 Сравнительная таблица

| Критерий | **Docker** | **Podman** |
|----------|------------|------------|
| **Архитектура** | Client-Server (демон `dockerd`) | Daemonless (прямой запуск) |
| **Root-контейнеры** | ✅ По умолчанию | ✅ Поддерживается |
| **Rootless-контейнеры** | ⚠️ Ограниченная поддержка | ✅ Native (первоклассная) |
| **Systemd интеграция** | ⚠️ Через external tools | ✅ Native (`podman generate systemd`) |
| **Docker Socket** | ✅ `/var/run/docker.sock` | ⚠️ Через `podman-docker` wrapper |
| **Docker Compose** | ✅ Native | ✅ Через `podman-compose` / `docker-compose` |
| **Kubernetes** | ⚠️ Через CRI (containerd) | ✅ Native (`podman play kube`) |
| **Registry аутентификация** | `~/.docker/config.json` | `/etc/containers/registries.conf` |
| **Storage drivers** | overlay2, aufs, btrfs, zfs | overlay, vfs, btrfs, zfs |
| **Network drivers** | bridge, host, macvlan, overlay | CNI (Container Network Interface) |
| **Лицензия** | Apache 2.0 + Docker EE commercial | Apache 2.0 (полностью open-source) |
| **Вендор** | Docker Inc. | Red Hat / IBM |

---

## 🏗️ Архитектурные различия

### Docker Architecture

```
┌─────────────────────────────────────────────────────┐
│  Docker Client (CLI)                                │
│  $ docker run nginx                                 │
└────────────────────────────────────────────────────┘
                 │ REST API
                 ▼
┌─────────────────────────────────────────────────────┐
│  Docker Daemon (dockerd) — ROOT PRIVILEGES          │
│  • Управление контейнерами                           │
│  • Сетевое конфигурирование                          │
│  • Хранение образов                                  │
│  • Логирование                                       │
└────────────────────────────────────────────────────┘
                 │
        ┌────────┴────────┐
        ▼                 ▼
┌───────────────┐   ┌───────────────┐
│  containerd   │   │    Network    │
│  (runtime)    │   │    Namespace  │
└───────────────┘   └───────────────┘
```

**Проблемы:**
- ❌ Единая точка отказа (демон)
- ❌ Требует root-прав
- ❌ Security risk (взлом демона = root на хосте)

---

### Podman Architecture

```
┌─────────────────────────────────────────────────────┐
│  Podman CLI                                         │
│  $ podman run nginx                                 │
└────────────────────────────────────────────────────┘
                 │ fork/exec (прямой вызов)
                 ▼
┌─────────────────────────────────────────────────────┐
│  OCI Runtime (runc/crun)                            │
│  • Запуск контейнера как обычного процесса           │
│  • Без центрального демона                           │
└────────────────────────────────────────────────────┘
                 │
        ┌────────┴────────
        ▼                 ▼
┌───────────────┐   ┌───────────────┐
│  Container    │   │  Network (CNI)│
│  Process      │   │               │
└───────────────┘   └───────────────┘
```

**Преимущества:**
- ✅ Нет демона (daemonless)
- ✅ Rootless по умолчанию
- ✅ Интеграция с systemd
- ✅ Лучшая безопасность

---

## 🔐 Безопасность: Rootless контейнеры

### Docker Rootless Mode

```bash
# ✅ Поддерживается, но с ограничениями
$ dockerd-rootless-setuptool.sh install

# ⚠️ Ограничения:
# - Нет поддержки cgroups v1
# - Ограниченная сетевая функциональность
# - Не все storage drivers работают
# - Сложная настройка
```

### Podman Rootless Mode

```bash
# ✅ Работает из коробки
$ podman run -d nginx

# ✅ Полный функционал:
# - Сеть (CNI)
# - Volumes
# - Все storage drivers
# - Простая настройка

# ✅ Проверка:
$ podman info | grep -i rootless
rootless: true
```

### Сравнение безопасности

| Аспект | Docker | Podman |
|--------|--------|--------|
| **Root по умолчанию** | ✅ Да | ❌ Нет (rootless) |
| **Взлом контейнера** | 🔴 Root на хосте | 🟢 Ограниченные права |
| **Namespaces** | ✅ Да | ✅ Да + user namespaces |
| **Capabilities** | ⚠️ По умолчанию много | ✅ Minimal по умолчанию |
| **SELinux/AppArmor** | ✅ Поддержка | ✅ Поддержка + усиленная |
| **seccomp** | ✅ Да | ✅ Да + строгие профили |

---

## 🛠️ CLI Совместимость

### Podman как drop-in replacement

```bash
# ✅ Podman эмулирует Docker CLI
alias docker=podman

# ✅ Те же команды:
$ docker run -d --name web -p 80:80 nginx
$ podman run -d --name web -p 80:80 nginx

# ✅ Docker Compose:
$ docker-compose up
$ podman-compose up  # или docker-compose с podman socket

# ⚠️ Некоторые различия:
$ docker build -t myapp .
$ podman build -t myapp .  # работает

$ docker system prune
$ podman system prune  # работает
```

### Podman-specific команды

```bash
# 🔹 Pods (группы контейнеров)
$ podman pod create --name mypod
$ podman run --pod mypod -d nginx
$ podman run --pod mypod -d redis

# 🔹 Kubernetes интеграция
$ podman generate kube mypod > pod.yaml
$ podman play kube pod.yaml

# 🔹 Systemd интеграция
$ podman generate systemd --new --name mycontainer
$ systemctl --user start libpod-mycontainer.service
```

---

## 📦 Работа с образами

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

## 🌐 Сеть

### Docker Networking

```bash
# Создать сеть
$ docker network create mynet

# Подключить контейнер
$ docker run -d --network mynet nginx

# Drivers: bridge, host, macvlan, overlay
$ docker network create -d macvlan --subnet=192.168.1.0/24 mymacvlan
```

### Podman Networking (CNI)

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

## 📊 Производительность

### Benchmarks (запуск контейнера)

| Операция | Docker | Podman | Разница |
|----------|--------|--------|---------|
| **Start container** | ~200ms | ~180ms | **Podman на 10% быстрее** |
| **Stop container** | ~150ms | ~140ms | **Podman на 7% быстрее** |
| **Image pull** | ~5s | ~5s | ≈ одинаково |
| **Build image** | ~30s | ~32s | Docker на 6% быстрее |
| **Memory overhead** | ~50MB (daemon) | ~0MB | **Podman экономит память** |

### Resource usage

```bash
# Docker — демон потребляет память
$ ps aux | grep dockerd
root  1234  0.5  1.2  50MB  dockerd

# Podman — нет демона
$ ps aux | grep podman
# (только процессы контейнеров)
```

---

## 🔄 Миграция с Docker на Podman

### Шаг 1: Установка Podman

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

### Шаг 2: Замена Docker CLI

```bash
# Вариант 1: Alias
$ alias docker=podman

# Вариант 2: Podman wrapper
$ sudo dnf install podman-docker  # создаёт symlink

# Вариант 3: Docker socket compatibility
$ podman system service --time=0 unix:///var/run/docker.sock &
```

### Шаг 3: Миграция образов

```bash
# Экспорт из Docker
$ docker save myapp:latest | gzip > myapp.tar.gz

# Импорт в Podman
$ gunzip -c myapp.tar.gz | podman load

# Или через registry
$ docker push myapp:latest
$ podman pull myapp:latest
```

### Шаг 4: Миграция volumes

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

### Шаг 5: Docker Compose

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

## 🎯 Use Cases: Когда что выбрать

### ✅ Выбирайте **Docker**, если:

| Сценарий | Почему Docker |
|----------|---------------|
| **Разработка на macOS/Windows** | Docker Desktop — лучшая интеграция |
| **CI/CD с Docker-in-Docker** | Зрелая поддержка DinD |
| **Enterprise с Docker EE** | Commercial support, Docker Swarm |
| **Команда уже знает Docker** | Меньше обучения |
| **Нужен Docker Swarm** | Native orchestration |
| **Legacy приложения** | Стабильность и совместимость |

### ✅ Выбирайте **Podman**, если:

| Сценарий | Почему Podman |
|----------|---------------|
| **Безопасность критична** | Rootless по умолчанию |
| **RHEL/CentOS/Fedora** | Native интеграция |
| **Kubernetes-first подход** | Native `play kube` |
| **Systemd интеграция** | `generate systemd` |
| **No daemon архитектура** | Меньше overhead |
| **Compliance (PCI-DSS, HIPAA)** | Лучшая security модель |
| **Edge/IoT устройства** | Меньше ресурсов |

---

## 🧪 Пример: Production deployment

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

## ⚠️ Известные ограничения

### Docker

| Ограничение | Описание |
|-------------|----------|
| **Root required** | По умолчанию требует root |
| **Daemon SPOF** | Падение демона = все контейнеры down |
| **Resource overhead** | Демон потребляет память/CPU |
| **Security** | Взлом демона = root на хосте |

### Podman

| Ограничение | Описание |
|-------------|----------|
| **Docker-in-Docker** | Ограниченная поддержка |
| **Docker Desktop** | Нет (но есть Podman Machine) |
| **Windows** | Ограниченная поддержка |
| **Docker Swarm** | Не поддерживается |
| **Maturity** | Меньше production кейсов |

---

## 📦 Экосистема

### Docker

- **Docker Hub** — largest registry
- **Docker Desktop** — macOS/Windows GUI
- **Docker Swarm** — orchestration
- **Docker Compose** — multi-container
- **Docker BuildKit** — advanced builds
- **Docker Scout** — security scanning

### Podman

- **Quay.io** — Red Hat registry
- **Podman Desktop** — GUI (в разработке)
- **Buildah** — image building
- **Skopeo** — image inspection/copy
- **Crictl** — CRI compatibility
- **Podman Machine** — macOS/Windows VM

---

## 📊 Decision Matrix

```
                    ┌─────────────────────────────────────────┐
                    │  Нужен ли rootless режим?                │
                    └─────────────────┬───────────────────────┘
                                      │
                    ┌─────────────────┴───────────────────────┐
                    │                                         │
                   ДА                                        НЕТ
                    │                                         │
                    ▼                                         ▼
    ┌───────────────────────────┐             ┌───────────────────────────┐
    │  Podman ✅                │             │  Docker или Podman        │
    │  (native rootless)        │             │  (оба подходят)           │
    └───────────────────────────┘             └──────────┬────────────────┘
                                                         │
                                            ┌────────────┴────────────┐
                                            │                         │
                                           ДА                        НЕТ
                                            │                         │
                                            ▼                         ▼
                            ┌───────────────────────┐   ┌───────────────────────┐
                            │  Docker Desktop       │   │  Podman               │
                            │  (macOS/Windows)      │   │  (Linux, security)    │
                            └───────────────────────┘   └───────────────────────┘
```

---

## 📌 Памятка

```
┌─────────────────────────────────────────────────────┐
│  Docker                                             │
│  ✅ Индустриальный стандарт                          │
│  ✅ Зрелая экосистема                                │
│  ✅ Docker Desktop для dev                           │
│  ⚠️ Требует root (по умолчанию)                     │
│  ⚠️ Daemon = single point of failure                │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  Podman                                             │
│  ✅ Daemonless архитектура                           │
│  ✅ Rootless по умолчанию                            │
│  ✅ Kubernetes-native                                │
│  ✅ Systemd интеграция                               │
│  ⚠️ Меньше mature для некоторых use cases           │
└─────────────────────────────────────────────────────┘
```

---

## 🚀 Итог

| Вопрос | Рекомендация |
|--------|--------------|
| **Что для production Linux?** | ✅ Podman (безопасность) |
| **Что для разработки (macOS/Windows)?** | ✅ Docker Desktop |
| **Что для Kubernetes?** | ✅ Podman |
| **Что если команда знает Docker?** | ⚠️ Docker или постепенная миграция |
| **Что если compliance критичен?** | ✅ Podman (rootless) |
| **Можно ли использовать оба?** | ✅ Да (совместимы) |

---

## 💡 Бонус: Hybrid подход

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

---

Если хочешь — могу показать:

- Как настроить **Podman в CI/CD** (GitHub Actions, GitLab CI)
- Как мигрировать **Docker Swarm → Kubernetes** с Podman
- Как настроить **rootless Podman в production**

Пишите — сделаю! 🚀