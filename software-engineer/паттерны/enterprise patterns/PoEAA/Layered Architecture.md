многослойная архитектура

>[!important] xxx
>Репозиторий ↔ `Entity` ↔ Сервисный слой ↔ `DTO` ↔ Контроллер  
>Репозиторий ↔  `Entity` ↔ Маппер ↔ DTO ↔ Сервисный слой ↔ DTO ↔ Контроллер  
>Model ↔ Controller ↔ View 

==MVC style==
Традиционно включает в себя:
- controller
- service
- dto – слой транспортных объектов
- mapper
- repository ~ model ~ entites 
Слои приложения в от верхнего.
## (0) **Интерфейсный слой**
**Интерфейсный слой —** UI Layer (Web Browser, JavaScript)
- может быть представвлен консолью ввода или Rest котроллером, любым иным клиентским интерфейсом
## [Слой аутентификации]
## (1) Контроллер
**Контроллер** — MVC Controller — **==controller==**
- Spring components annotated with `@Controller`
- получает команды от интерфейсного слоя и обращается к сервисному слою, бизнес-логике. Получает и передает **DTO**
## (2) Сервис
**Сервисный слой** — Service Layer — **==service==**
- , i.e. Spring components annotated with `@Service`
- слой бизнес логики
## [Маппер]
Маппер — ==**mapper**==
адаптер задач которого сопоставлять `Entity ⇔ DTO`
`org.mapstruct` — пакет для маппинга
## (3) Слой данных
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
            - [https://leodev.ru/blog/hibernate/аннотации-jpa-java-persistence-api/](https://leodev.ru/blog/hibernate/%D0%B0%D0%BD%D0%BD%D0%BE%D1%82%D0%B0%D1%86%D0%B8%D0%B8-jpa-java-persistence-api/)
Данные из репозитория поступают в сервисный слой в форме ==**DTO**==
DataTransferObject — **==DTO==**
Объект передачи данных. Настраивается для передачи между слоями или для передачи клиенту. Мы хотим контролировать какие данные получает клиент для безопасности и консистентности.
Для сопоставления (маппинга `Entity ⇔ DTO`) используется слой - маппер.