---
aliases:
  - JSON
  - RCL
  - Lua
  - RON
  - Nix
  - Properties
  - HCL
  - XML
  - TOML
  - YAML
  - INI
  - конфиги
  - форматы кофигов
  - config formats
---
### Форматы конфигурационных файлов
> config files formats

1. **Common config [[DSL]] formats**
	1. **YAML (YAML Ain’t Markup Language)** — формат сериализации данных, ориентированный на удобство для человека. Использует отступы вместо скобок и запятых. Поддерживает сложные структуры данных (списки, ассоциативные массивы). Популярнен в DevOps (Ansible, [[Kubernetes]]), [[IaC|IaC]]
	2. **Properties (Java Properties)** — простой формат пар «ключ=значение», преимущественно используется в Java-проектах для настройки приложения
	3. **TOML (Tom’s Obvious, Minimal Language)** — минималистичный формат конфигов с чёткой структурой. Сочетает простоту INI и поддержку сложных структур (таблицы, массивы). Используется в Rust (Cargo.toml), Python (pyproject.toml)
	4. **INI (Initialization File)** — один из старейших форматов, представляет настройки в виде пар «ключ=значение», сгруппированных по разделам (секциям). Используется в Windows-приложениях, для простых конфигураций

2. Универсальные форматы
	1. **JSON (JavaScript Object Notation)** — лёгкий формат обмена данными, основанный на синтаксисе JavaScript. Широко используется в веб-разработке, контейнерах (Docker), Kubernetes. Характеризуется простотой парсинга и читаемостью
	2. **XML (Extensible Markup Language)** — расширяемый язык разметки с иерархической структурой (теги, атрибуты). Подходит для сложных конфигураций, где важна строгая валидация. Применяется в корпоративных системах, legacy-коде
	3. *CFG (Configuration File)* — не формат, а расширение файла
		* Typical CFG files will use a standard format, such as XML or JSON, making it easier to edit. Some CFG files, however, use a custom format and style designed by the developers.

3. Cпециализированные форматы конфигов
	1. **HCL (HashiCorp Configuration Language)** — декларативный язык конфигов, разработанный HashiCorp для инструментов Terraform, Vault, Nomad. Оптимизирован для инфраструктуры как кода (IaC), сочетает читаемость и машинную обработку.
	2. **Nix** — язык конфигурации, совмещённый с менеджером пакетов (NixOS). Функциональный и декларативный, обеспечивает воспроизводимость сборок и откатов системы.
	3. **RON (Rust Object Notation)** — формат сериализации для систем, требующих сложной структуры данных. Используется в экосистеме Rust.
	4. **Lua (как DSL)** — язык сценариев, иногда используется как домен-специфический язык (DSL) для конфигураций в игровых движках, встраиваемых системах.
	5. **RCL (Racket Configuration Language)** — надмножество JSON с поддержкой списочных выражений и нативных типов данных. Используется в некоторых специализированных системах.
	6. **Также встречаются:**
		- кастомные JSON-подобные форматы с расширениями (например, `.cjson`, `.json5`);
		- форматы, специфичные для фреймворков/библиотек (например, `.yaml` с расширениями в Spring Boot);
		- XML-диалекты (например, `.xsd`, `.dtd` для схем валидации);
		- конфигурационные форматы баз данных, контейнеров, облачных сервисов (например, `docker-compose.yml`, `terraform.tf`).

---

📋 Сравнение основных форматов

| **Формат**     | **Простота** | **Сложность** | **Комментарии** | **Типы данных** | **Использование**            |
| -------------- | ------------ | ------------- | --------------- | --------------- | ---------------------------- |
| **INI**        | ★★★★★        | Очень низкая  | `;` или `#`     | Базовые         | Windows, простые конфиги     |
| **TOML**       | ★★★☆☆        | Средняя       | `#`             | Богатые         | Rust, современные приложения |
| **Properties** | ★★★★★        | Очень низкая  | `#` или `!`     | Строковые       | Java, Spring Boot            |
| **YAML**       | ★★☆☆☆        | Высокая       | `#`             | Очень богатые   | Kubernetes, Docker, DevOps   |

### Демо конфигурационных файлов разных форматов

