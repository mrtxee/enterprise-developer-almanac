# Grok в [[Logstash]] - полное руководство

## Что такое Grok?

**Grok** — это фильтр в [[Logstash]], который позволяет парсить и структурировать неструктурированные логовые данные с помощью шаблонов на основе регулярных выражений. Название "grok" происходит из научной фантастики и означает "понять что-то глубоко и интуитивно".

## Основные концепции

### **Синтаксис Grok**
```
%{SYNTAX:SEMANTIC[:FIELD_TYPE]}
```

- **SYNTAX** - имя шаблона (pattern)
- **SEMANTIC** - имя поля для сохранения результата
- **FIELD_TYPE** - тип данных (опционально)

### **Пример базового использования**
```ruby
filter {
  grok {
    match => { 
      "message" => "%{IP:client_ip} %{WORD:http_method} %{URIPATHPARAM:request}" 
    }
  }
}
```

## Встроенные шаблоны

### **Общие шаблоны**
```ruby
# IP адреса
%{IP:client_ip}
%{IPV6:client_ipv6}

# Числа
%{NUMBER:bytes_sent}
%{BASE10NUM:response_time}

# Даты и время
%{TIMESTAMP_ISO8601:timestamp}
%{HTTPDATE:log_timestamp}

# Строки
%{WORD:http_method}
%{NOTSPACE:filename}
%{DATA:user_agent}
%{GREEDYDATA:full_message}

# Сетевые
%{HOSTNAME:server_name}
%{URIPROTO:protocol}
%{URIHOST:domain}
%{URIPATHPARAM:request_uri}
```

### **Композитные шаблоны**
```ruby
# Стандартные комбинации
%{COMBINEDAPACHELOG}          # Полный Apache access log
%{COMMONAPACHELOG}            # Базовый Apache access log
%{SYSLOGLINE}                 # Стандартный syslog
```

## Практические примеры

### **Пример 1: Парсинг Apache логов**
```ruby
input {
  file {
    path => "/var/log/apache2/access.log"
  }
}

filter {
  grok {
    match => { 
      "message" => "%{COMBINEDAPACHELOG}" 
    }
  }
}

# После обработки лога:
# 192.168.1.100 - john [10/Oct/2023:14:30:01 +0000] "GET /index.html HTTP/1.1" 200 1234
#
# Будут созданы поля:
# clientip: 192.168.1.100
# ident: -
# auth: john
# timestamp: 10/Oct/2023:14:30:01 +0000
# verb: GET
# request: /index.html
# httpversion: 1.1
# response: 200
# bytes: 1234
```

### **Пример 2: Кастомный лог приложения**
```ruby
filter {
  grok {
    match => { 
      "message" => "\[%{TIMESTAMP_ISO8601:timestamp}\] \[%{LOGLEVEL:log_level}\] \[%{WORD:service}\] %{GREEDYDATA:log_message}" 
    }
  }
}

# Обрабатывает лог:
# [2023-10-10T14:30:01Z] [ERROR] [payment-service] Failed to process transaction 12345: insufficient funds
#
# Результат:
# timestamp: 2023-10-10T14:30:01Z
# log_level: ERROR
# service: payment-service
# log_message: Failed to process transaction 12345: insufficient funds
```

### **Пример 3: Многострочные логи**
```ruby
filter {
  grok {
    match => { 
      "message" => "Exception: %{WORD:exception_type}: %{GREEDYDATA:exception_message}\s+at %{GREEDYDATA:stack_trace}" 
    }
  }
}
```

## Создание кастомных шаблонов

### **Способ 1: Встроенные шаблоны в конфигурации**
```ruby
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { 
      "message" => "%{MY_CUSTOM_PATTERN:my_field}" 
    }
  }
}
```

### **Способ 2: Файл с кастомными шаблонами**
```bash
# Создаем файл ./patterns/custom_patterns
MY_TIMESTAMP %{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{HOUR}:%{MINUTE}:%{SECOND}
TRANSACTION_ID [A-Z0-9]{8}-[A-Z0-9]{4}-[A-Z0-9]{4}-[A-Z0-9]{4}-[A-Z0-9]{12}
EMAIL [a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}
```

### **Способ 3: Прямое использование regex**
```ruby
filter {
  grok {
    match => { 
      "message" => "(?<user_id>\d+) \[(?<action>\w+)\] %{GREEDYDATA:details}" 
    }
  }
}
```

## Расширенные возможности

### **Множественные шаблоны**
```ruby
filter {
  grok {
    match => { 
      "message" => [
        "%{COMBINEDAPACHELOG}",           # Попытка 1: Apache log
        "%{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST:hostname} %{DATA:program}: %{GREEDYDATA:message}",  # Попытка 2: Syslog
        "%{TIMESTAMP_ISO8601:timestamp} \[%{LOGLEVEL:level}\] %{GREEDYDATA:message}"  # Попытка 3: Кастомный формат
      ]
    }
    break_on_match => false
  }
}
```

### **Типы данных и преобразования**
```ruby
filter {
  grok {
    match => { 
      "message" => "%{NUMBER:response_code:int} %{NUMBER:response_time:float} %{NUMBER:bytes_sent:long}" 
    }
  }
}
```

