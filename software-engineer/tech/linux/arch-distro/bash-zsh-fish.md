---
aliases:
  - bash
  - fish
  - zsh
---
## fish

В Arch Linux установка и настройка оболочки **Fish** (Friendly Interactive Shell) выполняется через стандартный менеджер пакетов `pacman`. После установки вам потребуется настроить конфигурацию и, при необходимости, установить менеджер плагинов.

Вот краткое руководство по установке и первоначальной настройке.

### 📦 Установка Fish

Установите пакет `fish` из официального репозитория `extra`:

```bash
sudo pacman -S fish
```

### ⚙️ Настройка Fish в качестве основной оболочки

Чтобы Fish стала оболочкой по умолчанию при входе в систему, измените вашу оболочку по умолчанию.

```bash
chsh -s $(which fish)
```

> **Важно:** Эта команда изменит оболочку для вашего пользователя. Вам нужно будет выйти из системы и зайти снова, или перезапустить терминал, чтобы изменения вступили в силу.

### 🐟 Базовая настройка Fish

Конфигурационный файл Fish находится по пути `~/.config/fish/config.fish`. Если его нет, вы можете его создать.

**1. Настройка `$PATH`:**

Для добавления путей в `$PATH` в Fish используется команда `fish_add_path`. Например, чтобы добавить папку `~/.local/bin`:

```bash
fish_add_path -m ~/.local/bin
```

Чтобы сохранить это изменение навсегда, добавьте эту команду в ваш `~/.config/fish/config.fish`.

**2. Отключение приветствия:**

По умолчанию Fish показывает приветственное сообщение. Чтобы его отключить, выполните команду:

```bash
set -U fish_greeting
```

Эта команда установит универсальную переменную `fish_greeting` в пустое значение, отключая приветствие во всех сессиях.

**3. Настройка внешнего вида:**

Вы можете использовать встроенную веб-утилиту для настройки цветов и темы:

```bash
fish_config
```

Это откроет в браузере интерфейс, где вы сможете выбрать и применить тему.

### 🧩 Менеджеры пакетов и плагины

Один из способов управления плагинами для Fish — использовать **Fisher**. Это популярный менеджер плагинов.

**Установка Fisher:**

Выполните команду в терминале:

```bash
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher
```

Теперь вы можете устанавливать плагины с помощью Fisher:

```bash
fisher install ИМЯ_ПЛАГИНА
```

Примеры плагинов:

- `fisher install PatrickF1/fzf.fish` — интеграция с инструментом `fzf` для поиска.
- `fisher install jorgebucaran/autopair.fish` — автоматическое закрытие скобок и кавычек.

### ✍️ Создание пользовательских функций

Вы можете создавать свои собственные команды (функции). Для этого поместите файлы с именем `ИМЯ_ФУНКЦИИ.fish` в папку `~/.config/fish/functions/`. Fish автоматически загрузит их при запуске.

## zsh

Чтобы установить Zsh в Arch Linux, выполните следующие шаги:

### Установка Zsh

Используйте пакетный менеджер `pacman` для установки Zsh:

```bash
sudo pacman -S zsh
```

После установки проверьте, что Zsh установлен корректно, запустив его в терминале:

```bash
zsh
```

При первом запуске должен появиться скрипт `zsh-newuser-install`, который проведёт вас через базовую настройку. Если вы хотите пропустить эту настройку, нажмите `q`. Если скрипт не запустился автоматически, вызовите его вручную:

```bash
autoload -Uz zsh-newuser-install
zsh-newuser-install -f
```