#### INI
**Пример:** `config.ini`
```ini
[Database]
Host=localhost
Port=5432
Username=admin
Password=securepassword

[Logging]
LogLevel=DEBUG
LogFilePath=/var/logs/app.log

[UserSettings]
Theme=Dark
Language=en-US
Notifications=Enabled

; Комментарий — не влияет на работу
[AdvancedSettings]
MaxConnections=100
Timeout=30
```
**Возможности формата:**
* группировка параметров в секции (в квадратных скобках);
* пары «ключ=значение» для настройки параметров;
* комментарии с помощью `;` или `#`;
* простота и читаемость, подходит для базовых настроек.

#### TOML
**Пример:** `config.toml`
```toml
title = "Пример конфигурации TOML"

[database]
host = "localhost"
port = 5432
username = "admin"
password = "securepassword"
ssl_mode = "require"

[logging]
level = "DEBUG"
file_path = "/var/logs/app.log"

[user_settings]
theme = "Dark"
language = "en-US"
notifications = true

[advanced_settings]
max_connections = 100
timeout = 30

[features]
enabled = ["auth", "logging", "monitoring"]
disabled = ["debug_mode"]

[[servers]]
host = "server1.example.com"
port = 8080

[[servers]]
host = "server2.example.com"
port = 8081
# 333
; dfsdf
```
**Возможности формата:**
* иерархическая структура с вложенными секциями;
* поддержка булевых значений (`true`, `false`);
* списки и массивы (например, `enabled = ["auth", "logging"]`);
* массивы объектов (секция `[[servers]]` — список серверов);
* строгая типизация (числа, строки, булевы значения);
* читаемость и компактность.

#### Properties
**Пример:** `application.properties`
```properties
# Пример конфигурации Properties
title=Пример конфигурации Properties

# Настройки базы данных
database.host=localhost
database.port=5432
database.username=admin
database.password=securepassword
database.ssl_mode=require

# Настройки логирования
logging.level=DEBUG
logging.file_path=/var/logs/app.log

# Настройки пользователя
user.theme=Dark
user.language=en-US
user.notifications=true

# Расширенные настройки
advanced.max_connections=100
advanced.timeout=30

# Массив значений (через запятую)
features.enabled=auth,logging,monitoring
features.disabled=debug_mode
```
**Возможности формата:**
* простой синтаксис «ключ=значение»;
* комментарии с помощью `#`;
* поддержка базовых типов данных (строки, числа, булевы значения);
* группировка параметров через «точечную» нотацию (`database.host`);
* компактность, удобен для Java-приложений.

#### YAML
**Пример:** `config.yaml`
```yaml
title: "Пример конфигурации YAML"

database:
  host: localhost
  port: 5432
  username: admin
  password: securepassword
  ssl_mode: require

logging:
  level: DEBUG
  file_path: /var/logs/app.log

user_settings:
  theme: Dark
  language: en-US
  notifications: true

advanced_settings:
  max_connections: 100
  timeout: 30

features:
  enabled: [auth, logging, monitoring]
  disabled: [debug_mode]

servers:
  - host: server1.example.com
    port: 8080
  - host: server2.example.com
    port: 8081

# Комментарии начинаются с #
# Это пример комментария
```
**Возможности формата:**
* иерархическая структура с отступами (вместо скобок);
* поддержка списков и массивов (`enabled: [auth, logging, monitoring]`);
* вложенные объекты и массивы объектов (секция `servers`);
* поддержка булевых значений (`true`, `false`);
* комментарии с помощью `#`;
* гибкость и читаемость, популярен в DevOps и Kubernetes.

# Полное демо возможностей формата

## INI
> Initialization File

