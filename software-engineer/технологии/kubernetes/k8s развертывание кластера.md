# API-сервер Kubernetes
Для работы с системой в ОС должен быть **API-сервер Kubernetes**

Типовые решения:
* minikube 
* kind 
* k3d 
* Docker Desktop

# структура k8s проекта


Вы хотите **создать Deployment и Service** — но в списке файлов уже есть `events-service.yaml`, `movies-service.yaml`, `proxy-service.yaml`, `monolith.yaml` — вероятно, **это уже Deployment/Service**.

Но если вы хотите **понять, как именно создавать Deployment и Service** на основе этой структуры — давайте разберёмся **пошагово**, **как правильно применять** все файлы, и **как создавать новые** (если нужно).

---

## ✅ Шаг 1: Понимание структуры файлов

Ваша папка `kubernetes/` содержит:

| Файл | Назначение |
|------|------------|
| `namespace.yaml` | Создаёт пространство имён `cinemaabyss` (скорее всего) |
| `configmap.yaml` | Общие ConfigMap (например, для настроек приложений) |
| `postgres-init-configmap.yaml` | Инициализация PostgreSQL (скрипты, миграции) |
| `postgres.yaml` | Deployment + Service для PostgreSQL |
| `kafka/kafka.yaml` | Kafka (возможно, StatefulSet + Service) |
| `dockerconfigsecret.yaml` | Секрет для pull-образов из приватного registry |
| `secret.yaml` | Другие секреты (например, DB_PASSWORD, JWT_SECRET) |
| `events-service.yaml` | **Скорее всего — Service + Deployment для events-сервиса** |
| `movies-service.yaml` | **Скорее всего — Service + Deployment для movies-сервиса** |
| `proxy-service.yaml` | **Скорее всего — Service + Deployment для proxy-сервиса** |
| `monolith.yaml` | Возможно, монолитное приложение (все в одном) |
| `ingress.yaml` | Ingress-ресурс для маршрутизации HTTP-запросов |

> 💡 **Вывод**: У вас уже есть **Deployment и Service** для всех сервисов!  
> Файлы `*-service.yaml` — это, скорее всего, **Service + Deployment** вместе.

---

## ✅ Шаг 2: Как проверить, что уже есть Deployment и Service?

Выполните в терминале (предварительно настроив `kubectl` на кластер):

```bash
kubectl get all -n cinemaabyss
```

Это покажет:

- `deployments.apps` — ваши Deployment
- `services` — ваши Service
- `pods` — запущенные поды
- `ingresses` — ваш ingress

Пример вывода:

```
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/events-service 1/1     1            1           5d

NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/events-service    ClusterIP   10.96.123.45   <none>        8082/TCP   5d
```

Если вы видите такие записи — значит, **Deployment и Service уже созданы**!

---

## ✅ Шаг 3: Как создать Deployment и Service — если их ещё нет?

Допустим, вы хотите **создать новый сервис** — например, `auth-service`.  
Как это сделать правильно?

### 📁 1. Создайте файл `auth-service.yaml`

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
        - name: dockerconfigjson  # ← используем ваш секрет для pull
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

> ✅ Обратите внимание:
> - `Service` и `Deployment` разделены `---`
> - `selector` в Service **должен совпадать** с `labels` в Pod-шаблоне Deployment
> - `imagePullSecrets` — ссылается на ваш `dockerconfigsecret.yaml`
> - `env` использует `secret.yaml` и `configmap.yaml` (если они есть)

### 🚀 2. Примените файл

```bash
kubectl apply -f kubernetes/auth-service.yaml
```

### 🔍 3. Проверьте

```bash
kubectl get pods -n cinemaabyss
kubectl get svc -n cinemaabyss
kubectl get deploy -n cinemaabyss
```

---

## ✅ Шаг 4: Как создать Deployment и Service для уже существующих сервисов?

Если вы **не уверены**, что `events-service.yaml` — это Deployment и Service, откройте его.

### Пример правильного `events-service.yaml`:

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

> ✅ **Важно**:  
> - Если в файле **только Service** — значит, Deployment где-то в другом файле.  
> - Если в файле **два ресурса через `---`** — значит, всё в одном.  
> - Всё это **уже создано** в вашей папке, просто примените их.

---

## ✅ Шаг 5: Как правильно применить ВСЕ файлы?

### 🔁 Рекомендуемый порядок применения (важно!)

Некоторые ресурсы зависят от других. Порядок:

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

> 💡 **Почему так?**  
> - Приложения (`events-service`) используют `postgres`, `kafka`, `secret`, `configmap` — они должны быть созданы **до** приложений.  
> - Ingress использует `Service` — они должны быть созданы **до** Ingress.

---

## ✅ Шаг 6: Проверка успешного развертывания

После применения всех файлов:

```bash
# Проверить все ресурсы в namespace
kubectl get all -n cinemaabyss

# Проверить логи подов
kubectl logs -n cinemaabyss <pod-name>

# Проверить доступность сервисов внутри кластера
kubectl run -it --rm debug --image=busybox -n cinemaabyss -- sh
# внутри пода:
wget -qO- http://events-service:8082/api/events
```

