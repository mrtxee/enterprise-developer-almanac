# faq

Файл конфигурации композитора:
```bash
~/.config/cosmic/com.system76.CosmicComp/config.ron
```


## `win+v`
```bash
[a@al ~]$ pacman -Ss clipboard
extra/clipcat 0.25.0-1 # нет ui
    A clipboard manager
extra/cliphist 1:0.7.0-2 # нет ui
    wayland clipboard manager
extra/clipmenu 6.2.0-3 # нет ui
    Clipboard management using dmenu
extra/clipnotify 1.0.2-3 # не работает
    Polling-free clipboard notifier
extra/copyq 15.0.0-1 # не работает
    Clipboard manager with searchable and editable history
extra/deepin-clipboard 6.1.25-1 (deepin-extra)
    DDE clipboard manager component
extra/gpaste 45.4-1 # не работает
    Clipboard management system
extra/nwg-clipman 0.2.8-2 # не работает
    nwg-shell clipboard manager, a GTK3-based GUI for cliphist
extra/parcellite 1.2.5-3 # нет ui
    Lightweight GTK clipboard manager
extra/python-pyclip 0.7.0-9 # нет ui
    Cross-platform clipboard utilities supporting both binary and text data
extra/python-pyperclip 1.11.0-2
    A cross-platform clipboard module for Python
extra/texlive-latexextra 2026.1-1 (texlive)
    TeX Live - LaTeX additional packages
extra/wl-clip-persist 0.5.0-2
    Keep Wayland clipboard even after programs close
extra/wl-clipboard 1:2.3.0-1
    Command-line copy/paste utilities for Wayland
extra/xclip 0.13-6
    Command line interface to the X11 clipboard
extra/xfce4-clipman-plugin 1.7.0-1 (xfce4-goodies)
    A clipboard plugin for the Xfce4 panel
extra/xorg-xclipboard 1.1.5-1
    X clipboard manager
extra/yank 1.3.0-2
    Copy terminal output to **clipboard**
```

## `alt+shift` перключение раскладки

```bash
localectl set-x11-keymap us,ru pc104 , grp:alt_shift_toggle
```
аналогично это команде 
localectl set-x11-keymap us,ru pc104 , grp:alt_shift_toggle

напиши команду которые будут запускать:
cosmic terminal по нажатию на SUPER+R
cosmic file по нажатию на SUPER+E

localectl set-x11-keymap us,ru pc104 , grp:alt_shift_toggle

## установить вторым рядом с gnome
```bash
# Установка группы пакетов (если доступна в вашем зеркале)
sudo pacman -S cosmic

# Или вручную основные компоненты:
sudo pacman -S \
  cosmic-session \
  cosmic-files \
  cosmic-terminal \
  cosmic-text-editor \
  cosmic-wallpapers

cat /usr/share/wayland-sessions/cosmic.desktop

# 1. Установите greeter
sudo pacman -S cosmic-greeter

# 2. Отключите GDM, включите cosmic-greeter
sudo systemctl disable gdm.service
sudo systemctl enable cosmic-greeter.service

# 3. Перезагрузитесь
reboot
```