```ini
; INI формат - простой и понятный
; Основные возможности INI формата

; 1. Комментарии (начинаются с ; или #)
# Это тоже комментарий
; Секция обычно называется [Section]

; 2. Базовые секции
[Database]
; Простые пары ключ=значение
host = localhost
port = 5432
database = myapp
username = admin
password = secret123  ; Пароли обычно хранятся отдельно

; 3. Разные форматы значений
connection_string = postgresql://${host}:${port}/${database}
retry_count = 3
timeout = 30.5
enabled = true
null_value =  ; Пустое значение

; 4. Многострочные значения (некоторые парсеры поддерживают)
description = Это значение занимает
    несколько строк с
    отступами

; 5. Специальные символы
special_chars = C:\Program Files\MyApp\data
url = https://example.com/api/v1/users?page=1&limit=10

; 6. Секции с подсекциями (не стандартно, но некоторые парсеры поддерживают)
[Database.Primary]
host = db-primary.example.com
role = master

[Database.Replica]
host = db-replica.example.com
role = slave

; 7. Логическая группировка
[WebServer]
; Настройки веб-сервера
hostname = 0.0.0.0
port = 8080
ssl_enabled = false

[WebServer.SSL]
; Подсекция для SSL
cert_file = /path/to/cert.pem
key_file = /path/to/key.pem
protocols = TLSv1.2, TLSv1.3

; 8. Списки значений (не стандартно)
[Features]
; Обычно через запятую
enabled_features = auth,logging,caching
disabled_features = debug,profiling

; 9. Переменные окружения (расширение)
[Environment]
; Подстановка переменных окружения
home_dir = %USERPROFILE%  ; Windows
; или
home_dir = ${HOME}        ; Unix-like

; 10. Наследование секций (некоторые реализации)
[Production:Database]  ; Наследует от Database
host = prod-db.example.com
; остальные параметры берутся из [Database]

; 11. Секции без значений
[Dependencies]
; Только для группировки
requires_java = true
requires_python = false

; 12. Типы данных (все строки, но можно интерпретировать)
[Types]
integer_value = 42
float_value = 3.14
boolean_true = true
boolean_false = false
string_number = "123"  ; Явно строка
quoted_string = "Hello, World!"
empty_string = ""

; 13. Экранирование символов
[Paths]
; В некоторых реализациях
path_with_spaces = "C:\\Program Files\\My App"
path_with_equals = C\=Program Files\=MyApp  ; Альтернатива

; 14. Секции с повторяющимися ключами (не рекомендуется)
[Logging]
file = /var/log/app.log
file = /var/log/app2.log  ; Перезапишет предыдущее

; 15. Глобальная секция (без имени)
; Параметры без секции
app_name = MyApplication
version = 1.0.0

[DEFAULT]  ; Специальная секция для значений по умолчанию
timeout = 30
retries = 3

; 16. Включение других файлов (некоторые реализации)
#include "database.ini"
#include "security.ini"

; 17. Массивы (не стандартно, но некоторые парсеры понимают)
[Servers]
server[] = server1.example.com
server[] = server2.example.com
server[] = server3.example.com

; 18. Сложные структуры (ограниченно)
[User Preferences]
theme = dark
language = ru_RU
notifications.email = true
notifications.push = false

; 19. Интерполяция значений (в расширенных реализациях)
[Network]
base_url = https://api.example.com
users_endpoint = ${base_url}/v1/users
posts_endpoint = ${base_url}/v1/posts

; 20. Пример полной конфигурации приложения
[Application]
name = "My Awesome App"
version = "2.1.0"
debug_mode = false
log_level = INFO

[Database]
type = postgresql
host = localhost
port = 5432
name = production_db
user = ${DB_USER}  ; Переменная окружения
password = ${DB_PASSWORD}

[Redis]
host = redis.local
port = 6379
database = 0
password = ; Опционально

[API]
port = 8080
workers = 4
timeout = 30
cors_origins = https://example.com,http://localhost:3000

[Features]
enable_cache = true
enable_metrics = false
enable_tracing = true

[Monitoring]
prometheus_port = 9090
health_check_interval = 30
alert_email = admin@example.com

[Security]
jwt_secret = ${JWT_SECRET}
token_expiry = 3600
require_https = true
allowed_ips = 192.168.1.0/24,10.0.0.0/8
```

## TOML
> Tom's Obvious Minimal Language

