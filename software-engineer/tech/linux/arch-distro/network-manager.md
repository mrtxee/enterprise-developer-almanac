---
aliases:
  - linux-network-manager
  - network-manager
---

Ошибка "NetworkManager not running" в Arch Linux и GNOME, как правило, возникает по двум причинам: сервис не запущен или пакет не установлен вовсе. Это связано с тем, что NetworkManager не входит в группу пакетов `gnome` по умолчанию .

Вот пошаговое руководство, как это исправить.

## 1️⃣ Проверьте состояние и установку

Сначала нужно выяснить, установлен ли NetworkManager и запущен ли он. Откройте терминал и выполните команду:

```bash
systemctl status NetworkManager
```

* **Если статус `Active: inactive (dead)`**: Сервис установлен, но не запущен. Переходите к шагу 2.
* **Если статус `Unit NetworkManager.service not found`**: Пакет не установлен. Переходите к шагу 3.

## 2️⃣ Если установлен, но не запущен

В этом случае вам нужно просто включить и запустить сервис:

```bash
# Включить автозапуск
sudo systemctl enable NetworkManager

# Запустить сейчас
sudo systemctl start NetworkManager
```

После этого проверьте статус командой `systemctl status NetworkManager` — он должен быть `active (running)` .

## 3️⃣ Если не установлен

Установите NetworkManager через `pacman`:

```bash
sudo pacman -S networkmanager
```

Затем включите и запустите сервис, как описано в шаге 2 .

## 4️⃣ Дополнительно: возможные конфликты

GNOME по умолчанию тесно интегрирован с NetworkManager . Если у вас одновременно установлены и запущены другие менеджеры сети (например, `iwd` или `dhcpcd`), они могут конфликтовать с NetworkManager и мешать его корректной работе .

Убедитесь, что другие сервисы, управляющие сетью, отключены. Проверить их список можно командой:

```bash
systemctl list-units | grep -E "dhcpcd|iwd|systemd-networkd"
```

Если они есть и активны, остановите и отключите их:

```bash
sudo systemctl stop dhcpcd
sudo systemctl disable dhcpcd
```

После разрешения конфликтов перезапустите NetworkManager:

```bash
sudo systemctl restart NetworkManager
```

После этих шагов сетевой менеджер должен запуститься, а GNOME должен корректно отображать его состояние в настройках.
