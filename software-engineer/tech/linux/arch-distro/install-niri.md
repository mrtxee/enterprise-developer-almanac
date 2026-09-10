---
aliases:
  - install-niri
  - niri
---
## Установка и настройка Niri на Arch Linux с GNOME + GDM

**Niri** — это скроллящийся тайловый Wayland-композитор (не полноценное DE). Он может работать параллельно с GNOME, и вы сможете выбирать сессию при входе через GDM.

### Шаг 1: Установка Niri

Niri доступен в AUR. Установите его через ваш AUR-хелпер (например, `yay` или `paru`):

```bash
yay -S niri
```

Или соберите вручную:

```bash
git clone https://aur.archlinux.org/niri.git
cd niri
makepkg -si
```

### Шаг 2: Создание записи сессии для GDM

GDM читает файлы `.desktop` из директории `/usr/share/wayland-sessions/`. Нужно создать запись для Niri.

Создайте файл `/usr/share/wayland-sessions/niri.desktop`:

```bash
sudo nano /usr/share/wayland-sessions/niri.desktop
```

Добавьте следующее содержимое:

```ini
[Desktop Entry]
Name=Niri
Comment=Scrollable tiling Wayland compositor
Exec=niri-session
Type=Application
DesktopNames=niri
```

**Важно:** `niri-session` — это скрипт-обертка, который запускает Niri с правильной инициализацией. Если его нет в системе, создайте его:

```bash
sudo nano /usr/bin/niri-session
```

Содержимое скрипта:

```bash
#!/bin/sh
# Скрипт инициализации сессии Niri

# Экспортируем переменные окружения для Wayland
export XDG_SESSION_TYPE=wayland
export XDG_CURRENT_DESKTOP=niri
export XDG_SESSION_DESKTOP=niri

# Запускаем Niri
exec niri "$@"
```

Сделайте скрипт исполняемым:

```bash
sudo chmod +x /usr/bin/niri-session
```

### Шаг 3: Настройка конфигурации Niri

Создайте директорию для конфигурации и базовый файл:

```bash
mkdir -p ~/.config/niri
nano ~/.config/niri/config.kdl
```

Минимальная рабочая конфигурация:

```kdl
input {
    keyboard {
        xkb {
            layout "us,ru"
        }
    }
}

output "eDP-1" {
    // Настройки монитора (опционально)
}

window-rule {
    match app-id="firefox"
    open-maximized true
}

window-rule {
    match app-id="org.gnome.Nautilus"
    open-maximized true
}
```

### Шаг 4: Выбор сессии в GDM

1. Выйдите из текущей сессии GNOME (Logout)
2. На экране входа GDM нажмите на свое имя пользователя
3. **Перед вводом пароля** нажмите на иконку шестеренки (️) в правом нижнем углу
4. Выберите **"Niri"** из списка сессий
5. Введите пароль и войдите

Теперь вы можете переключаться между GNOME и Niri при каждом входе.

### Шаг 5: Интеграция с GNOME-сервисами (опционально)

Чтобы Niri мог использовать некоторые сервисы GNOME (например, уведомления, буфер обмена), добавьте в скрипт `niri-session`:

```bash
#!/bin/sh
export XDG_SESSION_TYPE=wayland
export XDG_CURRENT_DESKTOP=niri
export XDG_SESSION_DESKTOP=niri

# Запуск GNOME-сервисов (опционально)
/usr/lib/gnome-session-ctl --session &
/usr/lib/gnome-keyring-daemon --start --components=pkcs11,secrets,ssh &
/usr/lib/xdg-desktop-portal &

# Запускаем Niri
exec niri "$@"
```

### Шаг 6: Настройка горячих клавиш

Отредактируйте `~/.config/niri/config.kdl`, добавив горячие клавиши:

```kdl
input {
    keyboard {
        xkb {
            layout "us,ru"
        }
    }
}

binds {
    "Mod+Return" { spawn "kitty"; }
    "Mod+Shift+Q" { close-window; }
    "Mod+D" { spawn "wofi --show drun"; }
    "Mod+H" { focus-column-left; }
    "Mod+L" { focus-column-right; }
    "Mod+J" { focus-window-down; }
    "Mod+K" { focus-window-up; }
    "Mod+Shift+H" { move-column-left; }
    "Mod+Shift+L" { move-column-right; }
    "Mod+Shift+J" { move-window-down; }
    "Mod+Shift+K" { move-window-up; }
    "Mod+M" { maximize-column; }
    "Mod+Shift+M" { expand-column-to-available-width; }
    "Mod+T" { toggle-window-floating; }
}
```

### Шаг 7: Перезагрузка и тестирование

Перезагрузите компьютер или просто выйдите из сессии:

```bash
# Выход из текущей сессии
loginctl terminate-session $XDG_SESSION_ID
```

На экране GDM выберите **Niri** и войдите.

### Полезные команды

```bash
# Проверка версии Niri
niri --version

# Перезагрузка конфигурации без перезапуска
# (Niri автоматически подхватывает изменения config.kdl)

# Просмотр логов
journalctl --user -u niri

# Справка по конфигурации
niri validate
```

### Возможные проблемы

1. **Niri не появляется в списке сессий GDM**
   - Проверьте, что файл `/usr/share/wayland-sessions/niri.desktop` существует
   - Перезапустите GDM: `sudo systemctl restart gdm`

2. **Черный экран при входе**
   - Проверьте логи: `journalctl --user -b`
   - Убедитесь, что `niri-session` скрипт исполняемый

3. **Нет звука или уведомлений**
   - Добавьте запуск `pipewire` и `xdg-desktop-portal-gnome` в скрипт сессии

Теперь у вас есть две полноценные сессии: GNOME (традиционный DE) и Niri (тайловый композитор), между которыми можно переключаться при входе.
