---
aliases:
  - niri
  - niri compositor
  - Композитор niri
---

## Установка niri

**niri** — это новый **Wayland-композитор** с нерегулярной, скроллируемой компоновкой окон (scrollable tiling). Устанавливается из исходников и управляется через сервис niri.service.

## Базовый конфиг

Порядок установки: зависимости → компоненты проекта → запуск сервиса.

```mermaid
---
title: Порядок установки niri
---
flowchart LR
    Under["Зависимости (cargo, ninja)"] --> Repo["Сборка проекта"]
    Repo --> Conf["Конфигурация"]
    Conf --> Sys["Сервис niri.service"]
    Sys --> Run["Старт композитора"]
```

- **Зависимости:** `cargo`, `ninja`, `wayland`, `meson`, `cairo`, `pango` и др.
- **Сборка:** клонирование репозитория `YaLTeR/niri` и сборка через `cargo`.
- **Конфиг:** `~/.config/niri/config.toml`.
- **Сервис:** `systemctl --user enable --now niri.service`.

## Сохранение конфига

1. Создать каталог конфигурации:

```bash
mkdir -p ~/.config/niri
```

2. Сохранить файл конфигурации `config.toml` в `~/.config/niri/config.toml`.

## Первичный запуск

Запустить сервис в пользовательской сессии systemd:

```bash
systemctl --user start niri.service
```

## Создание рабочего пространства

```bash
systemctl --user start niri-workspace.service
```

## Перезапуск сервиса

```bash
systemctl --user restart niri.service
```

## Проверка работы

```bash
systemctl --user status niri.service
```

В выводе должно быть `Active: active (running)`.

## Проверка оборудования

Информация о графике:

```bash
sudo dmesg | grep -i amdgpu
```

Контроль разрешения:

- Включить EDID-проверку от гарнитуры/монитора при сбоях.
- При ошибках разрешения — проверить драйвер (amdgpu / nouveau) в `dmesg`.

## Типовые действия

| Действие | Команда |
|----------|---------|
| Старт композитора | `systemctl --user start niri.service` |
| Полный запуск сессии | `systemctl --user start niri-workspace.service` |
| Перезапуск | `systemctl --user restart niri.service` |
| Статус | `systemctl --user status niri.service` |
| Автостарт | `systemctl --user enable niri.service` |

## Возможные проблемы

| Проблема | Решение |
|----------|---------|
| Нет графики/чёрный экран | Проверить драйвер GPU и `dmesg`, включить подходящий параметр ядра |
| Сбито разрешение | Проверить EDID монитора, задать явное разрешение в конфиге |
| Сервис не активен | Проверить `journalctl --user -u niri.service` |
| Отсутствует workspace service | Запустить `niri-workspace.service` отдельно |

## Итог

- Конфиг — в `~/.config/niri/config.toml`.
- Запуск — через пользовательские сервисы systemd.
- Проблемы графики решаются проверкой драйвера и логов `dmesg`/`journalctl`.
