---
aliases:
  - Client for URLs
  - cURL
  - curl
---

## Client for URLs

curl (cURL) — кроссплатформенная служебная программа командной строки, которая позволяет взаимодействовать с серверами по различным протоколам с синтаксисом URL. Название расшифровывается как Client for URLs.

## Основные опции

### Запрос

- `-X, --request <method>` — указание HTTP-метода (GET, POST, PUT, DELETE и т.д.)
- `--url <url>` — URL для запроса

### Данные запроса

- `-d, --data <data>` — отправка данных (обычно для POST)
- `-F, --form <name=content>` — отправка multipart/form-data
- `--data-raw <data>` — отправка сырых данных
- `--data-ascii <data>` — отправка ASCII данных
- `--data-binary <data>` — отправка бинарных данных
- `--data-urlencode <data>` — URL-encode данных

### Заголовки

- `-H, --header <header>` — кастомные HTTP-заголовки
- `-A, --user-agent <name>` — строка User-Agent
- `-e, --referer <URL>` — заголовок Referer

### Cookies

- `-b, --cookie <data>` — отправка cookies
- `-c, --cookie-jar <filename>` — сохранение cookies в файл
- `-j, --junk-session-cookies` — игнорирование session cookies

### Аутентификация

- `-u, --user <user:password>` — базовая аутентификация
- `--basic` — использовать Basic auth
- `--digest` — использовать Digest auth
- `--ntlm` — использовать NTLM auth
- `--negotiate` — использовать Negotiate auth
- `--anyauth` — любой метод аутентификации

### SSL/TLS

- `-k, --insecure` — отключить проверку SSL-сертификатов
- `--cacert <file>` — CA-сертификат для проверки
- `--cert <certificate[:password]>` — клиентский SSL-сертификат
- `--cert-type <type>` — тип клиентского сертификата (PEM, DER, ENG)
- `--key <key>` — приватный ключ SSL
- `--key-type <type>` — тип приватного ключа
- `--pass <phrase>` — парольная фраза для приватного ключа
- `--engine <name>` — использование crypto engine
- `--capath <directory>` — директория с CA-сертификатами
- `--pinnedpubkey <hashes>` — фиксированный публичный ключ

## Вывод и отладка

### Вывод ответа

- `-i, --include` — включить заголовки ответа в вывод
- `-I, --head` — только заголовки (HEAD-запрос)
- `-D, --dump-header <file>` — сохранить заголовки в файл
- `-o, --output <file>` — сохранить вывод в файл
- `-O, --remote-name` — сохранить в файл с именем из URL
- `-J, --remote-header-name` — использовать имя файла из заголовков
- `-s, --silent` — тихий режим (без progress meter)
- `-S, --show-error` — показывать ошибки даже в silent-режиме

### Отладка

- `-v, --verbose` — подробный вывод
- `--trace <file>` — полная трассировка в файл
- `--trace-ascii <file>` — трассировка ASCII в файл
- `--trace-time` — добавить временные метки к trace
- `-w, --write-out <format>` — кастомный формат вывода

### Перенаправления

- `-L, --location` — следовать перенаправлениям
- `--location-trusted` — следовать перенаправлениям с отправкой аутентификации
- `--max-redirs <num>` — максимальное количество перенаправлений

## Сетевые настройки

### Прокси

- `-x, --proxy [protocol://]host[:port]` — использовать прокси
- `--proxy-user <user:password>` — аутентификация на прокси
- `--proxy-anyauth` — любой метод аутентификации на прокси
- `--proxy-basic` — Basic-аутентификация на прокси
- `--proxy-digest` — Digest-аутентификация на прокси
- `--proxy-ntlm` — NTLM-аутентификация на прокси
- `--proxy-negotiate` — Negotiate-аутентификация на прокси
- `--socks5 <host[:port]>` — SOCKS5-прокси
- `--socks5-basic` — Basic auth для SOCKS5
- `--socks5-gssapi` — GSS-API для SOCKS5

### Таймауты и лимиты

- `-m, --max-time <seconds>` — максимальное время выполнения
- `--connect-timeout <seconds>` — таймаут на подключение
- `--max-filesize <bytes>` — максимальный размер файла
- `--retry <num>` — количество повторных попыток
- `--retry-delay <seconds>` — задержка между попытками
- `--retry-max-time <seconds>` — максимальное время для повторных попыток

### Соединения

- `--keepalive-time <seconds>` — интервал keep-alive
- `--no-keepalive` — отключить keep-alive
- `-4, --ipv4` — использовать только IPv4
- `-6, --ipv6` — использовать только IPv6
- `--interface <name>` — использовать указанный интерфейс
- `--local-port <num/range>` — использовать указанный локальный порт
- `--resolve <host:port:address>` — кастомное разрешение хоста

## Файлы и передача данных

### Загрузка файлов

- `-T, --upload-file <file>` — загрузить файл (PUT)
- `-a, --append` — добавить к загружаемому файлу
- `--limit-rate <speed>` — ограничить скорость передачи
- `-C, --continue-at <offset>` — продолжить загрузку с позиции

### Чтение из файлов

- `-K, --config <file>` — чтение конфигурации из файла
- `--data @filename` — чтение данных из файла
- `--cert @filename` — чтение сертификата из файла
- `--key @filename` — чтение ключа из файла

## Разное

### Компрессия

- `--compressed` — запросить сжатый ответ
- `--compressed-ssh` — включить SSH-компрессию

### Протоколы

- `-0, --http1.0` — использовать HTTP/1.0
- `--http1.1` — использовать HTTP/1.1
- `--http2` — использовать HTTP/2
- `--http2-prior-knowledge` — HTTP/2 без upgrade
- `--http3` — использовать HTTP/3

