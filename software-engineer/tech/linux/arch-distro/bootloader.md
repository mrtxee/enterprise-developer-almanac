---
aliases:
  - boot-chains
  - bootloader
  - dracut
  - efibootmgr
  - GRUB
  - i801_smbus
  - initramfs
  - mkinitcpio
  - os-prober
  - plymouth
  - sbctl
  - Secure Boot
  - systemd-boot
  - systemd-stub
  - systemd-ukify
  - UEFI
  - UKI
  - unified kernel image
  - Единый образ ядра
  - Загрузка Arch Linux
  - Загрузчик
  - Первичный загрузчик
  - Файлы настройки загрузки
---
## Загрузка и загрузчики Arch Linux

### Сообщение i801_smbus при загрузке

Что означает сообщение при загрузке ОС Arch Linux и как его исправить:

```text
i801_smbus 0000:00:1f.4: SMBus is busy, can't use it!
```

### Где находится bootloader-logo

Логотип загрузчика:

```text
/usr/share/systemd/bootctl/splash-arch.bmp
```

**Обновление второстепенного конфига**

```bash
sudo nano /etc/mkinitcpio.d/linux.preset
sudo nano /etc/mkinitcpio.conf
sudo nano /etc/kernel/cmdline
# cat /proc/cmdline
sudo mkinitcpio -P
```

**Обновление главного конфига**

```bash
sudo nano /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

**Plymouth**

```bash
sudo plymouth-set-default-theme
sudo plymouth-set-default-theme -l
sudo plymouth-set-default-theme bgrt
```

### Переход с [[boot-chains]] на GRUB

Установка GRUB и перенос загрузки с systemd-boot:

```bash
sudo pacman -S grub efibootmgr os-prober

cat /etc/fstab # проверить адрес загрузочного тома /boot

sudo grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

```bash
sudo os-prober
```

Обнаружение Windows загрузчиком:

```text
/dev/nvme0n1p1@/efi/Microsoft/Boot/bootmgfw.efi:Windows Boot Manager:Windows:efi
```

Генерация конфигурации GRUB:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Удаление systemd-boot:

```bash
sudo bootctl remove
sudo rm -f /boot/EFI/Linux/*.efi
sudo rm -f /boot/EFI/systemd/*
sudo rm -rf /boot/loader/
```

Чистка загрузочных записей EFI:

```bash
efibootmgr
sudo grub-install --target=x86_64-efi --efi-directory=/boot --recheck
# sudo grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# удалить дубли
sudo efibootmgr -b 0001 -B
```

### Файлы настройки загрузки

Назначение каждого файла и его роль в процессе загрузки Linux (на примере Arch Linux).

#### /etc/mkinitcpio.d/linux.preset

**Назначение:** файл настроек для генерации initramfs (initial RAM filesystem) при обновлении ядра. Определяет, какие образы initramfs создавать и где их размещать.

**Что делает:**

- задаёт шаблоны для создания initramfs-образов (например, для разных ядер или конфигураций);
- указывает пути сохранения образов (обычно `/boot/initramfs-linux.img`);
- может настраивать создание Unified Kernel Images (UKI) для UEFI;
- определяет, какие дополнительные опции применять при сборке (например, включение splash-экрана).

**Когда используется:** при обновлении ядра (`pacman -Syu`) или ручном запуске `mkinitcpio`.

**Типичные параметры:**

- `PRESETS=('default' 'fallback')`;
- `ALL_kver="vmlinuz-linux"`;
- `ALL_initrd="/boot/initramfs-linux.img"`;
- `ALL_uki="/boot/EFI/Linux/arch.efi"` (для UKI).

#### /etc/mkinitcpio.conf

**Назначение:** основной конфигурационный файл для утилиты `mkinitcpio`, отвечающей за создание initramfs.

**Что определяет:**

- **HOOKS** — порядок и набор модулей, загружаемых на этапе initramfs (например, `base udev autodetect modconf block filesystems keyboard fsck`);
- **MODULES** — список дополнительных модулей ядра, которые нужно включить в initramfs;
- **FILES** — дополнительные файлы, которые нужно скопировать в initramfs;
- **BINARIES** — дополнительные бинарные файлы для включения.

**Роль в загрузке:** initramfs — временная файловая система в памяти, которая:

- загружает драйверы для доступа к дискам;
- монтирует корневой раздел;
- передаёт управление основной ОС.

**Когда редактируют:** при настройке шифрования диска, LVM, RAID, добавлении поддержки нестандартного оборудования.

#### /etc/kernel/cmdline

**Назначение:** файл с параметрами ядра Linux, которые передаются загрузчиком (GRUB, systemd-boot и т.д.) при старте системы.

**Что содержит:** строку параметров ядра, например:

