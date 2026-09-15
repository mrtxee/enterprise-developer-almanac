---
aliases:
  - AmneziaWG
  - Double Commander
  - VSCodium
  - arch-faq
  - celluloid
  - cutefish
  - fastfetch
  - Вопросы по Arch Linux
  - Настройки GNOME
  - Подбор графической оболочки
  - Установка кодеков
  - Частые вопросы
---
## Частые вопросы (FAQ)

### Удаление ненужных GNOME-приложений (del-crap)

```bash
sudo pacman -Rns gnome-tour malcontent-control showtime
```

### GNOME Console

```bash
gtk-launch org.gnome.Console
pacman -Ql gnome-console | grep bin/
gnome-console /usr/bin/
gnome-console /usr/bin/kgx
```

### VPN

#### Заметки

Ограничения AmneziaWG:

- не понятно, подключился или нет;
- нельзя закрывать приложение. Если закрыл — то точно понадобится ребут.

#### Детали

Установка:

```bash
yay -S asteriaray-bin
```

- code: `266092057852074`
- protocol: `AmneziaWG`

Конфиг AmsterdamS4.conf:

```ini
# AmsterdamS4 conf
[Interface]
Address = 10.95.174.93/32
PrivateKey = KB7qelx9kmVV4G0Vi7eWbKLEi+APW6D8fy9Yfxu+ZWE=
DNS = 1.1.1.1
Jc = 3
Jmin = 50
Jmax = 100
S1 = 16
S2 = 32
H1 = 114613668
H2 = 1859823361
H3 = 1618676693
H4 = 939078689

[Peer]
PublicKey = Z/lGYSTlnaEYMLu1Cx6+bIA0it719p+2N8CXeEpmWVw=
AllowedIPs = 0.0.0.0/0
Endpoint = 78.31.250.57:45497
PersistentKeepalive = 22
```

### Поиск AUR-пакетов

### Видеоплеер

**celluloid**

Конфиг input.conf:

```properties
# input.conf:
# Переключение треков
7 playlist_prev
9 playlist_next
# Управление громкостью
8 add volume +5
5 add volume -5
2 add volume -1
# Навигация по текущему треку
6 seek +15
4 seek -15
3 seek +3
1 seek -3
# misc
0 cycle pause
. cycle fullscreen
```

### Чтение информации о системе

```bash
# модная утилита
fastfetch

# Общая информация о дистрибутиве:
cat /etc/os-release

# Информация о ядре и архитектуре:
uname -a
uname -r # версия ядра;
uname -m # архитектура (x86_64 и т.д.).

# Информация о процессоре:
lscpu
# Оперативная память:
free -h
# Диски и разделы:
lsblk
# Более подробно:
fdisk -l
# Информация об оборудовании (подробно):
sudo lshw -short
# Версия BIOS/UEFI и сведения о материнской плате:
sudo dmidecode -t system
# Графический драйвер и видеокарта:
# lspci | grep -i vga
# Сеть:
ip addr
hostname -I
# Время работы системы и нагрузка:
uptime

# Проверить текущий масштаб
gsettings get org.gnome.desktop.interface scaling-factor
# проверить статус дробного масштабирования:
gsettings get org.gnome.mutter experimental-features
# Узнать, используете ли вы Wayland или Xorg:
echo $XDG_SESSION_TYPE
gnome-shell --version
```

### Двухпанельный файловый менеджер (tcmd)

Double Commander.

### Nautilus с правами администратора (sudo-nautilus)

Просто вставьте этот адрес в адресную строку уже запущенного Nautilus:

```text
admin:///
```

Вас один раз попросят ввести пароль, и вы получите полный доступ ко всей файловой системе в этом же окне. Никакого `sudo` не нужно. (На системах Arch с GNOME этот метод работает из коробки, так как в составе `gvfs` есть бэкенд для `admin`).

### Офисные пакеты

```bash
sudo pacman -Syu libreoffice-fresh libreoffice-still libreoffice-fresh-ru
```

### Установка кодеков

Рекомендации Arch Wiki: https://wiki.archlinux.org/title/Codecs_and_containers

- см. Tips and tricks.

Кодеки:

```text
aom dav1d rav1e svt-av1 libde265 libdv libmpeg2 schroedinger libtheora libvpx x264 x265 xvidcore
```

