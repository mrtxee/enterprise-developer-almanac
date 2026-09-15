---
aliases:
  - ClusterIP
  - ConfigMap
  - DaemonSet
  - Deployment
  - Docker Desktop
  - Ingress
  - Job
  - k3d
  - k8s
  - kind
  - kubectl
  - Kubernetes
  - LoadBalancer
  - minikube
  - NodePort
  - Pod
  - Secret
  - Service
  - StatefulSet
  - Кластер
---

## API-сервер Kubernetes

Для работы с системой в ОС должен быть API-сервер Kubernetes.

Типовые решения:
- minikube
- kind
- k3d
- Docker Desktop

## Типовая структура k8s-проекта

Создание Deployment и Service на основе существующей структуры: файлы `events-service.yaml`, `movies-service.yaml`, `proxy-service.yaml`, `monolith.yaml` уже являются Deployment/Service.

### Структура файлов

Папка `kubernetes/` содержит:

| Файл | Назначение |
|------|------------|
| `namespace.yaml` | Создаёт пространство имён `cinemaabyss` |
| `configmap.yaml` | Общие ConfigMap (например, для настроек приложений) |
| `postgres-init-configmap.yaml` | Инициализация PostgreSQL (скрипты, миграции) |
| `postgres.yaml` | Deployment + Service для PostgreSQL |
| `kafka/kafka.yaml` | Kafka (возможно, StatefulSet + Service) |
| `dockerconfigsecret.yaml` | Секрет для pull-образов из приватного registry |
| `secret.yaml` | Другие секреты (например, DB_PASSWORD, JWT_SECRET) |
| `events-service.yaml` | Service + Deployment для events-сервиса |
| `movies-service.yaml` | Service + Deployment для movies-сервиса |
| `proxy-service.yaml` | Service + Deployment для proxy-сервиса |
| `monolith.yaml` | Монолитное приложение (все в одном) |
| `ingress.yaml` | Ingress-ресурс для маршрутизации HTTP-запросов |

Файлы `*-service.yaml` — это, скорее всего, Service + Deployment вместе.

### Проверка существующих Deployment и Service

Проверка выполняется в терминале (предварительно настроив `kubectl` на кластер):

```bash
kubectl get all -n cinemaabyss
```

Это покажет:
- `deployments.apps` — Deployment
- `services` — Service
- `pods` — запущенные поды
- `ingresses` — Ingress

Пример вывода:

```text
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/events-service 1/1     1            1           5d

NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/events-service    ClusterIP   10.96.123.45   <none>        8082/TCP   5d
```

Если такие записи видны — Deployment и Service уже созданы.

### Создание Deployment и Service

Пример создания нового сервиса — `auth-service`.

#### Файл `auth-service.yaml`

```yaml
# auth-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: auth-service
  namespace: cinemaabyss
  labels:
    app: auth-service
spec:
  selector:
    app: auth-service
  ports:
    - protocol: TCP
      port: 8080        # порт, который слушает Service
      targetPort: 8080  # порт, на котором слушает контейнер
  type: ClusterIP       # внутри кластера

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
  namespace: cinemaabyss
  labels:
    app: auth-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: auth-service
  template:
    metadata:
      labels:
        app: auth-service
    spec:
      imagePullSecrets:
        - name: dockerconfigjson  # секрет для pull
      containers:
        - name: auth
          image: your-registry.com/cinemaabyss/auth-service:v1.2
          ports:
            - containerPort: 8080
          env:
            - name: DB_HOST
              value: "postgres.cinemaabyss.svc.cluster.local"
            - name: DB_PORT
              value: "5432"
            - name: DB_NAME
              valueFrom:
                secretKeyRef:
                  name: secret
                  key: db-name
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: secret
                  key: db-password
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
```

Обратите внимание:
- Service и Deployment разделены `---`
- `selector` в Service должен совпадать с `labels` в Pod-шаблоне Deployment
- `imagePullSecrets` ссылается на `dockerconfigsecret.yaml`
- `env` использует `secret.yaml` и `configmap.yaml`

