---
aliases:
  - Cluster
  - Deployment
  - k8s
  - kubernetes
  - Namespace
---

Отличный вопрос!

> Вы спрашиваете — **как устроена иерархия Kubernetes** и **как сущности связаны между собой** — это **фундамент** для понимания архитектуры.
> Давайте разберём **иерархию сущностей** на примерах из вашей переписки, а затем — **все ключевые понятия** с их ролями.

|                        |                                                                      |
| ---------------------- | -------------------------------------------------------------------- |
| `Cluster → Node`       | Кластер состоит из узлов                                             |
| `Node → Pod`           | На каждом узле запускаются поды                                      |
| `Pod → Deployment`     | Deployment управляет подами                                          |
| `Deployment → Service` | Service предоставляет доступ к подам, управляемым Deployment         |
| `Deployment → HPA`     | HPA масштабирует Deployment по метрикам                              |
| `Namespace → все`      | Namespace изолирует ресурсы: Nodes, Pods, Deployments, Services, HPA |

---

## 🌲 **Иерархия Kubernetes: От высшего к низшему (с Container)**

```
Namespace
│
├── Deployment
│   │
│   └── ReplicaSet (RS)
│       │
│       └── Pod (x3)
│           │
│           └── Container (Docker image >= 1)  ← Ваше приложение здесь!
│
└── Service
    │
    └── (направляет трафик → к Pod’ам через Selector)
```

> 💡 **Ключевое правило**:
> **Вы не управляете Pod’ами напрямую — вы управляете Deployment’ом.**
> **Вы не обращаетесь к Pod’ам — вы обращаетесь к Service.**
> **Вы не запускаете контейнеры вручную в K8s — они живут внутри Pod’ов.**

---

**🔍 Где именно находится Container?**

**Container** — это самая маленькая исполняемая единица. Он находится **внутри спецификации Pod’а**.

В YAML-файле это выглядит так:

```yaml
apiVersion: v1
kind: Pod          # Уровень 4: Pod
meta
  name: my-pod
spec:
  containers:      # ← Здесь начинается уровень Container
  - name: my-app   # Имя контейнера
    image: nginx   # Docker-образ
    ports:
    - containerPort: 80
```

**🧩 Как связаны pod и container?**

1. **Pod** предоставляет среду: IP-адрес, общие тома (Volumes), переменные окружения.
2. **Container** выполняет код: запускает Java/Node.js/Python приложение.
3. В одном Poде может быть **несколько контейнеров** (например, основное приложение + sidecar для логирования), и они будут видеть друг друга как `localhost`.

> ✅ **Аналогия**:
> **Pod** — это **квартира** (имеет адрес, электричество, воду).
> **Container** — это **жилец** в этой квартире, который делает полезную работу.

---

### 🔹 1. **Namespace** — *Логический изолированный контейнер для ресурсов*

> 📌 **Что это?**
> Виртуальный кластер внутри реального — чтобы изолировать проекты, окружения (dev/stage/prod), команды.

> ✅ **Пример из вашей переписки**:
> ```yaml
> apiVersion: v1
> kind: Namespace
> metadata:
>   name: cinemaabyss
> ```
> → Все ваши ресурсы (`events-service`, `postgres`, `ingress`) размещаются в `cinemaabyss`.

> 🔄 **Как использовать?**
> ```bash
> kubectl -n cinemaabyss get pods
> ```

> 📌 **Важно**:
> - Ресурсы в разных namespace **не видят друг друга** по умолчанию.
> - Это как “разные комнаты в одном доме”.

---

### 🔹 2. **Deployment** — *Управляющий контроллер для Pod’ов*

> 📌 **Что это?**
> Объект, который **гарантирует**, что нужное количество **идентичных Pod’ов** запущено и работает.
> Он **управляет ReplicaSet’ами** и обеспечивает **обновления**, **откаты**, **восстановление при сбое**.

> ✅ **Пример из вашей переписки**:
> В файле `events-service.yaml`:
> ```yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   name: events-service
> spec:
>   replicas: 1
>   selector:
>     matchLabels:
>       app: events-service
>   template:
>     metadata:
>       labels:
>         app: events-service
>     spec:
>       containers:
>       - name: events
>         image: ghcr.io/mrtxee/cinemaabysstest/events-service:v1.0.0
> ```

> 🔗 **Что делает Deployment?**
> - Создаёт **ReplicaSet** с именем `events-service-6cbbdd56fd` (хеш шаблона)
> - Этот ReplicaSet создаёт **Pod** `events-service-6cbbdd56fd-l8s4s`
> - Если Pod упал — Deployment **создаст новый**

