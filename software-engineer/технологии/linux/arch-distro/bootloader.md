# shit to fixe @ loadin
Что означатает сообщение при загрузке ОС archlinux и как это испрваить
i801_smbus 0000:00:1f.4: SMBus is busy, can't use it! ?`


# faq
## где находится bootloader-logo
```bash
# /usr/share/systemd/bootctl/splash-arch.bmp

# обновить второстепенный конфиг
sudo nano /etc/mkinitcpio.d/linux.preset
sudo nano /etc/mkinitcpio.conf
sudo nano /etc/kernel/cmdline
# cat /proc/cmdline
sudo mkinitcpio -P

# обновить главный конфиг
sudo nano /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg

# plymouth
sudo plymouth-set-default-theme
sudo plymouth-set-default-theme -l
sudo plymouth-set-default-theme bgrt

```
# с UKI на GRUB

```bash
sudo pacman -S grub efibootmgr os-prober

cat /etc/fstab # -- /boot ? проверить адрес загрузочного тома

[a@al ~]$ sudo grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
Installing for x86_64-efi platform.
done

[a@al ~]$ cat /etc/default/grub
# GRUB boot loader configuration
# ...

[a@al ~]$ sudo os-prober
/dev/nvme0n1p1@/efi/Microsoft/Boot/bootmgfw.efi:Windows Boot Manager:Windows:efi
[a@al ~]$ sudo grub-mkconfig -o /boot/grub/grub.cfg
Generating grub configuration file ...
done

[a@al ~]$ sudo bootctl remove
Removed '/boot/EFI/BOOT/BOOTX64.EFI'.
...
Removed directory "/boot/loader/entries".
Removed EFI variable LoaderSystemToken-4a67b082-0a4c-41cf-b6c7-440b29bb8c4f.

[a@al ~]$ sudo rm -f /boot/EFI/Linux/*.efi
[a@al ~]$ sudo rm -f /boot/EFI/systemd/*
[a@al ~]$ sudo rm -rf /boot/loader/

efibootmgr
sudo grub-install --target=x86_64-efi --efi-directory=/boot --recheck
# sudo grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# удалить дубли
sudo efibootmgr -b 0001 -B
```




---
Разберу назначение каждого файла подробно — с указанием их роли в процессе загрузки Linux (на примере Arch Linux).


## 1. `/etc/mkinitcpio.d/linux.preset`

**Назначение:** файл настроек для генерации initramfs (initial RAM filesystem) при обновлении ядра. Определяет, какие образы initramfs создавать и где их размещать.

**Что делает:**
* задаёт шаблоны для создания initramfs‑образов (например, для разных ядер или конфигураций);
* указывает пути сохранения образов (обычно `/boot/initramfs-linux.img`);
* может настраивать создание Unified Kernel Images (UKI) для UEFI;
* определяет, какие дополнительные опции применять при сборке (например, включение splash‑экрана).

**Когда используется:** при обновлении ядра (`pacman -Syu`) или ручном запуске `mkinitcpio`.

**Типичные параметры:**
* `PRESETS=('default' 'fallback')`;
* `ALL_kver="vmlinuz-linux"`;
* `ALL_initrd="/boot/initramfs-linux.img"`;
* `ALL_uki="/boot/EFI/Linux/arch.efi"` (для UKI).

---

## 2. `/etc/mkinitcpio.conf`

**Назначение:** основной конфигурационный файл для утилиты `mkinitcpio`, отвечающей за создание initramfs.

**Что определяет:**
* **HOOKS** — порядок и набор модулей, загружаемых на этапе initramfs (например, `base udev autodetect modconf block filesystems keyboard fsck`);
* **MODULES** — список дополнительных модулей ядра, которые нужно включить в initramfs;
* **FILES** — дополнительные файлы, которые нужно скопировать в initramfs;
* **BINARIES** — дополнительные бинарные файлы для включения.

**Роль в загрузке:** initramfs — временная файловая система в памяти, которая:
* загружает драйверы для доступа к дискам;
* монтирует корневой раздел;
* передаёт управление основной ОС.

**Когда редактируют:** при настройке шифрования диска, LVM, RAID, добавлении поддержки нестандартного оборудования.

---

## 3. `/etc/kernel/cmdline`

**Назначение:** файл с параметрами ядра Linux, которые передаются загрузчиком (GRUB, systemd‑boot и т. д.) при старте системы.

**Что содержит:** строку параметров ядра, например:
```
root=PARTUUID=... rw quiet loglevel=3
```

**Основные параметры:**
* `root=` — указывает корневой раздел (`PARTUUID`, `UUID` или путь);
* `rw` — монтировать корень в режиме чтения‑записи;
* `quiet` — скрыть сообщения ядра;
* `loglevel=` — уровень детализации логов;
* другие опции (поддержка модулей, настройки драйверов и т. д.).

