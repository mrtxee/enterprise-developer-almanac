

# GNU/Linux OS Architecture

```mermaid
---
title: GNU/Linux OS Architecture
---
flowchart TB
 subgraph Core["Базовые подсистемы ядра"]
        Core_Features["Планирование задач (CFS, EEVDF, RT)<br>Управление памятью (MMU, NUMA, swap)<br>IPC (сигналы, каналы, сокеты, futex)<br>Виртуальная файловая система (VFS)<br>Сетевой стек (TCP/IP, Netfilter)"]
  end
 subgraph Modules["Загружаемые модули ядра (LKM)"]
        Modules_Features["Драйверы устройств<br>Файловые системы (ext4, XFS, btrfs)<br>Сетевые протоколы и фильтры"]
  end
 subgraph Security["Подсистемы безопасности"]
        Security_Features["LSM (SELinux, AppArmor)<br>Namespaces, cgroups<br>Capabilities, seccomp"]
  end
 subgraph System_Layer["Системное окружение (Userspace)"]
    direction TB
        Init["Системы инициализации<br>(systemd, OpenRC, runit)"]
        Libraries["Системные библиотеки<br>(glibc, musl, динамический линкер)"]
        Shell["Командная оболочка<br>(bash, zsh, fish, dash)"]
        Coreutils["GNU Coreutils / busybox<br>(ls, cp, mv, mkdir, cat, rm)"]
        Daemons["Системные демоны<br>(cron, syslog, dbus, NetworkManager)"]
  end
 subgraph Linux_Kernel["Linux Kernel (монолитное ядро)"]
    direction LR
        Core
        Modules
        Security
  end
 subgraph Base["GNU/Linux Base OS"]
    direction LR
        Linux_Kernel
        System_Layer
  end
 subgraph EndUser["Пользовательские дистрибутивы и DE"]
    direction TB
        Distros["Ubuntu, Debian, Fedora, Arch Linux, ..."]
        GUI["Графические серверы и рабочие окружения<br>(X11/Wayland, GNOME, KDE, XFCE, приложения)"]
  end
    EndUser --o Base
    Core --> Modules & Security
    Modules --> Security

    Core_Features@{ shape: comment}
    Modules_Features@{ shape: comment}
    Security_Features@{ shape: comment}
    Init@{ shape: braces}
    Libraries@{ shape: braces}
    Shell@{ shape: braces}
    Coreutils@{ shape: braces}
    Daemons@{ shape: braces}
    GUI@{ shape: comment}
```

**Пояснения к элементам схемы:**

- **Пользовательские дистрибутивы и DE** – конечные продукты (Ubuntu, Fedora и др.) вместе с графическими серверами (X11/Wayland) и рабочими окружениями (GNOME, KDE). 
- **GNU/Linux Base** – полноценная операционная система, объединяющая ядро Linux и системное окружение пользователя (утилиты, библиотеки, демоны). Аналог Darwin: так же включает ядро и системный userspace.
- **Linux Kernel** – монолитное ядро, включающее базовые подсистемы (планировщик, память, IPC, VFS, сеть), загружаемые модули (драйверы, ФС) и подсистемы безопасности (LSM, cgroups). В отличие от XNU, нет чёткого разделения на микроядро и BSD-слой – всё работает в одном адресном пространстве.
- **Системное окружение (Userspace)** – библиотеки, оболочки, coreutils и системные демоны. Аналог «Darwin System Components» на схеме macOS.


---
# GNU/Linux layers

```mermaid
---
title: GNU/Linux layers
---
flowchart TB
 subgraph UserOS["Пользовательское пространство (End-user OS)"]
    direction TB
        Distros["Дистрибутивы<br>(Ubuntu, Arch, Debian, Alpine и др.)"]
        GUI["Графические серверы и оболочки<br>(Wayland, X11, GNOME, KDE, XFCE)"]
        Apps["Пользовательские приложения<br>(браузеры, IDE, мультимедиа)"]
  end
 subgraph SystemComponents["Системные компоненты (User Space)"]
    direction TB
        Libraries["Системные библиотеки<br>(glibc, musl, runtime)"]
        Init["Инициализация и IPC<br>(systemd, D-Bus, udev)"]
        Shell["Командная оболочка<br>(bash, zsh, sh)"]
        Utilities["Системные утилиты<br>(GNU coreutils: ls, cp, grep, top, sysctl)"]
  end
 subgraph LinuxKernel["Linux Kernel (Монолитное ядро)"]
    direction LR
        Kernel_Features["Планирование задач (CFS)<br>Управление памятью (MMU)<br>Сетевой стек (TCP/IP, Netfilter)<br>Виртуальная файловая система (VFS)<br>Системные вызовы (syscalls)<br>Модули ядра (драйверы устройств)"]
  end

    UserOS --> SystemComponents
    SystemComponents --> LinuxKernel

    Kernel_Features@{ shape: comment}
    Libraries@{ shape: braces}
    Init@{ shape: braces}
    Shell@{ shape: braces}
    Utilities@{ shape: braces}
    GUI@{ shape: comment}
    Distros@{ shape: hex}
    Apps@{ shape: hex}
```