> 💡 **Вы не видите ReplicaSet напрямую — но он всегда существует!**

> ✅ Проверить:
> ```bash
> kubectl -n cinemaabyss get deploy,rs,pod -l app=events-service
> ```

> Вывод:
> ```
> NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
> deployment.apps/events-service   0/1     1            0           1h
>
> NAME                                        DESIRED   CURRENT   READY   AGE
> replicaset.apps/events-service-6cbbdd56fd   1         1         0       1h
>
> NAME                                   READY   STATUS             RESTARTS   AGE
> pod/events-service-6cbbdd56fd-l8s4s    0/1     CrashLoopBackOff   7          1h
> ```

> 🎯 **Deployment → ReplicaSet → Pod**

---

### 🔹 3. **ReplicaSet (RS)** — *Физический механизм создания Pod’ов*

> 📌 **Что это?**
> Это **реализация** Deployment’а.
> ReplicaSet — это **группа Pod’ов**, которые должны быть **одинаковыми** (по шаблону).
> Deployment **управляет ReplicaSet’ами**, а ReplicaSet — **создаёт и убивает Pod’ы**.

> ✅ **Почему несколько ReplicaSet’ов?**
> Когда вы обновляете Deployment (например, меняете образ), Kubernetes создаёт **новый ReplicaSet** — и **постепенно** заменяет старые Pod’ы.
> Это называется **rolling update**.

> ✅ **Пример из вашей переписки**:
> Вы видели два ReplicaSet’а:
> - `events-service-6cbbdd56fd` — новый (с ошибкой)
> - `events-service-5fd575b977` — старый (уже неактивный)

> 💡 Это **не ошибка** — это **нормальное поведение** при обновлении.

> 🔍 Проверить:
> ```bash
> kubectl -n cinemaabyss get replicaset
> ```

> ⚠️ **Вы не создаете ReplicaSet вручную** — только Deployment.

---

### 🔹 4. **Pod** — *Самая маленькая единица развертывания*

> 📌 **Что это?**
> **Группа из одного или нескольких контейнеров**, которые:
> - Работают на одном узле
> - Делят сеть, storage, IPC
> - Имеют **один IP-адрес в кластере**

> ✅ **Пример из вашей переписки**:
> `events-service-6cbbdd56fd-l8s4s` — это **один Pod**, в котором запущен контейнер `events-service`.

> 💡 **Pod — это "лампочка"**.
> **Deployment — это "выключатель", который включает 3 лампочки**.
> **Service — это "табличка" на двери, которая ведёт к лампочкам**.

> ❌ **Вы не должны создавать Pod’ы вручную** — только через Deployment/StatefulSet.

> 🔍 Проверить:
> ```bash
> kubectl -n cinemaabyss get pods -o wide
> ```

В Kubernetes **Pod не может существовать без хотя бы одного контейнера**.

🛑 Почему?

1. **Определение Pod'а**: Pod — это логическая оболочка для **одного или нескольких контейнеров**. Если убрать все контейнеры, то "оболочке" нечего будет запускать.
2. **Валидация API**: Если вы попытаетесь создать YAML-файл с пустым списком `containers: []`, API-сервер Kubernetes отвергнет запрос с ошибкой валидации:
    > `spec.containers: Required value`

⚠️ Нюанс: Init Containers

В Pod'е могут быть два типа контейнеров:

1. **Init Containers** (`initContainers`) — запускаются первыми для подготовки.
2. **Main Containers** (`containers`) — основные рабочие контейнеры.

Даже если у вас есть только `initContainers`, но нет основных `containers`, Pod перейдет в статус `Completed` (завершен) сразу после выполнения инициализации, так как ему нечего поддерживать в рабочем состоянии. Но технически он всё равно содержит контейнеры.

**💡 Исключение: Ephemeral Containers (Эфемерные контейнеры)**

Существует концепция **Ephemeral Containers** (добавлена в K8s v1.22+). Они используются **только для отладки** уже запущенных Pod'ов (например, чтобы добавить `curl` или `tcpdump` в образ, где их нет).

- Они добавляются к **уже существующему** Pod'у, в котором уже есть обычные контейнеры.
- Они не могут быть определены при создании Pod'а.
- Они не считаются частью основной спецификации приложения.

✅ Вывод

