---
aliases:
  - "*.iml"
  - "*.ipr"
  - "*.iws"
  - idea
  - iml
  - ipr
  - iws
---
Файлы `*.iml`, `*.ipr` и `*.iws` — это **конфигурационные файлы проекта IntelliJ IDEA** (и других IDE JetBrains). [intellij-support.jetbrains](https://intellij-support.jetbrains.com/hc/en-us/articles/206544839-How-to-manage-projects-under-Version-Control-Systems)

## Расшифровка

- **`.iml`** — **IntelliJ Module** (файл модуля). [intellij-support.jetbrains](https://intellij-support.jetbrains.com/hc/en-us/articles/206544839-How-to-manage-projects-under-Version-Control-Systems)
- **`.ipr`** — **IntelliJ Project** (файл проекта, старый формат). [intellij-support.jetbrains](https://intellij-support.jetbrains.com/hc/en-us/articles/206544839-How-to-manage-projects-under-Version-Control-Systems)
- **`.iws`** — **IntelliJ Workspace** (файл рабочей области, старый формат). [intellij-support.jetbrains](https://intellij-support.jetbrains.com/hc/en-us/articles/206544839-How-to-manage-projects-under-Version-Control-Systems)

## Назначение

- **`.iml`** описывает **один модуль**: его исходные директории, зависимости, уровень языка, настройки сборки и т. п.. [intellij-support.jetbrains](https://intellij-support.jetbrains.com/hc/en-us/articles/206544839-How-to-manage-projects-under-Version-Control-Systems)
- **`.ipr`** (устаревший, file‑based формат) хранил **настройки всего проекта**: список модулей, компилятор, SDK, пути и прочее в одном XML‑файле. [intellij-support.jetbrains](https://intellij-support.jetbrains.com/hc/en-us/articles/206544839-How-to-manage-projects-under-Version-Control-Systems)
- **`.iws`** (тоже устаревший) содержал **персональные настройки рабочей области**: открытые вкладки, расположение окон, история и т. п., специфичные для конкретного пользователя/машины. [intellij-support.jetbrains](https://intellij-support.jetbrains.com/hc/en-us/articles/206544839-How-to-manage-projects-under-Version-Control-Systems)

## Обобщающее понятие

Их можно назвать **IDE‑специфичными конфигурационными файлами проекта** или **файлами метаданных проекта IntelliJ IDEA**. [intellij-support.jetbrains](https://intellij-support.jetbrains.com/hc/en-us/articles/206544839-How-to-manage-projects-under-Version-Control-Systems)

## Современный статус

Сейчас JetBrains рекомендует **directory‑based формат**: настройки проекта хранятся в папке `.idea/`, а для модулей остаются `.iml` файлы. [jetbrains](https://www.jetbrains.com/help/idea/creating-and-managing-projects.html)
Формат `.ipr`/`.iws` считается **устаревшим (legacy)** и не рекомендуется для новых проектов; `.iws` обычно не коммитят в VCS, так как он содержит пользовательские настройки. [intellij-support.jetbrains](https://intellij-support.jetbrains.com/hc/en-us/articles/206544839-How-to-manage-projects-under-Version-Control-Systems)