**Важно:** для работы `zsh-newuser-install` размер терминала должен быть не менее 72×15 символов. [```1```](https://wiki.archlinux.org/title/Zsh_%28%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9%29)[```2```](https://wiki.archlinux.org/title/Zsh)

### Установка дополнительных пакетов

Для расширения возможностей автодополнения команд рекомендуется установить пакет `zsh-completions`:

```bash
sudo pacman -S zsh-completions
```

### Смена оболочки по умолчанию

Чтобы сделать Zsh оболочкой по умолчанию, используйте команду `chsh`:

```bash
chsh -s /bin/zsh
```

После этого потребуется выйти из текущей сессии и войти снова, чтобы изменения вступили в силу. Проверить текущую оболочку можно командой:

```bash
echo $SHELL
```

Если после смены оболочки в выводе всё ещё указана старая оболочка, попробуйте перезагрузить систему. [```3```](https://davidtsadler.com/posts/arch/2020-09-07/installing-zsh-and-powerlevel10k-on-arch-linux/)[```4```](https://www.linuxfordevices.com/tutorials/linux/make-arch-terminal-awesome)

### Настройка

Конфигурационные файлы Zsh обычно располагаются в домашнем каталоге пользователя. Основные файлы:

- `~/.zshrc` — для интерактивных оболочек;
- `~/.zprofile` — для оболочек входа.

Для более тонкой настройки можно ознакомиться с руководством пользователя Z-Shell, где подробно описаны интерактивные оболочки, оболочки входа и файлы запуска. [```1```](https://wiki.archlinux.org/title/Zsh_%28%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9%29)

### Дополнительные возможности

- **Плагины и темы.** Можно установить популярные плагины (например, `zsh-syntax-highlighting` для подсветки синтаксиса или `zsh-autosuggestions` для предложений команд) и темы оформления (например, Powerlevel10k).
- **Обработка неизвестных команд.** С помощью утилиты `pkgfile` можно настроить поиск команд в официальных репозиториях при вводе неизвестной команды. Для этого добавьте в `~/.zshrc` строку `source /usr/share/doc/pkgfile/command-not-found.zsh` и синхронизируйте базу данных `pkgfile`. [```1```](https://wiki.archlinux.org/title/Zsh_%28%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9%29)

Если вам нужна дополнительная информация по настройке Zsh, обратитесь к официальной документации ArchWiki или другим руководствам.

Powerlevel10k — это высокопроизводительная и гибкая тема для Zsh, которая позволяет настроить внешний вид и функциональность командной строки. Для настройки можно использовать встроенный мастер конфигурации или редактировать конфигурационный файл вручную. [```7```](https://vk.com/wall-226121191_166)[```5```](https://z.niceos.ru/blog/cto-takoe-powerlevel10k-i-kak-ego-ustanovit)

### Установка Powerlevel10k

Если вы используете Oh My Zsh, выполните следующие шаги:

1. Клонируйте репозиторий Powerlevel10k в каталог тем Oh My Zsh:
   ```bash
   git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
   ```
2. Откройте файл `~/.zshrc` и замените строку, задающую тему, на:
   ```bash
   ZSH_THEME="powerlevel10k/powerlevel10k"
   ```
3. Перезагрузите конфигурацию Zsh:
   ```bash
   source ~/.zshrc
   ```

Если Oh My Zsh не используется, можно клонировать репозиторий напрямую в локальную директорию и подключить файл темы из `.zshrc`. [```3```](https://alexhost.com/ru/faq/installation-and-usage-of-powerlevel10k-in-zsh/)

### Запуск мастера настройки

После активации темы запустите мастер настройки:

```bash
p10k configure
```

Если мастер не запустился автоматически, выполните эту команду вручную. Мастер проведёт вас через серию вопросов, предлагая выбрать варианты настроек, например:

- тип шрифта;
- стиль отображения времени выполнения команд;
- отображение статуса Git-репозитория и другие параметры. [```7```](https://vk.com/wall-226121191_166)

На основе ваших ответов мастер создаст файл `~/.p10k.zsh`, который содержит настройки темы. [```1```](https://github.com/romkatv/powerlevel10k)[```6```](https://github.com/romkatv/powerlevel10k/blob/master/README.md)

### Ручная настройка через `~/.p10k.zsh`

Файл `~/.p10k.zsh` содержит множество комментариев, которые помогут сориентироваться в опциях конфигурации. В нём можно:

- добавлять или удалять сегменты командной строки (например, отображение текущего пользователя, статуса Git, информации о системе);
- изменять цвета, иконки и поведение сегментов в зависимости от контекста. [```5```](https://z.niceos.ru/blog/cto-takoe-powerlevel10k-i-kak-ego-ustanovit)

Пример содержимого файла:

```bash
# Generated by Powerlevel10k configuration wizard on 2024-06-04 at 12:34 UTC.
# Based on romkatv/powerlevel10k/config/p10k-classic.zsh, checksum 1234567890abcdef.
# Wizard options: nerdfont-complete + powerline+awesome-patched font, small icons.

# P10K prompt customization options.
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon dir vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status command_execution_time background_jobs time)

# os_icon: Show OS icon.
POWERLEVEL9K_OS_ICON_BACKGROUND='black'
POWERLEVEL9K_OS_ICON_FOREGROUND='white'

# dir: Show current directory.
POWERLEVEL9K_DIR_BACKGROUND='blue'
POWERLEVEL9K_DIR_FOREGROUND='white'

# vcs: Show git status.
POWERLEVEL9K_VCS_BACKGROUND='green'
POWERLEVEL9K_VCS_FOREGROUND='black'

# status: Show status of the last command.
POWERLEVEL9K_STATUS_BACKGROUND='red'
POWERLEVEL9K_STATUS_FOREGROUND='white'

# command_execution_time: Show command execution time.
POWERLEVEL9K_COMMAND_EXECUTION_TIME_BACKGROUND='magenta'
POWERLEVEL9K_COMMAND_EXECUTION_TIME_FOREGROUND='white'

# background_jobs: Show number of background jobs.
POWERLEVEL9K_BACKGROUND_JOBS_BACKGROUND='cyan'
POWERLEVEL9K_BACKGROUND_JOBS_FOREGROUND='black'

# time: Show current time.
POWERLEVEL9K_TIME_BACKGROUND='yellow'
POWERLEVEL9K_TIME_FOREGROUND='black'
```

### Дополнительные настройки

**Шрифты.** Для корректного отображения иконок и символов в Powerlevel10k рекомендуется установить шрифт Nerd Font, который поддерживает расширенные глифы. Один из таких шрифтов — MesloLGS NF. После установки выберите этот шрифт в настройках вашего терминала. [```8```](https://dev.to/pratik_kale/customise-your-terminal-using-zsh-powerlevel10k-1og5)[```5```](https://z.niceos.ru/blog/cto-takoe-powerlevel10k-i-kak-ego-ustanovit)

**Мгновенный промпт.** Powerlevel10k поддерживает мгновенный промпт, который ускоряет загрузку командной строки. Для его включения добавьте в `~/.zshrc` следующий код:

```bash
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi
```

Важно: любой код в `.zshrc`, который записывает в стандартный вывод до инициализации мгновенного промпта, вызовет предупреждение. Перенесите такой код после строки `source ~/.p10k.zsh` или подавите его вывод во время инициализации мгновенного промпта. [```3```](https://alexhost.com/ru/faq/installation-and-usage-of-powerlevel10k-in-zsh/)

**Сегменты с условием отображения.** В конфигураторе по умолчанию для нескольких сегментов промпта активируется опция `show on command`. Это означает, что такие сегменты будут отображаться только при вводе определённых команд. Чтобы настроить это поведение, откройте `~/.p10k.zsh`, найдите параметры с `SHOW_ON_COMMAND` и либо удалите их (чтобы сегменты отображались всегда), либо измените значения. [```6```](https://github.com/romkatv/powerlevel10k/blob/master/README.md)

Если вам нужна дополнительная помощь или возникнут проблемы, обратитесь к официальной документации Powerlevel10k или создайте issue на GitHub. [```1```](https://github.com/romkatv/powerlevel10k)