> **Pod = Контейнер(ы) + Общие ресурсы (сеть, диск).**
> Без контейнера Pod — это просто пустая мета-информация, которая не имеет смысла в архитектуре Kubernetes.

---

### 🔹 5. **Service** — *Сетевой доступ к группе Pod’ов*

> 📌 **Что это?**
> **Стабильный IP и DNS-имя**, которое **всегда ведёт к группе Pod’ов**, даже если они пересоздаются.

> ✅ **Пример из вашей переписки**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: movies-service
spec:
  selector:
    app: movies-service
  ports:
    - port: 8081
      targetPort: 8081
  type: ClusterIP
```

> 🔗 Как работает:
> - Service смотрит на Pod’ы с меткой `app: movies-service`
> - Любой запрос к `movies-service.cinemaabyss.svc.cluster.local:8081` → балансируется между всеми Pod’ами
> - Если Pod упал — Service **автоматически перестаёт его учитывать**

> ✅ **Почему нельзя обращаться к Pod’у напрямую?**
> Pod’ы имеют **динамические IP** — они меняются при перезапуске.
> Service — **имеет постоянный IP и DNS**.

✅ **Как вы обращались к нему?**

```bash
kubectl -n cinemaabyss run debug --rm -i --image=curlimages/curl -- sh -c "curl http://movies-service:8081/api/movies"
```

→ Вы обращались **к Service**, а не к Pod’у — **это правильно!**

🌐 **Типы Service**:

| Тип            | Где доступен                      | Использовали?                      |
| -------------- | --------------------------------- | ---------------------------------- |
| `ClusterIP`    | Только внутри кластера            | ✅ Да — `movies-service`            |
| `NodePort`     | На каждом узле (порт 30000–32767) | ❌ Нет                              |
| `LoadBalancer` | Внешний балансировщик (в облаке)  | ❌ Нет                              |
| `Ingress`      | HTTP/HTTPS через прокси           | ✅ Да — через `cinemaabyss-ingress` |

---

## 🌟 Другие сущности Kubernetes

| Понятие | Что это? | Пример из вашей переписки |
|--------|----------|-----------------------------|
| **Ingress** | HTTP-маршрутизатор — **внешний вход** в кластер | `cinemaabyss-ingress` — направляет `cinemaabyss.example.com/api/events` → `events-service` |
| **Ingress Controller** | Реальный прокси-сервер (NGINX, Traefik), который **реализует** Ingress | Установлен через `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/...` |
| **Secret** | Хранение чувствительных данных (пароли, токены) | `dockerconfigjson` — для доступа к GitHub Container Registry |
| **ConfigMap** | Хранение конфигураций (не секреты) | `postgres-init-configmap.yaml` — SQL-скрипты для инициализации БД |
| **StatefulSet** | Управление **состоянием** (базы данных, Kafka) | `kafka-0`, `zookeeper-0`, `postgres-0` — это **StatefulSet’ы** (не Deployment!) |
| **PersistentVolume (PV) / PersistentVolumeClaim (PVC)** | Постоянное хранилище для данных | PostgreSQL сохраняет данные на PVC — **даже если Pod упал — данные не исчезают** |
| **HorizontalPodAutoscaler (HPA)** | Автоматическое масштабирование по CPU/Memory | Вы не использовали — но это `kubectl autoscale deployment/events-service --cpu-percent=70 --min=2 --max=10` |
| **Job / CronJob** | Запуск задачи один раз или по расписанию | Вы не использовали — но могли бы запустить миграцию БД как Job |
| **Role / RoleBinding / ClusterRole / ClusterRoleBinding** | RBAC — права доступа | Вы не настраивали — но в продакшене **обязательны** |

---

## 🧩 Полная иерархия с примерами из вашей системы

```
NAMESPACE: cinemaabyss
│
├── INGRESS: cinemaabyss-ingress
│   │
│   └── (направляет трафик от cinemaabyss.example.com → к Service)
│
├── SERVICE: events-service → (ClusterIP: 10.96.123.45)
│   │
│   └── (выбирает Pod'ы по метке app=events-service)
│       │
│       └── DEPLOYMENT: events-service
│           │
│           └── REPLICASET: events-service-6cbbdd56fd
│               │
│               └── POD: events-service-6cbbdd56fd-l8s4s (0/1 CrashLoopBackOff)
│
├── SERVICE: movies-service → (ClusterIP: 10.96.123.46)
│   │
│   └── (выбирает Pod'ы по метке app=movies-service)
│       │
│       └── DEPLOYMENT: movies-service
│           │
│           └── REPLICASET: movies-service-586c87cbc7
│               │
│               └── POD: movies-service-586c87cbc7-khrzk (1/1 Running)
│
├── SERVICE: proxy-service → (ClusterIP: 10.96.123.47)
│   │
│   └── DEPLOYMENT: proxy-service → ReplicaSet → Pod (Running)
│
├── SERVICE: monolith → (ClusterIP: 10.96.123.48)
│   │
│   └── DEPLOYMENT: monolith → ReplicaSet → Pod (Running)
│
├── STATEFULSET: postgres-0
│   │
│   ├── POD: postgres-0 (1/1 Running)
│   └── PVC: postgres-data (хранит данные БД)
│
├── STATEFULSET: kafka-0
│   │
│   ├── POD: kafka-0 (1/1 Running)
│   └── PVC: kafka-data
│
├── STATEFULSET: zookeeper-0
│   │
│   ├── POD: zookeeper-0 (1/1 Running)
│   └── PVC: zookeeper-data
│
├── SECRET: dockerconfigjson → (для доступа к ghcr.io)
├── SECRET: secret → (логины, пароли)
├── CONFIGMAP: postgres-init-configmap → (SQL-скрипты)
└── CONFIGMAP: configmap → (общие настройки)
```

> 🔥 **Ключевая мысль**:
> **Вы никогда не работаете напрямую с Pod’ами — только через Deployment/Service/Ingress.**
> **Pod — это "результат", а не "цель".**

---

## ✅ Практический совет: Как быстро понять, что где находится?

| Что вы хотите узнать? | Как узнать? |
|------------------------|-------------|
| Какой Pod запущен? | `kubectl -n cinemaabyss get pods` |
| Какой Deployment управляет этим Pod’ом? | `kubectl -n cinemaabyss get deploy -o wide` → посмотрите имя |
| Какой Service обращается к этому Pod’у? | `kubectl -n cinemaabyss get svc -o wide` → смотрите `SELECTOR` и совпадает ли с `LABELS` Pod’а |
| Какой Secret используется? | `kubectl -n cinemaabyss get deployment <name> -o yaml | grep imagePullSecrets` |
| Какой Ingress направляет трафик? | `kubectl -n cinemaabyss get ingress` → смотрите `HOSTS` и `PATHS` |
| Какой порт слушает Pod? | `kubectl -n cinemaabyss exec <pod> -- netstat -tlnp` |

---

## 🎯 Резюме: Иерархия в 3 пунктах

| Уровень | Роль | Кто управляет кем? |
|--------|------|-------------------|
| **1. Namespace** | Изоляция | Содержит все остальные |
| **2. Deployment / StatefulSet** | Управление жизненным циклом | Создаёт ReplicaSet |
| **3. ReplicaSet** | Физическое создание | Создаёт Pod’ы |
| **4. Service** | Сетевой доступ | Направляет трафик к Pod’ам через метки |
| **5. Ingress** | Внешний вход | Направляет HTTP-запросы к Service |

> ✅ **Вы — администратор** — вы **управляете Deployment’ами и Service’ами**.
> **Kubernetes — ваш помощник** — он **создаёт ReplicaSet’ы и Pod’ы** за вас.

---

## 💡 Бонус: Как запомнить?

> 🔁 **Deployment → ReplicaSet → Pod** — **"Создаём"**
> 🌐 **Service → Pod** — **"Доступаем"**
> 🚪 **Ingress → Service** — **"Внешний вход"**
> 🔐 **Secret / ConfigMap** — **"Настройки и пароли"**
> 🗃️ **StatefulSet + PVC** — **"Базы данных и состояние"**

---

✅ **Теперь вы знаете:**

- Как устроена иерархия Kubernetes
- Почему нельзя обращаться к Pod’ам напрямую
- Как связаны Deployment, ReplicaSet, Service и Ingress
- Какие ещё важные сущности есть в кластере

Это — **фундамент**, на котором строится **любая продакшен-архитектура в Kubernetes**.

Если захотите — я могу сделать **визуальную схему этой иерархии** в виде ASCII-диаграммы или PNG-картинки (описание для генерации).

## Ingress (вход)

`Интернет → Ingress Controller (NGINX/Traefik) → Ingress (правила) → Service → Pod`

## Init Containers

**Init Containers** (инициализационные контейнеры) — это специальные контейнеры, которые запускаются **до** основных (`containers`) и выполняются **строго последовательно**.

Их главная задача — **подготовить среду** для основного приложения. Если любой Init-контейнер падает, Pod не перейдет в статус `Running`, а будет перезапускать этот Init-контейнер до успеха.

---

### 🔑 Ключевые отличия от обычных контейнеров

| Характеристика              | Init Containers                                    | Обычные Containers                 |
| :-------------------------- | :------------------------------------------------- | :--------------------------------- |
| **Запуск**                  | Последовательно (один за другим)                   | Параллельно (все сразу после init) |
| **Жизненный цикл**          | Запускается → Выполняет задачу → **Завершается**   | Запускается → Работает постоянно   |
| **Условие старта основных** | Все Init-контейнеры должны завершиться с кодом `0` | -                                  |
| **Пробы (Probes)**          | ❌ Не поддерживают Liveness/Readiness               | ✅ Поддерживают                     |
| **Ресурсы**                 | Используются только во время инициализации         | Используются всё время жизни Pod'а |

---

### 🎯 Типичные сценарии использования

1. **Ожидание зависимостей** (самый частый кейс):
    Ждать, пока PostgreSQL/Kafka/Redis станут доступными, прежде чем запускать приложение.
2. **Генерация конфигов**:
    Скачать секреты из Vault, сгенерировать `nginx.conf` или сертификаты.
3. **Миграции БД**:
    Выполнить Flyway/Liquibase миграции *до* старта приложения.
4. **Настройка прав доступа**:
    Изменить владельца файлов на Persistent Volume (`chown`).
5. **Регистрация в Service Discovery**:
    Зарегистрировать сервис в Consul/Eureka перед стартом.

---

### 📝 Пример YAML

```yaml
apiVersion: v1
kind: Pod
meta
  name: my-app-pod
spec:
  # 1️⃣ Init-контейнеры выполняются ПОСЛЕДОВАТЕЛЬНО
  initContainers:
    - name: wait-for-db
      image: busybox:1.36
      command: ['sh', '-c', 'until nc -z postgres 5432; do echo waiting for db; sleep 2; done']

    - name: run-migrations
      image: my-app:migrate-v2
      command: ['./migrate', '--path=/migrations', 'up']
      envFrom:
        - secretRef:
            name: db-credentials

  # 2️⃣ Основные контейнеры запускаются ТОЛЬКО после успешного завершения ВСЕХ init-контейнеров
  containers:
    - name: my-app
      image: my-app:v2
      ports:
        - containerPort: 8080
```

### ⏱ Порядок выполнения

```
Pod создаётся
     │
     ▼
initContainer: wait-for-db ──→ Успех (exit 0)
     │
     ▼
initContainer: run-migrations ──→ Успех (exit 0)
     │
     ▼
container: my-app ──→ Running ✅
```

> ⚠️ **Если `run-migrations` упадёт**: Pod останется в статусе `Init:Error` или `Init:CrashLoopBackOff`. Основной контейнер `my-app` **никогда не запустится**, пока миграции не пройдут успешно.

---

### 💡 Best Practices

1. **Используйте минимальные образы**: `busybox`, `alpine`, `curlimages/curl` — не тащите тяжелый образ приложения только ради `nc` или `wget`.
2. **Не делайте бесконечных циклов без таймаута**: Всегда добавляйте максимальное время ожидания, иначе Pod может зависнуть навсегда.
3. **Idempotentность**: Init-контейнер может быть перезапущен при сбое. Убедитесь, что повторный запуск не ломает систему (например, миграции должны быть идемпотентными).
4. **Логируйте прогресс**: Поскольку Init-контейнеры блокируют старт приложения, хорошие логи помогают быстро понять, *что именно* ждёт Pod.
5. **Ресурсы**: Задавайте `resources.limits` даже для Init-контейнеров, чтобы они не съели все ресурсы ноды во время инициализации.

---

### 🆚 Init Container vs Startup Probe

| Сценарий | Используйте |
|----------|-------------|
| Нужно выполнить **одноразовую подготовку** (миграции, генерация конфигов) | ✅ **Init Container** |
| Приложение **медленно стартует** (Java/Spring Boot грузится 60 сек) | ✅ **Startup Probe** |
| Нужно ждать внешнюю зависимость **перед каждым перезапуском** | ✅ **Init Container** |
| Нужно проверять готовность приложения **постоянно** | ✅ **Readiness Probe** |

> 💡 **Комбинация**: Часто используют **оба**. Init Container ждёт базу данных, а Startup Probe даёт приложению время на прогрев кэша после запуска.
