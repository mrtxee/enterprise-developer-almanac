```bash
sudo pacman -S cliphist wofi wl-clipboard wtype
# resolving dependencies...
nano ~/.config/autostart/cliphist.service
nano ~/.local/clipmenu/clipmenu
chmod +x ~/.local/clipmenu/clipmenu
touch ~/.config/cliphist/config
nano ~/.config/cliphist/config
```


##### `~/.config/cliphist/config`
```bash
max-items 94
```
##### `~/.config/autostart/cliphist.service`
```bash
[Desktop Entry]
Type=Application
Name=Cliphist
Exec=bash -c "wl-paste --watch cliphist store"
X-GNOME-Autostart-enabled=true
```
##### `~/.local/clipmenu/clipmenu`
```bash
#!/usr/bin/env bash

# 1. Получаем полный список с ID (для внутреннего использования)
FULL_LIST=$(cliphist list)

# 2. Показываем в wofi ТОЛЬКО текст (без ID)
SELECTED_TEXT=$(echo "$FULL_LIST" | cut -f2- | wofi \
    --dmenu \
    --normal-window \
    --height	 480 \
    --width 400 \
    --insensitive \
    --prompt "clipboard search ")

# Если отмена → выход
[ -z "$SELECTED_TEXT" ] && exit 0

# 3. Находим соответствующий ID по тексту
#    (grep ищем строку, где после TAB идёт выбранный текст)
SELECTED_ID=$(echo "$FULL_LIST" | grep -P "\t\Q$SELECTED_TEXT\E$" | cut -f1)

# Если ID не найден → выход
[ -z "$SELECTED_ID" ] && exit 0

# 4. Декодируем по ID
cliphist decode "$SELECTED_ID" | wl-copy
sleep 0.1
wtype -M ctrl v -m ctrl

```

`super+v` –> `~/.local/clipmenu/clipmenu`

### версия 2

```bash
# ограничить историю до 30
systemctl --user edit --full cliphist.service
```