```toml
# TOML - Tom's Obvious Minimal Language
# Полная демонстрация возможностей формата

# 1. Комментарии (только #)
# Это комментарий в TOML
title = "TOML Example"

# 2. Строки
str = "I'm a string. \"You can quote me\". Name\tJos\u00E9\nLocation\tSF."
literal_str = 'C:\Users\nodejs\templates'  # Без экранирования
multiline_str = """
Это многострочная
строка в TOML.
"""

# 3. Числа
int1 = +99
int2 = 42
int3 = 0
int4 = -17
int5 = 1_000_000  # С разделителями
float1 = 3.1415
float2 = -0.01
float3 = 5e+22
float4 = 1e6
float5 = -2E-2
float6 = 6.626e-34

# 4. Булевы значения
bool_true = true
bool_false = false

# 5. Дата и время
date1 = 1979-05-27T07:32:00Z
date2 = 1979-05-27T00:32:00.999999-07:00
date3 = 1979-05-27 07:32:00Z  # Альтернативный формат

# 6. Массивы
array1 = [1, 2, 3]
array2 = ["red", "yellow", "green"]
array3 = [[1, 2], [3, 4, 5]]  # Массив массивов
array4 = [
  "Все",
  'строки',
  """одинакового""",
  '''типа''',
]
array5 = [
  { x = 1, y = 2 },
  { x = 3, y = 4 },
]

# 7. Таблицы (основные объекты)
[database]
enabled = true
ports = [8000, 8001, 8002]
data = [ ["delta", "phi"], [3.14] ]
temp_targets = { cpu = 79.5, case = 72.0 }

[database.connection]
max_connections = 500
timeout_seconds = 3.5

[database.connection.credentials]
username = "admin"
password = "secret"

# 8. Вложенные таблицы (dot notation)
[servers]
[servers.alpha]
ip = "10.0.0.1"
role = "frontend"

[servers.beta]
ip = "10.0.0.2"
role = "backend"

# 9. Массив таблиц (массив объектов)
[[products]]
name = "Hammer"
sku = 738594937

[[products]]
name = "Nail"
sku = 284758393
color = "gray"

# 10. Сложные массивы таблиц
[[fruit]]
name = "apple"

[fruit.physical]
color = "red"
shape = "round"

[[fruit.variety]]
name = "red delicious"

[[fruit.variety]]
name = "granny smith"

[[fruit]]
name = "banana"

[[fruit.variety]]
name = "plantain"

# 11. Inline таблицы (компактный синтаксис)
name = { first = "Tom", last = "Preston-Werner" }
point = { x = 1, y = 2 }
animal = { type.name = "pug" }

# 12. Многоуровневые inline таблицы
[net]
ip = "192.168.1.1"
ports = [80, 443]
net.connection = { enabled = true, timeout = 30 }

# 13. Специальные символы в ключах
# Ключи могут быть в кавычках
"127.0.0.1" = "value"
"character encoding" = "value"
"ʎǝʞ" = "value"
'quoted "value"' = "value"

# 14. Расширенный пример приложения
[app]
name = "My Application"
version = "1.0.0"
description = """
Многострочное описание
приложения на русском языке.
"""

# 15. Конфигурация БД с разными окружениями
[production.database]
host = "prod-db.example.com"
port = 5432
pool_size = 20

[staging.database]
host = "staging-db.example.com"
port = 5432
pool_size = 10

[development.database]
host = "localhost"
port = 5432
pool_size = 5

# 16. Настройки логирования
[log]
level = "INFO"
format = "json"

[log.app]
file = "/var/log/app.log"
max_size = "100MB"
backup_count = 5

[log.access]
file = "/var/log/access.log"
enabled = true

# 17. Настройки кэширования
[cache]
default_ttl = 300  # секунд
max_size = "1GB"

[cache.redis]
enabled = true
host = "localhost"
port = 6379
database = 0

[cache.memcached]
enabled = false
servers = ["cache1:11211", "cache2:11211"]

# 18. API конфигурация
[api]
version = "v1"
base_path = "/api/v1"

[api.auth]
enabled = true
jwt_secret = "your-secret-key"
token_expiry = 3600

[api.rate_limit]
enabled = true
requests_per_minute = 100
burst_size = 20

# 19. Мониторинг и метрики
[monitoring]
enabled = true

[monitoring.prometheus]
port = 9090
path = "/metrics"

[monitoring.health]
liveness_path = "/health"
readiness_path = "/ready"

# 20. Уведомления
[notifications]
enabled = true

[notifications.email]
smtp_host = "smtp.gmail.com"
smtp_port = 587
username = "user@gmail.com"
password = "password"  # В реальности из переменных окружения

[notifications.slack]
webhook_url = "https://hooks.slack.com/services/..."
channel = "#alerts"
username = "Bot"

# 21. Секреты (обычно из переменных окружения)
[secrets]
database_url = "${DATABASE_URL}"
api_key = "${API_KEY}"

# 22. Кастомные типы через строки
[types]
duration = "30s"
size = "100MB"
percentage = "95%"

# 23. Пример комплексной конфигурации
[application]
title = "Production Configuration"
created = 2024-01-15T10:30:00Z
author = { name = "Admin", email = "admin@example.com" }

[application.features]
dark_mode = true
notifications = ["email", "push"]
supported_languages = ["en", "ru", "es"]

[[application.servers]]
name = "web1"
role = "load_balancer"
ports = [80, 443]
metadata = { region = "us-east-1", zone = "a" }

[[application.servers]]
name = "web2"
role = "application"
ports = [8080]
metadata = { region = "us-east-1", zone = "b" }

[application.dependencies]
python = ">=3.8"
packages = ["flask>=2.0", "sqlalchemy>=1.4"]

# 24. Специальные значения
[special]
inf = inf  # Бесконечность
neg_inf = -inf
nan = nan  # Не число

# 25. Экранирование в строках
[escaping]
str1 = "Значение с \"кавычками\""
str2 = 'Одинарные кавычки: isn\'t'
str3 = """
Многострочная
с "разными" 'кавычками'
"""
```

