---
aliases:
  - Affinity
  - Anti-Affinity
  - k8s
  - kubernetes
  - Node
  - Node Selector
  - Pod
  - Scheduler
  - Taint
  - Toleration
  - Нода
  - Под
  - Узел
---

# Pod и Node в Kubernetes

## Определение Pod и Node

| Понятие | Описание |
|---------|----------|
| **Pod** | Самая маленькая единица развёртывания. Это **группа из одного или нескольких контейнеров**, которые работают вместе на одном узле. |
| **Node (нода)** | Физическая или виртуальная машина (сервер), на которой **запускаются Pod'ы**. Это **рабочая машина** в кластере. |

**Простая аналогия:**

- **Node** — это **компьютер** (например, сервер в облаке).
- **Pod** — это **контейнер с приложением**, запущенный на этом компьютере.
- **Node = сервер в дата-центре**; **Pod = Docker-контейнер с приложением**, запущенный на этом сервере.

---

## Как связаны Pod и Node

**Основная связь:**

- Каждый Pod запускается на одной Node.
- Один Node может запускать множество Pod'ов.
- Один Pod не может быть запущен на нескольких Node одновременно.

**Пример**

Кластер с 3 нодами:

| Node | IP | Запущенные Pod'ы |
|------|-----|------------------|
| Node 1 | 192.168.1.10 | events-service-0, proxy-service-0 |
| Node 2 | 192.168.1.11 | movies-service-0, postgres-0 |
| Node 3 | 192.168.1.12 | kafka-0, zookeeper-0 |

- ✅ Каждый Pod **привязан к одной ноде**.
- ✅ Каждая нода может содержать **десятки Pod'ов** (если хватает ресурсов).

---

## Как Kubernetes выбирает ноду для Pod

Kubernetes использует **Scheduler** — специальный компонент, который **автоматически выбирает подходящую ноду** для каждого Pod'а.

**Что учитывает Scheduler**

| Фактор | Объяснение |
|--------|------------|
| **Ресурсы** | Хватает ли на ноде CPU и RAM? Если Pod запрашивает 500m CPU — нода должна иметь хотя бы 500m свободных |
| **Толерансы (Tolerations)** | Pod может быть запущен только на нодах, которые «терпят» его метки (например, `dedicated=database`) |
| **Селекторы узлов (Node Selectors)** | Pod может требовать ноду с определённой меткой (например, `node-type=high-memory`) |
| **Аффинитет / Анти-аффинитет** | Хочется запустить Pod рядом с другим Pod'ом (аффинитет) или подальше от него (анти-аффинитет) — например, чтобы не ставить две базы данных на одну ноду |
| **Приоритеты / Quotas** | Некоторые Pod'ы имеют приоритет выше — их запускают первыми |
| **Запросы и лимиты** | Если Pod запрашивает больше, чем есть, он остаётся в состоянии `Pending` |

**Пример: Pod с запросами ресурсов**

```yaml
spec:
  containers:
  - name: events
    image: events-service
    resources:
      requests:
        cpu: "500m"
        memory: "512Mi"
```

Scheduler ищет ноду, где:

- Свободно ≥ 500m CPU.
- Свободно ≥ 512Mi RAM.
- Нет других ограничений (например, `nodeSelector`).

Если таких нод нет — Pod остаётся в состоянии **`Pending`**.

---

## Поведение при отказе ноды

Pod'ы, запущенные на этой ноде, автоматически удаляются.

**Что делает Kubernetes**

1. **Мастер-контроллер (Control Plane)** замечает, что нода **не отвечает** (например, через `NodeStatus: NotReady`).
2. Через ~5 минут (по умолчанию) нода помечается как **`Unreachable`**.
3. Kubernetes **перезапускает все Pod'ы** с этой ноды на **других доступных нодах**.
4. Это делает **Deployment** (или StatefulSet) — они следят за нужным количеством Pod'ов.

Это называется **«самовосстановление»** — ключевое преимущество Kubernetes.

---

## Как посмотреть, где запущены Pod'ы

### Команда: `kubectl get pods -o wide`

```bash
kubectl -n cinemaabyss get pods -o wide
```

Вывод:

```text
NAME                              READY   STATUS    RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
events-service-6cbbdd56fd-l8s4s   0/1     Running   0          15m   10.244.1.5   docker-desktop  <none>           <none>
kafka-0                           1/1     Running   0          2h    10.244.1.6   docker-desktop  <none>           <none>
postgres-0                        1/1     Running   0          2h    10.244.1.7   docker-desktop  <none>           <none>
```

Столбец **`NODE`** содержит **имя ноды**, на которой запущен Pod. В Docker Desktop это `docker-desktop` (одна нода), в облаке — что-то вроде `gke-cluster-1-default-pool-1234abcd`.

### Команда: `kubectl get nodes`

```bash
kubectl get nodes
```

Вывод:

```text
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   3d    v1.30.0
```

Здесь видны **все ноды в кластере**. В Minikube — одна нода, в EKS/GKE — 3–10+ нод.

