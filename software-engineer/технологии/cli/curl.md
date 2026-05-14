---
aliases:
  - curl
  - cURL
  - Client for URLs
---
# Client for URLs

**CURL (cURL)** — кроссплатформенная служебная программа командной строки, которая позволяет взаимодействовать с серверами по различным протоколам с синтаксисом URL. Название расшифровывается как Client for URLs.

Вот **полный список основных атрибутов/опций `curl`**, сгруппированных по категориям:

## **ОСНОВНЫЕ ОПЦИИ**

### **Запрос**
- `-X, --request <method>` - Указание HTTP метода (GET, POST, PUT, DELETE и т.д.)
- `--url <url>` - URL для запроса

### **Данные запроса**
- `-d, --data <data>` - Отправка данных (обычно для POST)
- `-F, --form <name=content>` - Отправка multipart/form-data
- `--data-raw <data>` - Отправка сырых данных
- `--data-ascii <data>` - Отправка ASCII данных
- `--data-binary <data>` - Отправка бинарных данных
- `--data-urlencode <data>` - URL-encode данных

### **Заголовки**
- `-H, --header <header>` - Кастомные HTTP заголовки
- `-A, --user-agent <name>` - User-Agent строка
- `-e, --referer <URL>` - Referer заголовок

### **Cookies**
- `-b, --cookie <data>` - Отправка cookies
- `-c, --cookie-jar <filename>` - Сохранение cookies в файл
- `-j, --junk-session-cookies` - Игнорирование session cookies

### **Аутентификация**
- `-u, --user <user:password>` - Базовая аутентификация
- `--basic` - Использовать Basic auth
- `--digest` - Использовать Digest auth
- `--ntlm` - Использовать NTLM auth
- `--negotiate` - Использовать Negotiate auth
- `--anyauth` - Любой метод аутентификации

### **SSL/TLS**
- `-k, --insecure` - Отключить проверку SSL сертификатов
- `--cacert <file>` - CA сертификат для проверки
- `--cert <certificate[:password]>` - Клиентский SSL сертификат
- `--cert-type <type>` - Тип клиентского сертификата (PEM, DER, ENG)
- `--key <key>` - Приватный ключ SSL
- `--key-type <type>` - Тип приватного ключа
- `--pass <phrase>` - Парольная фраза для приватного ключа
- `--engine <name>` - Использование crypto engine
- `--capath <directory>` - Директория с CA сертификатами
- `--pinnedpubkey <hashes>` - Фиксированный публичный ключ

## **ВЫВОД И ОТЛАДКА**

### **Вывод ответа**
- `-i, --include` - Включить заголовки ответа в вывод
- `-I, --head` - Только заголовки (HEAD запрос)
- `-D, --dump-header <file>` - Сохранить заголовки в файл
- `-o, --output <file>` - Сохранить вывод в файл
- `-O, --remote-name` - Сохранить в файл с именем из URL
- `-J, --remote-header-name` - Использовать имя файла из заголовков
- `-s, --silent` - Тихий режим (без progress meter)
- `-S, --show-error` - Показывать ошибки даже в silent режиме

### **Отладка**
- `-v, --verbose` - Подробный вывод
- `--trace <file>` - Полная трассировка в файл
- `--trace-ascii <file>` - Трассировка ASCII в файл
- `--trace-time` - Добавить временные метки к trace
- `-w, --write-out <format>` - Кастомный формат вывода

### **Перенаправления**
- `-L, --location` - Следовать перенаправлениям
- `--location-trusted` - Следовать перенаправлениям с отправкой аутентификации
- `--max-redirs <num>` - Максимальное количество перенаправлений

## **СЕТЕВЫЕ НАСТРОЙКИ**

### **Прокси**
- `-x, --proxy [protocol://]host[:port]` - Использовать прокси
- `--proxy-user <user:password>` - Аутентификация на прокси
- `--proxy-anyauth` - Любой метод аутентификации на прокси
- `--proxy-basic` - Basic аутентификация на прокси
- `--proxy-digest` - Digest аутентификация на прокси
- `--proxy-ntlm` - NTLM аутентификация на прокси
- `--proxy-negotiate` - Negotiate аутентификация на прокси
- `--socks5 <host[:port]>` - SOCKS5 прокси
- `--socks5-basic` - Basic auth для SOCKS5
- `--socks5-gssapi` - GSS-API для SOCKS5

