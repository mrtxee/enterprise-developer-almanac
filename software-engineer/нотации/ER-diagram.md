---
aliases:
  - Entity-Relationship diagram
  - erd
  - ER-diagram
  - ER diagram
  - Диаграмма связей
---
**ER-диаграмма (Entity-Relationship diagram)** — это визуальное представление структуры базы данных, которое показывает сущности, их атрибуты и связи между ними. Она используется для проектирования и документирования структуры данных.

**Основные компоненты ER-диаграммы**
1. **Сущности (Entities)** — прямоугольники
	- Представляют таблицы в БД
	- Пример: `Пользователь`, `Заказ`, `Товар`
2. **Атрибуты (Attributes)** — овалы
	- Поля таблиц
	- Пример: `id`, `имя`, `email`, `дата_создания`
 3. **Связи (Relationships)** — ромбы
	- Отношения между сущностями    
	- Пример: "один-ко-многим", "многие-ко-многим"
4. **Ключи**
	- **Первичный ключ (PK)** — уникальный идентификатор
	- **Внешний ключ (FK)** — ссылка на первичный ключ другой таблицы

```mermaid
---
title: Диаграмма связей базы фильмов
config:
  layout: elk
  look: handDrawn
  theme: forest
  elk:
    mergeEdges: true
    nodePlacementStrategy: LINEAR_SEGMENTS

---
erDiagram 
    CINEMA ||--o{ THEATER : "Один кинотеатр → много залов"
    THEATER ||--o{ SHOWING : "Один зал → много сеансов"
    SHOWING }o--|| MOVIE : "Много сеансов → один фильм"

    CINEMA {
        int Cinema_ID PK
        string Cinema_Name
        string Cinema_Address
        string Cinema_Phone
    }

    THEATER {
        int Theater_ID PK
        int Theater_Capacity
        int Cinema_ID FK
    }

    SHOWING {
        int Showing_Time PK
        int Showing_Date PK
        int Theater_ID PK, FK
        int Movie_ID PK, FK
        int Showing_Attendance
    }

    MOVIE {
        int Movie_ID PK, FK
        string Movie_Name
        string Movie_Director
        string Movie_Rating
    }
```