#### Применение файла

```bash
kubectl apply -f kubernetes/auth-service.yaml
```

#### Проверка

```bash
kubectl get pods -n cinemaabyss
kubectl get svc -n cinemaabyss
kubectl get deploy -n cinemaabyss
```

### Изменение существующих сервисов

Проверка формата `events-service.yaml` — чтобы понять, что это Deployment и Service.

Пример правильного `events-service.yaml`:

```yaml
# events-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: events-service
  namespace: cinemaabyss
  labels:
    app: events-service
spec:
  selector:
    app: events-service
  ports:
    - protocol: TCP
      port: 8082
      targetPort: 8082
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: events-service
  namespace: cinemaabyss
  labels:
    app: events-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: events-service
  template:
    metadata:
      labels:
        app: events-service
    spec:
      imagePullSecrets:
        - name: dockerconfigjson
      containers:
        - name: events
          image: your-registry.com/cinemaabyss/events-service:latest
          ports:
            - containerPort: 8082
          env:
            - name: KAFKA_BOOTSTRAP_SERVERS
              value: "kafka.cinemaabyss.svc.cluster.local:9092"
            - name: DB_HOST
              value: "postgres.cinemaabyss.svc.cluster.local"
          resources:
            requests:
              memory: "256Mi"
              cpu: "500m"
```

Важно:
- Если в файле только Service — Deployment находится в другом файле.
- Если в файле два ресурса через `---` — всё в одном файле.

### Порядок применения файлов

Некоторые ресурсы зависят от других. Рекомендуемый порядок:

```bash
# 1. Создать namespace
kubectl apply -f kubernetes/namespace.yaml

# 2. Создать секреты и configmaps (они нужны для Deployment)
kubectl apply -f kubernetes/dockerconfigsecret.yaml
kubectl apply -f kubernetes/secret.yaml
kubectl apply -f kubernetes/configmap.yaml
kubectl apply -f kubernetes/postgres-init-configmap.yaml

# 3. Создать базы данных и Kafka (они должны быть доступны до приложений)
kubectl apply -f kubernetes/postgres.yaml
kubectl apply -f kubernetes/kafka/kafka.yaml

# 4. Создать сервисы (Deployment + Service)
kubectl apply -f kubernetes/events-service.yaml
kubectl apply -f kubernetes/movies-service.yaml
kubectl apply -f kubernetes/proxy-service.yaml
kubectl apply -f kubernetes/monolith.yaml

# 5. Создать Ingress (внешний доступ)
kubectl apply -f kubernetes/ingress.yaml
```

Почему так:
- Приложения (`events-service`) используют `postgres`, `kafka`, `secret`, `configmap` — они должны быть созданы до приложений.
- Ingress использует `Service` — они должны быть созданы до Ingress.

### Проверка успешного развертывания

После применения всех файлов:

```bash
# Проверить все ресурсы в namespace
kubectl get all -n cinemaabyss

# Проверить логи подов
kubectl logs -n cinemaabyss <pod-name>

# Проверить доступность сервисов внутри кластера
kubectl run -it --rm debug --image=busybox -n cinemaabyss -- sh
```

```text
# внутри пода:
wget -qO- http://events-service:8082/api/events
```

### Шаблон для нового сервиса

Шаблон для создания нового сервиса (например, `notifications`):

```yaml
# notifications-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: notifications-service
  namespace: cinemaabyss
  labels:
    app: notifications-service
spec:
  selector:
    app: notifications-service
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notifications-service
  namespace: cinemaabyss
  labels:
    app: notifications-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notifications-service
  template:
    metadata:
      labels:
        app: notifications-service
    spec:
      imagePullSecrets:
        - name: dockerconfigjson
      containers:
        - name: notifications
          image: your-registry.com/cinemaabyss/notifications:latest
          ports:
            - containerPort: 8081
          env:
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: secret
                  key: db-host
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
```

