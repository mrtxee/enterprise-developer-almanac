---
aliases:
  - blkid
  - bootctl
  - fdisk
  - filefrag
  - fstab
  - GRUB
  - grub-mkconfig
  - hibernation
  - initramfs
  - linux hibernation
  - lsblk
  - mkinitcpio
  - mkinitcpio.conf
  - partition
  - Plymouth
  - resume
  - sd-resume
  - swap
  - swapon
  - systemctl
  - systemd
  - UKI
  - zram
  - Гибернация
  - Подкачка
  - Раздел
  - Раздел подкачки
---

## Структура томов

Проверить, что при установке правильно создана структура разделов.

Пример правильной структуры:

```bash
sudo fdisk -l
Disk /dev/nvme0n1: ...
Disk model: FORESEE XP2100F001T
...
Disk identifier: 70BE49B5-C97F-4A63-828F-8CF25EF996F2

Device             Start        End    Sectors   Size Type
# /boot; boot efi -- том с загрузчиком ОС, grub
/dev/nvme0n1p4 357156864  359254015    2097152     1G EFI System
# ; ; swap-linux  -- том для гибернации и подкачки, д.б. >= ОЗУ
/dev/nvme0n1p5 359254016  426362879   67108864    32G Linux swap
# /; ext4         -- том для root пользователя
/dev/nvme0n1p6 426362880  615106559  188743680    90G Linux root (x86-64)
# /home; ext4;    -- том для пользователей
/dev/nvme0n1p7 615106560  719964159  104857600    50G Linux home
```

Проверка swap-разделов:

```bash
swapon --show
NAME           TYPE      SIZE USED PRIO
/dev/zram0     partition   4G   0B  100
/dev/nvme0n1p5 partition  32G   0B   -1

sudo blkid | grep swap
/dev/nvme0n1p5: UUID="967bf664-29ba-4aa1-bf85-d18fdd3d47ec" TYPE="swap" PARTUUID="3ccb641b-0674-4892-9a92-b90e3918f15c"
/dev/zram0: LABEL="zram0" UUID="858c9205-7189-417c-b83d-1ba82fb9c20d" TYPE="swap"
```

[[swap-partition|swap-partition]] есть и виден системе, но не прописан в `/etc/fstab`. Ниже — пошаговая инструкция по настройке гибернации.

---

## Добавляем swap-раздел в /etc/fstab

Это нужно, чтобы после каждой перезагрузки дисковый swap включался автоматически.

Откройте файл `/etc/fstab` для редактирования:

```bash
sudo nano /etc/fstab
```

Добавьте в самый конец файла следующую строку (используя UUID вашего раздела):

```text
UUID=967bf664-29ba-4aa1-bf85-d18fdd3d47ec none swap defaults 0 0
```

Сохраните файл (`Ctrl+O`, `Enter`, `Ctrl+X`).

Теперь swap будет автоматически подключаться при загрузке. Для немедленной активации можно выполнить:

```bash
sudo swapon -a
```

Хотя он и так уже активен, это не помешает.

---

## Добавление хука resume в mkinitcpio

Теперь добавим в начальный загрузочный образ модуль, который сможет восстанавливать состояние.

Откройте `/etc/mkinitcpio.conf`:

```bash
sudo nano /etc/mkinitcpio.conf
```

Найдите строку `HOOKS=(...)`. Внутри скобок добавьте слово `resume` **после `block` и перед `filesystems`**. Например, если у вас строка выглядит так:

```text
HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block filesystems fsck)
```

Измените её на:

```text
HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block resume filesystems fsck)
```

> Если вы используете `systemd` вместо `udev` (в строке есть `systemd`), то добавляйте `resume` после `systemd`, но тоже перед `filesystems`.

Сохраните файл и пересоберите initramfs:

```bash
sudo mkinitcpio -P
```

---

## Настройка параметра resume для GRUB

Этот параметр укажет ядру, где искать образ гибернации.

> Имеет смысл только если несколько swap-разделов. Иначе параметр избыточен.

Откройте файл `/etc/default/grub`:

```bash
sudo nano /etc/default/grub
```