## Properties
> Java Properties

```properties
# Java Properties File Format
# Полная демонстрация возможностей формата

# 1. Комментарии (начинаются с # или !)
! Это тоже комментарий
# Основные настройки приложения

# 2. Базовые пары ключ-значение
app.name=My Application
app.version=1.0.0
app.environment=production

# 3. Строковые значения
string.value=Hello World
quoted.string="Quoted Value"
empty.string=
spaces.in.value=Value with spaces

# 4. Числовые значения (хранятся как строки)
integer.value=42
negative.number=-100
float.value=3.14159
scientific.notation=1.23e+4

# 5. Булевы значения
boolean.true=true
boolean.false=false
boolean.yes=yes
boolean.no=no
boolean.on=on
boolean.off=off

# 6. Списки через запятую
server.hosts=host1.example.com,host2.example.com,host3.example.com
supported.locales=en_US,ru_RU,fr_FR,de_DE
enabled.features=auth,logging,cache,monitoring

# 7. Многострочные значения (обратный слэш в конце строки)
long.description=Это очень длинное описание, \
    которое продолжается на следующей строке, \
    и еще на одной.

# 8. Специальные символы (экранирование)
path=C\:\\Program Files\\MyApp
url=https\\://example.com/path\\?param=value
special.chars=\\t\\n\\r\\f (таб, новая строка, возврат каретки, перевод формата)

# 9. Unicode символы
unicode.hello=\u041F\u0440\u0438\u0432\u0435\u0442
unicode.snowman=\u2603

# 10. Пустые значения
empty.value=
null.value=null
none.value=None

# 11. Дата и время (как строки)
start.date=2024-01-15
timestamp=2024-01-15T10:30:00Z

# 12. Иерархия через точки
database.host=localhost
database.port=5432
database.name=production
database.credentials.username=admin
database.credentials.password=secret123

# 13. Массивы (не стандартно, но используется)
server.ports=8080,8081,8082
server.names.0=web-server-1
server.names.1=web-server-2
server.names.2=web-server-3

# 14. Логические группы
# Группа: Настройки веб-сервера
server.hostname=0.0.0.0
server.port=8080
server.context.path=/api
server.servlet.session.timeout=30m

# Группа: Настройки базы данных
db.driver=org.postgresql.Driver
db.url=jdbc:postgresql://${database.host}:${database.port}/${database.name}
db.username=${database.credentials.username}
db.password=${database.credentials.password}
db.pool.size=10
db.pool.timeout=30000

# 15. Переменные (подстановка значений)
base.url=https://api.example.com
users.endpoint=${base.url}/v1/users
posts.endpoint=${base.url}/v1/posts
comments.endpoint=${base.url}/v1/posts/{id}/comments

# 16. Окружение (environment-specific)
# application.properties
spring.profiles.active=prod

# application-prod.properties
logging.level.root=WARN
server.port=8443
security.require-ssl=true

# application-dev.properties
logging.level.root=DEBUG
server.port=8080
security.require-ssl=false

# 17. Системные свойства и переменные окружения
user.home=${user.home}
java.home=${JAVA_HOME}
app.data.dir=${APP_DATA}/storage
temp.dir=${java.io.tmpdir}/myapp

# 18. Булевы флаги с разными значениями
feature.auth.enabled=true
feature.cache.enabled=yes
feature.debug.enabled=on
feature.maintenance.enabled=false
feature.legacy.enabled=no
feature.experimental.enabled=off

# 19. Размеры и единицы измерения
cache.size.max=1024MB
heap.size.initial=512m
heap.size.max=2G
file.upload.max.size=10MB
session.timeout=30m
connection.timeout=30s

# 20. Цвета и форматы
ui.theme=dark
ui.primary.color=#FF5733
ui.font.family=Arial, sans-serif
ui.font.size=14px

# 21. Полный пример конфигурации Spring Boot приложения
# Основные настройки приложения
spring.application.name=myapp
spring.profiles.active=prod

# Настройки сервера
server.port=8080
server.servlet.context-path=/api
server.servlet.session.timeout=30m
server.compression.enabled=true
server.compression.mime-types=text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json

# Настройки базы данных
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=admin
spring.datasource.password=secret
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=30000
spring.datasource.hikari.connection-timeout=20000

# JPA/Hibernate настройки
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.jdbc.batch_size=20

# Кэширование
spring.cache.type=redis
spring.cache.redis.time-to-live=600000
spring.cache.redis.cache-null-values=true

# Redis настройки
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.password=
spring.redis.database=0
spring.redis.timeout=2000ms

# Логирование
logging.level.root=INFO
logging.level.com.myapp=DEBUG
logging.level.org.springframework.web=WARN
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
logging.file.name=/var/log/myapp/application.log
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

# Безопасность
spring.security.user.name=admin
spring.security.user.password=admin123
spring.security.user.roles=ADMIN,USER
security.jwt.secret=mySecretKey
security.jwt.expiration=86400000

# Мейл
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=myapp@gmail.com
spring.mail.password=emailpassword
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true

# Межсервисная связь
app.services.auth.url=http://auth-service:8080
app.services.payment.url=http://payment-service:8080
app.services.notification.url=http://notification-service:8080

# Настройки интеграции
integration.external.api.base-url=https://api.external.com/v1
integration.external.api.key=${EXTERNAL_API_KEY}
integration.external.api.timeout=5000
integration.external.api.retry.max-attempts=3
integration.external.api.retry.delay=1000

# Здоровье и мониторинг
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=when_authorized
management.metrics.export.prometheus.enabled=true
management.metrics.tags.application=${spring.application.name}
management.metrics.tags.environment=${spring.profiles.active}

# Расписание задач
app.scheduler.report-generation.cron=0 0 6 * * *
app.scheduler.data-cleanup.cron=0 0 0 * * *
app.scheduler.backup.cron=0 0 2 * * 0

# Кастомные настройки приложения
app.features.enable-experimental=false
app.features.enable-multi-tenant=true
app.features.default-language=ru_RU
app.features.supported-currencies=RUB,USD,EUR
app.ui.theme=dark
app.ui.timezone=Europe/Moscow
```

