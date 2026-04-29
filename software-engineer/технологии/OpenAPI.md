---
aliases:
  - OpenAPI
  - Swagger
---
## Синтаксис OpenAPI (Swagger): основы и структура

OpenAPI Specification (OAS) — стандарт описания REST‑API в формате **JSON** или **YAML**. Позволяет автоматически генерировать документацию, клиентский код и тесты.

### Версии спецификации

- **OpenAPI 3.0.x** — актуальная версия (рекомендуется).
- **Swagger 2.0** — устаревшая, но ещё встречается.

В примерах ниже — синтаксис **OpenAPI 3.0**.

### Базовая структура документа

```yaml
openapi: 3.0.0                   # версия спецификации
info:                          # метаданные API
  title: My API                # название
  version: 1.0.0               # версия API
  description: Описание API     # краткое описание
  contact:                     # контакты (опционально)
    name: API Support
    email: support@example.com
paths:                         # эндпоинты (пути)
  /users:                     # путь
    get:                     # HTTP-метод
      summary: Получить список пользователей
      responses:
        '200':
          description: Успешно
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
components:                    # повторно используемые компоненты
  schemas:                   # схемы данных
    UserList:
      type: array
      items:
        $ref: '#/components/schemas/User'
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
```

### Ключевые секции

1. **`openapi`**  
   Указывает версию спецификации.  
   ```yaml
   openapi: 3.0.0
   ```

2. **`info`**  
   Метаданные API: название, версия, описание, контакты.  
   ```yaml
   info:
     title: My API
     version: 1.0.0
     description: API для управления пользователями
   ```

3. **`paths`**  
   Описание эндпоинтов и методов (GET, POST и др.).  
   ```yaml
   paths:
     /users:
       get:
         summary: Получить список пользователей
         responses:
           '200':
             description: Успешно
   ```

4. **`components`**  
   Хранит повторно используемые схемы, параметры, ответы.  
   ```yaml
   components:
     schemas:
       User:
         type: object
         properties:
           id: { type: integer }
           name: { type: string }
   ```

5. **`servers`**  
   Базовые URL для API (например, для разных окружений).  
   ```yaml
   servers:
     - url: https://api.example.com/v1
       description: Основной сервер
   ```

### Описание эндпоинта (пример GET)

```yaml
paths:
  /users/{id}:
    get:
      summary: Получить пользователя по ID
      parameters:                  # параметры запроса
        - name: id
          in: path                # параметр в URL
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: Пользователь найден
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: Пользователь не найден
```

### Параметры запросов

- **`in`** — место параметра:  
  - `path` — в URL (`/users/{id}`),  
  - `query` — в строке запроса (`?page=1`),  
  - `header` — в заголовке,  
  - `cookie` — в cookie.
- **`required`** — обязателен ли параметр.
- **`schema`** — тип данных (string, integer, boolean и др.).

Пример:
```yaml
parameters:
  - name: page
    in: query
    required: false
    schema:
      type: integer
      default: 1
```

### Схемы данных (schemas)

Описывают структуру объектов в запросах/ответах.

```yaml
components:
  schemas:
    User:
      type: object
      required: [id, name]
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
          minLength: 1
        email:
          type: string
          format: email
```

### Ответы (responses)

Для каждого кода HTTP-статуса указывается:
- `description` — описание,
- `content` — формат и схема данных.

```yaml
responses:
  '201':
    description: Пользователь создан
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/User'
  '400':
    description: Ошибка валидации
```

### Безопасность (security)

Пример для Bearer‑токена:
```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

### Ссылки на компоненты (`$ref`)

Позволяют повторно использовать схемы, параметры и др.:
```yaml
schema:
  $ref: '#/components/schemas/User'
```

### Важные нюансы

1. **YAML vs JSON**  
   YAML удобнее для чтения, но JSON тоже поддерживается.

2. **Отступы в YAML**  
   Критичны! Используйте пробелы (не табуляции).

3. **Коды статусов**  
   Указывайте как строки (`'200'`, `'404'`).

4. **Валидация**  
   Проверяйте спецификацию через:  
   - [Swagger Editor](https://editor.swagger.io) (онлайн),  
   - утилиты `swagger-cli` или `openapi-validator`.

5. **Генерация документации**  
   Используйте Swagger UI или Redoc для интерактивной документации.

### Где применять

- Автогенерация документации (Swagger UI, Redoc).
- Создание клиентского кода (через Swagger Codegen).
- Тестирование API (Postman, Newman).
- Валидация контрактов API в CI/CD.