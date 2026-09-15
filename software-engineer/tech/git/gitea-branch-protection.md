---
aliases:
  - branch protection
  - Branch Protection
  - Git Hooks
  - Gitea
  - gitea-branch-protection
  - pre-receive hook
  - Push Rules
  - защита веток
---

## Ограничение имён веток в Gitea

Задача — разрешить создание веток только с префиксами `release`, `feature`, `bugfix` и запретить остальные.

Разрешённые префиксы:

```text
release/...
feature/...
bugfix/...
```

Разрешённые примеры веток:

```text
release/1.2.0
feature/CITADEL-1234-add-button
bugfix/CITADEL-5678-fix-login
```

Запрещённые примеры веток:

```text
my-branch
test
hotfix/urgent
```

---

## Вариант с Push Rules или Rules

В разных версиях Gitea интерфейс называется по-разному:

```text
Repository → Settings → Push Rules
```

```text
Repository → Settings → Rules
```

```text
Repository → Settings → Branches
```

Если там есть правило для имени ветки, используется regex:

```regex
^(release|feature|bugfix)/.+$
```

Если нужно разрешить ещё основную ветку `main`:

```regex
^(main|(release|feature|bugfix)/.+)$
```

Если нужно разрешить и `main`, и `develop`:

```regex
^(main|develop|(release|feature|bugfix)/.+)$
```

Если поле принимает glob-паттерны, а не regex, разрешённые группы описываются так:

```text
release/*
feature/*
bugfix/*
```

Обычно одним правилом это описывают именно regex-ом.

---

## Вариант с Branch Protection

```text
Repository → Settings → Branches → Add branch protection
```

Там можно защитить ветки по маскам `release/*`, `feature/*`, `bugfix/*`:

| Protected branch | Что настроить |
| --- | --- |
| `release/*` | разрешить push нужной группе/команде |
| `feature/*` | разрешить push нужной группе/команде |
| `bugfix/*` | разрешить push нужной группе/команде |

Branch Protection в Gitea защищает совпадающие ветки, но не всегда надёжно решает задачу «запретить создание всех остальных веток». Если защитить только `release/*`, `feature/*`, `bugfix/*`, это не гарантирует, что пользователь не создаст ветку `some-branch`. Для полноценного запрета «создавать можно только такие группы веток» лучше использовать серверный Git-hook.

---

## Надёжный способ: pre-receive hook

Для настройки Git Hooks:

```text
Repository → Settings → Git Hooks → pre-receive
```

Добавьте скрипт:

```sh
#!/bin/sh

ZERO="0000000000000000000000000000000000000000"

while read -r oldrev newrev refname; do
  case "$refname" in
    refs/heads/*)
      branch="${refname#refs/heads/}"

      # Удаление ветки разрешаем.
      # Если нужно запретить удаление, уберите этот if.
      if [ "$newrev" = "$ZERO" ]; then
        continue
      fi

      # Если нужно проверять ТОЛЬКО создание ветки,
      # раскомментируйте строку ниже.
      # Тогда обновление уже существующих веток проверяться не будет.
      # if [ "$oldrev" != "$ZERO" ]; then continue; fi

      case "$branch" in
        release/*|feature/*|bugfix/*)
          ;;
        main|master|develop)
          # Разрешаем служебные ветки.
          # Если они не нужны, удалите эту строку.
          ;;
        *)
          echo "Ошибка: запрещено создавать ветку '$branch'." >&2
          echo "Разрешены только префиксы: release/, feature/, bugfix/." >&2
          exit 1
          ;;
      esac
      ;;
    *)
      # Теги и другие refs пропускаем.
      ;;
  esac
done

exit 0
```

Этот hook:

1. Проверяет только ветки, то есть `refs/heads/*`.
2. Разрешает ветки `release/*`, `feature/*`, `bugfix/*`.
3. Дополнительно разрешает, при необходимости, `main`, `master`, `develop`.
4. Запрещает создание любой другой ветки.

---

## Разрешение только создания, но не обновления веток

Чтобы запретить только создание новых веток вне групп `release`, `feature`, `bugfix`, в hook нужно оставить проверку только для создания ветки:

```sh
if [ "$oldrev" != "$ZERO" ]; then
  continue
fi
```

Тогда проверка срабатывает только когда `oldrev` равен нулю, то есть ветка создаётся впервые.

На практике обычно лучше проверять все push-операции в ветку, чтобы нельзя было случайно или специально работать с ветками неправильного формата.

---

## Запрет удаления веток

В примере выше удаление веток разрешено. Чтобы запретить удаление любых веток, уберите этот блок:

```sh
if [ "$newrev" = "$ZERO" ]; then
  continue
fi
```

Тогда при попытке удалить ветку hook тоже выполнит проверку имени.

---

## Итоговая рекомендация

Если в версии Gitea есть Push Rules / Rules, используйте regex:

```regex
^(release|feature|bugfix)/.+$
```

или с разрешённым `main`:

```regex
^(main|(release|feature|bugfix)/.+)$
```

Если такого пункта нет, самый надёжный способ — серверный pre-receive hook.

Примеры разрешённых веток:

```text
release/1.0.0
release/2024.12
feature/CITADEL-1234-add-report
feature/add-new-api
bugfix/CITADEL-5678-fix-export
bugfix/null-pointer-fix
```

Примеры запрещённых веток:

```text
my-branch
test
fix-login
hotfix/urgent
release-1.0.0
feature_add_report
```
