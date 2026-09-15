---
aliases:
  - GLib
  - GNOME
  - dconf
  - gsettings
  - GVariant
  - Каталог настроек
  - Настройки GNOME
  - Хранилище конфигурации
---
# dconf и gsettings

dconf — низкоуровневая система хранения настроек, используемая GNOME и другими приложениями. gsettings — интерфейс командной строки для доступа к этим настройкам.

## Основные команды gsettings

**Просмотр всех настроек схемы:**

```bash
gsettings list-recursively org.gnome.desktop.interface
```

**Просмотр конфигураций всех приложений GNOME:**

```bash
gsettings list-recursively | grep -i monospace
```

**Получение одного значения:**

```bash
gsettings get org.gnome.desktop.interface font-name
```

**Установка значения:**

```bash
gsettings set org.gnome.desktop.interface font-name 'Ubuntu 11'
```

## dconf и gsettings

dconf — низкоуровневая система хранения настроек, используемая GNOME. gsettings — интерфейс командной строки для доступа к настройкам.

dconf хранит настройки напрямую в каталоге, а gsettings использует gio, чтобы обращаться к этим настройкам.

> Некоторые приложения требуют проверить, что настройки применяются на самом деле. Для этого можно использовать `dconf watch /path/to/key` — он показывает, какое приложение изменяет настройки.

**Как установить значение через dconf:**

```bash
dconf write /org/example/key 'value'
```

**Как сбросить значение через dconf:**

```bash
dconf reset /org/example/key
```

**Как прочитать значение через dconf:**

```bash
dconf read /org/example/key
```

## Команды gsettings для работы с настройками

**Список изменённых ключей (не значения по умолчанию):**

```bash
gsettings list-recursively | grep font
```

**Получить значение конкретного ключа:**

```bash
gsettings get org.gnome.desktop.interface cursor-size
```

**Установить значение:**

```bash
gsettings set org.gnome.desktop.interface cursor-size 30
```

**Искать по схемам:**

```bash
gsettings list-schemas | grep -i system
```

**Watch — просмотр изменений:**

```bash
gsettings monitor org.gnome.desktop.interface
```

**Reset — сброс ключа:**

```bash
gsettings reset org.gnome.desktop.interface font-name
```

## Использование dconf для тематизации

Если используется сторонняя тема и она ломает «официальный» вид, можно применить настройку через `dconf`:

```bash
dconf write /org/gnome/desktop/interface/gtk-theme 'Some Theme Name'
```

## Резервное копирование настроек dconf

**Резервное копирование всех настроек:**

```bash
dconf dump / > ~/dconf-backup.txt
```

**Резервное копирование конкретной папки:**

```bash
dconf dump /org/gnome/ > ~/dconf-gnome-backup.txt
```

**Формат:** текстовый формат `key=value`, похож на INI-файлы.

**Восстановление из резервной копии:**

```bash
dconf load / < ~/dconf-backup.txt
```

## Управление настройками через ключи и полные пути

Эти термины — то же самое: `path = key` при обращении через gsettings/dconf.

Если используется `dconf` — принято использовать полный путь от корня:

```bash
dconf read /org/gnome/desktop/interface/cursor-size
```

Если используется `gsettings` — прописывается схема и полный путь:

```bash
gsettings get org.gnome.desktop.interface cursor-size
```

В gsettings путь `org.gnome.desktop.interface/cursor-size` = путь dconf `/org/gnome/desktop/interface/cursor-size`.

## Версии dconf

**Версия командной строки dconf:**

```bash
dconf --version
```

**Версия бинарного файла gsettings:**

```bash
gsettings --version
```

**Версия библиотеки libdconf:**

```bash
pkg-config --modversion dconf
```

## UI Elements

**UI@font_name / symbol / unit**

Параметры `font_name`, `symbol` и `unit` — элементы интерфейса:

```text
@font_name 'Ubuntu 11'
@symbol λ
@unit px
```

Например:

```bash
gsettings set org.gnome.desktop.interface font-name "'Ubuntu 11'"
```