### **Условная обработка**
```ruby
filter {
  if [type] == "apache" {
    grok {
      match => { 
        "message" => "%{COMBINEDAPACHELOG}" 
      }
    }
  } else if [type] == "application" {
    grok {
      match => { 
        "message" => "\[%{TIMESTAMP_ISO8601:timestamp}\] %{LOGLEVEL:level} %{GREEDYDATA:message}" 
      }
    }
  }
}
```

## Отладка и тестирование

### **Grok Debugger**
```ruby
filter {
  grok {
    match => { 
      "message" => "%{COMBINEDAPACHELOG}" 
    }
    # Добавляем отладочную информацию
    add_tag => ["grok_debug"]
    break_on_match => false
  }
}
```

### **Тестирование шаблонов**
```bash
# Использование онлайн Grok Debugger в Kibana
# Или локальное тестирование:

# Пример тестовой строки:
echo '192.168.1.100 - john [10/Oct/2023:14:30:01 +0000] "GET /index.html HTTP/1.1" 200 1234' | \
logstash -e 'input { stdin { } } filter { grok { match => { "message" => "%{COMBINEDAPACHELOG}" } } } output { stdout { codec => rubydebug } }'
```

## Распространенные проблемы и решения

### **Проблема 1: GrokParseFailure**
```ruby
filter {
  grok {
    match => { 
      "message" => "%{COMBINEDAPACHELOG}" 
    }
    # Добавляем тег при ошибке парсинга
    tag_on_failure => ["_grokparsefailure"]
  }
  
  # Обработка неудачных парсингов
  if "_grokparsefailure" in [tags] {
    # Альтернативная обработка или логирование
  }
}
```

### **Проблема 2: Производительность**
```ruby
filter {
  grok {
    match => { 
      "message" => "%{COMBINEDAPACHELOG}" 
    }
    # Оптимизация для высоконагруженных систем
    break_on_match => true
    timeout_millis => 10000
  }
}
```

## Продвинутые техники

### **Вложенные парсеры**
```ruby
filter {
  grok {
    match => { 
      "message" => "%{SYSLOGBASE} %{GREEDYDATA:message}" 
    }
  }
  
  # Дополнительный парсинг извлеченного поля
  grok {
    match => { 
      "message" => "Error: %{WORD:error_type}: %{GREEDYDATA:error_details}" 
    }
  }
}
```

### **Парсинг ключ-значение**
```ruby
filter {
  grok {
    match => { 
      "message" => "%{WORD:key1}=%{QUOTEDSTRING:value1} %{WORD:key2}=%{NUMBER:value2}" 
    }
  }
}
```

### **Использование conditionals с grok**
```ruby
filter {
  # Сначала извлекаем базовую информацию
  grok {
    match => { 
      "message" => "\[%{TIMESTAMP_ISO8601:timestamp}\] %{LOGLEVEL:level} %{GREEDYDATA:message}" 
    }
  }
  
  # Затем парсим специфичные форматы в зависимости от уровня лога
  if [level] == "ERROR" {
    grok {
      match => { 
        "[message]" => "Exception: %{WORD:exception} in %{WORD:class}: %{GREEDYDATA:details}" 
      }
    }
  }
}
```

## Best Practices

### **1. Начинайте с простых шаблонов**
```ruby
# Вместо сложного шаблона сразу
# Разбейте на несколько этапов:

# Этап 1: Базовый парсинг
grok {
  match => { 
    "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}" 
  }
}

# Этап 2: Детальный парсинг сообщения
grok {
  match => { 
    "[message]" => "User %{NOTSPACE:username} performed %{WORD:action} on %{WORD:resource}" 
  }
}
```

### **2. Используйте meaningful имена полей**
```ruby
# Хорошо
%{IP:client_ip}
%{NUMBER:response_code}

# Плохо
%{IP:ip1}
%{NUMBER:field2}
```

### **3. Тестируйте шаблоны**
```ruby
filter {
  grok {
    match => { 
      "message" => "%{COMBINEDAPACHELOG}" 
    }
    # Временное поле для отладки
    add_field => { 
      "grok_pattern_used" => "COMBINEDAPACHELOG" 
    }
  }
}
```

### **4. Обрабатывайте ошибки**
```ruby
filter {
  grok {
    match => { 
      "message" => [
        "%{COMBINEDAPACHELOG}",
        "%{COMMONAPACHELOG}",
        "^%{GREEDYDATA:raw_message}"  # Fallback pattern
      ]
    }
    tag_on_failure => []  # Не добавлять тег при ошибке для fallback
  }
}
```

## Производительность Grok

### **Оптимизация шаблонов**
```ruby
filter {
  grok {
    # Используйте более специфичные шаблоны первыми
    match => { 
      "message" => [
        "%{IP:ip} %{WORD:method} %{URIPATHPARAM:request}",  # Самый частый случай
        "%{WORD:service} %{NUMBER:code} %{GREEDYDATA:msg}"  # Менее частый
      ]
    }
    break_on_match => true  # Остановиться при первом успешном совпадении
  }
}
```

Grok - это мощный инструмент для преобразования неструктурированных логов в структурированные данные, что делает их пригодными для анализа, поиска и визуализации в Elasticsearch и Kibana.