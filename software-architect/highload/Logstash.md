# Logstash - полное руководство

## Что такое Logstash?

**Logstash** — Конвейер (pipeline) обработки данных
* **Logstash** — это инструмент с открытым исходным кодом для приема, обработки и передачи данных в реальном времени. Является ключевым компонентом стека [[ELK]] ([[Elasticsearch]], [[Logstash]], [[Kibana]]).

## Архитектура Logstash

**Конвейер обработки данных (Pipeline)**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Inputs    │───▶│  Filters    │───▶│   Outputs   │
│             │    │             │    │             │
│ • File      │    │ • Grok      │    │ • Elastic-  │
│ • Beats     │    │ • Mutate    │    │   search    │
│ • Kafka     │    │ • Date      │    │ • Kafka     │
│ • HTTP      │    │ • GeoIP     │    │ • File      │
│ • JDBC      │    │ • UserAgent │    │ • S3        │
└─────────────┘    └─────────────┘    └─────────────┘
```

## Основные компоненты

### 1. **Inputs (Входы)**
Прием данных из различных источников.

```ruby
# Примеры входных плагинов
input {
  # Чтение из файла
  file {
    path => "/var/log/nginx/access.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
  
  # Прием из Filebeat
  beats {
    port => 5044
  }
  
  # HTTP endpoint
  http {
    port => 8080
  }
  
  # Kafka topic
  kafka {
    bootstrap_servers => "kafka:9092"
    topics => ["logs"]
  }
  
  # База данных
  jdbc {
    jdbc_connection_string => "jdbc:mysql://localhost:3306/db"
    jdbc_user => "user"
    jdbc_password => "password"
    statement => "SELECT * FROM logs"
  }
}
```

### 2. **Filters (Фильтры)**
Обработка и преобразование данных.

```ruby
filter {
  # Парсинг JSON
  if [message] =~ /^{.*}$/ {
    json {
      source => "message"
    }
  }
  
  # Парсинг логов с помощью Grok
  grok {
    match => { 
      "message" => "%{COMBINEDAPACHELOG}" 
    }
  }
  
  # Парсинг даты
  date {
    match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
  
  # Геолокация по IP
  geoip {
    source => "clientip"
    target => "geoip"
  }
  
  # Анализ User-Agent
  useragent {
    source => "user_agent"
    target => "user_agent_info"
  }
  
  # Мутация полей
  mutate {
    convert => { "response" => "integer" }
    gsub => [ "message", "\r", "" ]
    rename => { "host" => "server_host" }
    remove_field => [ "headers" ]
  }
  
  # Условная обработка
  if [response] == 404 {
    mutate { add_tag => "not_found" }
  }
}
```

### 3. **Outputs (Выходы)**
Отправка обработанных данных в целевые системы.

```ruby
output {
  # В Elasticsearch
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
    user => "elastic"
    password => "password"
  }
  
  # В файл (для отладки)
  file {
    path => "/tmp/processed_logs.log"
  }
  
  # В Kafka
  kafka {
    codec => json
    topic_id => "processed_logs"
    bootstrap_servers => "kafka:9092"
  }
  
  # В Amazon S3
  s3 {
    bucket => "my-log-bucket"
    region => "us-east-1"
    time_file => 5
    codec => "json"
  }
  
  # В stdout для отладки
  stdout { 
    codec => rubydebug 
  }
}
```

## Полные примеры конфигураций

**Пример 1: Обработка веб-логов**
```ruby
input {
  file {
    path => "/var/log/nginx/access.log"
    type => "nginx-access"
    start_position => "beginning"
  }
}

filter {
  # Парсинг Apache-подобного лога
  grok {
    match => { 
      "message" => "%{COMBINEDAPACHELOG}" 
    }
  }
  
  # Преобразование даты
  date {
    match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    remove_field => [ "timestamp" ]
  }
  
  # Геолокация
  geoip {
    source => "clientip"
  }
  
  # User-Agent анализ
  useragent {
    source => "agent"
    target => "user_agent"
  }
  
  # Преобразование полей
  mutate {
    convert => { 
      "response" => "integer"
      "bytes" => "integer" 
    }
    rename => { 
      "request" => "url"
      "agent" => "user_agent_raw" 
    }
    remove_field => [ "message" ]
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "nginx-access-%{+YYYY.MM.dd}"
  }
  
  stdout { codec => rubydebug }
}
```

**Пример 2: Обработка JSON логов приложения**
```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  # Парсинг JSON
  json {
    source => "message"
  }
  
  # Добавление timestamp
  date {
    match => [ "log_timestamp", "ISO8601" ]
    target => "@timestamp"
  }
  
  # Обогащение данных
  mutate {
    add_field => { 
      "environment" => "%{[fields][env]}"
      "application" => "%{[fields][app]}" 
    }
  }
  
  # Обработка ошибок
  if [level] == "ERROR" {
    mutate {
      add_tag => [ "error", "alert" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
    document_type => "_doc"
  }
}
```

## [[Grok]] Patterns

**Создание кастомных паттернов**
```ruby
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { 
      "message" => "%{MY_LOG_PATTERN}" 
    }
  }
}
```

```bash
# custom_patterns file
MY_LOG_PATTERN \[%{TIMESTAMP_ISO8601:timestamp}\] %{LOGLEVEL:level} %{GREEDYDATA:message}
```

**Популярные Grok паттерны**
```ruby
# Apache access log
%{COMBINEDAPACHELOG}

