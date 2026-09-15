---
aliases:
  - OpenSSL
  - p12
  - PFX
  - PKCS#12
  - PKCS12
  - X.509
---

## OpenSSL

**OpenSSL** — библиотека и набор командной строки для работы с криптографией: генерация ключей, создание CSR, самоподписанные сертификаты, проверка и конвертация сертификатов (в том числе в формат PKCS#12).

## Основные возможности

- Генерация закрытых ключей (RSA, EC).
- Создание запроса на сертификат (CSR).
- Выпуск самоподписанных сертификатов (X.509).
- Просмотр содержимого ключей и сертификатов.
- Конвертация между форматами (PEM, DER, PKCS#12).
- Проверка цепочек и сроков действия.

## Генерация закрытого ключа

```bash
openssl genrsa -out private.key 2048
```

или для EC-ключа:

```bash
openssl ecparam -genkey -name prime256v1 -out ec.key
```

## Создание CSR

```bash
openssl req -new -key private.key -out request.csr
```

Реквизиты (CN, O, OU) запрашиваются интерактивно.

## Самоподписанный сертификат

```bash
openssl req -x509 -new -key private.key -days 365 -out certificate.crt
```

## Просмотр содержимого

Ключ:

```bash
openssl rsa -in private.key -text -noout
```

Сертификат:

```bash
openssl x509 -in certificate.crt -text -noout
```

CSR:

```bash
openssl req -in request.csr -text -noout
```

## Формат PKCS#12

**PKCS#12** (расширения `.p12`, `.pfx`) — контейнер, объединяющий **сертификат, закрытый ключ и (опционально) цепочку доверия** в одном файле с паролем.

Создание из PEM:

```bash
openssl pkcs12 -export -out bundle.p12 -inkey private.key -in certificate.crt -certfile ca-chain.crt
```

**Флаги:**

- `-export` — создать контейнер
- `-inkey` — ключ
- `-in` — сертификат
- `-certfile` — цепочка доверия (опционально)
- `-out` — выходной файл
- `-passout` — пароль на контейнер

Просмотр содержимого p12:

```bash
openssl pkcs12 -in bundle.p12 -info -noout
```

Извлечение данных из p12:

```bash
openssl pkcs12 -in bundle.p12 -clcerts -nokeys -out certs.pem
openssl pkcs12 -in bundle.p12 -nocerts -nodes -out key.pem
```

## Проверка пары «ключ — сертификат»

```bash
openssl x509 -in certificate.crt -noout -modulus | openssl md5
openssl rsa -in private.key -noout -modulus | openssl md5
```

Совпадение хэшей означает, что ключ соответствует сертификату.

## Типовые ошибки

| Ошибка | Причина/решение |
|--------|-----------------|
| `unable to load private key` | Неверный путь или формат ключа |
| `no start line` | Файл не PEM — проверить формат (PEM vs DER) |
| `PKCS12: wrong password` | Неверный пароль контейнера |
| `X509 certificate mismatch` | Сертификат не соответствует ключу |

## Итог

OpenSSL — стандартный инструмент для работы с ключами и сертификатами: генерация, CSR, самоподписанные сертификаты, конвертация и проверка. Формат PKCS#12 удобен для переноса «ключ + сертификат» единым файлом.
