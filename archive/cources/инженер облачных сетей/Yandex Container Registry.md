## Yandex Container Registry
Если вы работаете с Yandex.Cloud, лучше всего использовать сервис [Yandex Container Registry](https://cloud.yandex.ru/docs/container-registry/)
- **Реестр Yandex Container Registry** — это хранилище Docker-образов
- **Репозиторий Yandex Container Registry** — набор образов с одинаковыми именами (т. е. версий образа).
### Нейминг образов
* Запись для обращения к образу:
	* `cr.yandex/<реестр>/<имя образа>:<тег>`
	* Пример полного имени: `cr.yandex/my-registry/my-app:latest`.
	* можно использовать регулярные выражения `cr.yandex/my-registry/my-app:test.*`
### Автоматическое удаление
Политики автоматического удаления настраиваются для каждого репозитория отдельно. Пример rules.json
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
* Удаление образа — это ответственное действие. Поэтому после настройки правил проверьте, как они будут работать в автоматическом режиме. Вам поможет тестовый запуск политики: `dry-run`.