Применение:

```bash
kubectl apply -f kubernetes/notifications-service.yaml
```

### Резюме

| Задача | Действие |
|-------|----------|
| Уже есть Deployment и Service? | Да — в файлах `*-service.yaml`, `monolith.yaml` |
| Как проверить? | `kubectl get deploy,svc,pods -n cinemaabyss` |
| Как создать? | `kubectl apply -f kubernetes/имя-файла.yaml` |
| Порядок важен? | Да — сначала namespace → secrets → DB → apps → ingress |
| Как создать новый сервис? | YAML с `Service` + `Deployment`, как в примере выше |
| Где взять `image`? | Из registry (Docker Hub, GitHub CR, ECR и т. д.) |

**Финальный совет**

Для управления всеми ресурсами как единым приложением используется **Helm** или **Kustomize**. При обучении достаточно применить все YAML-файлы по порядку, как в разделе «Порядок применения файлов».

## Kubectl

**[[kubectl]]** — утилита командной строки (CLI-инструмент).

## Pod vs Deployment vs Service

**Суть**
Pod, Deployment и Service — три фундаментальных понятия в Kubernetes: Pod — наименьшая единица развертывания (контейнеры), Deployment — контроллер, управляющий набором Pod, Service — абстракция стабильного сетевого доступа к группе Pod.

### Pod — самая маленькая единица развертывания

**Что это**

Pod — это наименьший объект в Kubernetes. Это логическая группа из одного или нескольких контейнеров, которые разделяют ресурсы: сеть, хранилище, IPC и т. д.

- Один Pod = один IP-адрес в кластере.
- Контейнеры внутри Pod работают вместе — как «одна машина».

**Пример**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  containers:
  - name: web
    image: nginx:latest
  - name: log-agent
    image: fluentd
```

Здесь:
- `nginx` — веб-сервер
- `fluentd` — агент логирования
- Они работают в одном Pod, на одном IP, делят volume и могут общаться через `localhost`

**Почему не просто контейнер**

В реальности контейнеры часто идут парами:
- основной сервис + sidecar (логи, прокси, мониторинг)
- например: `nginx` + `cert-manager` для обновления SSL-сертификатов

**Важно**
- Pod — это временный объект. Он может умереть, пересоздаться, переместиться.
- Pod вручную обычно не создаётся — используется Deployment для управления ими.

Pod — это «лампочка». Лампочками управляют не напрямую, а через выключатель (Deployment).

### Deployment — управление Pod

**Что это**

Deployment — это контроллер, который управляет набором идентичных Pod. Он обеспечивает:
- нужное количество реплик (например, 3)
- обновление приложений без простоя (rolling update)
- откат на предыдущую версию
- автоматический перезапуск при сбое

**Пример**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: web
        image: nginx:latest
```

Здесь:
- Deployment создаст 3 одинаковых Pod с меткой `app: my-app`
- если один Pod упадёт — Deployment создаст новый автоматически
- если изменить `image: nginx:1.25` — Deployment плавно обновит все Pod, один за другим

**Когда использовать**
- когда нужно запустить 1+ копий приложения
- когда нужно обновлять/откатывать приложение
- когда нужно автоматическое восстановление

Deployment — это «выключатель с кнопкой «включить 3 лампочки» и «обновить их без отключения света»».

**Нельзя использовать Deployment, если:**
- нужен один Pod (например, база данных) → используется **StatefulSet**
- нужен Pod, который работает один раз → используется **Job**
- нужен Pod, который работает на каждом узле → используется **DaemonSet**

### Service — сетевой доступ к Pod

**Что это**

Service — это абстракция, которая предоставляет стабильный IP-адрес и DNS-имя для доступа к группе Pod.

- Pod меняют IP при пересоздании.
- Service не меняет IP, даже если Pod умирают и пересоздаются.

**Пример**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80          # порт, по которому Service доступен внутри кластера
      targetPort: 8080  # порт, на котором слушает контейнер в Pod
  type: ClusterIP       # доступен только внутри кластера