---

## Как переместить Pod на другую ноду

Вручную перенести Pod нельзя — он **привязан к ноде**.

**Как «переместить» Pod**

Удалите Pod — Kubernetes создаст его заново на другой ноде:

```bash
kubectl -n cinemaabyss delete pod events-service-6cbbdd56fd-l8s4s
```

Kubernetes **создаст новый Pod**, и Scheduler **выберет другую ноду**, если текущая перегружена или недоступна. Если в кластере **одна нода** (например, Docker Desktop) — Pod будет создан **на той же ноде**.

---

## Почему один Pod не может быть запущен на нескольких нодах

Pod — это логическая единица, привязанная к одному узлу. Он использует:

- Один IP-адрес
- Один сетевой интерфейс
- Один набор дисков (PVC)
- Один процессор и память

Это **не контейнер в Docker**, который можно запустить везде, а **связанная группа контейнеров**, которая работает как **одна машина**.

**Аналогия:** нельзя запустить **один компьютер** на двух столах одновременно — он **физически находится в одном месте**.

---

## Управление размещением Pod'ов на нодах

### Node Selector

Запуск Pod только на нодах с определённой меткой:

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

Pod запустится **только на нодах**, у которых есть метка `disktype=ssd`:

```bash
kubectl label node node-name disktype=ssd
```

### Affinity / Anti-Affinity

**Пример: Anti-affinity** — не запускать два `postgres` на одной ноде:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - postgres
      topologyKey: kubernetes.io/hostname
```

Это гарантирует, что **каждый Pod `postgres` будет на отдельной ноде** — повышается отказоустойчивость.

**Пример: Affinity** — запускать `events-service` рядом с `kafka`:

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - kafka
      topologyKey: kubernetes.io/hostname
```

Это снижает задержку — если приложение и Kafka на одной ноде, коммуникация быстрее.

### Taints and Tolerations

«Запретить» запуск на ноде, если Pod не «терпит» её тайнт. Пример: зарезервировать ноду **только для баз данных**:

```bash
kubectl taint nodes node-name dedicated=database:NoSchedule
```

Теперь **никакой Pod не запустится на этой ноде**, **если только** он не имеет **toleration**:

```yaml
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "database"
  effect: "NoSchedule"
```

Используется для:

- Выделения нод под критичные сервисы.
- Разделения dev/prod.
- Использования специальных нод (GPU, high-memory).

---

## Важные термины

| Термин | Что означает |
|--------|--------------|
| **Node Selector** | Простой способ сказать: «запусти на ноде с этой меткой» |
| **Affinity** | «Постарайся запустить рядом с этим Pod'ом» |
| **Anti-Affinity** | «Не запускай на той же ноде, что и этот Pod'» |
| **Taint** | «Эта нода — не для всех, только для тех, кто её терпит» |
| **Toleration** | «Я — тот, кто терпит этот тайнт» |
| **Topology Key** | `kubernetes.io/hostname` — значит «по ноде» (а не по зоне/региону) |

---

## Резюме: связь Pod и Node

| Вопрос | Ответ |
|--------|-------|
| Что такое Pod? | Группа контейнеров, работающих как одна единица |
| Что такое Node? | Физическая/виртуальная машина, где запускаются Pod'ы |
| Сколько Pod'ов на одной Node? | Много — пока хватает ресурсов (CPU, RAM, IP) |
| Может ли Pod быть на нескольких Node? | ❌ Нет — один Pod = одна нода |
| Может ли Node запускать несколько Pod'ов? | ✅ Да — это нормально и ожидаемо |
| Кто решает, на какую ноду запустить Pod? | **Scheduler** — автоматически, на основе ресурсов, меток, аффинити |
| Что происходит, если нода упала? | Все Pod'ы на ней удаляются — Kubernetes пересоздаёт их на других нодах |
| Как увидеть, где запущен Pod? | `kubectl get pods -o wide` — смотрите столбец `NODE` |
| Как управлять размещением? | `nodeSelector`, `affinity`, `anti-affinity`, `taints/tolerations` |

---

## Диагностика: почему Pod не запускается

```bash
kubectl -n cinemaabyss describe pod events-service-6cbbdd56fd-l8s4s
```

Ищите в выводе:

```text
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  2m    default-scheduler  0/1 nodes are available: 1 Insufficient cpu.
```

Это значит: **на всех нодах не хватает CPU**. Нужно:

- Уменьшить `requests` в Pod'е.
- Увеличить размер нод.
- Добавить новые ноды (Cluster Autoscaler).

---

## Финальный совет

> **Pod — это приложение. Node — это дом, где оно живёт.**
>
> Kubernetes — это **умный управляющий**, который:
> - Знает, **сколько места** есть на каждом доме.
> - Знает, **какие приложения не любят жить вместе**.
> - Переселяет приложения, если дом сгорел.
> - Добавляет новые дома, если все переполнены.

**Понимание связи Pod ↔ Node — ключ к надёжности, производительности и экономии в Kubernetes.**