### **Таймауты и лимиты**
- `-m, --max-time <seconds>` - Максимальное время выполнения
- `--connect-timeout <seconds>` - Таймаут на подключение
- `--max-filesize <bytes>` - Максимальный размер файла
- `--retry <num>` - Количество повторных попыток
- `--retry-delay <seconds>` - Задержка между попытками
- `--retry-max-time <seconds>` - Максимальное время для повторных попыток

### **Соединения**
- `--keepalive-time <seconds>` - Интервал keep-alive
- `--no-keepalive` - Отключить keep-alive
- `-4, --ipv4` - Использовать только IPv4
- `-6, --ipv6` - Использовать только IPv6
- `--interface <name>` - Использовать указанный интерфейс
- `--local-port <num/range>` - Использовать указанный локальный порт
- `--resolve <host:port:address>` - Кастомное разрешение хоста

## **ФАЙЛЫ И ПЕРЕДАЧА ДАННЫХ**

### **Загрузка файлов**
- `-T, --upload-file <file>` - Загрузить файл (PUT)
- `-a, --append` - Добавить к загружаемому файлу
- `--limit-rate <speed>` - Ограничить скорость передачи
- `-C, --continue-at <offset>` - Продолжить загрузку с позиции

### **Чтение из файлов**
- `-K, --config <file>` - Чтение конфигурации из файла
- `--data @filename` - Чтение данных из файла
- `--cert @filename` - Чтение сертификата из файла
- `--key @filename` - Чтение ключа из файла

## **РАЗНОЕ**

### **Компрессия**
- `--compressed` - Запросить сжатый ответ
- `--compressed-ssh` - Включить SSH compression

### **Протоколы**
- `-0, --http1.0` - Использовать HTTP/1.0
- `--http1.1` - Использовать HTTP/1.1
- `--http2` - Использовать HTTP/2
- `--http2-prior-knowledge` - HTTP/2 без upgrade
- `--http3` - Использовать HTTP/3

### **Другие**
- `-f, --fail` - Не выводить HTML страницы об ошибках
- `--ftp-account <data>` - FTP account data
- `--ftp-alternative-to-user <command>` - Альтернативная FTP команда
- `--ftp-create-dirs` - Создать директории при FTP загрузке
- `--ftp-method <method>` - FTP метод (multicwd, nocwd, singlecwd)
- `--ftp-pasv` - Использовать пассивный режим FTP
- `-P, --ftp-port <address>` - Использовать активный режим FTP
- `--ftp-skip-pasv-ip` - Пропуск IP в PASV ответе
- `--ftp-ssl` - Использовать SSL/TLS для FTP
- `--ftp-ssl-ccc` - Отправить CCC после аутентификации
- `--ftp-ssl-control` - Использовать SSL/TLS для FTP control connection
- `-l, --list-only` - Только список файлов (FTP)
- `-B, --use-ascii` - Использовать ASCII transfer (FTP/LDAP)
- `--crlf` - Конвертировать LF в CRLF (FTP)
- `--mail-from <address>` - Отправитель для SMTP
- `--mail-rcpt <address>` - Получатель для SMTP
- `--mail-auth <address>` - Аутентификация для SMTP
- `-Q, --quote <command>` - Отправить команду на сервер (FTP/SFTP)
- `--range <range>` - Запросить диапазон байтов
- `--raw` - Отключить декодирование ответа
- `--post301` - Не конвертировать POST в GET после 301
- `--post302` - Не конвертировать POST в GET после 302
- `--post303` - Не конвертировать POST в GET после 303
- `-N, --no-buffer` - Отключить буферизацию вывода
- `-Y, --speed-limit <speed>` - Минимальная скорость для остановки
- `-y, --speed-time <seconds>` - Время для измерения скорости
- `-z, --time-cond <time>` - Условие на время (If-Modified-Since)
- `-r, --range <range>` - Запросить диапазон байтов
- `-R, --remote-time` - Установить время файла как на сервере
- `--tcp-nodelay` - Включить TCP_NODELAY
- `--tr-encoding` - Поддержка Transfer-Encoding
- `--aws-sigv4 <provider1[:provider2[:region[:service]]]>` - AWS V4 signature