```

Здесь:
- Service «смотрит» на Pod с меткой `app: my-app`
- любой запрос на `my-app-service:80` автоматически направляется на любой из Pod (балансировка)
- внутри кластера доступ идёт как `http://my-app-service:80`

**Типы Service**

| Тип | Где доступен | Когда использовать |
|-----|-------------|-------------------|
| `ClusterIP` | Только внутри кластера | По умолчанию, для внутренних сервисов |
| `NodePort` | На каждом узле кластера (на порту 30000–32767) | Для тестирования |
| `LoadBalancer` | Внешний балансировщик (в облаке) | Для публичного доступа (AWS, GCP) |
| `Ingress` | Через HTTP-прокси (NGINX, Traefik) | Для HTTP/HTTPS-запросов |

Service — это «дверь с табличкой «my-app-service»», которая ведёт к группе Pod. Даже если Pod меняются — дверь остаётся на том же месте.

### Сравнение

| Характеристика | Pod | Deployment | Service |
|----------------|-----|------------|---------|
| **Что это?** | Самая маленькая единица (контейнеры + ресурсы) | Управление группой Pod | Сетевой доступ к группе Pod |
| **Создаётся вручную?** | Да, но не рекомендуется | Да — основной способ | Да — обязательно |
| **Отвечает за:** | Запуск контейнеров | Масштабирование, обновление, восстановление | Балансировка трафика, стабильный доступ |
| **IP-адрес меняется?** | Да (при пересоздании) | Нет (управляет Pod) | Нет (постоянный) |
| **Может ли быть один?** | Да | Да (реплика=1) | Да |
| **Для чего нужен?** | Базовая единица выполнения | Управление жизненным циклом приложения | Доступ к приложению изнутри/вне кластера |
| **Аналогия** | Лампочка | Выключатель с автоподдержкой | Табличка «Свет» на двери |

### Как они работают вместе

Веб-приложение:
1. Deployment создаёт 3 Pod с приложением (например, `nginx`).
2. Каждый Pod имеет свой уникальный IP (например, `10.244.1.5`, `10.244.1.6`, `10.244.1.7`).
3. Service («дверь») получает стабильный IP (`10.96.123.45`) и DNS-имя (`my-app-service`).
4. При обращении к `my-app-service:80` Kubernetes балансирует запрос между 3 Pod.
5. Если один Pod упал, Deployment создаст новый, и Service автоматически его подключит.

Связь: `Deployment` → управляет → Pod → которые доступны через → `Service`.

### Частые ошибки

| Ошибка | Правильно |
|-------|----------|
| Создание Pod вручную | Используйте Deployment — он автоматически восстановит Pod |
| Обращение к Pod по IP | Используйте Service — IP Pod меняется |
| Пропуск Service | Без Service нельзя обращаться к Pod из других приложений |
| `selector` в Service не совпадает с метками в Deployment | Service не найдёт Pod — трафик не пойдёт |

### Практический пример

```yaml
# 1. Deployment — управляемые Pod
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-web-app
  template:
    metadata:
      labels:
        app: my-web-app
    spec:
      containers:
      - name: web
        image: nginx:latest
        ports:
        - containerPort: 80

# 2. Service — доступ к этим Pod
apiVersion: v1
kind: Service
metadata:
  name: my-web-service
spec:
  selector:
    app: my-web-app
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

Применение:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Проверка:

```bash
kubectl get pods,deploy,svc
```

Проверка доступа из другого Pod:

```bash
kubectl run -it --rm debug --image=busybox -- sh
```

```text
# внутри:
wget -qO- http://my-web-service
```

**Запомните как фразу**

> «Deployment запускает Pod, Service их открывает»

- Pod — это приложение (контейнеры).
- Deployment — «менеджер», который следит, чтобы их было ровно столько, сколько нужно.
- Service — «портал», через который к ним можно обратиться.
