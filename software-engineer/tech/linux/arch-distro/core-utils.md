---
aliases:
  - Arch Linux
  - BusyBox
  - bat
  - Core utilities
  - core-utils
  - fd
  - fzf
  - GNU Core Utilities
  - GNU coreutils
  - Heirloom Toolchest
  - Linux Core Utilities
  - POSIX
  - POSIX utilities
  - Toybox
  - Unix
  - eza
  - linux core utils
  - moreutils
  - plocate
  - ripgrep
  - sbase
  - uutils
  - zoxide
  - 9base
  - Базовые утилиты Linux
  - Основные утилиты Linux
  - Утилиты POSIX
---
## Core utilities из Arch Wiki

**Core utilities** — это базовые, фундаментальные инструменты системы GNU/Linux. Статья даёт неполный обзор этих утилит, ссылки на их документацию и полезные альтернативы. Входит, но не ограничивается, [GNU Core Utilities]. Большинство core utilities — традиционные инструменты Unix, многие стандартизированы POSIX, но были развиты дальше для предоставления большего числа возможностей.

Большинство интерфейсов командной строки документировано в man-страницах, утилиты проекта GNU документируются в первую очередь в Info-руководствах, некоторые оболочки предоставляют команду `help` для встроенных команд оболочки. Кроме того, большинство утилит печатают информацию об использовании при запуске с флагом `--help`.

### Essentials

Каталог важных утилит, которые должны быть знакомы пользователям Arch Linux.

| Package | Utility | Description | Documentation | Alternatives |
| ------- | ------- | ----------- | ------------- | ------------ |
| shell built-ins | cd | change directory | cd(1p) | cd alternatives |
| GNU coreutils | ls | list directory | ls(1), info | tree, ls alternatives |
| GNU coreutils | cat | concatenate files to stdout | cat(1), info | tac(1), cat alternatives |
| GNU coreutils | mkdir | make directory | mkdir(1), info | — |
| GNU coreutils | rmdir | remove empty directory | rmdir(1), info | — |
| GNU coreutils | rm | remove files or directories | rm(1), info | shred, unlink(1) |
| GNU coreutils | cp | copy files or directories | cp(1), info | cp alternatives |
| GNU coreutils | mv | move files or directories | mv(1), info | — |
| GNU coreutils | ln | make hard or symbolic links | ln(1), info | sln(8) (soname recovery) |
| GNU coreutils | chown | change file owner and group | chown(1), info | chgrp(1) |
| GNU coreutils | chmod | change file permissions | chmod(1), info | — |
| GNU coreutils | dd | convert and copy a file | dd(1), info | dd alternatives |
| GNU coreutils | df | report file system disk space usage | df(1), info | df alternatives |
| GNU coreutils | du | estimate disk space used by files and directories | du(1), info | du alternatives |
| GNU tar | tar | tar archiver | tar(1), info | archivers |
| GNU less | less | terminal pager | less(1) | terminal pagers |
| GNU findutils | find | search files or directories | find(1), info, GregsWiki | find alternatives |
| GNU diffutils | diff | compare files line by line | diff(1), info | diff alternatives |
| GNU grep | grep | print lines matching a pattern | grep(1), info | grep alternatives |
| GNU sed | sed | stream editor | sed(1), info, one-liners | sad, sd |
| GNU AWK (gawk) | AWK | pattern scanning and processing language | gawk(1), info, one-liners | alternative implementations |
| util-linux | dmesg | print or control the kernel ring buffer | dmesg(1) | systemd journal |
| util-linux | lsblk | list block devices | lsblk(8) | — |
| util-linux | mount | mount a filesystem | mount(8) | — |
| util-linux | umount | unmount a filesystem | umount(8) | — |
| util-linux | su | substitute user | su(1) | sudo, doas |
| procps-ng | kill | terminate a process | kill(1) | pkill(1), killall(1) |
| procps-ng | pgrep | look up processes by name or attributes | pgrep(1) | pidof(1) |
| procps-ng | ps | show information about processes | ps(1) | top(1), system monitors |
| procps-ng | free | display amount of free and used memory | free(1) | — |

#### Preventing data loss

`rm`, `mv`, `cp` и перенаправления оболочки безоговорочно удаляют или перезаписывают файлы. `rm`, `mv` и `cp` поддерживают флаг `-i` для запроса подтверждения перед каждым удалением или перезаписью. Некоторые пользователи включают `-i` по умолчанию через алиасы. Полагаться на эти опции опасно: привыкнув к ним, легко потерять данные при работе на системе, где они не настроены. Лучший способ предотвратить потерю данных — создание резервных копий.

### Nonessentials

Часто полезные core utilities.

