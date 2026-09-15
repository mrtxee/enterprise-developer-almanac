---
aliases:
  - bash
  - fish
  - Fisher
  - Oh My Zsh
  - p10k
  - Powerlevel10k
  - zsh
---

## fish

В Arch Linux установка и настройка оболочки Fish (Friendly Interactive Shell) выполняется через пакетный менеджер `pacman`.

### Установка fish

Пакет `fish` находится в официальном репозитории `extra`:

```bash
sudo pacman -S fish
```

### Настройка fish в качестве основной оболочки

Чтобы fish стала оболочкой по умолчанию:

```bash
chsh -s $(which fish)
```

> Команда изменит оболочку для текущего пользователя. Нужно выйти из системы и войти снова или перезапустить терминал, чтобы изменения вступили в силу.

### Базовая настройка fish

Конфигурационный файл fish находится по пути `~/.config/fish/config.fish` (создаётся при необходимости).

**Настройка `$PATH`**

Для добавления путей в `$PATH` используется команда `fish_add_path`. Например, чтобы добавить папку `~/.local/bin`:

```bash
fish_add_path -m ~/.local/bin
```

Чтобы сохранить изменение навсегда, эта команда добавляется в `~/.config/fish/config.fish`.

**Отключение приветствия**

По умолчанию fish показывает приветственное сообщение. Чтобы его отключить:

```bash
set -U fish_greeting
```

Команда устанавливает универсальную переменную `fish_greeting` в пустое значение, отключая приветствие во всех сессиях.

**Настройка внешнего вида**

Встроенная веб-утилита настраивает цвета и тему:

```bash
fish_config
```

Команда открывает в браузере интерфейс, где можно выбрать и применить тему.

### Менеджеры пакетов и плагины

Для управления плагинами fish используется менеджер **Fisher**.

**Установка Fisher:**

```bash
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher
```

**Установка плагинов:**

```bash
fisher install ИМЯ_ПЛАГИНА
```

Примеры плагинов:

- `fisher install PatrickF1/fzf.fish` — интеграция с инструментом `fzf` для поиска
- `fisher install jorgebucaran/autopair.fish` — автоматическое закрытие скобок и кавычек

### Создание пользовательских функций

Свои команды (функции) создаются как файлы с именем `ИМЯ_ФУНКЦИИ.fish` в папке `~/.config/fish/functions/`. Fish автоматически загружает их при запуске.

---

## zsh

### Установка Zsh

```bash
sudo pacman -S zsh
```

Для проверки установки запустите `zsh`. При первом запуске появляется скрипт `zsh-newuser-install`, который проводит через базовую настройку. Чтобы пропустить настройку, нажмите `q`. Если скрипт не запустился автоматически, вызовите его вручную:

```bash
autoload -Uz zsh-newuser-install
zsh-newuser-install -f
```

> Для работы `zsh-newuser-install` размер терминала должен быть не менее 72×15 символов.

### Установка дополнительных пакетов

Для расширения возможностей автодополнения команд рекомендуется установить `zsh-completions`:

```bash
sudo pacman -S zsh-completions
```

### Смена оболочки по умолчанию

```bash
chsh -s /bin/zsh
```

После этого потребуется выйти из сессии и войти снова. Проверить текущую оболочку:

```bash
echo $SHELL
```

Если в выводе всё ещё указана старая оболочка, перезагрузите систему.

### Настройка

Конфигурационные файлы Zsh располагаются в домашнем каталоге пользователя:

- `~/.zshrc` — для интерактивных оболочек;
- `~/.zprofile` — для оболочек входа.

Более тонкая настройка описана в [ArchWiki: Zsh (русский)](https://wiki.archlinux.org/title/Zsh_%28%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9%29).

### Дополнительные возможности

- **Плагины и темы.** Популярные плагины: `zsh-syntax-highlighting` (подсветка синтаксиса), `zsh-autosuggestions` (предложения команд); популярная тема — Powerlevel10k.
- **Обработка неизвестных команд.** С помощью утилиты `pkgfile` выполняется поиск команды в официальных репозиториях при вводе неизвестной команды. Для этого добавьте в `~/.zshrc` строку `source /usr/share/doc/pkgfile/command-not-found.zsh` и синхронизируйте базу данных `pkgfile`.

### Powerlevel10k

Powerlevel10k — высокопроизводительная и гибкая тема для Zsh. Она настраивается через встроенный мастер конфигурации или редактированием конфигурационного файла вручную. Официальный репозиторий: [Powerlevel10k](https://github.com/romkatv/powerlevel10k).

**Установка Powerlevel10k через Oh My Zsh:**

1. Клонируйте репозиторий в каталог тем Oh My Zsh:
   ```bash
   git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
   ```
2. В файле `~/.zshrc` замените строку, задающую тему:
   ```bash
   ZSH_THEME="powerlevel10k/powerlevel10k"
   ```
3. Перезагрузите конфигурацию Zsh:
   ```bash
   source ~/.zshrc
   ```

Если Oh My Zsh не используется, репозиторий клонируется в локальную директорию и файл темы подключается из `.zshrc`.

**Запуск мастера настройки**

```bash
p10k configure
```

Мастер проводит через серию вопросов — например:

- тип шрифта;
- стиль отображения времени выполнения команд;
- отображение статуса Git-репозитория и другие параметры.

На основе ответов мастер создаст файл `~/.p10k.zsh` с настройками темы.

**Ручная настройка через `~/.p10k.zsh`**

Файл содержит множество комментариев. В нём можно:

- добавлять или удалять сегменты командной строки (например, текущий пользователь, статус Git, информация о системе);
- изменять цвета, иконки и поведение сегментов в зависимости от контекста.

Пример содержимого:

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

### Дополнительные настройки Powerlevel10k

**Шрифты**

Для корректного отображения иконок в Powerlevel10k рекомендуется шрифт Nerd Font с поддержкой расширенных глифов (например, MesloLGS NF). После установки шрифт выбирается в настройках терминала.

**Мгновенный промпт**

Powerlevel10k поддерживает мгновенный промпт, который ускоряет загрузку командной строки. Для включения добавьте в `~/.zshrc`:

```bash
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi
```

> Любой код в `.zshrc`, который записывает в стандартный вывод до инициализации мгновенного промпта, вызовет предупреждение. Такой код переносится после строки `source ~/.p10k.zsh` или его вывод подавляется во время инициализации мгновенного промпта.

**Сегменты с условием отображения**

В конфигураторе по умолчанию для нескольких сегментов промпта активируется опция `show on command` — такие сегменты отображаются только при вводе определённых команд. Чтобы изменить поведение, откройте `~/.p10k.zsh`, найдите параметры с `SHOW_ON_COMMAND` и либо удалите их (сегменты будут отображаться всегда), либо измените значения.
