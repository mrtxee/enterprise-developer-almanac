---
aliases:
  - Controller
  - DTO
  - Data Access Layer
  - Data Transfer Object
  - Entity
  - Layered Architecture
  - MVC
  - Mapper
  - Model
  - Repository
  - Service Layer
  - Контроллер
  - Маппер
  - Многослойная архитектура
  - Объект передачи данных
  - Репозиторий
  - Сервисный слой
  - Слой данных
  - Сущность
---

# Layered Architecture

многослойная архитектура

> [!important] xxx
> Репозиторий ↔ `Entity` ↔ Сервисный слой ↔ `DTO` ↔ Контроллер
> Репозиторий ↔ `Entity` ↔ Маппер ↔ DTO ↔ Сервисный слой ↔ DTO ↔ Контроллер
> Model ↔ Controller ↔ View

==MVC style==

Традиционно включает в себя:

- controller
- service
- dto – слой транспортных объектов
- mapper
- repository ~ model ~ entites

Слои приложения в от верхнего.

## Интерфейсный слой

**Интерфейсный слой —** UI Layer (Web Browser, JavaScript)

- может быть представвлен консолью ввода или Rest котроллером, любым иным клиентским интерфейсом

## \[Слой аутентификации\]

## Контроллер

**Контроллер** — MVC Controller — **==controller==**

- Spring components annotated with `@Controller`
- получает команды от интерфейсного слоя и обращается к сервисному слою, бизнес-логике. Получает и передает **DTO**

## Сервис

**Сервисный слой** — Service Layer — **==service==**

- , i.e. Spring components annotated with `@Service`
- слой бизнес логики

## \[Маппер\]

Маппер — ==**mapper**==

адаптер задач которого сопоставлять `Entity ⇔ DTO`

`org.mapstruct` — пакет для маппинга

## Слой данных

**Слой данных** — Data Access Layer — ==**repository, mapper, model, dto**==

Слой взаимодействия с данными. При реализации может представлен репозиторием.

Репозиторий — класс который умеет общаться с хранилищем данных — **==repository==**

1. Spring components annotated with `@Repository`
2. По MVC-паттерну относится к **model**

Репозиторий возвращает сущности — `@Entity` — Сущность (по JPA)

- паттерн - ActiveRecord
- Spring components annotated with `@Entity`
  - `@Data`, `@Entity`, `@Table(name = "client")`
    - Entity — сущность в JPA — бизнес объект, хранимый в базе данных
      - По спецификации JPA все активрекорды должны быть замаркированы этой аннотацией

Данные из репозитория поступают в сервисный слой в форме ==**DTO**==

DataTransferObject — **==DTO==**

Объект передачи данных. Настраивается для передачи между слоями или для передачи клиенту. Мы хотим контролировать какие данные получает клиент для безопасности и консистентности.

Для сопоставления (маппинга `Entity ⇔ DTO`) используется слой - маппер.