| Package | Utility | Description | Documentation | Alternatives |
| ------- | ------- | ----------- | ------------- | ------------ |
| shell built-ins | alias | define or display aliases | alias(1p) | — |
| shell built-ins | type | print the type of a command | type(1p) | command(1p), whereis(1), which(1) |
| shell built-ins | time | time a command | time(1p) | — |
| GNU coreutils | tee | read stdin and write to stdout and files | tee(1), info | pee(1) |
| GNU coreutils | mktemp | make a temporary file or directory | mktemp(1), info | — |
| GNU coreutils | mknod | create named pipe or device node | mknod(1), mkfifo(1), info | — |
| GNU coreutils | truncate | shrink or extend the size of a file | truncate(1), info | fallocate(1) |
| GNU coreutils | basenc | encoding input and output it | basenc(1), base64(1), info | — |
| GNU coreutils | cut | print selected parts of lines | cut(1), info | colrm(1), hck, choose |
| GNU coreutils | tr | translate or delete characters | tr(1), info | uconv(1) |
| GNU coreutils | od | dump files in octal and other formats | od(1), info | hexdump(1), vim, xxd(1) |
| GNU coreutils | sort | sort lines | sort(1), info | — |
| GNU coreutils | uniq | report or omit repeated lines | uniq(1), info | anewer, runiq, huniq-git |
| GNU coreutils | comm | compare two sorted files line by line | comm(1), info | zet |
| GNU coreutils | head | output the first part of files | head(1), info | — |
| GNU coreutils | join | join lines of two inputs on a common field | join(1), info | combine(1), zet |
| GNU coreutils | md5sum | calculate cryptography hash functions of inputs and output | sha256sum(1), sha512sum(1), info | shasum(1), rhash(1) |
| GNU coreutils | tail | output the last part of files, or follow files | tail(1), info | — |
| GNU coreutils | wc | print newline, word and byte count | wc(1), info | — |
| GNU binutils | strings | print printable characters in binary files | strings(1), info | stringsext |
| util-linux | column | columnate file, optionally pretty-printing in table with grid | column(1) | paste(1), csview |
| GNU findutils | xargs | combine or template arguments from stdin to invoke external command | xargs(1) | parallel(1), parallel_alternatives(7) |
| GNU glibc | iconv | convert character encodings | iconv(1) | recode, uconv(1) |
| GNU sharutils | uuencode | encode file into email friendly text | uuencode(1), uudecode(1), info | uudeview(1) |
| file | file | guess file type | file(1) | — |

Пакет moreutils предоставляет полезные инструменты, такие как `sponge(1)`, отсутствующие в GNU coreutils.

### Alternatives

Альтернативные core utilities предоставляются следующими пакетами:

- **9base** — порт различных оригинальных инструментов Plan9 на unix.
- **BusyBox** — утилиты для rescue- и встраиваемых систем.
- **Heirloom Toolchest** — традиционные реализации стандартных утилит Unix.
- **sbase** — suckless-вариант *nix core utilities.
- **Toybox** — «всё-в-одном» командная строка Linux.
- **ubase** — расширение утилит sbase.
- **uutils** — кроссплатформенная Rust-переписанная версия GNU coreutils.

#### cat alternatives

- **bat** — клон cat с подсветкой синтаксиса и интеграцией с Git.

#### cd alternatives

- **autojump** — быстрый способ навигации по файловой системе из командной строки.
- **zoxide** — умный cd, который запоминает привычки и позволяет переходить в любое место в несколько нажатий.

См. также Bash: авто"cd" при вводе пути и Zsh: запоминание недавних директорий.

#### date alternatives

- **dateutils** — удобные утилиты для расчёта и конвертации дат в командной строке.
- **pdd** — крошечный калькулятор разницы дат.

#### cp alternatives

Использование rsync как альтернативы cp/mv позволяет возобновить прерванную передачу, показать статус передачи, пропустить уже существующие файлы и проверить целостность файлов назначения с помощью чексумм.

#### ls alternatives

- **broot** — новый способ просмотра и навигации по деревьям директорий.
- **clifm** — файловый менеджер, который может выводить списки как ls(1) (плюс иконки и RGB-цвета).
- **eza** — замена ls с поддержкой цвета, tree view, интеграцией с git. Основан на exa, который больше не поддерживается.
- **lsd** — современный ls с множеством цветов и иконок.

#### find alternatives

- **fd** — простая и быстрая альтернатива find. Игнорирует скрытые файлы и файлы из `.gitignore` по умолчанию.
- **fuzzy-find** — нечёткое дополнение для поиска файлов.
- **plocate** — гораздо более быстрый locate.
- **rawhide** — поиск файлов с использованием C-подобных выражений.
- **uutils-findutils** — Rust-переписанная версия findutils.

Для графических поисковиков файлов см. List of applications/Utilities — File searching.

#### diff alternatives

- **uutils-diffutils** — Rust-переписанная версия diffutils.

Так как diffutils не предоставляет пословного сравнения, другие программы делают это:

- **cwdiff** — обёртка GNU wdiff с цветным выводом.
- **dwdiff** — фронтенд пословного diff с поддержкой цветов.
- **git diff** — умеет делать пословный diff с `--color-words`, через `--no-index` работает и с файлами вне Git-деревьев.
- **git-delta** — пейджер для git, diff и grep output с подсветкой синтаксиса.
- **icdiff** — цветной diff, написанный на Python. «Improved color diff» дополняет обычный diff.
- **wdiff** — пословная реализация GNU diff без поддержки цветов.

