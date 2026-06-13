# todo


- [+] настроить видео плеер
	- [x] взять исходники, исправить кнопки, скомпилировать, установить
- [x] визуальное и консольные удаление пакетов
- [x] инструкция: установки кодеков
- [x] взлом жетбраинс
- [x] установка jdk
- [x] установка hdmn
- [ ] nearby share
- [x] включить гибернацию
- [x] настроить tiling

Extensions
- AppIndicator and KStatus …
- ArcMenu
	- Unity
- ClipboardIdicator
- Power off options
- Tilling Shell

 ---

Utils
https://wiki.archlinux.org/title/Core_utilities#Essentials

# faq

### gnome-conole
```
gtk-launch org.gnome.Console
pacman -Ql gnome-console | grep bin/
gnome-console /usr/bin/
gnome-console /usr/bin/kgx
```

### vpn

#### nb
ограничения asteriaray:
* не понятно подключился или нет
* нельзя закрывать приложение. Если закрыл то точно понадобится ребут.
#### details
`yay -S asteriaray-bin `
https://hide-my-name.cloud/faq/vpn/vpn-installation-and-configuration/third-party-applications/wireguard-for-linux/
* code: `266092057852074`
* protocol: `AmneziaWG`

AmsterdamS4.conf
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


### поиск AUR пакеторв
https://aur.archlinux.org/packages?O=0&SeB=nd&K=AmneziaWG&outdated=&SB=p&SO=d&PP=50&submit=Go
### video player
**celluloid**

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


### read stuff
```bash
# модная утилита
fastfetch

# Общая информация о дистрибутиве:
cat /etc/os-release

# Информация о ядре и архитектуре:
uname -a
uname -r # версия ядра;
uname -m # архитектура (x86_64 и т. д.).

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
#Узнать, используете ли вы Wayland или Xorg:
echo $XDG_SESSION_TYPE
gnome-shell --version
```


### tcmd
Double Commander
### sudo-nautilus
Просто вставьте этот адрес в адресную строку уже запущенного Nautilus:
```
admin:///
```
Вас один раз попросят ввести пароль, и вы получите полный доступ ко всей файловой системе в этом же окне. Никакого `sudo` не нужно. (На системах Arch с GNOME этот метод работает из коробки, так как в составе `gvfs` есть бэкенд для `admin`).

### office
```bash
sudo pacman -Syu libreoffice-fresh libreoffice-still libreoffice-fresh-ru
```

### codec-install

https://wiki.archlinux.org/title/Codecs_and_containers
* см. Tips and tricks

aom dav1d rav1e svt-av1 libde265 libdv libmpeg2 schroedinger libtheora libvpx x264 x265 xvidcore

mkvtoolnix-cli, mkvtoolnix-gui ogmtools

xine-lib libavcodec gst-libav

### установка VSCode

https://wiki.archlinux.org/title/Visual_Studio_Code
```bash
yay -S vscodium-bin
```
### установка GNOME Software
> Apps for GNOME

`sudo pacman -S gnome-software gnome-software-packagekit-plugin flatpak`

### Activities Overview как в Windows

see `gnome-shell-extensions \ ArcMenu`

## запустить archlinux в режиме консоли
запустить archlinux в режиме консоли, без gui
`ctrl+alt+f3`

## настройки gnome

### alt+shift
```bash
gsettings set org.gnome.desktop.wm.keybindings switch-input-source "['<Alt>Shift_L']"

напиши аналогичную команду для 
 - открытия gnome console на SUPER+R
 - открытия gnome files на SUPER+E
 - свернуть все окна на SUPER+D


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
### firefox не реагирует на alt

1. Запустите Firefox.
2. В адресной строке введите `about:config` и нажмите Enter.
3.  `ui.key.menuAccessKey` — обычно его значение равно `18` (код клавиши Alt). --> `0` (отключение клавиши).
4. `ui.key.menuAccessKeyFocuses` --> `false`
	* `ui.key.menuAccessKey = 0` — отключает клавишу‑активатор меню (в т. ч. Alt).
	* `ui.key.menuAccessKeyFocuses = false` — запрещает фокусировку на меню при нажатии клавиши‑активатора.
5. Перезапустите Firefox


# подбор графической оболочки

###### потребление память оболочками
* cosmic-* 520
* plasma-shell 380
* gnome-* 750
* cinnamon-* 540
###### ==
✅ ❌

###### cosmic
Хорошо. Но не позволяет полностью отказаться от gnome.
###### cinnamon
Приятненько работать. Все ок с расширениями.

###### cutefish
````bash
yay -S cutefish-meta
````
* из-за yay установка оболочки занимает дольше чем установка ОС
* и не установился по непонятной причине

###### budgie
*  ✅ 3 GB утилизация ОЗУ, 1% cpu, 400 MB distr
*  ✅ масштабируется на мониторы
*  ✅ подходит для работы
* ❌ alt+shift приводит к залипанию в firefox
* ❌ SUPER + search не работает без клика мышью по input
* ❌ не нашел решения, чтобы окно с консолью было в едином DM оформлении с выбранной темой
###### hyprland
*  ✅ щадящий расход ресурсов
* ❌ управление мониторами только через ручне конфиги
* ❌ все действия только через горячие клавиши, которые надо с нуля изобретать
* ❌ скорее всего будет неудобно в работе то что вся верстка полностью едет при открытии нового окна. Проще решать это вручную

```plain

```

###### xfce
```bash
sudo pacman -S xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
sudo systemctl enable lightdm
sudo systemctl start lightdm
sudo reboot
```

 - ✅ подходит для работы
 - ✅ 2 GB утилизация ОЗУ, 1% cpu, 180 MB distr
 - ❌ простоватый дизайн
 - ❌ не поддерживает type-c дисплеи из коробки


###### kde plasma 
 - ✅ удобнее для работы чем gnome
 - ❌ ручное управление многими мониторами
 - ❌ 4 GB утилизация ОЗУ
 - ❌ перегружено настройками
 - ❌ история буфера обмена не работает из коробки

###### gnome
1. apply-2
	* ok
2. apply-1
	 - ✅ чистый эстетичный дизайн
	 - ✅ 3 GB утилизация ОЗУ
	 - ❌ док приложений смахивает открытые окна
	 - ❌ не доступно atl+shift для переключения раскладки
	 - ❌ история буфера обмена не работает из коробки

# подбор приложений
```yaml
useful:
 - btop
 - gedit
needed:
 - idea
```

---

