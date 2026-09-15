---
aliases:
  - bash
  - cli
  - docker
  - docker-compose
  - git
  - grep
  - kafka
  - keytool
  - maven
  - mvn
  - PowerShell
  - ssh
  - ssh config
  - ssh-keygen
  - Командная строка
---

# CLI

## SSH config

Файл **`~/.ssh/config`** нужен, чтобы закешировать «адресную книгу» с доступами к разным SSH-хостам.

По умолчанию конфигурационный файл SSH может не существовать, поэтому его нужно создать командой `touch ~/.ssh/config`.

**Пример конфигурации**

```text
Host hostname1
    SSH_OPTION value
    SSH_OPTION value
Host hostname2
    HostName dev.example.com
    User john
    Port 2322
    IdentityFile ~/.ssh/targaryen.key
Host *ell
    user oberyn
Host * !martell
    LogLevel INFO
    Compression yes
    LogLevel INFO
Host *
    SSH_OPTION value
```

Потом в терминале можно инициировать подключение, например, `ssh hostname2`.

- Чтобы использовать все остальные опции, но подключиться как пользователь `root` вместо `john`, достаточно указать пользователя в командной строке:

  ```text
  ssh -o "User=root" dev
  ```

- Опция `-F` (`configfile`) позволяет указать альтернативный конфигурационный файл для пользователя. Чтобы клиент `ssh` игнорировал все опции конфигурационного файла:

  ```text
  ssh -F /dev/null user@example.com
  ```

## Bash

### Версия Linux

```bash
cat /etc/os-release
uname -a
```

### Работа с файлами

- `ls -la` — все файлы списком в столбик.
- `mc` — midnight commander, работа с файловой системой в интерфейсе.
- `cat` — прочитать содержимое файла.
- `pwd` — текущая папка, адрес в файловой системе.
- `rm` — удалить файл; `rm -rfv` — рекурсивно (recursive), принудительно (force), подробно (verbose).
- `touch file1 file2 file3` — обновить временные метки существующих файлов, а также создать новые пустые файлы.
- `stat <file_name>` — показать статус файла, включая временные метки.

### ssh-keygen: смена пароля приватного ключа

`ssh-keygen -p -P oldPw -N newPw -f /path/to/private_key`

### grep: фильтр по регулярным выражениям

Search **g**lobally for lines matching the **r**egular **e**xpression, and **p**rint them.

`grep [опции] шаблон [<путь к файлу или папке>]`

Можно применить фильтр к стандартному выводу другой команды:

`команда | grep [опции] шаблон`

```bash
grep --version | grep grep
# grep (GNU grep) 2.5.1-FreeBSD
grep "^[a-zA-Z]" pgm.s
```

### Переменные

```bash
#!/bin/bash
# данные в переменную
CUR_DIR=$(pwd)
# вставить переменную
COMPILE_OPTS="-D maven.repo.local=${CUR_DIR}"
# конкатенация переменной
COMPILE_OPTS="${COMPILE_OPTS} -llr"
```

### Vim

Связанная тема: [[vim]].

## SSH

SSH — Secure Shell — протокол удалённого управления компьютером с операционной системой Linux.

- Настройки сервера SSH находятся в файлах `/etc/ssh/sshd_config`, `/etc/ssh/sshd.conf`.
- По умолчанию SSH работает на порту 22.

Основные опции клиента `ssh`:

- **f** — перевести ssh в фоновый режим;
- **g** — разрешить удалённым машинам обращаться к локальным портам;
- **l** — имя пользователя в системе;
- **n** — перенаправить стандартный вывод в /dev/null;
- **p** — порт ssh на удалённой машине;
- **q** — не показывать сообщения об ошибках;
- **v** — режим отладки;
- **x** — отключить перенаправление X11;
- **X** — включить перенаправление X11;
- **C** — включить сжатие.

## Maven

### Скачивание зависимости

```bash
mvn dependency:resolve
# или скачать одну зависимость:
mvn dependency:get -Dartifact=groupId:artifactId:version
```

### Сборка определённого модуля

```bash
-pl, --projects
  # build specified reactor projects instead of all projects
  # use colon if you are referencing an artifactId which differs from directory name
-am, --also-make
  # if project list is specified, also build projects required by the list
mvn package -pl :my-module -am
mvn install -pl :my-module -am
```

### Продолжение сборки («build resume»)