См. также List of applications/Utilities — Comparison, diff, merge.

#### grep alternatives

- **mgrep** — многострочный grep.
- **pdfgrep** — инструмент поиска текста в PDF-файлах.
- **ripgrep-all** — поиск в plain text, а также в PDF, электронных книгах, офисных документах, zip, tar.gz.

##### Code searchers

Эти инструменты заменяют grep для поиска по коду. Рекурсивный поиск по умолчанию, пропуск бинарных файлов и уважение `.gitignore`.

- **ack** — замена grep на Perl, ориентированная на большие деревья разнородного исходного кода.
- **pcre2grep** — grep, совместимый с Perl, использует библиотеку регулярных выражений PCRE2.
- **ripgrep (rg)** — поисковый инструмент, сочетающий удобство ag со скоростью grep.
- **The Silver Searcher (ag)** — код поисковый инструмент, похожий на ack, но быстрее.
- **ugrep (ug)** — сверхбыстрый grep с интерактивным интерфейсом, fuzzy search, boolean query, hexdumps.

См. также: cscope.

##### Interactive filters

- **fnf** — интерактивный fuzzy finder для терминала.
- **fzf** — универсальный терминальный fuzzy finder.
- **fzy** — быстрый, простой fuzzy text selector с улучшенным алгоритмом оценки.
- **peco** — простой интерактивный инструмент фильтрации.
- **percol** — добавляет интерактивную фильтрацию к традиционному pipeline UNIX.
- **skim** — fuzzy finder на Rust, похожий на fzf.

#### dd alternatives

См. также: dd и ddrescue.

##### Alternative dd implementations

Реализации _dd_, чей интерфейс и поведение по умолчанию в основном соответствуют спецификации POSIX.

- **ddpt** — переносимая переписанная версия sg_dd(8) от мейнтейнера подсистемы SCSI ядра Linux, с поддержкой специализированного аппаратного ввода-вывода (SCSI command sets) и многими другими функциями.
- **sdd** — реализация dd, переносимая между UNIX-окружениями, от Joerg Schilling, умеет проверять контрольные суммы скопированных данных и повторять чтение повреждённых блоков.

###### Fork GNU dd

GNU-реализация _dd_ из coreutils также соответствует POSIX. Эта подсекция перечисляет её форки.

- **dc3dd** — патченная версия GNU dd от отдела киберкриминалистики Минобороны США (DC3) с целями, похожими на dcfldd.
- **dcfldd** — форк GNU dd для криминалистики и сценариев безопасности, включает попутное хеширование, гибкие затирания, контроль записи, вывод на несколько целей одновременно, split и piped output.

##### Modernised dd analogues

Аналоги _dd_, не соответствующие POSIX (по JCL-подобному синтаксису командной строки и поведению по умолчанию).

- **dd_rescue** — модернизированный, нагруженный функционалом аналог dd, подходящий для повседневных сценариев, клонирования дисков и восстановления данных.
- **rw** — минимальный и переносимый аналог _dd_ с обычными флагами командной строки.

##### buffer spin-offs

Форки buffer — обобщённой утилиты буферизации ввода-вывода, похожей на _dd_, но с динамическим буфером. Поддерживает блочный ввод-вывод и применяется при работе с LTO-лентами для избежания «shoe shining».

- **mbuffer** — продолжение утилиты buffer с threading и другими функциями.

#### df alternatives

- **duf** — утилита использования/освобождения диска.

#### du alternatives

- **cdu** — обёртка du с цветами и симпатичной гистограммой.
- **dua** — быстрый анализатор использования диска, поддерживает удаление файлов, написан на Rust.
- **dust** — более интуитивная версия du на Rust.
- **gdu** — анализатор использования дисков с консольным интерфейсом, написан на Go.
- **ncdu** — чрезвычайно лёгкий и простой анализатор использования дисков на основе ncurses, написан на Zig.

См. также List of applications/Utilities — Disk usage display.

### POSIX shell utilities

Многие распространённые пакеты уже устанавливают большинство популярных POSIX-утилит как зависимости, но метапакет `posix` можно установить, чтобы гарантировать их наличие.

Кроме обязательных утилит есть метапакеты для некоторых опциональных категорий:

- posix-c-development
- posix-software-development
- posix-user-portability
- posix-xsi

**Примечание:** не все опциональные утилиты из данной категории обязательно присутствуют в соответствующем метапакете.

### Tips and tricks

#### Override or add missing coreutils

Некоторые команды (`arch`, `kill` и др.) отсутствуют в coreutils или берутся из других пакетов. Для совместимости установите uutils-coreutils и выполните:

```bash
# ln -sf /usr/bin/uu-coreutils /usr/local/bin/arch
# echo -e "#compdef arch=uu-arch\n_uu-arch" > /usr/local/share/zsh/site-functions/_arch
# echo "complete -c arch -w uu-arch" > /usr/local/share/fish/vendor_completions.d/arch.fish
```

### See also

- GNU Coreutils documentation
- GNU Coreutils FAQ
- Coreutils Gotchas: заметки мейнтейнера GNU coreutils о спорном поведении некоторых компонентов
- POSIX utilities
