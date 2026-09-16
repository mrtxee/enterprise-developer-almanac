---
aliases:
  - Container
  - Container Registry
  - Containerization
  - Docker
  - Docker Hub
  - Docker image
  - Docker pull
  - Docker push
  - Docker run
  - Docker-образ
  - Dockerfile
  - ENTRYPOINT
  - FROM
  - Guest OS
  - Host OS
  - Nginx
  - NGINX
  - RUN
  - Ubuntu
  - Virtual machine
  - Virtualization
  - Yandex Cloud
  - Yandex Container Registry
  - виртуализация
  - виртуальная машина
  - гостевая ОС
  - контейнер
  - контейнеризация
  - образ
  - репозиторий Docker-образов
  - хостовая ОС
---

## Виртуализация и контейнеризация

```mermaid
---
title: Сравнение контейнеризации и виртуализации
---
flowchart TB
 subgraph s1["Контейнеризация"]
    direction TB
        B1("Библиотеки")
        A1("Сервис 1")
        A2("Сервис 2")
        A3("Сервис 3")
        C1("Контейнеризация")
        D1("Хостовая ОС")
        E1("Инфраструктура")
  end
 subgraph s2["Виртуализация"]
    direction TB
        G1("Библиотеки")
        F1("Сервис 1")
        G2("Библиотеки")
        F2("Сервис 2")
        G3("Библиотеки")
        F3("Сервис 3")
        H1("Гостевая ОС")
        H2("Гостевая ОС")
        H3("Гостевая ОС")
        I1("Хостовая ОС")
        J1("Инфраструктура")
  end
    A1 --> B1
    A2 --> B1
    A3 --> B1
    B1 --> C1
    C1 --> D1
    D1 --> E1
    F1 --> G1
    F2 --> G2
    F3 --> G3
    G1 --> H1
    G2 --> H2
    G3 --> H3
    H1 --> I1
    H2 --> I1
    H3 --> I1
    I1 --> J1

     B1:::Peach
     A1:::Sky
     A2:::Sky
     A3:::Sky
     C1:::Pine
     D1:::Rose
     E1:::Aqua
     G1:::Peach
     F1:::Sky
     G2:::Peach
     F2:::Sky
     G3:::Peach
     F3:::Sky
     H1:::Ash
     H2:::Ash
     H3:::Ash
     I1:::Rose
     J1:::Aqua
    classDef Ash stroke-width:1px, stroke-dasharray:none, stroke:#999999, fill:#EEEEEE, color:#000000
    classDef Aqua stroke-width:1px, stroke-dasharray:none, stroke:#46EDC8, fill:#DEFFF8, color:#378E7A
    classDef Pine stroke-width:1px, stroke-dasharray:none, stroke:#254336, fill:#27654A, color:#FFFFFF
    classDef Sky stroke-width:1px, stroke-dasharray:none, stroke:#374D7C, fill:#E2EBFF, color:#374D7C
    classDef Rose stroke-width:1px, stroke-dasharray:none, stroke:#FF5978, fill:#FFDFE5, color:#8E2236
    classDef Peach stroke-width:1px, stroke-dasharray:none, stroke:#FBB35A, fill:#FFEFDB, color:#8F632D
```

Контейнеры выигрывают по сравнению с виртуальными машинами тем, что:

- не требуют развертывания полноценной ОС для среды приложения
- контейнеры изолированы
- контейнеры быстрее [[orchestration|оркестрировать]], чем виртуальные машины
- экономия ресурсов железа

## Docker

[[Docker|Docker]] работает так: приложение упаковывается со всеми зависимостями — библиотеками, интерпретаторами, файлами — в Docker-образ и отправляется в репозиторий (хранилище). Чтобы развернуть приложение, нужно скачать из репозитория образ и создать из него контейнер на рабочем сервере.

Хранилища Docker-образов бывают публичными и приватными. Самое известное публичное хранилище — это Docker Hub. В Yandex Cloud удобно использовать собственное хранилище облака — `Yandex Container Registry`.

### Создание образа

Пример `Dockerfile` для образа с ОС Ubuntu и веб-сервером NGINX:

```dockerfile
FROM ubuntu:latest
RUN apt-get update -y
RUN apt-get install -y nginx
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

Каждая инструкция создаёт новый слой образа, и эти слои накладываются друг на друга. В конце задаётся команда — исполняемый файл, который запускается при старте Docker-контейнера.

### Создание контейнера

Основные команды для работы с репозиторием:

```bash
# положить образ в репозиторий
docker push my-image
# взять образ из репозитория и запустить
docker pull my-image
docker run my-image
```

---

[[yandex-container-registry|Yandex Container Registry]]