---

## ✅ Бонус: Как создать Deployment и Service "из головы" — шаблон

Если вам нужно **создать новый сервис** (например, `notifications`), используйте этот шаблон:

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

Примените:

```bash
kubectl apply -f kubernetes/notifications-service.yaml
```

---

## ✅ Резюме: Что делать с вашей папкой `kubernetes/`

| Задача | Действие |
|-------|----------|
| ✅ Уже есть Deployment и Service? | Да — в файлах `*-service.yaml`, `monolith.yaml` |
| 🔍 Как проверить? | `kubectl get deploy,svc,pods -n cinemaabyss` |
| 🚀 Как создать? | `kubectl apply -f kubernetes/имя-файла.yaml` |
| ⚠️ Порядок важен? | Да — сначала namespace → secrets → DB → apps → ingress |
| 💡 Как создать новый сервис? | Сделайте YAML с `Service` + `Deployment`, как в примере выше |
| 📦 Где взять `image`? | Из вашего registry (Docker Hub, GitHub CR, ECR и т.д.) |

---

## ✅ Финальный совет

Если вы хотите **управлять всеми ресурсами как единым приложением**, используйте **Helm** или **Kustomize**.  
Но пока вы в стадии обучения — **просто примените все YAML-файлы по порядку**, и всё заработает.

👉 **Сделайте так:**

```bash
cd kubernetes
kubectl apply -f namespace.yaml
kubectl apply -f dockerconfigsecret.yaml
kubectl apply -f secret.yaml
kubectl apply -f configmap.yaml
kubectl apply -f postgres-init-configmap.yaml
kubectl apply -f postgres.yaml
kubectl apply -f kafka/kafka.yaml
kubectl apply -f events-service.yaml
kubectl apply -f movies-service.yaml
kubectl apply -f proxy-service.yaml
kubectl apply -f monolith.yaml
kubectl apply -f ingress.yaml
```

✅ И ваша система `cinemaabyss` будет полностью развёрнута!

Если нужна помощь с конкретным файлом — пришлите его содержимое, и я скажу, что в нём делает Deployment и Service.
# Kubectl
**[[Kubectl]]** — **утилита командной строки (CLI-инструмент)**

# Pod vs Deployment vs Service
Отличный вопрос! Это **три фундаментальных понятия** в Kubernetes, которые часто путают. Давайте разберём их **просто, понятно и с аналогиями**, чтобы вы всегда знали, **когда что использовать**.

---

## 🧩 1. **Pod** — *самая маленькая единица развертывания*

### 💡 Что это?
**Pod — это наименьший объект в Kubernetes**.  
Это **логическая группа из одного или нескольких контейнеров**, которые **разделяют ресурсы**: сеть, хранилище, IPC и т.д.

> ✅ **Один Pod = один IP-адрес в кластере**  
> ✅ **Контейнеры внутри Pod работают вместе** — как "одна машина"

### 📌 Пример:
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
- Они работают **в одном Pod**, **на одном IP**, **делят volume**, могут общаться через `localhost`.

### 🔍 Почему не просто контейнер?
Потому что в реальности **контейнеры часто идут парами**:
- Основной сервис + sidecar (логи, прокси, мониторинг)
- Например: `nginx` + `cert-manager` для обновления SSL-сертификатов

### ⚠️ Важно!
- **Pod — это временный**. Он может умереть, пересоздаться, переместиться.
- **Вы не создаете Pod вручную** — вы используете **Deployment**, чтобы управлять ими.

> ✅ **Pod — это "лампочка". Вы не управляете лампочками напрямую — вы управляете выключателем (Deployment).**

---

## 🏗️ 2. **Deployment** — *управление Pod'ами*

### 💡 Что это?
**Deployment — это контроллер**, который **управляет набором идентичных Pod’ов**.  
Он обеспечивает:
- Нужное количество реплик (например, 3)
- Обновление приложений без простоя (rolling update)
- Откат на предыдущую версию
- Автоматический перезапуск при сбое

### 📌 Пример:
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
- Deployment создаст **3 одинаковых Pod’а** с меткой `app: my-app`
- Если один Pod упадёт — Deployment **создаст новый автоматически**
- Если вы измените `image: nginx:1.25` — Deployment **плавно обновит все Pod’ы**, один за другим

### 🔍 Когда использовать?
- Когда вам нужно **запустить 1+ копий приложения**
- Когда нужно **обновлять/откатывать** приложение
- Когда нужно **автоматическое восстановление**

> ✅ **Deployment — это "выключатель с кнопкой «включить 3 лампочки» и «обновить их без отключения света»"**

### ❌ Нельзя использовать Deployment, если:
- Вам нужен **один Pod** (например, база данных) → используйте **StatefulSet**
- Вам нужен **Pod, который работает один раз** → используйте **Job**
- Вам нужен **Pod, который работает на каждом узле** → используйте **DaemonSet**

---