**`@font_name`** — имя шрифта. **`@symbol`** — символ. **`@unit`** — единица измерения.

## Схемы и профили gsettings

**Схемы** — типизированные объекты (`GLib.Variant`), которые используются в qsettings; хранятся в бинарном виде.

**Профили** — используются для отслеживания изменений настроек для дальнейшей синхронизации.

Чтобы RGBA-цвет можно было использовать в глобальной схеме, его нужно объявить в `.gschema.xml`.

**Схема для конкретного приложения ggsetting (пример):**

```xml
<schemalist>
  <schema>
    <key name="color" type="(ddd)">
      <default>(1.0, 0.0, 0.0)</default>
      <summary>Цвет фона</summary>
      <description>Цвет фона в формате RGB 0.0–1.0</description>
    </key>
  </schema>
</schemalist>
```

## Экспорт и импорт настроек

**Синхронизация настроек между машинами:**

```bash
dconf dump / > dconf-dump.txt
dconf load / < dconf-dump.txt
```

**Бэкап настроек GNOME Tools или приложений:**

```bash
dconf dump /org/gnome/tweaks/ > dconf-tweaks.txt
```

## Полный список специальных символов в dconf

| Символ | Описание |
| ------ | -------- |
| `@as` | массив строк |
| `@ay` | массив байтов |
| `@b` | булево значение |
| `@d` | число с плавающей точкой двойной точности |
| `@s` | строка |
| `@u` | беззнаковое целое |

**Пример:** значение `@b true` — булево `true`.

## Резюме по dconf

dconf и gsettings — мощные инструменты для управления конфигурацией GNOME. gsettings — более высокоуровневый интерфейс, удобный для большинства задач; dconf — низкоуровневый и гибкий, позволяет работать с точным путём ключа и выполнять резервное копирование.

- для простого изменения настроек используйте `gsettings set`;
- для сложных операций (бэкап, миграция, мониторинг) — `dconf dump`, `dconf load`, `dconf watch`.

**Итог: настройка GNOME через терминал — это просто, если знать пару команд. А dconf — это нижний слой, на котором всё это основано.**

## dconf для управления настройками GNOME

dconf может управлять настройками приложений GNOME, используя текстовый файл с парами `key=value` (INI-формат).

### Резервное копирование всех настроек

```bash
dconf dump / > dconf-backup-file.txt
```

### Восстановление настроек

```bash
dconf load / < dconf-backup-file.txt
```

### Экспорт настроек в файл

```bash
dconf dump / > dconf-export.txt
```

### Импорт настроек из файла

```bash
dconf load / < dconf-export.txt
```

### Мониторинг изменений настроек

```bash
dconf watch /
```

### Установка значения для конкретного приложения

```bash
dconf write /org/gnome/shell/app-picker-view 'icon-grid'
```

## GNOME gsettings

### Список всех изменённых настроек

```bash
gsettings list-changes org.gnome.desktop.interface
```

### Сброс всех настроек на значения по умолчанию

```bash
gsettings reset-recursively org.gnome.desktop.interface
```

### Экспорт настроек в файл

```bash
dconf dump /org/gnome/desktop/interface/ > gnome-interface-settings.conf
```

### Импорт настроек из файла

```bash
dconf load /org/gnome/desktop/interface/ < gnome-interface-settings.conf
```

## gsettings для работы с настройками GNOME

gsettings — команда для управления настройками приложений GNOME. Хранит настройки в dconf (бинарном backend 1:1 на диске).

### Путь записи и ключ

Путь к настройке:

```text
/org/gnome/desktop/interface/font-name
```

**Ключи** записываются с помощью `key.name`. Полный путь ключа — `/org/gnome/desktop/interface/font-name`.

### Просмотр всех ключей и значений

```bash
gsettings list-recursively
```

### Установка значения

```bash
gsettings set org.gnome.desktop.interface font-name 'Ubuntu 11'
```

### Получение значения

```bash
gsettings get org.gnome.desktop.interface font-name
```

### Сброс ключа на значение по умолчанию

