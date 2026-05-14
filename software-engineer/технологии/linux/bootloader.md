Bootloaders (загрузчики) — это программы, которые запускают операционную систему. Они загружают ядро и initramfs в память перед запуском системы. Рассмотрим различия между systemd-boot, GRUB, efistub, Limine и rEFInd. [```3```](https://wiki.archlinux.org/title/Arch_boot_process_%28%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9%29)

## systemd-boot

Часть экосистемы systemd, минималистичный загрузчик, предназначенный только для систем с UEFI. Не поддерживает BIOS. [```1```](https://wiki.cachyos.org/installation/boot_managers/)[```2```](https://www.dotlinux.net/blog/6-best-linux-boot-loaders/)

**Особенности:**
* Простая конфигурация: загрузочные записи разделены на несколько файлов, что упрощает управление. [```1```](https://wiki.cachyos.org/installation/boot_managers/)
* Автоматически обнаруживает ядра в каталоге `/boot` (например, `vmlinuz-linux` и `initramfs-linux.img`). [```2```](https://www.dotlinux.net/blog/6-best-linux-boot-loaders/)
* Поддерживает зашифрованные разделы `/boot`. [```1```](https://wiki.cachyos.org/installation/boot_managers/)
* Интегрирован с systemd: для управления записями и обновлениями используется утилита `bootctl`. [```2```](https://www.dotlinux.net/blog/6-best-linux-boot-loaders/)
* Измеряет TPM PCR во время загрузки. [```1```](https://wiki.cachyos.org/installation/boot_managers/)
* Подходит для быстрых и простых установок UEFI, а также как запасной вариант для материнских плат MSI с проблемами UEFI. [```1```](https://wiki.cachyos.org/installation/boot_managers/)[```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)

## GRUB (GRand Unified Bootloader)

Один из самых распространённых загрузчиков, поддерживает как BIOS, так и UEFI. [```2```](https://www.dotlinux.net/blog/6-best-linux-boot-loaders/)[```14```](https://www.linux.org/threads/linux-bootloaders-installing-and-configuring-refind-limine-and-grub.57750/)

**Особенности:**
* Гибкий и многофункциональный: поддерживает многозадачность, RAID, LUKS1, LVM. [```2```](https://www.dotlinux.net/blog/6-best-linux-boot-loaders/)[```3```](https://wiki.archlinux.org/title/Arch_boot_process_%28%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9%29)
* Имеет встроенный генератор конфигурации `grub-mkconfig`, который автоматически обнаруживает другие операционные системы. [```4```](https://wiki.alpinelinux.org/wiki/Bootloaders)
* Поддерживает широкий спектр файловых систем, включая ext2, ext3, ext4, btrfs, zfs, minix, iso9660, xfs, NTFS и FAT32. [```5```](https://www.ubuntupit.com/best-linux-bootloaders/)
* Подходит для сложных сценариев, например, для зашифрованного `/boot`, BIOS или работы с Btrfs. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Может быть медленнее других загрузчиков, особенно на старых системах с BIOS. [```2```](https://www.dotlinux.net/blog/6-best-linux-boot-loaders/)

## efistub (EFI boot stub)

Метод, который позволяет загружать ядро Linux напрямую через UEFI без дополнительного загрузчика. [```8```](https://dzen.ru/a/ZzEFePdYWmh8FbZ8)

**Особенности:**
* Ядро должно быть собрано с флагом `CONFIG_EFI_STUB=y`. [```6```](https://wiki.archlinux.org/title/EFI_boot_stub)
* Минимальная задержка: отсутствие промежуточного звена в цепочке загрузки ускоряет процесс. [```8```](https://dzen.ru/a/ZzEFePdYWmh8FbZ8)
* Пользователь сам решает, передавать ли параметры ядру. [```8```](https://dzen.ru/a/ZzEFePdYWmh8FbZ8)
* Требует настройки UEFI: обычно достаточно указать местоположение ядра и параметры для его запуска. [```8```](https://dzen.ru/a/ZzEFePdYWmh8FbZ8)

## Limine

Современный, продвинутый и портативный многопротокольный загрузчик. Служит эталонной реализацией протокола загрузки Limine, поддерживает Linux и цепную загрузку других загрузчиков. [```1```](https://wiki.cachyos.org/installation/boot_managers/)[```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)

**Особенности:**
* Поддерживает несколько протоколов загрузки, включая Multiboot2 и протокол загрузки Linux. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Работает как на системах UEFI, так и на BIOS. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Имеет возможности кастомизации тем, схожие с GRUB. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Поддерживает снимки Btrfs через `limine-snapper-sync`. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Требует, чтобы `/boot` использовал FAT12/16/32 или ISO9660; другие файловые системы требуют дополнительной настройки. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Не добавляет автоматически запись в NVRAM UEFI — это нужно делать вручную с помощью `efibootmgr` или `limine-entry-tool`. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)

## rEFInd (reFormatted EFI Indicator)

Графический загрузчик, ориентированный на UEFI. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)[```14```](https://www.linux.org/threads/linux-bootloaders-installing-and-configuring-refind-limine-and-grub.57750/)

**Особенности:**
* Автоматически обнаруживает все операционные системы и ядра на накопителях. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Имеет графический интерфейс, напоминающий селектор загрузки в macOS. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Поддерживает темы, с опциональной поддержкой сенсорного экрана. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Может считывать образы для загрузки с файловых систем EFI (FAT12/16/32), а также EXT4 и BTRFS. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Не поддерживает системы с BIOS. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)
* Не измеряет регистры PCR модуля TPM. [```12```](https://wiki.cachyos.org/ru/installation/boot_managers/)

## Сравнение

| Загрузчик | Поддержка UEFI/BIOS | Особенности | Сценарии использования |
|---|---|---|---|
| systemd-boot | Только UEFI | Минималистичный, быстрая загрузка, интеграция с systemd | Простые установки UEFI, системы с проблемами UEFI на MSI |
| GRUB | UEFI и BIOS | Гибкий, поддержка расширенных функций (шифрование, RAID, LVM) | Сложные сценарии, зашифрованный `/boot`, многозадачность |
| efistub | Только UEFI | Прямая загрузка ядра через UEFI | Минимализм, скорость, контроль над параметрами ядра |
| Limine | UEFI и BIOS | Многопротокольный, поддержка Btrfs-снапшотов | Современные системы, dual-boot с Windows, поддержка обоих типов прошивки |
| rEFInd | Только UEFI | Графический интерфейс, автоматическое обнаружение ОС | Мультизагрузка, простота использования, акцент на внешний вид |

Выбор загрузчика зависит от конкретных требований: простоты, гибкости, поддержки определённых функций или внешнего вида интерфейса.


## Windows Boot Manager `WBM`

Windows использует собственный загрузчик — Windows Boot Manager `WBM`, сокращённо bootmgr или winload.efi, который работает по‑разному в системах с [[BIOS UEFI|BIOS]] и UEFI. Разберу подробно.