`mvn package -rf :my-module`

### Сборка артефакта в IntelliJ IDEA

1. В главном меню выберите Build | Build Artifacts.
2. Укажите созданный .jar (HelloWorld:jar) и выберите Build. После этого в папке `out/artifacts` появится .jar-файл.

## Kafka

### Прочитать / записать пару в Kafka

```bash
/bin/kafka-console-producer \
  --bootstrap-server localhost:9092 \
  --topic time2 \
  --property "parse.key=true" \
  --property "key.separator=:"
/bin/kafka-console-consumer \
  --topic time2 \
  --from-beginning \
  --bootstrap-server localhost:9092 \
  --property "print.key=true"
```

### Файл конфигурации Kafka

`/opt/kafka/config/server.properties`

## Docker

### Деплой проекта в Docker

```bash
# выключить проект с набором контейнеров
docker-compose -p stand3100 -f loans.yml down
# скопировать приложение
scp C:/path/my-app-1.0.1.war stend:/PPRB/stand3100/loans/
# выключить проект с набором контейнеров
docker-compose -p stand3100 -f loans.yml up -d
# читать логи запуска приложения
docker-compose -p stand3100 -f loans.yml logs -f loans | grep 'loans-for-business'
```

### Создать контейнер из docker-compose.yml

```bash
cd <dir>
docker-compose up -d
```

### Подключиться к консоли контейнера

Чтобы открыть интерактивную оболочку внутри контейнера Docker (например, для изучения файловой системы или отладки процессов), используется `docker exec` с флагами `-i` и `-t`. Флаг `-i` держит ввод открытым, флаг `-t` создаёт псевдотерминал:

`docker exec -it container-name sh`

Эта команда запустит оболочку `sh` в указанном контейнере. Для выхода введите `exit` и нажмите `Enter`. Если образ содержит более продвинутую оболочку, например `bash`, можно заменить `sh` на `bash`.

### Рестарт контейнера

`docker restart my_container`

### Логи контейнера

`docker container logs [OPTIONS] CONTAINER_NAME`

## Git

### Обычный цикл

```bash
# regular cycle
git fetch
git checkout <branchname>
git pull
git commit -m "new feature added"
git push
# seldom cycle
git cherrypick #?
```

### git stash — припрятать

```bash
git stash        # спрятать на полку
git stash show   # показать содержимое стека
git stash pop    # вытащить с полки в проект, очистить полку
git stash apply  # вытащить с полки в проект, оставить копию на полке
git stash clear  # очистить полку
git stash drop   # очистить полку в случае конфликта при выполнении git stash pop
```

### Опубликовать новую ветку

```bash
git push -u origin <local-branch-name>
git push -u origin feature/mySuperFeature
# git push upstream ... # когда локальная ветка «детачед» от удалённой
```

### Исключить файлы из отслеживания

```bash
git rm --cached /path/to/files
```

Связанная тема: [[git]].

## Batch (cmd)

### Аналоги между bash, batch (cmd) и PowerShell

| Действие | bash | batch (cmd) | PowerShell |
|----------|------|-------------|------------|
| Перевод строки | `\` | `^` | `` ` `` |
| Вывод в файл | — | `<что-то> > <путь к файлу>` | — |
| Список файлов | `ls` | `dir` | — |
| Очистить экран | `clear` | `cls` | — |
| Запуск в фоне | — | `call` | — |

### Работа с файлами

```bash
dir            # список файлов, ls -la
%cd%           # current working directory
%__appdir__%   # expands to the executable that runs the current script
```

### Упаковать jar-артефакт

```bash
cd <into your package directory> # then use:
jar -cfv my-artifact-21.0.1.jar *
    # c -- create new archive
    # f -- specify filename
    # v -- verbose output
    # x -- extract
# -- распаковать
jar -xfv my-artifact-21.0.1.jar

# -- упаковать при помощи Maven
cd <path>
mvn package
```

### Запись данных о сертификатах в файл

`<что-то> > <путь к файлу>`

`keytool -v -list .\jdks\cacerts > c:\cacerts-list.txt`

### Прочитать переменную среды

`echo %JAVA_HOME%`

### Очистка консоли

`cls`

## Keytool cacerts — хранилище сертификатов

**cacerts** — файл с хранилищем сертификатов для доступа к репозиториям.

**keytool** — утилита для манипуляции сертификатами.