```bash
gsettings reset org.gnome.desktop.interface font-name
```

### Список схем

```bash
gsettings list-schemas
```

### Поиск путей

```bash
gsettings list-relocatable-schemas
```

`relocatable-schemas` — схема, путь которой не зафиксирован (например, приложение может указать любой путь).

### Монитор изменений

```bash
gsettings monitor org.gnome.desktop.interface
```

При изменении настроек инструмент выводит:

```text
font-name: 'Ubuntu 12'
```

## Где хранятся настройки (dconf binary / каталог)

**dconf binary** — бинарный файл, в котором gsettings хранит настройки.

**Каталог** — каталог, в котором эти настройки сохраняются.

| Поле | dconf binary | Каталог |
| ---- | ------------ | ------- |
| Формат | бинарный | текстовый |
| Быстрота | быстрый | быстрее |
| Чтение | только чтение | чтение-запись |
| Просмотр | редактором | cat/Nano |
| Применение | gsettings | dconfed |

## Примеры использования настройки GNOME на примере resursor-size

Рекурсивный вывод настроек с фильтром:

```bash
gsettings list-recursively | grep -i cursur
```

Получение значений:

```bash
gsettings get org.gnome.desktop.interface cursor-size
dconf read /org/gnome/desktop/interface/cursor-size
```

Установка значения:

```bash
gsettings set org.gnome.desktop.interface cursor-size 30
dconf write /org/gnome/desktop/interface/cursor-size 30
```

**dconf-editor** — графический интерфейс для dconf, позволяет редактировать настройки визуально. В дистрибутивах ставится отдельно:

```bash
sudo apt install dconf-editor
```

## Работа с GNOME Shell Extensions

### Список всех дополнений

```bash
gnome-extensions list
```

### Включение (по имени)

```bash
gnome-extensions enable ['extension-name']
```

### Отключение

```bash
gnome-extensions disable ['extension-name']
```

### Перезагрузка GNOME Shell

```bash
alt+F2 → r → Enter
```

### Расширения GNOME Shell (библиотеки)

Вместо кнопки `extension-name` можно использовать `--extension-id`:

```bash
gnome-extensions enable --extension-id 307
```

### Изменение настроек расширений

```bash
gsettings set org.gnome.shell.extensions.[extension-name] [path] [value]
```

### Открытие расширения в браузере

```bash
xdg-open https://extensions.gnome.org/extension/307/extensions-organizer/
```

При настройке расширений используются стандартные правила изменения настроек приложений GNOME.

## Изменение конфигурации приложений через dconf/gsettings

Все настройки GNOME и приложений (например, Gedit, Nautilus) хранятся в dconf. Изменяются через:

```bash
gsettings set org.gnome.<app> <key> <value>
```

## Keyboard Shortcuts

### Просмотр всех клавиатурных сочетаний

```bash
gsettings list-recursively org.gnome.desktop.wm.keybindings
```

### Установка пользовательского сочетания

```bash
gsettings set org.gnome.desktop.wm.keybindings cycle-windows "['<Alt>Tab']"
```

### Сброс сочетаний на значения по умолчанию

```bash
gsettings reset-recursively org.gnome.desktop.wm.keybindings
```

## Автозагрузка приложений при старте GNOME

**Через gsettings (не рекомендуется):**

```bash
gsettings set org.gnome.desktop.session idle-delay 0
```

**Через startup-programs:**

```bash
org.gnome.desktop.background: picture-uri
```

**Через файлы .desktop в ~/.config/autostart:**

```text
~/.config/autostart/application.desktop
```

**Пример файла:**

```ini
[Desktop Entry]
Type=Application
Name=Example
Exec=example
X-GNOME-Autostart-enabled=true
```

## Дополнительные команды dconf и gsettings

- проверить, что все изменения применились, без перезагрузки — `dconf watch`;
- сделать дамп всех настроек — `dconf dump /`;
- сбросить настройки приложения — `gsettings reset-recursively`;

**Настройка GNOME через терминал — это просто, если знать пару команд, а dconf — нижний слой, на котором всё основано.**