## YAML
> YAML Ain't Markup Language

```yaml
# YAML - YAML Ain't Markup Language
# Полная демонстрация возможностей формата

# 1. Комментарии
# Это комментарий в YAML
key: value  # Инлайновый комментарий

# 2. Скалярные типы (scalars)
string: "Hello, World!"  # Двойные кавычки
simple_string: Простая строка  # Без кавычек
single_quoted: 'Экранирование \'внутри\''  # Одинарные кавычки
boolean_true: true
boolean_false: false
integer: 42
float: 3.14159
scientific: 1.23e+4
null_value: null  # или ~
infinity: .inf  # Бесконечность
not_a_number: .nan  # Не число

# 3. Многострочные строки
plain: |
  Это многострочная
  строка с сохранением
  переносов строк.

folded: >
  Это свернутая строка,
  где переносы заменяются
  пробелами.

literal_block: |
  Сохраняет все символы:
    - отступы
    - пробелы
    - переносы строк

# 4. Якоря и алиасы (anchors & aliases)
defaults: &defaults
  adapter: postgresql
  host: localhost
  port: 5432

development:
  <<: *defaults  # Мерджинг якоря
  database: dev_db

test:
  <<: *defaults
  database: test_db

# 5. Списки/последовательности (sequences)
simple_list:
  - apple
  - banana
  - cherry

nested_list:
  - 
    - 1
    - 2
  - 
    - 3
    - 4

inline_list: [apple, banana, cherry]  # Inline синтаксис

mixed_list:
  - строка
  - 42
  - 3.14
  - true
  - null

# 6. Словари/отображения (mappings)
simple_dict:
  key1: value1
  key2: value2
  nested:
    inner_key: inner_value

inline_dict: {key1: value1, key2: value2}  # Inline синтаксис

complex_nesting:
  level1:
    level2:
      level3:
        - item1
        - item2
        - item3

# 7. Типы данных YAML
# !!str - явное указание типа
string_type: !!str "123"
# !!int
integer_type: !!int "123"  # Строка преобразуется в число
# !!float
float_type: !!float "3.14"
# !!bool
bool_type: !!bool "true"
# !!null
null_type: !!null "null"
# !!timestamp
timestamp: !!timestamp 2024-01-15T10:30:00Z
# !!binary
binary_data: !!binary |
  R0lGODlhDAAMAIQAAP//9/X17unp5WZmZgAAAOfn515eXvPz7Y6OjuDg4J+fn5
  OTk6enp56enmlpaWNjY6Ojo4SEhP/++f/++f/++f/++f/++f/++f/++f/++f/+
  +f/++f/++f/++f/++f/++SH+Dk1hZGUgd2l0aCBHSU1QACwAAAAADAAMAAAFLC
  AgjoEwnuNAFOhpEMTRiggcz4BNJHrv/zCFcLiwMWYNG84BwwEeECcgggoBADs=

# 8. Кастомные теги
!CustomTag
  data: значение
  number: 42

# 9. Документы в потоке (multiple documents)
---
# Первый документ
app: Application One
version: 1.0.0
...
---
# Второй документ
app: Application Two  
version: 2.0.0
...

# 10. Сложные структуры с типами
configuration: !!map
  database: !!map
    type: !!str "postgresql"
    connection: !!map
      host: !!str "localhost"
      port: !!int 5432
    credentials: !!map
      username: !!str "admin"
      password: !!str "secret"

# 11. Пример полной конфигурации приложения
# Application Configuration
app:
  name: "My Awesome App"
  version: "2.1.0"
  description: |
    Многострочное описание
    приложения на русском языке
    с поддержкой Unicode.
  
  metadata:
    created: 2024-01-15T10:30:00Z
    author:
      name: "Иван Петров"
      email: "ivan@example.com"
      department: "Engineering"
    
    tags: &app_tags
      - web
      - api
      - microservices
      - production

# Server Configuration  
server:
  host: "0.0.0.0"
  port: 8080
  ssl:
    enabled: true
    certificate: "/path/to/cert.pem"
    key: "/path/to/key.pem"
    protocols:
      - "TLSv1.2"
      - "TLSv1.3"
  
  cors:
    allowed_origins:
      - "https://example.com"
      - "http://localhost:3000"
    allowed_methods: ["GET", "POST", "PUT", "DELETE", "OPTIONS"]
    allowed_headers: ["Content-Type", "Authorization"]
    allow_credentials: true

# Database Configuration
database:
  primary: &db_config
    adapter: "postgresql"
    host: ${DB_HOST:-localhost}
    port: ${DB_PORT:-5432}
    database: "production_db"
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    pool: 20
    timeout: 30
    ssl: true
    
    connection_params:
      connect_timeout: 10
      application_name: "myapp"
      sslmode: "require"

  replica:
    <<: *db_config
    host: "replica-db.example.com"
    role: "read-only"
    pool: 10

# Cache Configuration  
cache:
  default_ttl: 300  # seconds
  
  redis:
    enabled: true
    hosts:
      - host: "redis-1.example.com"
        port: 6379
        weight: 1
      - host: "redis-2.example.com"
        port: 6379
        weight: 2
    
    sentinel:
      enabled: false
      master: "mymaster"
      nodes:
        - "sentinel1:26379"
        - "sentinel2:26379"
    
    cluster:
      enabled: true
      max_redirects: 3
    
    connection:
      timeout: 2000
      retry_attempts: 3
      retry_delay: 100

# Logging Configuration
logging:
  level: "INFO"
  
  appenders:
    - type: "file