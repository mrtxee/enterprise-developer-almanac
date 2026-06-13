### струткура томов
> проверить что при установке правильно создана струткура разделов

пример правильной структуры:
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

## проверка sweap разделов
swapon --show
NAME           TYPE      SIZE USED PRIO
/dev/zram0     partition   4G   0B  100
/dev/nvme0n1p5 partition  32G   0B   -1

sudo blkid | grep swap
/dev/nvme0n1p5: UUID="967bf664-29ba-4aa1-bf85-d18fdd3d47ec" TYPE="swap" PARTUUID="3ccb641b-0674-4892-9a92-b90e3918f15c"
/dev/zram0: LABEL="zram0" UUID="858c9205-7189-417c-b83d-1ba82fb9c20d" TYPE="swap"
```

У вас отличная ситуация: [[swap-partition]] есть, он виден системе, но не прописан в `/etc/fstab`. Это легко исправить. Вот пошаговая инструкция, как настроить гибернацию.

---

### 1. Добавляем swap-раздел в `/etc/fstab`

Это нужно, чтобы после каждой перезагрузки дисковый swap включался автоматически.

Откройте файл `/etc/fstab` для редактирования:
```bash
sudo nano /etc/fstab
```

Добавьте в самый конец файла следующую строку (используя UUID вашего раздела):
```
UUID=967bf664-29ba-4aa1-bf85-d18fdd3d47ec none swap defaults 0 0
```

Сохраните файл (`Ctrl+O`, `Enter`, `Ctrl+X`).

Теперь swap будет автоматически подключаться при загрузке. Для немедленной активации можно выполнить:
```bash
sudo swapon -a
```
(хотя он и так уже активен, это не помешает).

---
### 2. Добавление хука `resume` в `mkinitcpio`

Теперь добавим в начальный загрузочный образ модуль, который сможет восстанавливать состояние.

Откройте `/etc/mkinitcpio.conf`:
```bash
sudo nano /etc/mkinitcpio.conf
```

Найдите строку `HOOKS=(...)`. Внутри скобок добавьте слово `resume` **после `block` и перед `filesystems`**. Например, если у вас строка выглядит так:
```
HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block filesystems fsck)
```
Измените её на:
```
HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block resume filesystems fsck)
```

> Если вы используете `systemd` вместо `udev` (в строке есть `systemd`), то добавляйте `resume` после `systemd`, но тоже перед `filesystems`.

Сохраните файл и пересоберите initramfs:
```bash
sudo mkinitcpio -P
```

---

### ~~3. Настройка параметра `resume` для GRUB~~
> имеет смысл, только если несколько SWAP-разделов. Иначе избыточен.

Этот параметр укажет ядру, где искать образ гибернации.

Откройте файл `/etc/default/grub`:
```bash
sudo nano /etc/default/grub
```

Найдите строку `GRUB_CMDLINE_LINUX_DEFAULT=`. Внутри кавычек добавьте `resume=UUID=967bf664-29ba-4aa1-bf85-d18fdd3d47ec`. Если там уже есть `quiet`, добавьте после пробела. Например:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet resume=UUID=967bf664-29ba-4aa1-bf85-d18fdd3d47ec"
```

Сохраните файл и перегенерируйте конфигурацию GRUB:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

---


### 4. Проверка и тестирование

Перезагрузите систему:
```bash
reboot
```

~~После перезагрузки проверьте, что параметр `resume` был передан ядру~~: *только если 2. пригодился*
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

### ⚠️ Важное замечание про `/dev/zram0`

У вас включён zram (сжатая оперативная память) с высоким приоритетом. Он используется как быстрый swap, но **не мешает гибернации**, потому что ядро знает, куда сохранять образ – на диск (указано в `resume`). При пробуждении zram будет создан заново.

**Если хотите отключить zram** (например, чтобы освободить место в оперативной памяти), можно удалить пакет `zram-generator` или отключить соответствующий systemd-сервис. Но в целом он не мешает.

---