```text
root=PARTUUID=... rw quiet loglevel=3
```

**Основные параметры:**

- `root=` — указывает корневой раздел (`PARTUUID`, `UUID` или путь);
- `rw` — монтировать корень в режиме чтения-записи;
- `quiet` — скрыть сообщения ядра;
- `loglevel=` — уровень детализации логов;
- другие опции (поддержка модулей, настройки драйверов и т.д.).

**Где используется:**

- systemd-boot читает этот файл напрямую;
- GRUB может использовать его содержимое через переменные (если настроено).

**Когда меняют:** при настройке splash-экранов, отладке загрузки, изменении корневого раздела.

#### /proc/cmdline

**Назначение:** виртуальный файл в псевдофайловой системе `/proc`, отображающий текущие параметры ядра, с которыми система была загружена.

**Ключевые особенности:**

- не редактируется напрямую (это снимок состояния ядра);
- создаётся ядром при загрузке;
- отражает фактическую командную строку, переданную загрузчиком.

**Как использовать:** для проверки, с какими параметрами работает текущая сессия:

```bash
cat /proc/cmdline
```

**Пример вывода:**

```text
BOOT_IMAGE=/boot/vmlinuz-linux root=PARTUUID=... rw quiet loglevel=3
```

**Зачем нужен:**

- диагностика проблем загрузки;
- проверка, применились ли изменения из `/etc/kernel/cmdline` или `/etc/default/grub`;
- определение режима монтирования корня (`rw`/`ro`).

#### /etc/default/grub

**Назначение:** главный конфигурационный файл загрузчика GRUB в дистрибутивах на базе Debian/Arch.

**Что настраивает:**

- параметры командной строки ядра (`GRUB_CMDLINE_LINUX`);
- внешний вид GRUB (фон, шрифты, цвета);
- таймаут меню;
- порядок загрузки ОС;
- дополнительные опции GRUB.

**Важные переменные:**

- `GRUB_DEFAULT` — пункт меню по умолчанию;
- `GRUB_TIMEOUT` — время ожидания выбора в меню;
- `GRUB_CMDLINE_LINUX` — параметры ядра для Linux (например, `quiet splash`);
- `GRUB_BACKGROUND` — путь к фоновому изображению;
- `GRUB_GFXMODE` — разрешение экрана GRUB.

**Когда применяется:** после редактирования файла нужно выполнить:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Это сгенерирует новый конфигурационный файл GRUB на основе шаблона.

**Роль:** связывает настройки пользователя с реальным конфигурационным файлом загрузчика (`grub.cfg`).

#### Сравнение файлов настройки загрузки

| Файл | Тип | Назначение | Редактируется вручную? | Когда применяется |
| ---- | --- | ---------- | ---------------------- | ----------------- |
| `/etc/mkinitcpio.d/linux.preset` | Конфиг | Шаблоны для сборки initramfs | Да | При обновлении ядра |
| `/etc/mkinitcpio.conf` | Конфиг | Настройки сборки initramfs | Да | При настройке модулей/драйверов |
| `/etc/kernel/cmdline` | Конфиг | Параметры ядра для загрузчиков | Да | При настройке параметров загрузки |
| `/proc/cmdline` | Виртуальный | Текущие параметры ядра (только чтение) | Нет | Для диагностики |
| `/etc/default/grub` | Конфиг | Настройки GRUB | Да | Перед генерацией `grub.cfg` |

#### Взаимосвязь файлов в процессе загрузки

1. `/etc/mkinitcpio.conf` и `/etc/mkinitcpio.d/linux.preset` используются `mkinitcpio` для создания `initramfs-linux.img`.
2. `/etc/kernel/cmdline` или `/etc/default/grub` задают параметры ядра (`root=`, `quiet` и т.д.).
3. GRUB (настроенный через `/etc/default/grub`) передаёт параметры ядра и загружает `vmlinuz` + `initramfs`.
4. Ядро запускается с параметрами из `/proc/cmdline` (копия переданных настроек).
5. initramfs монтирует корневой раздел и передаёт управление ОС.

### Архитектуры загрузки: GRUB с initramfs и systemd-boot с UKI

Переход на `systemd-boot + UKI` с отказом от `mkinitcpio` возможен, но требует изменения подхода к сборке ядра. Вместо `mkinitcpio` используется `systemd-ukify` — встроенный инструмент `systemd` для создания UKI-образов.

#### Архитектура UKI, создаваемая через systemd-ukify

Архитектура опирается на `systemd-stub` (загрузочный загрузчик), который вшит в сам UKI-образ.

**Цепочка загрузки**

1. **UEFI** → загружает `.efi`-файл UKI напрямую.
2. **systemd-stub** → минимальный загрузчик внутри UKI, распаковывает ядро и initramfs.
3. **Ядро Linux** → загружается и запускает `init`.
4. **systemd (PID 1)** → инициализирует систему.