Найдите строку `GRUB_CMDLINE_LINUX_DEFAULT=`. Внутри кавычек добавьте `resume=UUID=967bf664-29ba-4aa1-bf85-d18fdd3d47ec`. Если там уже есть `quiet`, добавьте после пробела. Например:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet resume=UUID=967bf664-29ba-4aa1-bf85-d18fdd3d47ec"
```

Сохраните файл и перегенерируйте конфигурацию GRUB:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Проверка и тестирование

Перезагрузите систему:

```bash
reboot
```

После перезагрузки проверьте, что параметр `resume` был передан ядру (только если шаг про GRUB применился):

```bash
cat /proc/cmdline
```

Вы должны увидеть `resume=UUID=...`.

Теперь протестируйте гибернацию:

```bash
sudo systemctl hibernate
```

Система выключится. При следующем включении она должна загрузиться с сохранённым состоянием (открытые программы, документы).

---

## Замечание про /dev/zram0

У вас включён zram (сжатая оперативная память) с высоким приоритетом. Он используется как быстрый swap, но **не мешает гибернации**, потому что ядро знает, куда сохранять образ — на диск (указано в `resume`). При пробуждении zram будет создан заново.

**Если хотите отключить zram** (например, чтобы освободить место в оперативной памяти), можно удалить пакет `zram-generator` или отключить соответствующий systemd-сервис. Но в целом он не мешает.

---

## Логотип Plymouth при выходе из гибернации (Arch Linux)

При выходе из гибернации Plymouth часто не отображается, потому что хук восстановления (`resume`) запускается **до** инициализации графической заставки. Решение — правильная последовательность хуков в `mkinitcpio.conf`.

### Ключевая причина проблемы

При гибернации ядро при включении обнаруживает образ в swap и **немедленно** начинает восстановление. Если хук `resume` стоит **до** `plymouth`, экран уже занят текстом восстановления, и анимация не появляется.

**Правило:** `plymouth` должен стоять **до** `resume`/`sd-resume`.

### Исправьте порядок хуков

```bash
sudo nano /etc/mkinitcpio.conf
```

Поскольку у вас `systemd`-based хуки, используйте `sd-resume`:

```text
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole plymouth sd-resume block filesystems fsck)
```

| Позиция | Хук | Зачем |
|---------|-----|-------|
| 1–3 | `base systemd autodetect` | Базовая инициализация |
| 4–7 | `microcode modconf kms keyboard sd-vconsole` | Драйверы, видео, ввод |
| 8 | `plymouth` | Запуск анимации ДО восстановления |
| 9 | `sd-resume` | Восстановление из гибернации (под анимацией) |
| 10–11 | `block filesystems fsck` | Монтирование дисков |

> Если используете классические хуки (без `systemd`), замените `sd-resume` на `resume`:
>
> ```text
> HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont plymouth resume block filesystems fsck)
> ```

### Убедитесь, что параметр resume в cmdline

```bash
sudo nano /etc/kernel/cmdline
```

Строка должна содержать `resume=`:

```text
root=PARTUUID=aea8c00a-... zswap.enabled=0 rw rootfstype=ext4 resume=PARTUUID=<swap-partuuid> quiet splash loglevel=3 vt.global_cursor_default=0
```

Найдите PARTUUID swap:

```bash
lsblk -o NAME,PARTUUID,FSTYPE | grep swap
```

Для swap-**файла** (не раздела):

```text
resume=/dev/nvme0n1pX resume_offset=12345678
```

Offset:

```bash
sudo filefrag -v /swapfile | awk 'NR==4{print $4}' | tr -d '.'
```

### Пересоберите initramfs и UKI

```bash
sudo mkinitcpio -P
sudo bootctl update
```

Проверка, что `plymouth` и `sd-resume` попали в образ:

```bash
lsinitcpio /boot/EFI/Linux/arch-linux.efi | grep -E "plymouth|resume"
```

Ожидаемый вывод:

```text
usr/lib/initcpio/hooks/plymouth
usr/lib/initcpio/install/plymouth
usr/lib/initcpio/hooks/sd-resume
usr/lib/initcpio/install/sd-resume
```

### Настройка служб systemd

```bash
# Plymouth запускается рано при загрузке/resume
sudo systemctl enable plymouth-start.service

# Plymouth завершается только после полного восстановления
sudo systemctl enable plymouth-quit-wait.service

