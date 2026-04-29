---
aliases:
  - ycloud
  - yc
  - Yandex Cloud CLI
---
# [[DevOps]] автоматизация

# Yandex Cloud CLI — аналог putty для ycloud

Cегодня одни из самых нагруженных сервисов — это поисковики, именно эти компании стали «гуру» в сфере DevOps. Так, Google собрал ключевые знания в книге SRE Book (SRE — Site Reliability Engineering).

см. [https://yandex.cloud/ru/docs/cli/quickstart](https://yandex.cloud/ru/docs/cli/quickstart)

## yc команды

Типовые сервисы

- `yc resource-manager …` — управление облаками и каталогами;
- `yc compute …` — управление ВМ;
- `yc load-balancer …` — управление балансировщиками нагрузки.

Управление кластерами баз данных:

- `yc managed-mysql …` — MySQL;
- `yc managed-postgresql …` — PostgreSQL;
- `yc managed-clickhouse …` — ClickHouse.

Есть ещё небольшая группа служебных команд:

- `yc init` — первоначальная настройка CLI;
- `yc version` — показывает версию CLI;
- `yc help` — выводит описание всех команд или справку о команде.

[https://practicum.yandex.ru/trainer/ycloud/lesson/3ced8e83-a7c0-497a-82d1-02aab9ed43f4/](https://practicum.yandex.ru/trainer/ycloud/lesson/3ced8e83-a7c0-497a-82d1-02aab9ed43f4/)
