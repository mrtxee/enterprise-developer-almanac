---
aliases:
  - kubernetes
  - Liveness Probe
  - Probe
  - Probes
  - Readiness Probe
  - Startup Probe
  - Зондирование
  - Проба
---

## Kubernetes Probes

В Kubernetes существуют три основных вида проб (probe — англ. **зонд**), каждый из которых решает свою задачу:

| Проба | Задача | Что произойдёт при провале |
|-------|--------|----------------------------|
| **Liveness Probe** | Проверяет, «жив» ли контейнер, работает ли корректно | Kubernetes автоматически перезапустит контейнер |
| **Readiness Probe** | Определяет, готов ли контейнер принимать входящий сетевой трафик | Контейнер исключается из балансировки нагрузки (не получает трафик) |
| **Startup Probe** | Используется для приложений, которым требуется длительное время на запуск | Позволяет избежать преждевременного перезапуска или исключения из маршрутизации |

> Логика взаимодействия: Startup Probe выполняется при старте, и только после её успеха включаются Liveness и Readiness пробы. Приложения, которым требуется время на прогрев (например, Java/Spring Boot), обычно задают Startup Probe с большим периодом, чтобы Liveness не перезапускала контейнер во время загрузки.

### Виды проверок

Каждая проба может проверять состояние тремя способами:

| Способ | Описание | Пример |
|--------|----------|--------|
| `exec` | Выполнение команды внутри контейнера; успех — код выхода 0 | `cat /tmp/healthy` |
| `httpGet` | HTTP GET-запрос к контейнеру; успех — код 2xx/3xx | `GET /healthz:8080` |
| `tcpSocket` | TCP-подключение к порту; успех — соединение установлено | `connect 3306` |

### Параметры проб

```yaml
livenessProbe:
  httpGet:               # Проверка HTTP-запросом
    path: /healthz
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  initialDelaySeconds: 5     # Задержка перед первой проверкой
  periodSeconds: 10          # Периодичность проверок
  timeoutSeconds: 1          # Таймаут проверки
  successThreshold: 1        # Успешные проверки, чтобы контейнер стал Ready
  failureThreshold: 3        # Неудачи, чтобы контейнер стал Not Ready
```

```yaml
readinessProbe:
  exec:                  # Проверка командой
    command:
    - cat
    - /tmp/healthy
```

### Пример пробы для Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: my-app:v1
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 5
```

### Какую пробу выбрать

| Сценарий | Используйте |
|----------|-------------|
| Контейнер завис или упал | Liveness Probe |
| Проверка готовности приложения принимать трафик | Readiness Probe |
| Приложение медленно стартует (Java/Spring Boot грузится 60 сек) | Startup Probe |
| Нужна одноразовая подготовка (миграции, генерация конфигов) | Init-контейнер |

**Связь с пружинными Actuator:** смотрите [[spring-actuator-probes|spring-actuator-probes]].

**См. также:** при использовании Startup Probe вместе с Init-контейнерами Pod ждёт сначала завершения init-контейнеров, а затем — успешного старта приложения.
