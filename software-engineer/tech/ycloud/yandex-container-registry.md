---
aliases:
  - Auto-deletion
  - Container Registry
  - Docker
  - Docker Image
  - Docker Registry
  - Docker образ
  - Docker-image
  - Dry Run
  - Expire Period
  - Image Name
  - Image Tag
  - Lifecycle Policy
  - Registry ID
  - Repository
  - Retained Top
  - Retention Policy
  - Tag Regexp
  - Untagged
  - Yandex Cloud
  - Yandex Container Registry
  - YC
  - автоматическое удаление
  - имя образа
  - имя тега
  - неотмеченные образы
  - образ Docker
  - политика жизни
  - политика удаления
  - реестр контейнеров
  - репозиторий контейнеров
  - тег
  - удержание последних
---

## Yandex Container Registry

Если вы работаете с Yandex.Cloud, лучше всего использовать сервис [Yandex Container Registry](https://cloud.yandex.ru/docs/container-registry/)

**Реестр** — хранилище Docker-образов; **Репозиторий** — набор образов с одинаковыми именами (т. е. версий образа).

### Нейминг образов

Запись для обращения к образу:
- `cr.yandex/<реестр>/<имя образа>:<тег>`
- Пример полного имени: `cr.yandex/my-registry/my-app:latest`.
- можно использовать регулярные выражения `cr.yandex/my-registry/my-app:test.*`

### Автоматическое удаление

Политики автоматического удаления настраиваются для каждого репозитория отдельно.

Пример rules.json:

```json
[
    {
     "description": "Delete prod Docker images older than 30 days but retain 20 last ones",
     "tag_regexp": "prod",
     "expire_period": "30d",
     "retained_top": 20
    },
    {
     "description": "delete all test Docker images except 10 last ones",
     "tag_regexp": "test.*",
     "retained_top": 10
    },
    {
     "description": "delete all untagged Docker images older than 48 hours",
     "untagged": true,
     "expire_period": "48h"
    }
]
```

Удаление образа — ответственное действие. Поэтому после настройки правил проверьте, как они будут работать в автоматическом режиме. Вам поможет тестовый запуск политики: `dry-run`.
