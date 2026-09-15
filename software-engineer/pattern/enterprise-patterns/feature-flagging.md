---
aliases:
  - A/B test
  - AWS AppConfig
  - Canary Release
  - Canary
  - Feature Flag
  - Feature Flagging
  - Feature Toggle
  - Feature Toggling
  - Flagsmith
  - LaunchDarkly
  - Split.io
  - Unleash
  - Канареечный релиз
  - Переключение функций
  - Управление функциями
  - Флаг функции
---

**Feature Flagging** и **[[feature-toggling]]** — это одно и то же, но с разницей в оттенке смысла.

На практике их часто используют как синонимы, но есть тонкое различие — особенно в контексте DevOps, CI/CD и масштабируемых систем.

## Краткий ответ

| Аспект | Feature Flagging | Feature Toggling |
| --- | --- | --- |
| Суть | Более широкий термин: стратегия управления функциями | Часто означает техническую реализацию «включения/выключения» |
| Фокус | Управление выпуском, тестирование, безопасность | Технический механизм «переключения» |
| Контекст | DevOps, CI/CD, продукт-менеджмент | Разработка, архитектура, программирование |
| Аналогия | Стратегия использования переключателя | Сам выключатель |

> Feature Flagging = Feature Toggling + стратегия, контроль, метрики, безопасность.

---

## Подробное сравнение

### Feature Toggling

**Определение**

> Feature Toggling — технический паттерн, при котором определённая функция в коде включается или выключается с помощью логического условия.

**Пример на Go:**

```go
if config.IsFeatureEnabled("new_checkout") {
    useNewCheckoutFlow()
} else {
    useOldCheckoutFlow()
}
```

**Характеристики:**

- простая реализация;
- управление через конфиг-файл или переменные окружения;
- может быть «зашит» в коде;
- часто используется для временного отключения багов;
- не всегда поддерживает A/B-тесты или таргетинг по пользователям.

> Toggling — это просто «вкл/выкл», без аналитики и безопасности.

---

### Feature Flagging

**Определение**

> Feature Flagging — практика управления жизненным циклом функций с помощью централизованной системы, где флаги можно:
> - включать/выключать в runtime;
> - таргетировать по пользователям, регионам, ролям;
> - проводить A/B-тесты;
> - делать canary-выпуски;
> - измерять влияние на бизнес-метрики.

**Пример: LaunchDarkly / Flagsmith / Unleash**

```go
user := lduser.NewUser("123")
flag := client.BoolVariation("new-checkout", user, false)

if flag {
    useNewCheckoutFlow()
} else {
    useOldCheckoutFlow()
}
```

Флаг `new-checkout` может быть:

- включён только для `role=admin`;
- включён для 5% пользователей (canary);
- включён только в `region=eu-west`;
- отключён при падении метрик ошибок > 1%.

---

## Сравнение Feature Flagging и Feature Toggling

| Критерий | Feature Toggling | Feature Flagging |
| --- | --- | --- |
| Где управляется | В коде, конфигах, `application.yml` | Централизованная система (LaunchDarkly, Flagsmith) |
| Время активации | При перезапуске сервиса | Мгновенно (в runtime) |
| Таргетинг | Ручной (`if role == admin`) | По пользователю, группе, версии, стране |
| A/B тесты | Нет | 50% видят новую форму |
| Canary Release | Ручная реализация | Готово — 1%, 5%, 10%, 100% |
| Мониторинг и метрики | Нет | Интеграция с Prometheus, Grafana, Amplitude |
| Безопасность | Легко сломать | Аудит, роли, доступ, rollback за секунды |
| Использование | Разработчики, DevOps | Product Manager, QA, DevOps |
| Примеры инструментов | `if`, `config.getProperty()` | LaunchDarkly, Flagsmith, Unleash, Split.io |
| Цель | Скрыть неготовый код | Управление рисками, тестирование, безопасный выпуск |

> Feature Flagging — это профессиональный подход. Feature Toggling — его простая версия.

---

## Когда использовать

| Сценарий | Рекомендация |
| --- | --- |
| Временно скрыть фичу | Feature Toggling (простой `if`) |
| MVP | Feature Toggling достаточно |
| 50+ фич, A/B-тесты, canary | Feature Flagging (инструмент) |
| Большая команда | Feature Flagging — чтобы не ломать друг друга |
| Нужен контроль и аудит | Feature Flagging |
| Отключить фичу при высокой нагрузке | Feature Flagging — автоматический fallback |
| Нет времени на внедрение | Feature Toggling — быстрее и проще |