**Где используется:**
* systemd‑boot читает этот файл напрямую;
* GRUB может использовать его содержимое через переменные (если настроено).

**Когда меняют:** при настройке splash‑экранов, отладке загрузки, изменении корневого раздела.

---

## 4. `/proc/cmdline`

**Назначение:** виртуальный файл в псевдофайловой системе `/proc`, отображающий **текущие параметры ядра**, с которыми система была загружена.

**Ключевые особенности:**
* не редактируется напрямую (это снимок состояния ядра);
* создаётся ядром при загрузке;
* отражает фактическую командную строку, переданную загрузчиком.

**Как использовать:** для проверки, с какими параметрами работает текущая сессия:
```bash
cat /proc/cmdline
```
**Пример вывода:**
```
BOOT_IMAGE=/boot/vmlinuz-linux root=PARTUUID=... rw quiet loglevel=3
```

**Зачем нужен:**
* диагностика проблем загрузки;
* проверка, применились ли изменения из `/etc/kernel/cmdline` или `/etc/default/grub`;
* определение режима монтирования корня (`rw`/`ro`).

---

## 5. `/etc/default/grub`

**Назначение:** главный конфигурационный файл загрузчика **GRUB** в дистрибутивах на базе Debian/Arch.

**Что настраивает:**
* параметры командной строки ядра (`GRUB_CMDLINE_LINUX`);
* внешний вид GRUB (фон, шрифты, цвета);
* таймаут меню;
* порядок загрузки ОС;
* дополнительные опции GRUB.

**Важные переменные:**
* `GRUB_DEFAULT` — пункт меню по умолчанию;
* `GRUB_TIMEOUT` — время ожидания выбора в меню;
* `GRUB_CMDLINE_LINUX` — параметры ядра для Linux (например, `quiet splash`);
* `GRUB_BACKGROUND` — путь к фоновому изображению;
* `GRUB_GFXMODE` — разрешение экрана GRUB.

**Когда применяется:** после редактирования файла нужно выполнить:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
Это сгенерирует новый конфигурационный файл GRUB на основе шаблона.

**Роль:** связывает настройки пользователя с реальным конфигурационным файлом загрузчика (`grub.cfg`).

---

## Краткий итог: сравнение файлов

| Файл | Тип | Назначение | Редактируется вручную? | Когда применяется |
|------|------|----------|---------------------|----------------|
| `/etc/mkinitcpio.d/linux.preset` | Конфиг | Шаблоны для сборки initramfs | Да | При обновлении ядра |
| `/etc/mkinitcpio.conf` | Конфиг | Настройки сборки initramfs | Да | При настройке модулей/драйверов |
| `/etc/kernel/cmdline` | Конфиг | Параметры ядра для загрузчиков | Да | При настройке параметров загрузки |
| `/proc/cmdline` | Виртуальный | Текущие параметры ядра (только чтение) | Нет | Для диагностики |
| `/etc/default/grub` | Конфиг | Настройки GRUB | Да | Перед генерацией `grub.cfg` |

---

## Взаимосвязь файлов в процессе загрузки


1. **`/etc/mkinitcpio.conf`** и **`/etc/mkinitcpio.d/linux.preset`** → используются `mkinitcpio` для создания `initramfs-linux.img`.
2. **`/etc/kernel/cmdline`** или **`/etc/default/grub`** → задают параметры ядра (`root=`, `quiet` и т. д.).
3. **GRUB** (настроенный через `/etc/default/grub`) → передаёт параметры ядра и загружает `vmlinuz` + `initramfs`.
4. **Ядро** → запускается с параметрами из `/proc/cmdline` (копия переданных настроек).
5. **initramfs** → монтирует корневой раздел и передаёт управление ОС.

Хотите, я помогу разобрать какой‑то конкретный файл подробнее или подскажу, как изменить настройки под вашу задачу?

# архитектуры загрузки: GRUB + initramfs → systemd-boot + UKI
Переход на `systemd-boot + UKI` с отказом от `mkinitcpio` возможен, но он требует изменения подхода к сборке ядра. Вместо `mkinitcpio` используется `systemd-ukify` — это встроенный инструмент `systemd` для создания UKI-образов.

### 🚀 Архитектура UKI, создаваемая через `systemd-ukify`

Эта архитектура опирается на **`systemd-stub`** (загрузочный загрузчик), который вшит в сам UKI-образ.

**Цепочка загрузки:**

1.  **UEFI** → загружает `.efi`-файл UKI напрямую.
2.  **systemd-stub** → минимальный загрузчик внутри UKI, распаковывает ядро и initramfs.
3.  **Ядро Linux** → загружается и запускает `init`.
4.  **systemd (PID 1)** → инициализирует систему.

Все компоненты упакованы в единый `.efi`-файл, подписанный для Secure Boot.

---

### 🛠️ Пошаговая инструкция по переходу

#### ⚠️ Важные предупреждения

