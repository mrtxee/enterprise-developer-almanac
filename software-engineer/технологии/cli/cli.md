---
aliases:
  - cli
  - ssh
  - bash
---
## ssh config file
Файл **.ssh/config** нужен чтобы закешировать”адресную книгу” с доступами к разным ssh хостам.
By default, the SSH configuration file may not exist, so you may need to create it using the touch command touch `~/.ssh/config`
```Bash
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
потом в терминале можно инициировать подключение при помощи eg. `ssh hostname2`
- if you want to use all other options but to connect as user `root` instead of `john` simply specify the user on the command line:
    
    ```Plain
    ssh -o "User=root" dev
    ```
    
- The `-F` (`configfile`) option allows you to specify an alternative per-user configuration file. To tell the `ssh` client to ignore all of the options specified in the ssh configuration file, use:
    
    ```Plain
    ssh -F /dev/null user@example.com
    ```
    
## bash
### bash core
#### узнать версию linux
```Bash
cat /etc/os-release
uname -a
```
#### работа с файлами
```bash
la -la — все файлы списком в столбик
mc — midnight commander / fs via gui
cat — прочитать содержимое файла
pwd — текущая папка, адрес в фс
rm — удалить файл
	- rm -rfv
	    - recursice
	    - force
	    - verbose
touch file1 file2 file3 — to update the timestamps on existing files 
		and directories as well as creating new, empty files
stat <file_name> — to display the file status including the timestamps  
```

#### ssh-keygen изменить пароль приватного ключа
`ssh-keygen -p -P oldPw -N newPw -f /path/to/private_key`
#### grep — фильтр по регулярным выражениям
search **g**lobally for lines matching the **r**egular **e**xpression, and **p**rint them
`$ grep [опции] шаблон [<путь к файлу или папке>]`
можно применить фильтр к стандартному выводу другой команды:  
  
`$ команда | grep [опции] шаблон`
```Bash
grep --version | grep grep
# grep (GNU grep) 2.5.1-FreeBSD
grep "^[a-zA-Z]" pgm.s
...
```
#### переменные
```Bash
#!/bin/bash
\#данные в перменную
CUR_DIR={pwd}
\#вставить перменную
COMPILE_OPTS="-D maven.repo.local=${CUR_DIR}"
\#конкатенации переменной
COMPILE_OPTS="${COMPILE_OPTS} -llr"
```

### [[vim]]

### ssh
SSH — Secure Shell — протокол удаленного управления компьютером с операционной системой Linux.
- Настройки сервера SSH находятся в файле `/etc/ssh/sshd_config`, `/etc/ssh/sshd.conf`
- По умолчанию ssh работает на порту 22
основные команды
- **f** - перевести ssh в фоновый режим;
- **g** - разрешить удаленным машинам обращаться к локальным портам;
- **l** - имя пользователя в системе;
- **n** - перенаправить стандартный вывод в /dev/null;
- **p** - порт ssh на удаленной машине;
- **q** - не показывать сообщения об ошибках;
- **v** - режим отладки;
- **x** - отключить перенаправление X11;
- **X** - включить перенаправление Х11;
- **C** - включить сжатие.
## maven
### dependency get
```Bash
mvn dependency:resolve
\#Or to download a single dependency:
mvn dependency:get -Dartifact=groupId:artifactId:version
```
### build certain module
```Bash
-pl, --projects
  # build specified reactor projects instead of all projects
  # use colon if you are referencing an artifactId which differs from directory name
-am, --also-make
  # if project list is specified, also build projects required by the list
