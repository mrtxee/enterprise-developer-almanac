---
aliases:
  - git
  - gitea
  - gitea-branch-protection
---

Скорее всего вы имеете в виду **Gitea**.

Формулировка «ветки только в группах `release`, `feature`, `bugfix`» обычно означает запрет на создание веток вне префиксов:

```text
release/...
feature/...
bugfix/...
```

То есть разрешены, например:

```text
release/1.2.0
feature/CITADEL-1234-add-button
bugfix/CITADEL-5678-fix-login
```

и запрещены, например:

```text
my-branch
test
hotfix/urgent
```

---

## Вариант 1. Если в вашей версии Gitea есть Push Rules / Rules

В разных версиях Gitea интерфейс может называться немного по-разному:

```text
Repository → Settings → Push Rules
```

или

```text
Repository → Settings → Rules
```

или

```text
Repository → Settings → Branches
```

Если там есть правило для имени ветки, используйте regex:

```regex
^(release|feature|bugfix)/.+$
```

Если нужно разрешить ещё основную ветку, например `main`, используйте:

```regex
^(main|(release|feature|bugfix)/.+)$
```

Если нужно разрешить и `main`, и `develop`:

```regex
^(main|develop|(release|feature|bugfix)/.+)$
```

Если поле принимает glob-паттерны, а не regex, то разрешённые группы описываются так:

```text
release/*
feature/*
bugfix/*
```

Но обычно одним правилом это описывают именно regex-ом.

---

## Вариант 2. Через Branch Protection в Gitea

В Gitea есть:

```text
Repository → Settings → Branches → Add branch protection
```

Там можно защитить ветки по маскам:

```text
release/*
feature/*
bugfix/*
```

Например:

| Protected branch | Что настроить |
|---|---|
| `release/*` | разрешить push нужной группе/команде |
| `feature/*` | разрешить push нужной группе/команде |
| `bugfix/*` | разрешить push нужной группе/команде |

Но важный момент:

> Branch Protection в Gitea защищает совпадающие ветки, но не всегда надёжно решает задачу «запретить создание всех остальных веток».

То есть если вы просто защитите `release/*`, `feature/*`, `bugfix/*`, это не гарантирует, что пользователь не создаст ветку `some-branch`.

Для полноценного запрета «создавать можно только такие группы веток» лучше использовать серверный Git-hook.

---

## Вариант 3. Надёжный способ: pre-receive hook

Если в вашем Gitea доступны Git Hooks:

```text
Repository → Settings → Git Hooks → pre-receive
```

добавьте скрипт:

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
2. Разрешает ветки:

   ```text
   release/*
   feature/*
   bugfix/*
   ```

3. Дополнительно разрешает, при необходимости:

   ```text
   main
   master
   develop
   ```

4. Запрещает создание любой другой ветки.

---

## Если нужно разрешить только создание, но не обновление других веток

Если вам нужно именно:

> запрещать только создание новых веток вне групп `release`, `feature`, `bugfix`

то в hook нужно оставить проверку только для создания ветки:

```sh
if [ "$oldrev" != "$ZERO" ]; then
  continue
fi
```

Тогда проверка будет срабатывать только когда `oldrev` равен нулю, то есть ветка создаётся впервые.

Но на практике обычно лучше проверять все push-операции в ветку, чтобы нельзя было случайно или специально работать с ветками неправильного формата.

---

## Если нужно запретить и удаление веток

В примере выше удаление веток разрешено.

Если нужно запретить удаление любых веток, уберите этот блок:

```sh
if [ "$newrev" = "$ZERO" ]; then
  continue
fi
```

Тогда при попытке удалить ветку hook тоже выполнит проверку имени.

---

## Итоговая рекомендация

Если в вашей версии Gitea есть **Push Rules / Rules**, используйте regex:

```regex
^(release|feature|bugfix)/.+$
```

или с разрешённым `main`:

```regex
^(main|(release|feature|bugfix)/.+)$
```

Если такого пункта нет, самый надёжный способ — серверный `pre-receive` hook.

Пример разрешённых веток:

```text
release/1.0.0
release/2024.12
feature/CITADEL-1234-add-report
feature/add-new-api
bugfix/CITADEL-5678-fix-export
bugfix/null-pointer-fix
```

Пример запрещённых веток:

```text
my-branch
test
fix-login
hotfix/urgent
release-1.0.0
feature_add_report
```