# Syslog
%{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST:hostname} %{DATA:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:message}

# JSON log
^{"timestamp":"%{TIMESTAMP_ISO8601:timestamp}","level":"%{LOGLEVEL:level}","message":"%{GREEDYDATA:message}"}
```

## Управление и эксплуатация

**Запуск Logstash**
```bash
# Базовая команда
bin/logstash -f config_file.conf

# С проверкой конфигурации
bin/logstash -f config_file.conf --config.test_and_exit

# Автоматическая перезагрузка при изменении конфигурации
bin/logstash -f config_file.conf --config.reload.automatic

# С указанием pipeline workers
bin/logstash -f config_file.conf --pipeline.workers 4
```

**Конфигурация JVM**
```yaml
# config/jvm.options
-Xms2g
-Xmx2g
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly
```

**Мониторинг Logstash**
```ruby
input {
  # Встроенный мониторинг
  http {
    port => 9600
  }
}

# API мониторинга
# GET http://localhost:9600/_node/stats
# GET http://localhost:9600/_node/pipelines
```

## Продвинутые техники

**Работа с многолинейными логами**
```ruby
input {
  file {
    path => "/var/log/java/app.log"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601} "
      negate => true
      what => "previous"
    }
  }
}
```

**Клонирование событий**
```ruby
filter {
  # Клонирование для разных обработок
  clone {
    clones => [ "debug_copy" ]
  }
}

output {
  if [type] == "debug_copy" {
    # Отправка в отдельный индекс для отладки
    elasticsearch {
      index => "debug-logs-%{+YYYY.MM.dd}"
    }
  }
}
```

**Агрегация событий**
```ruby
filter {
  aggregate {
    task_id => "%{session_id}"
    code => "
      map['events'] ||= []
      map['events'] << event.get('event_type')
      map['start_time'] ||= event.get('@timestamp')
      map['end_time'] = event.get('@timestamp')
    "
    push_previous_map_as_event => true
    timeout => 300
  }
}
```

## Оптимизация производительности

**Настройки pipeline**
```ruby
# pipelines.yml
- pipeline.id: main
  path.config: "/etc/logstash/conf.d/*.conf"
  pipeline.workers: 4
  pipeline.batch.size: 125
  pipeline.batch.delay: 50
```

**Рекомендации по настройке**
```yaml
# Для high-throughput систем:
pipeline.workers: количество ядер CPU
pipeline.batch.size: 125-250
pipeline.batch.delay: 50-100 ms

# Мониторинг очереди
queue.type: persisted
queue.max_bytes: 4gb
```

## Troubleshooting и отладка

**Логи Logstash**
```bash
# Просмотр логов
tail -f /var/log/logstash/logstash-plain.log

# Включение debug логов
--log.level debug
```

**Тестирование конфигурации**
```bash
# Валидация конфигурации
bin/logstash -f config.conf --config.test_and_exit

# Сухой запуск
bin/logstash -f config.conf --config.reload.automatic --debug
```

## Интеграция с ELK стеком

**Полный pipeline ELK**
```
Application Logs → Filebeat → Logstash → Elasticsearch → Kibana
```

**Конфигурация Filebeat → Logstash**
```yaml
# filebeat.yml
output.logstash:
  hosts: ["logstash:5044"]
  loadbalance: true
```

## Best Practices

**Структура конфигураций**
```
/etc/logstash/
├── conf.d/
│   ├── 01-inputs.conf
│   ├── 10-filters.conf
│   └── 30-outputs.conf
├── patterns/
│   └── custom_patterns
└── pipelines.yml
```

**Безопасность**
```ruby
# SSL/TLS для beats input
beats {
  port => 5044
  ssl => true
  ssl_certificate_authorities => ["/path/to/ca.crt"]
  ssl_certificate => "/path/to/server.crt"
  ssl_key => "/path/to/server.key"
}
```

Logstash предоставляет мощный и гибкий инструмент для обработки данных, который может масштабироваться от простых задач до сложных ETL-процессов в реальном времени.