## **СПЕЦИАЛЬНЫЕ ФЛАГИ**

### **Для безопасности**
- `--ssl` - Попытаться использовать SSL
- `--ssl-reqd` - Требовать использование SSL
- `--ssl-allow-beast` - Работа с SSL BEAST уязвимостью
- `--ssl-no-revoke` - Отключить проверку отзыва сертификатов
- `--proxy-ssl-allow-beast` - То же для прокси
- `--tlsv1.0`, `--tlsv1.1`, `--tlsv1.2`, `--tlsv1.3` - Версия TLS
- `--tls13-ciphers <list>` - Шифры для TLS 1.3
- `--tlsauthtype <type>` - Тип TLS аутентификации
- `--tlspassword <string>` - Пароль для TLS аутентификации
- `--tlsuser <name>` - Имя пользователя для TLS аутентификации

### **Для отладки сетевого стека**
- `--trace <file>` - Полный дамп трафика в файл
- `--trace-ascii <file>` - ASCII дамп трафика
- `--suppress-connect-headers` - Не показывать заголовки CONNECT
- `--proto <protocols>` - Ограничить протоколы
- `--proto-redir <protocols>` - Ограничить протоколы для перенаправлений
- `--proxy1.0 <host:port>` - HTTP/1.0 прокси
- `--socks4 <host[:port]>` - SOCKS4 прокси
- `--socks4a <host[:port]>` - SOCKS4a прокси
- `--socks5-hostname <host[:port]>` - SOCKS5 с разрешением на сервере

## **ФОРМАТЫ ВЫВОДА (для -w)**

Переменные для `-w, --write-out`:
- `%{content_type}` - Content-Type ответа
- `%{filename_effective}` - Имя конечного файла
- `%{ftp_entry_path}` - Начальный путь FTP
- `%{http_code}` - Числовой код ответа
- `%{http_connect}` - Код ответа CONNECT
- `%{local_ip}` - Локальный IP
- `%{local_port}` - Локальный порт
- `%{num_connects}` - Количество соединений
- `%{num_redirects}` - Количество перенаправлений
- `%{proxy_ssl_verify_result}` - Результат проверки SSL прокси
- `%{redirect_url}` - URL перенаправления
- `%{remote_ip}` - Удаленный IP
- `%{remote_port}` - Удаленный порт
- `%{response_code}` - Код ответа (устаревший)
- `%{scheme}` - Схема URL
- `%{size_download}` - Размер загрузки
- `%{size_header}` - Размер заголовков
- `%{size_request}` - Размер запроса
- `%{size_upload}` - Размер загрузки
- `%{speed_download}` - Скорость загрузки (байт/сек)
- `%{speed_upload}` - Скорость выгрузки (байт/сек)
- `%{ssl_verify_result}` - Результат проверки SSL
- `%{time_appconnect}` - Время SSL handshake
- `%{time_connect}` - Время установки TCP соединения
- `%{time_namelookup}` - Время DNS lookup
- `%{time_pretransfer}` - Время до начала передачи
- `%{time_redirect}` - Время перенаправлений
- `%{time_starttransfer}` - Время до первого байта
- `%{time_total}` - Общее время
- `%{url_effective}` - Конечный URL
- `%{urlnum}` - Номер URL в цепочке

## Примеры

### **Для отладки API**
```bash
curl -v -X POST \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer token" \
     -d '{"key":"value"}' \
     https://api.example.com/endpoint
```

### **Для скачивания файлов**
```bash
curl -L -O -C - \
     --retry 3 \
     --retry-delay 5 \
     https://example.com/largefile.zip
```

### **Для тестирования производительности**
```bash
curl -w "
Time total: %{time_total}s
DNS lookup: %{time_namelookup}s
Connect: %{time_connect}s
SSL handshake: %{time_appconnect}s
Start transfer: %{time_starttransfer}s
Speed download: %{speed_download} B/s
" -o /dev/null -s https://example.com
```

### **Для работы с прокси**
```bash
curl -x http://proxy:8080 \
     --proxy-user user:pass \
     --proxy-ntlm \
     https://target.com
```

**Примечание:** Это основные опции. Полный список можно увидеть с помощью `curl --help all` или `man curl`.