### Другие опции

- `-f, --fail` — не выводить HTML-страницы об ошибках
- `--ftp-account <data>` — FTP account data
- `--ftp-alternative-to-user <command>` — альтернативная FTP-команда
- `--ftp-create-dirs` — создать директории при FTP-загрузке
- `--ftp-method <method>` — FTP-метод (multicwd, nocwd, singlecwd)
- `--ftp-pasv` — использовать пассивный режим FTP
- `-P, --ftp-port <address>` — использовать активный режим FTP
- `--ftp-skip-pasv-ip` — пропуск IP в PASV-ответе
- `--ftp-ssl` — использовать SSL/TLS для FTP
- `--ftp-ssl-ccc` — отправить CCC после аутентификации
- `--ftp-ssl-control` — использовать SSL/TLS для FTP control connection
- `-l, --list-only` — только список файлов (FTP)
- `-B, --use-ascii` — использовать ASCII transfer (FTP/LDAP)
- `--crlf` — конвертировать LF в CRLF (FTP)
- `--mail-from <address>` — отправитель для SMTP
- `--mail-rcpt <address>` — получатель для SMTP
- `--mail-auth <address>` — аутентификация для SMTP
- `-Q, --quote <command>` — отправить команду на сервер (FTP/SFTP)
- `--range <range>` — запросить диапазон байтов
- `--raw` — отключить декодирование ответа
- `--post301` — не конвертировать POST в GET после 301
- `--post302` — не конвертировать POST в GET после 302
- `--post303` — не конвертировать POST в GET после 303
- `-N, --no-buffer` — отключить буферизацию вывода
- `-Y, --speed-limit <speed>` — минимальная скорость для остановки
- `-y, --speed-time <seconds>` — время для измерения скорости
- `-z, --time-cond <time>` — условие на время (If-Modified-Since)
- `-R, --remote-time` — установить время файла как на сервере
- `--tcp-nodelay` — включить TCP_NODELAY
- `--tr-encoding` — поддержка Transfer-Encoding
- `--aws-sigv4 <provider1[:provider2[:region[:service]]]>` — AWS V4 signature

## Специальные флаги

### Для безопасности

- `--ssl` — попытаться использовать SSL
- `--ssl-reqd` — требовать использование SSL
- `--ssl-allow-beast` — работа с SSL BEAST-уязвимостью
- `--ssl-no-revoke` — отключить проверку отзыва сертификатов
- `--proxy-ssl-allow-beast` — то же для прокси
- `--tlsv1.0`, `--tlsv1.1`, `--tlsv1.2`, `--tlsv1.3` — версия TLS
- `--tls13-ciphers <list>` — шифры для TLS 1.3
- `--tlsauthtype <type>` — тип TLS-аутентификации
- `--tlspassword <string>` — пароль для TLS-аутентификации
- `--tlsuser <name>` — имя пользователя для TLS-аутентификации

### Для отладки сетевого стека

- `--suppress-connect-headers` — не показывать заголовки CONNECT
- `--proto <protocols>` — ограничить протоколы
- `--proto-redir <protocols>` — ограничить протоколы для перенаправлений
- `--proxy1.0 <host:port>` — HTTP/1.0-прокси
- `--socks4 <host[:port]>` — SOCKS4-прокси
- `--socks4a <host[:port]>` — SOCKS4a-прокси
- `--socks5-hostname <host[:port]>` — SOCKS5 с разрешением на сервере

## Переменные вывода (для -w)

Переменные для `-w, --write-out`:

- `%{content_type}` — Content-Type ответа
- `%{filename_effective}` — имя конечного файла
- `%{ftp_entry_path}` — начальный путь FTP
- `%{http_code}` — числовой код ответа
- `%{http_connect}` — код ответа CONNECT
- `%{local_ip}` — локальный IP
- `%{local_port}` — локальный порт
- `%{num_connects}` — количество соединений
- `%{num_redirects}` — количество перенаправлений
- `%{proxy_ssl_verify_result}` — результат проверки SSL-прокси
- `%{redirect_url}` — URL перенаправления
- `%{remote_ip}` — удалённый IP
- `%{remote_port}` — удалённый порт
- `%{response_code}` — код ответа (устаревший)
- `%{scheme}` — схема URL
- `%{size_download}` — размер загрузки
- `%{size_header}` — размер заголовков
- `%{size_request}` — размер запроса
- `%{size_upload}` — размер выгрузки
- `%{speed_download}` — скорость загрузки (байт/сек)
- `%{speed_upload}` — скорость выгрузки (байт/сек)
- `%{ssl_verify_result}` — результат проверки SSL
- `%{time_appconnect}` — время SSL handshake
- `%{time_connect}` — время установки TCP-соединения
- `%{time_namelookup}` — время DNS lookup
- `%{time_pretransfer}` — время до начала передачи
- `%{time_redirect}` — время перенаправлений
- `%{time_starttransfer}` — время до первого байта
- `%{time_total}` — общее время
- `%{url_effective}` — конечный URL
- `%{urlnum}` — номер URL в цепочке

## Примеры

**Для отладки API**

```bash
curl -v -X POST \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer token" \
     -d '{"key":"value"}' \
     https://api.example.com/endpoint
```

**Для скачивания файлов**

```bash
curl -L -O -C - \
     --retry 3 \
     --retry-delay 5 \
     https://example.com/largefile.zip
```

**Для тестирования производительности**

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

**Для работы с прокси**

```bash
curl -x http://proxy:8080 \
     --proxy-user user:pass \
     --proxy-ntlm \
     https://target.com
```

**Примечание**: это основные опции. Полный список можно увидеть с помощью `curl --help all` или `man curl`.