---

## Пример: канареечный выпуск

**Без feature flagging:**

```yaml
# deployment.yaml
image: myapp:v2  # Все 100 pod'ов получают v2
```

Если v2 падает — падает всё.

**С feature flagging:**

```go
if flags.IsOn("new-checkout", user) {
    // v2 logic
} else {
    // v1 logic
}
```

В LaunchDarkly:

- 1% пользователей → новая корзина;
- если error rate > 1% → автоматический откат;
- можно включить только для `beta-testers`.

---

## Преимущества Feature Flagging

| Плюс | Объяснение |
| --- | --- |
| Безопасный выпуск | Запуск постепенно: 1% → 5% → 100% |
| A/B тестирование | Сравнение поведения двух версий |
| Немедленный откат | Одна кнопка — и фича выключена |
| Работа без деплоя | Менеджер включает фичу — разработчику ничего не нужно делать |
| Автоматизация | Связь с мониторингом: если p95 растёт — отключить фичу |
| Таргетинг | Только для сотрудников, только для Москвы, только для iOS 17 |

---

## Инструменты для Feature Flagging

| Инструмент | Особенности |
| --- | --- |
| LaunchDarkly | Коммерческий, мощный UI, SDK для всех языков |
| Flagsmith | Open Source, self-hosted, дешевле |
| Unleash | Open Source, легко интегрируется с Kubernetes |
| Split.io | Аналитика + флаги |
| AWS AppConfig | Managed-сервис AWS |
| Azure App Configuration | Microsoft |

---

## Лучшие практики

| Практика | Объяснение |
| --- | --- |
| Не оставляйте «мёртвые» флаги | Удаляйте флаг после того, как фича стабильна |
| Давайте понятные имена | `checkout-redesign-v2`, а не `feature-x` |
| Добавьте срок жизни флага | Например: `expires: 2025-06-30` |
| Интегрируйте с CI/CD | Автоматическое включение после успешного тестирования |
| Используйте в production | Не только для разработки |
| Не используйте для конфигурации | Флаги ≠ конфиги. Config Server для `db.url`, флаги — для поведения |

---

## Пример: Feature Flag в реальной системе

```json
{
  "key": "new-onboarding-flow",
  "enabled": true,
  "strategies": [
    {
      "name": "gradualRollout",
      "parameters": { "percentage": 10 }
    },
    {
      "name": "userWithId",
      "parameters": { "ids": ["admin-123"] }
    }
  ]
}
```

Эта фича:

- включена для 10% пользователей;
- всегда включена для `admin-123`;
- может быть изменена через UI без перезапуска.

---

## Финальный вывод: что выбирать

| Ваша цель | Решение |
| --- | --- |
| Скрыть фичу до готовности | Feature Toggling (простой `if`) |
| Управлять фичами извне | Feature Flagging (LaunchDarkly и др.) |
| Запустить canary | Feature Flagging |
| A/B тесты | Feature Flagging |
| Быстро отключить фичу | Feature Flagging |
| Маленькая команда | Toggling (дёшево и быстро) |
| Крупная компания | Flagging (без него — хаос) |

---

## Цитата от эксперта

> «Feature toggles are a code smell if you don’t have a system to manage them.»
> — Martin Fowler

> Если много флагов — нужна система управления ими. Иначе они станут техническим долгом.

---

## Заключение

| | Feature Toggling | Feature Flagging |
| --- | --- | --- |
| Что это? | Паттерн в коде | Платформа + стратегия |
| Как работает? | `if (flag)` | Через внешний сервис |
| Где используется? | Разработка | DevOps, продукт, релизы |
| Лучше для | MVP, временное решение | Production, большие команды |
| Вывод | Начало | Профи-подход |

> Feature Toggling — это инструмент. Feature Flagging — это культура.

---

- [Martin Fowler — Feature Toggle](https://martinfowler.com/bliki/FeatureToggle.html)
- [LaunchDarkly Docs](https://docs.launchdarkly.com/)
- [Unleash Open Source](https://unleash.github.io/)
- YouTube: «Feature Flags Explained» — TechWorld with Nana

---

Для микросервисов, CI/CD и DevOps Feature Flagging — следующий шаг: он превращает релиз из рискованного события в непрерывный процесс.