Все компоненты упакованы в единый `.efi`-файл, подписанный для Secure Boot.

#### Пошаговая инструкция по переходу

##### Важные предупреждения

- Сделайте резервную копию загрузочного раздела (ESP) и важных файлов.
- Процесс требует создания файлов в ESP, которые могут перезаписать существующие.
- Подготовьте загрузочную флешку с Arch на случай, если что-то пойдёт не так.

##### Установка необходимых пакетов

`systemd-ukify` уже входит в состав `systemd`, но для создания UKI потребуется установить дополнительные утилиты.

```bash
sudo pacman -S systemd-ukify sbctl
```

- `systemd-ukify` — утилита для создания UKI.
- `sbctl` — для управления Secure Boot и подписью.

##### Настройка параметров ядра и initramfs

Создание файла `/etc/kernel/cmdline` с параметрами командной строки ядра (вместо набора в `/etc/default/grub`):

```bash
sudo tee /etc/kernel/cmdline << EOF
root=PARTUUID=aea8c00a-d01c-46f8-8449-5c8813c94ce9 rw quiet loglevel=3
EOF
```

`PARTUUID` указывается для корневого раздела.

Для создания initramfs без `mkinitcpio` нужно либо использовать глобальный `initramfs` (если ядро его поддерживает), либо сгенерировать его с помощью `dracut`. `mkinitcpio` в Arch тесно интегрирован с ядром, поэтому полный отказ от него не рекомендуется и может привести к проблемам с загрузкой модулей. Однако можно использовать `mkinitcpio` для генерации образа, а затем упаковать его в UKI с помощью `ukify`.

##### Генерация initramfs (через mkinitcpio)

```bash
sudo mkinitcpio -P
```

Это создаст `/boot/initramfs-linux.img`.

##### Создание UKI через systemd-ukify

```bash
sudo ukify build \
    --linux=/boot/vmlinuz-linux \
    --initrd=/boot/initramfs-linux.img \
    --cmdline=/etc/kernel/cmdline \
    --splash=/usr/share/systemd/bootctl/splash-arch.bmp \
    --output=/boot/EFI/Linux/arch-linux.efi
```

**Опции:**

- `--linux` — путь к ядру.
- `--initrd` — путь к initramfs.
- `--cmdline` — файл с параметрами загрузки.
- `--splash` — картинка-заставка (опционально).
- `--output` — куда сохранить UKI.

##### Установка systemd-boot

```bash
bootctl install
```

Он создаст записи в UEFI и скопирует `systemd-bootx64.efi` в `/boot/EFI/systemd/`.

##### Создание загрузочных записей для UKI

`systemd-boot` автоматически обнаружит все `.efi`-файлы в `/boot/EFI/Linux/`. Если поместить `arch-linux.efi` туда, он появится в меню загрузки. Для Windows или других ОС создаются отдельные `.conf`-файлы в `/boot/loader/entries/`.

##### Настройка Secure Boot (опционально)

Если Secure Boot включён, подпишите UKI с помощью `sbctl`.

```bash
# Установка собственных ключей (один раз)
sbctl create-keys
sbctl enroll-keys -m

# Подпись UKI
sbctl sign -s /boot/EFI/Linux/arch-linux.efi
```

##### Удаление GRUB

```bash
sudo pacman -Rns grub
```

##### Настройка таймера для автоматического обновления UKI

Чтобы UKI обновлялся при каждом обновлении ядра, создайте хук для `pacman`. Создание файла `/etc/pacman.d/hooks/update-uki.hook`:

```ini
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = linux

[Action]
Description = Updating UKI...
When = PostTransaction
Exec = /usr/bin/ukify build --linux=/boot/vmlinuz-linux --initrd=/boot/initramfs-linux.img --cmdline=/etc/kernel/cmdline --splash=/usr/share/systemd/bootctl/splash-arch.bmp --output=/boot/EFI/Linux/arch-linux.efi
```

#### Финальная очистка

Полный отказ от `mkinitcpio` в процессе сборки (не только UKI) требует перехода на `dracut` или другой генератор initramfs. Это отдельный сложный процесс, не рекомендуется для новичков.

**Краткий итог:** UKI через `systemd-ukify` — современный, минималистичный и безопасный подход. Процесс перехода несложен при аккуратном выполнении шагов по созданию UKI и установке `systemd-boot`. Полный отказ от `mkinitcpio` — отдельная задача, требующая либо замены генератора initramfs, либо ручного создания образа. В большинстве случаев использование `mkinitcpio` для генерации initramfs, а `systemd-ukify` для упаковки в UKI — оптимальный и безопасный путь.