# Маскируем преждевременное завершение
sudo systemctl mask plymouth-quit.service
```

### Перезагрузка и тест

```bash
reboot
```

После входа:

```bash
# Тест гибернации
sudo systemctl hibernate
```

При включении вы должны увидеть **ту же анимацию Plymouth**, что и при обычной загрузке, а не текстовый лог восстановления.

### Если анимация всё равно не появляется

| Симптом | Диагностика | Решение |
|---------|-------------|---------|
| Текстовый лог при resume | `cat /proc/cmdline` — нет `quiet splash` | Добавьте в `/etc/kernel/cmdline`, пересоберите UKI |
| Чёрный экран при resume | `dmesg \| grep -i "resume\|hibernate"` | Проверьте `resume=` параметр и swap |
| Plymouth есть при boot, но не при resume | `lsinitcpio ... \| grep plymouth` — пусто | Пересоберите: `sudo mkinitcpio -P` |
| Анимация мелькает на 1 секунду | `plymouth-quit.service` не замаскирован | `sudo systemctl mask plymouth-quit.service` |
| `sd-resume` не найден | `pacman -Qs mkinitcpio` | Обновите: `sudo pacman -Syu mkinitcpio` |

### Итоговый чек-лист

```bash
# 1. Хуки: plymouth ПЕРЕД sd-resume
grep "^HOOKS" /etc/mkinitcpio.conf
# HOOKS=(base systemd ... plymouth sd-resume block filesystems fsck)

# 2. Cmdline: resume + quiet splash
cat /etc/kernel/cmdline
# ... resume=PARTUUID=... quiet splash loglevel=3 vt.global_cursor_default=0

# 3. Пересборка
sudo mkinitcpio -P
sudo bootctl update

# 4. Службы
sudo systemctl enable plymouth-start.service plymouth-quit-wait.service
sudo systemctl mask plymouth-quit.service

# 5. Тест
sudo systemctl hibernate
```

После этого при выходе из гибернации вы увидите ту же анимацию логотипа, что и при обычной загрузке — Plymouth перехватит экран до начала восстановления из swap.

---

## Ошибка: Hook 'sd-resume' cannot be found

Ошибка:

```text
==> ERROR: Hook 'sd-resume' cannot be found
```

возникает при выполнении команды:

```bash
sudo mkinitcpio -P
```

и означает, что в конфигурации `mkinitcpio.conf` указан хук **`sd-resume`**, но **он не существует** в системе — такого модуля/хука нет в Arch Linux.

### Причина

Вы, скорее всего, **скопировали конфигурацию** `mkinitcpio.conf` из другого дистрибутива (например, **[[linux-distros|EndeavourOS]]**, **Garuda**, **Manjaro**) или из примера, где используется **`systemd`-ориентированная сборка initramfs**. Хуки вроде `sd-resume`, `sd-encrypt`, `sd-lvm` — **это псевдонимы для systemd-хуков**, которые **не входят в стандартный пакет `mkinitcpio` в Arch Linux**.

### Решение

**Шаг 1: откройте конфигурацию**

```bash
sudo nano /etc/mkinitcpio.conf
```

**Шаг 2: найдите строку `HOOKS=`**

Пример **неправильной** строки:

```bash
HOOKS=(base udev autodetect modconf keyboard sd-vconsole sd-resume filesystems)
```

Проблема: `sd-resume` — **не существует** в стандартном Arch.

**Шаг 3: замените `sd-resume` на `resume`**

Правильная версия:

```bash
HOOKS=(base udev autodetect modconf keyboard vconsole resume filesystems)
```

> Также замените `sd-vconsole` на `vconsole`, если он не работает.

### Объяснение замены

| Было (неправильно) | Стало (правильно) | Описание |
|--------------------|-------------------|--------|
| `sd-resume` | `resume` | Стандартный хук Arch для восстановления из гибернации |
| `sd-vconsole` | `vconsole` | Настройка консоли (раскладка, шрифт) |

`sd-*` хуки — это **не часть официального Arch Linux**, а **кастомизация в производных дистрибутивах**.

**Шаг 4: сохраните файл и пересоберите образ**

```bash
sudo mkinitcpio -P
```

Теперь ошибка исчезнет, и образы `initramfs` будут успешно созданы.

### Дополнительно: нужен ли resume вообще

Хук `resume` нужен **только если вы используете гибернацию (suspend to disk)** и указали `resume=PARTUUID=...` в параметрах ядра.

Если **гибернация не используется** — хук `resume` можно **безопасно убрать** из `HOOKS`.

### Проверка: работает ли гибернация?

Узнайте, какой swap используется для `resume`:

```bash
cat /proc/swaps
```

Убедитесь, что в `/etc/default/grub` есть:

```text
GRUB_CMDLINE_LINUX_DEFAULT="... resume=PARTUUID=3ccb641b-..."
```

И пересоберите grub, если меняли:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### Итог

- Ошибка: `sd-resume` не существует в Arch Linux.
- Решение: заменить `sd-resume` → `resume`, `sd-vconsole` → `vconsole` в `/etc/mkinitcpio.conf`.
- Используйте только стандартные хуки из официальной документации Arch Wiki.

После исправления — `sudo mkinitcpio -P` выполнится без ошибок.