## 🌐 3. **Service** — *сетевой доступ к Pod’ам*

### 💡 Что это?
**Service — это абстракция, которая предоставляет стабильный IP-адрес и DNS-имя для доступа к группе Pod’ов.**

> ❗ Pod’ы **меняют IP** при пересоздании.  
> ❗ Service **не меняет IP**, даже если Pod’ы умирают и пересоздаются.

### 📌 Пример:
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
      targetPort: 8080  # порт, на котором слушает контейнер в Pod'е
  type: ClusterIP       # доступен только внутри кластера
```

Здесь:
- Service "смотрит" на Pod’ы с меткой `app: my-app`
- Любой запрос на `my-app-service:80` → автоматически направляется на любой из Pod’ов (балансировка)
- Внутри кластера вы обращаетесь к нему как к `http://my-app-service:80`

### 🔍 Типы Service (кратко)
| Тип | Где доступен | Когда использовать |
|-----|-------------|-------------------|
| `ClusterIP` | Только внутри кластера | По умолчанию, для внутренних сервисов |
| `NodePort` | На каждом узле кластера (на порту 30000–32767) | Для тестирования |
| `LoadBalancer` | Внешний балансировщик (в облаке) | Для публичного доступа (AWS, GCP) |
| `Ingress` | Через HTTP-прокси (NGINX, Traefik) | Для HTTP/HTTPS-запросов |

> ✅ **Service — это "дверь с табличкой «my-app-service»", которая ведёт к группе Pod’ов.**  
> Даже если Pod’ы меняются — дверь остаётся на том же месте.

---

## 🧠 Сравнение в одной таблице

| Характеристика | **Pod** | **Deployment** | **Service** |
|----------------|---------|----------------|-------------|
| **Что это?** | Самая маленькая единица (контейнеры + ресурсы) | Управление группой Pod’ов | Сетевой доступ к группе Pod’ов |
| **Создаётся вручную?** | Да, но **не рекомендуется** | Да — **основной способ** | Да — **обязательно** |
| **Отвечает за:** | Запуск контейнеров | Масштабирование, обновление, восстановление | Балансировка трафика, стабильный доступ |
| **IP-адрес меняется?** | ✅ Да (при пересоздании) | ❌ Нет (управляет Pod’ами) | ❌ Нет (постоянный) |
| **Может ли быть один?** | ✅ Да | ✅ Да (реплика=1) | ✅ Да |
| **Для чего нужен?** | Базовая единица выполнения | Управление жизненным циклом приложения | Доступ к приложению изнутри/вне кластера |
| **Аналогия** | Лампочка | Выключатель с автоподдержкой | Табличка «Свет» на двери |

---

## 🔄 Как они работают вместе? (Пример)

Представьте, что вы запускаете веб-приложение:

1. **Deployment** создаёт **3 Pod’а** с вашим приложением (например, `nginx`)
2. Каждый Pod имеет свой **уникальный IP** (например, `10.244.1.5`, `10.244.1.6`, `10.244.1.7`)
3. **Service** ("дверь") получает стабильный IP (`10.96.123.45`) и DNS-имя (`my-app-service`)
4. Когда вы обращаетесь к `my-app-service:80` — Kubernetes **балансирует запрос** между 3 Pod’ами
5. Если один Pod упал — Deployment **создаст новый**, и Service **автоматически его подключит**

> 🔗 **Связь**:  
> `Deployment` → управляет → `Pod’ами` → которые доступны через → `Service`

---

## 🚫 Частые ошибки

| Ошибка | Правильно |
|-------|----------|
| ✅ Создаю Pod вручную | ❌ Используйте Deployment — он автоматически восстановит Pod |
| ✅ Обращаюсь к Pod по IP | ❌ Обращайтесь к Service — IP Pod’ов меняется |
| ✅ Не создаю Service | ❌ Без Service вы не сможете обращаться к Pod’ам из других приложений |
| ✅ Пишу `selector` в Service, но не совпадает с метками в Deployment | ❌ Service не найдёт Pod’ы — трафик не пойдёт |

---

## ✅ Практический совет: Как создать приложение правильно?

```yaml
# 1. Deployment — управляемые Pod’ы
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

# 2. Service — доступ к этим Pod’ам
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

👉 Примените:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

👉 Проверьте:
```bash
kubectl get pods,deploy,svc
```

👉 Попробуйте зайти из другого Pod:
```bash
kubectl run -it --rm debug --image=busybox -- sh
# внутри:
wget -qO- http://my-web-service
```

---

## 💡 Запомните как фразу:

> **"Deployment запускает Pod’ы, Service их открывает"**

- **Pod** — это ваше приложение (контейнеры).
- **Deployment** — это "менеджер", который следит, чтобы их было ровно столько, сколько нужно.
- **Service** — это "портал", через который к ним можно обратиться.

---

✅ Теперь вы **понимаете разницу** и сможете **правильно проектировать** архитектуру в Kubernetes!  
Если нужно — могу показать, как добавить **Ingress** и **ConfigMap** в этот пример.