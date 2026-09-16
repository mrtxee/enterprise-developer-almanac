---
aliases:
  - Arch Linux
  - arch-maintance
  - paccache
  - paccache.timer
  - pacman-contrib
  - paru
  - systemd timer
  - systemd-таймер
  - topgrade
  - yay
  - Автоматическое обновление Arch Linux
  - Обслуживание Arch Linux
---

---

## Обслуживание Arch Linux

В сообществе Arch Linux нет единого «официального» скрипта, который делал бы всё сразу, но есть множество проверенных пользовательских решений. Чаще всего используют системный таймер `systemd` для вызова обновлений и утилиту `paccache` для автоматической очистки кэша.

Есть два популярных подхода, которые легко адаптировать под свои нужды.

## Подход 1: готовый таймер paccache

Самый простой способ автоматически чистить кэш пакетов — таймер из состава `pacman-contrib`. Утилита автоматически удаляет старые версии пакетов из `/var/cache/pacman/pkg/`, оставляя несколько последних (по умолчанию — 3). Это предотвращает бесконтрольный рост папки с кэшем.

**Включение таймера**

1. Убедитесь, что установлен пакет `pacman-contrib`: `sudo [[pacman]] -S pacman-contrib`.
2. Включите таймер: `sudo systemctl enable --now paccache.timer`.
3. Проверьте статус: `sudo systemctl status paccache.timer`.

paccache.timer — стандартный инструмент, поддерживаемый разработчиками Arch. Он выполняет только очистку кэша, но не обновляет систему.

## Подход 2: собственный скрипт с systemd-таймером

Более гибкий способ, позволяющий автоматизировать не только очистку, но и само обновление: комбинация `[[pacman]] -Syu` и `paccache` внутри одного таймера.

**Скрипт обновления (`/usr/local/bin/auto-update.sh`):**

```bash
#!/usr/bin/env bash

# Уведомление о начале (опционально)
notify-send "Arch Linux" "Начинается автоматическое обновление..."

# Обновление базы и пакетов
pacman -Syu --noconfirm

# Очистка кэша (оставить 2 последние версии)
paccache -rvk2

# Удаление orphan-пакетов (опционально)
pacman -Rns $(pacman -Qtdq) 2>/dev/null

notify-send "Arch Linux" "Обновление завершено!"
```

**Таймер systemd (`/etc/systemd/system/auto-update.timer`):**

```ini
[Unit]
Description=Еженедельное обновление Arch Linux

[Timer]
OnCalendar=weekly
Persistent=true

[Install]
WantedBy=multi-user.target
```

**Сервис systemd (`/etc/systemd/system/auto-update.service`):**

```ini
[Unit]
Description=Скрипт автоматического обновления Arch Linux

[Service]
Type=oneshot
ExecStart=/usr/local/bin/auto-update.sh
```

**Установка**

1. Создайте файлы скрипта, таймера и сервиса с указанным содержимым.
2. Сделайте скрипт исполняемым: `chmod +x /usr/local/bin/auto-update.sh`.
3. Включите таймер: `sudo systemctl enable --now auto-update.timer`.

Подход полностью прозрачен, легко кастомизируется (email-уведомления, логирование в файл, уведомления на рабочий стол) и использует стандартные механизмы systemd, а не сторонние cron-скрипты.

## Другие утилиты

- **`topgrade`** — обновляет не только пакеты Arch, но и сторонние менеджеры ([[pacman-yay-faltpak|Flatpak]], Snap, Pip, Rust и др.). Умеет запускаться по таймеру и выводить уведомления об успешном завершении. Устанавливается через [[aur|AUR]] (`yay -S topgrade`).
- **`yay` / `paru`** — AUR-помощники со встроенными флагами для автоматического обновления, например `yay -Syu --noconfirm`. Их можно использовать в пользовательских скриптах.

## Итог

Основной рецепт от сообщества Arch Linux — комбинация `paccache.timer` для очистки кэша и собственного скрипта с `systemd`-таймером для полного цикла обслуживания. Такой подход даёт полный контроль и использует только стандартные инструменты системы.