mvn package -pl :my-module -am
mvn install -pl :my-module -am
```
### build resume from
`mvn package -rf :my-module`
### build artifact with intellij idea
[https://www.jetbrains.com/help/idea/compiling-applications.html#add_file_to_jar](https://www.jetbrains.com/help/idea/compiling-applications.html#add_file_to_jar)
1. In the main menu, go to Build | Build Artifacts.
2. Point to the created .jar (HelloWorld:jar) and select Build.
    
    If you now look at the out/artifacts folder, you'll find your .jar file there.
    
## kafka
### прочитать / записать пару в kafka
```Bash
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
### файл конфигурации kafka
`/opt/kafka/config/server.properties`
## docker
### деплой проекта в докер
```Bash
\#bash
\#выключить проект с набором контейнеров
docker-compose -p stand3100 -f loans.yml down 
\#скопировать приложение
scp C:/path/my-app-1.0.1.war stend:/PPRB/stand3100/loans/
\#выключить проект с набором контейнеров
docker-compose -p stand3100 -f loans.yml up -d
\#читать логи запуска приложения
docker-compose -p stand3100 -f loans.yml logs -f loans|grep 'loans-for-business'
```
### создать контейнер из docker-compose.yml
```Bash
cd <dir>
docker-compose up -d
```
### подключиться к консоли контейнериа
If you need to start an interactive shell inside a Docker Container,  
perhaps to explore the filesystem or debug running processes, use  
`docker exec` with the `-i` and `-t` flags.
The `-i` flag keeps input open to the container, and the `-t` flag creates a pseudo-terminal to which the shell can attach. These flags can be combined like this:
`docker exec -it container-name sh`
This will run the `sh` shell in the specified container, giving you a basic shell prompt. To exit back out of the container, type `exit` then press `ENTER`:
`# / exit`
If your container image includes a more advanced shell such as `bash` , you could replace `sh` with `bash` above.
### рестарт контейнера
`docker restart my_container`
### docker container logs
`docker container logs [OPTIONS] CONTAINER_NAME`
## git
```Bash
# regular cycle
git fetch
git checkout <branchname>
git pull
git commit -m "new feature added"
git push
# seldom cycle
git cherrypick #?
# seldom cycle
```
### git stash — припрятать
```Bash
git stash        \#спрятать на полку
git stash show   \#спрятать на полку
git stash pop    \#вытащить с полки в проект, очистить полку
git stash apply  \#вытащить с полки в проект, оставить копию на полке
git stash clear  \#очистить полку
git stash drop   \#очистить полку в случае конфликта при выполнении git stash pop
```
### опубликовать новую ветку
```Bash
git push -u origin <local-branch-name>
git push -u origin feature/mySuperFeature
# git push upstram ... # когда локальная ветка детачед от удаленной
```
### исключить файлы из трэкед
```Bash
git rm --cached /path/to/files
```
[[git]]
# batch
## batch vs bash vs PowerShell
||bash|batch (cmd)|PowerShell|
|---|---|---|---|
|перевод строки|`\`|`^`|`` ` ``|
|вывод в файл||`<whatever> > <path to file>`||
|список фаайлов|`ls`|`dir`||
|очистить экран|`clear`|`cls`||
|run in background||`call`||
|||||
## работа с файлами
```Bash
dir            # список файлов, ls -la
%cd%           # current working directory 
%__appdir__%   # expands to the executable that runs the current script
```
## упаковать jar-артифакт
```Shell
cd <into your package directory> \#then use:
jar -cfv my-artifact-21.0.1.jar *
    # c -- create new archive
    # f -- specify filename
    # v -- verbose output
    # x -- extract
# -- рапаковать
jar -xfv my-artifact-21.0.1.jar
    
# -- упаковать при помощи MAVEN
cd <path>
mvn package
```
## запись данных о сертификатах в файл
`<whatever> > <path to file>`
`keytool -v -list .\jdks\cacerts > c:\cacerts-list.txt`
## прочитать перменную среды
`echo %JAVA_HOME%`
## очистка консоли
`cls`
# keytool cacerts — хранилище сертификатов
cacerts — файл с хранилищем сертификатов для доступа к репозиториям
keytool — утилита для манипулации сертификатами