*   **Сделайте резервную копию** загрузочного раздела (ESP) и важных файлов.
*   Процесс требует создания файлов в ESP, которые могут перезаписать существующие.
*   Подготовьте загрузочную флешку с Arch на случай, если что-то пойдёт не так.

---

#### 1. Установка необходимых пакетов

`systemd-ukify` уже входит в состав `systemd`, но для создания UKI потребуется установить дополнительные утилиты.

```bash
sudo pacman -S systemd-ukify sbctl
```
*   `systemd-ukify` — утилита для создания UKI.
*   `sbctl` — для управления Secure Boot и подписью.

---

#### 2. Настройка параметров ядра и initramfs

Создайте файл `/etc/kernel/cmdline`, который будет содержать параметры командной строки ядра (вместо сложного набора в `/etc/default/grub`):

```bash
sudo tee /etc/kernel/cmdline << EOF
root=PARTUUID=aea8c00a-d01c-46f8-8449-5c8813c94ce9 rw quiet loglevel=3
EOF
```
(Замените `PARTUUID` на ваш корневой раздел).

Для создания initramfs **без `mkinitcpio`** нужно либо использовать **глобальный `initramfs`** (если ядро его поддерживает), либо сгенерировать его с помощью `dracut`. Но `mkinitcpio` в Arch тесно интегрирован с ядром, поэтому полный отказ от него **не рекомендуется** и может привести к проблемам с загрузкой модулей. Однако можно использовать `mkinitcpio` для генерации образа, а затем упаковать его в UKI с помощью `ukify`.

---

#### 3. Генерация initramfs (через mkinitcpio, если он всё ещё используется)

Если вы планируете полностью отказаться от `mkinitcpio`, вам нужно будет собрать initramfs вручную. В Arch это сложно, поэтому **я рекомендую оставить `mkinitcpio` только для генерации initramfs**, а UKI собирать через `ukify`.

```bash
sudo mkinitcpio -P
```
Это создаст `/boot/initramfs-linux.img`.

---

#### 4. Создание UKI через `systemd-ukify`

Создайте UKI-образ из существующего ядра и initramfs, используя `ukify`.

```bash
sudo ukify build \
    --linux=/boot/vmlinuz-linux \
    --initrd=/boot/initramfs-linux.img \
    --cmdline=/etc/kernel/cmdline \
    --splash=/usr/share/systemd/bootctl/splash-arch.bmp \
    --output=/boot/EFI/Linux/arch-linux.efi
```

**Опции:**
*   `--linux` — путь к ядру.
*   `--initrd` — путь к initramfs.
*   `--cmdline` — файл с параметрами загрузки.
*   `--splash` — картинка-заставка (опционально).
*   `--output` — куда сохранить UKI.

---

#### 5. Установка `systemd-boot`

Теперь установите сам загрузчик `systemd-boot`. Поскольку вы удаляете GRUB, важно, чтобы `systemd-boot` был правильно установлен в ESP.

```bash
bootctl install
```
Он создаст записи в UEFI и скопирует `systemd-bootx64.efi` в `/boot/EFI/systemd/`.

---

#### 6. Создание загрузочных записей для UKI

`systemd-boot` автоматически обнаружит все `.efi`-файлы в `/boot/EFI/Linux/`. Поэтому, если вы поместили `arch-linux.efi` туда, он появится в меню загрузки. Однако для Windows или других ОС создайте отдельные `.conf` файлы в `/boot/loader/entries/`.

---

#### 7. (Опционально) Настройка Secure Boot

Если Secure Boot включён, подпишите UKI с помощью `sbctl`.

```bash
# Установка собственных ключей (один раз)
sbctl create-keys
sbctl enroll-keys -m

# Подпись UKI
sbctl sign -s /boot/EFI/Linux/arch-linux.efi
```

---

#### 8. Удаление GRUB

Теперь, когда `systemd-boot` работает, можно удалить GRUB.

```bash
sudo pacman -Rns grub
```

---

#### 9. Настройка таймера для автоматического обновления UKI

Чтобы UKI обновлялся при каждом обновлении ядра, создайте хук для `pacman`. Создайте файл `/etc/pacman.d/hooks/update-uki.hook`:

```
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

---

### 🧹 Финальная очистка

Если вы хотите полностью отказаться от использования `mkinitcpio` в процессе сборки (не только UKI), вам нужно будет перейти на `dracut` или другой генератор initramfs. Это отдельный сложный процесс, который не рекомендуется для новичков.

**Краткий итог:** UKI через `systemd-ukify` — это современный, минималистичный и безопасный подход. Процесс перехода несложный, если вы аккуратно выполните шаги по созданию UKI и установке `systemd-boot`. Однако полный отказ от `mkinitcpio` — это отдельная задача, которая требует либо замены генератора initramfs, либо ручного создания образа. В большинстве случаев использование `mkinitcpio` для генерации initramfs, а `systemd-ukify` для упаковки в UKI — это оптимальный и безопасный путь.