Инструменты для видео:

```text
mkvtoolnix-cli, mkvtoolnix-gui ogmtools
```

Библиотеки декодирования:

```text
xine-lib libavcodec gst-libav
```

### Установка VSCodium

Документация Arch Wiki: https://wiki.archlinux.org/title/Visual_Studio_Code

```bash
yay -S vscodium-bin
```

### Установка GNOME Software

Установка из официального репозитория:

```bash
sudo pacman -S gnome-software gnome-software-packagekit-plugin flatpak
```

### Activities Overview как в Windows

См. `gnome-shell-extensions \ ArcMenu`.

## Запуск Arch Linux в режиме консоли

Запуск без GUI:

```text
ctrl+alt+f3
```

## Настройки GNOME

### Переключение раскладки (Alt+Shift)

```bash
gsettings set org.gnome.desktop.wm.keybindings switch-input-source "['<Alt>Shift_L']"
```

Открытие GNOME Console на SUPER+R, GNOME Files на SUPER+E, сворачивание всех окон на SUPER+D:

```bash
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/']"
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ name "Open Terminal"
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ command "gnome-console kgx"
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ binding "<Super>r"
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom1/']"
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom1/ name "Open Files"
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom1/ command "nautilus"
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom1/ binding "<Super>e"
gsettings set org.gnome.desktop.wm.keybindings show-desktop "['<Super>d']"
```

### Firefox не реагирует на Alt

1. Запустите Firefox.
2. В адресной строке введите `about:config` и нажмите Enter.
3. `ui.key.menuAccessKey` — обычно его значение равно `18` (код клавиши Alt) → `0` (отключение клавиши).
4. `ui.key.menuAccessKeyFocuses` → `false`
   - `ui.key.menuAccessKey = 0` — отключает клавишу-активатор меню (в т.ч. Alt).
   - `ui.key.menuAccessKeyFocuses = false` — запрещает фокусировку на меню при нажатии клавиши-активатора.
5. Перезапустите Firefox.

## Подбор графической оболочки

### Потребление памяти оболочками

- cosmic-* — 520 МБ
- plasma-shell — 380 МБ
- gnome-* — 750 МБ
- cinnamon-* — 540 МБ

### Cosmic

Хорошо. Но не позволяет полностью отказаться от GNOME.

### Cinnamon

Приятно работать. Все ок с расширениями.

### Cutefish

```bash
yay -S cutefish-meta
```

- из-за yay установка оболочки занимает дольше, чем установка ОС;
- не установился по непонятной причине.

### Budgie

Плюсы:

- 3 GB утилизация ОЗУ, 1% CPU, 400 MB distr;
- масштабируется на мониторы;
- подходит для работы.

Минусы:

- alt+shift приводит к залипанию в Firefox;
- SUPER + search не работает без клика мышью по input;
- не нашёл решения, чтобы окно с консолью было в едином DM оформлении с выбранной темой.

### Hyprland

Плюсы:

- щадящий расход ресурсов.

Минусы:

- управление мониторами только через ручные конфиги;
- все действия только через горячие клавиши, которые надо с нуля изобретать;
- скорее всего будет неудобно в работе то, что вся вёрстка полностью едет при открытии нового окна. Проще решать это вручную.

### Xfce

```bash
sudo pacman -S xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
sudo systemctl enable lightdm
sudo systemctl start lightdm
sudo reboot
```

Плюсы:

- подходит для работы;
- 2 GB утилизация ОЗУ, 1% CPU, 180 MB distr.

Минусы:

- простоватый дизайн;
- не поддерживает type-c дисплеи из коробки.

### KDE Plasma

Плюсы:

- удобнее для работы, чем GNOME.

Минусы:

- ручное управление многими мониторами;
- 4 GB утилизация ОЗУ;
- перегружено настройками;
- история буфера обмена не работает из коробки.

### GNOME

Плюсы:

- чистый эстетичный дизайн;
- 3 GB утилизация ОЗУ.

Минусы:

- док приложений смахивает открытые окна;
- не доступен alt+shift для переключения раскладки;
- история буфера обмена не работает из коробки.

## Подбор приложений

**Полезные приложения:**

- btop
- gedit

**Нужные:**

- IntelliJ IDEA
