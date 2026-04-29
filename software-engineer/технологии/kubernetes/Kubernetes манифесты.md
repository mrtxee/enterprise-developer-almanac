# Синтаксис манифестов Kubernetes - полное руководство

## Базовая структура манифеста

### **Обязательные поля**
```yaml
apiVersion: v1          # Версия API Kubernetes
kind: Pod               # Тип ресурса (Pod, Deployment, Service и т.д.)
metadata:               # Метаданные ресурса
  name: my-pod          # Уникальное имя ресурса
  namespace: default    # Пространство имен (опционально)
spec:                   # Спецификация ресурса - самая важная часть
  # Конфигурация конкретного типа ресурса
```

## Основные виды ресурсов Kubernetes

### **1. Pod (Под)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: production
spec:
  containers:
  - name: nginx-container
    image: nginx:1.21
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
      protocol: TCP
    env:
    - name: ENV_VAR
      value: "production"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
  restartPolicy: Always
  nodeSelector:
    disktype: ssd
```

### **2. Deployment (Развертывание)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3  # Количество реплик Pod'ов
  selector:    # Как Deployment находит Pod'ы для управления
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate  # Стратегия обновления
    rollingUpdate:
      maxSurge: 1        # Максимум Pod'ов сверх replicas при обновлении
      maxUnavailable: 0  # Максимум недоступных Pod'ов при обновлении
  minReadySeconds: 5     # Минимальное время готовности Pod'а
  revisionHistoryLimit: 3 # Сколько старых ReplicaSet'ов хранить
  template:              # Шаблон Pod'а (как в Pod манифесте)
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        livenessProbe:   # Проверка живости контейнера
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:  # Проверка готовности контейнера
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 5
```

### **3. Service (Сервис)**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:          # На какие Pod'ы направить трафик
    app: nginx
  type: ClusterIP   # Тип сервиса
  # Другие типы: NodePort, LoadBalancer, ExternalName
  ports:
  - name: http
    port: 80        # Порт сервиса внутри кластера
    targetPort: 80  # Порт контейнера
    protocol: TCP
  # Для NodePort:
  # - nodePort: 30080  # Порт на ноде (30000-32767)
  # Для LoadBalancer:
  # externalTrafficPolicy: Local
  # loadBalancerIP: "192.168.0.100"
```

### **4. ConfigMap (Конфигурационная карта)**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:  # Данные в формате ключ-значение
  # Простые значения
  database.host: "mysql.default.svc.cluster.local"
  database.port: "3306"
  
  # Файлы конфигурации
  nginx.conf: |
    server {
        listen 80;
        server_name localhost;
        
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
  
  application.properties: |
    spring.datasource.url=jdbc:mysql://${database.host}:${database.port}/mydb
    spring.datasource.username=admin
    spring.datasource.password=secret
    logging.level.root=INFO
```

### **5. Secret (Секрет)**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque  # Тип секрета
data:         # Данные в base64
  # echo -n 'admin' | base64
  username: YWRtaW4=       # admin
  password: cGFzc3dvcmQ=   # password
  
  # Для TLS сертификатов
  # type: kubernetes.io/tls
  # tls.crt: <base64-encoded-cert>
  # tls.key: <base64-encoded-key>
  
stringData:  # Альтернатива data - не требует base64 кодирования
  api-token: "abc123-def456-ghi789"
```

## Продвинутые ресурсы

### **6. StatefulSet (Для stateful приложений)**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: mysql  # Обязательное поле для StatefulSet
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:  # Динамическое создание PVC для каждого Pod'а
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "fast-ssd"
      resources:
        requests:
          storage: 10Gi
```

### **7. PersistentVolumeClaim (PVC)**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce  # Может монтироваться только одной нодой
    # Другие режимы: ReadOnlyMany, ReadWriteMany
  storageClassName: "fast-ssd"  # Использовать StorageClass
  resources:
    requests:
      storage: 10Gi  # Объем хранилища
  selector:  # Опциональные критерии выбора PV
    matchLabels:
      type: ssd
      environment: production
```

### **8. HorizontalPodAutoscaler (HPA)**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:  # На какой ресурс применяется автоподстройка
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2   # Минимальное количество реплик
  maxReplicas: 10  # Максимальное количество реплик
  metrics:         # Метрики для масштабирования
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50  # Масштабировать при 50% использовании CPU
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods  # Кастомные метрики
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: 1000
  behavior:  # Поведение при масштабировании (Kubernetes 1.18+)
    scaleDown:
      stabilizationWindowSeconds: 300  # Окно стабилизации для уменьшения
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```

### **9. Ingress**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx  # Класс Ingress контроллера
  tls:  # Настройки TLS
  - hosts:
    - myapp.example.com
    secretName: myapp-tls-secret
  rules:  # Правила маршрутизации
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
  - host: admin.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
```

### **10. ServiceAccount**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
  namespace: default
automountServiceAccountToken: false  # Не монтировать токен автоматически
secrets:  # Секреты, связанные с ServiceAccount
- name: my-serviceaccount-token-xyz
```

### **11. Role и RoleBinding**
```yaml
# Role - набор разрешений в namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]  # Core API group
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list"]

# RoleBinding - связывает Role с субъектом
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: alice@example.com
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: my-serviceaccount
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Декларативное управление ресурсами

### **Мульти-ресурсные манифесты**
```yaml
# Можно объединять несколько ресурсов в одном файле через ---
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  log_level: "INFO"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:latest
        env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: log_level
---
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

### **Использование переменных и функций**

Kubernetes манифесты не поддерживают переменные напрямую, но есть обходные пути:

```yaml
# 1. Использование Helm Charts (рекомендуется)
# {{ .Values.replicaCount }}
# {{ .Values.image.tag }}

# 2. Kustomize (встроен в kubectl)
# kustomization.yml:
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yml
- service.yml
replicas:
- name: app-deployment
  count: 3
images:
- name: myapp
  newTag: v1.2.3
configMapGenerator:
- name: app-config
  literals:
  - ENVIRONMENT=production
```

## Поля, общие для всех ресурсов

### **metadata**
```yaml
metadata:
  name: my-resource      # Обязательное поле
  namespace: production  # Если не указан - default
  labels:               # Метки для селекции
    app: myapp
    version: "1.0"
    environment: production
  annotations:          # Аннотации (не используются для селекции)
    description: "Main application deployment"
    maintainer: "team@company.com"
    kubernetes.io/change-cause: "Updated to version 1.2.3"
  finalizers:          # Контролируют процесс удаления
  - foregroundDeletion
  ownerReferences:     # Ссылки на владельцев (для сборщика мусора)
  - apiVersion: apps/v1
    kind: Deployment
    name: parent-deployment
    uid: 12345-abcde
  generation: 1        # Увеличивается при каждом изменении spec
```

### **spec и status**
```yaml
# spec - желаемое состояние (declarative configuration)
spec:
  # Конфигурация, которую вы хотите применить
  replicas: 3
  containers: [...]
  
# status - фактическое состояние (read-only, заполняется Kubernetes)
status:
  availableReplicas: 3
  conditions:
  - type: Available
    status: "True"
    lastUpdateTime: "2024-01-15T10:30:00Z"
  observedGeneration: 1
```

## Специальные поля и конструкции

### **Селекторы (Selectors)**
```yaml
selector:
  matchLabels:        # Точное совпадение меток
    app: nginx
    tier: frontend
  
  matchExpressions:   # Более сложные условия
  - key: environment
    operator: In
    values: [production, staging]
  - key: version
    operator: NotIn
    values: [v1.0, v1.1]
  - key: ready
    operator: Exists  # Проверка наличия метки
```

### **Проб (Probes)**
```yaml
livenessProbe:    # Проверка, жив ли контейнер
  exec:           # Выполнение команды
    command:
    - cat
    - /tmp/healthy
  httpGet:        # HTTP GET запрос
    path: /healthz
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  tcpSocket:      # TCP подключение
    port: 3306
  initialDelaySeconds: 5    # Задержка перед первой проверкой
  periodSeconds: 10         # Периодичность проверок
  timeoutSeconds: 1         # Таймаут проверки
  successThreshold: 1       # Успешные проверки для перехода в Ready
  failureThreshold: 3       # Неудачи для перехода в Not Ready
```

### **Толерантности и привязки (Taints & Tolerations)**
```yaml
# На ноде (через kubectl taint)
# kubectl taint nodes node1 key=value:NoSchedule

# В Pod spec:
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
  tolerationSeconds: 3600  # Сколько терпеть taint
  
# Привязки к нодам (nodeAffinity)
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:  # Жесткое требование
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values: [ssd, nvme]
    preferredDuringSchedulingIgnoredDuringExecution:  # Предпочтение
    - weight: 1
      preference:
        matchExpressions:
        - key: zone
          operator: In
          values: [zone-a]
```

### **Security Context**
```yaml
securityContext:  # На уровне Pod
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault
  
containers:
- name: secure-container
  securityContext:  # На уровне контейнера
    privileged: false
    allowPrivilegeEscalation: false
    capabilities:
      drop: ["ALL"]
      add: ["NET_BIND_SERVICE"]
    readOnlyRootFilesystem: true
    seLinuxOptions:
      level: "s0:c123,c456"
```

## Пример полного приложения

```yaml
# full-application.yml
---
# 1. Конфигурация
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/mydb"
  redis_url: "redis://redis:6379"
---
# 2. Секреты
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  db_password: c2VjcmV0Cg==
  api_key: YXBpX2tleQo=
---
# 3. PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
# 4. База данных (StatefulSet)
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db_password
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
---
# 5. Backend приложение
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: myapp/backend:v1.0
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: redis_url
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: api_key
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
---
# 6. Сервис для backend
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
---
# 7. Frontend приложение
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: myapp/frontend:v1.0
        ports:
        - containerPort: 3000
        env:
        - name: BACKEND_URL
          value: "http://backend-service"
---
# 8. Сервис для frontend
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 3000
---
# 9. Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
```

## Best Practices

### **Валидация манифестов**
```bash
# Проверка синтаксиса
kubectl apply --dry-run=client -f manifest.yaml
kubectl diff -f manifest.yaml

# Валидация схемы
kubeval manifest.yaml
kube-score score manifest.yaml

# Линтинг
kube-linter lint manifest.yaml
```

### **Структура проекта**
```
k8s/
├── base/                    # Базовые конфигурации
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/               # Окружения
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   ├── staging/
│   └── production/
├── charts/                 # Helm charts
│   └── myapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
└── crds/                   # Custom Resource Definitions
```

Синтаксис манифестов Kubernetes предоставляет декларативный способ описания желаемого состояния кластера, что является фундаментальным принципом работы Kubernetes.

___
# Краткое назначение основных ресурсов Kubernetes

## 🎯 Уровень 1: Рабочая нагрузка (Workloads)

### **Pod (Под)**
> **Минимальная единица развертывания** - группа из одного или нескольких контейнеров, которые разделяют сеть и хранилище.

- **Что делает**: Запускает контейнер(ы) на ноде
- **Аналог**: Виртуальная машина для контейнеров
- **Когда использовать**: Для отладки, одноразовых задач, если не нужны фичи Deployment
- **Пример**: `web-server + log-collector` в одном Pod

### **Deployment (Развертывание)**
> **Управление жизненным циклом Pod'ов** с гарантиями доступности, обновлений и откатов.

- **Что делает**: Создает и управляет ReplicaSet'ами Pod'ов
- **Аналог**: "Автопилот" для Pod'ов
- **Когда использовать**: Для stateless приложений (90% случаев)
- **Пример**: Веб-приложение с 3 репликами, обновляемое без downtime

### **StatefulSet (Набор с состоянием)**
> **Deployment для stateful приложений** с гарантированным порядком, стабильными сетевыми идентификаторами и постоянным хранилищем.

- **Что делает**: Управляет Pod'ами с state (базы данных, очереди)
- **Аналог**: Deployment + стабильные имена + персистентное хранилище
- **Когда использовать**: Базы данных (MySQL, PostgreSQL, MongoDB), Kafka, Elasticsearch
- **Пример**: MySQL кластер с мастер-репликой

## 🔗 Уровень 2: Сеть и доступ

### **Service (Сервис)**
> **Стабильная точка доступа** к группе Pod'ов, абстрагирует их IP-адреса.

- **Что делает**: Балансирует трафик между Pod'ами, предоставляет стабильный DNS
- **Аналог**: Load Balancer внутри кластера
- **Когда использовать**: Всегда, когда Pod'ов больше одного
- **Пример**: `backend-service → 3 Pod'а backend`
- **Типы**: 
  - `ClusterIP` - только внутри кластера
  - `NodePort` - наружу через порт ноды
  - `LoadBalancer` - облачный LB
  - `ExternalName` - CNAME запись

### **Ingress (Вход)**
> **Маршрутизатор HTTP/HTTPS трафика** в кластере, управляет внешним доступом.

- **Что делает**: Роутинг по доменам и путям, SSL termination
- **Аналог**: Nginx/Apache на уровне кластера
- **Когда использовать**: Нужен роутинг по доменам или сложные правила
- **Пример**: `app.com → frontend`, `api.app.com → backend`

## ⚙️ Уровень 3: Конфигурация

### **ConfigMap (Конфигурационная карта)**
> **Хранилище неконфиденциальных конфигураций** в формате ключ-значение.

- **Что делает**: Отделяет конфиг от образа приложения
- **Аналог**: Файлы `.properties`, `.yaml`, `.json`
- **Когда использовать**: Переменные окружения, конфигурационные файлы
- **Пример**: Настройки логгирования, URL API, порты

### **Secret (Секрет)**
> **Безопасное хранилище конфиденциальных данных** (пароли, токены, ключи).

- **Что делает**: Хранит чувствительные данные (base64 encoded)
- **Аналог**: Хранилище паролей для Pod'ов
- **Когда использовать**: Пароли БД, TLS сертификаты, API токены
- **Типы**: `Opaque`, `docker-registry`, `tls`, `bootstrap-token`

## 💾 Уровень 4: Хранилище

### **PersistentVolume (PV)**
> **Абстракция физического хранилища** в кластере (диск, NAS, облачное хранилище).

- **Что делает**: Представляет кусок хранилища в кластере
- **Аналог**: Физический диск/том
- **Когда использовать**: Администратор создает для использования Pod'ами
- **Пример**: NFS share, AWS EBS volume, локальный SSD

### **PersistentVolumeClaim (PVC)**
> **Запрос на выделение хранилища** из пула PersistentVolume.

- **Что делает**: Запрашивает хранилище у кластера для Pod'а
- **Аналог**: Аренда дискового пространства
- **Когда использовать**: Приложению нужно постоянное хранилище
- **Пример**: База данных запрашивает 100GB диска

## 📊 Сводная таблица

| Ресурс | Для чего? | Аналог | Когда использовать |
|--------|-----------|--------|-------------------|
| **Pod** | Запуск контейнера(ов) | Виртуальная машина | Отладка, задачи без Deployment |
| **Deployment** | Управление stateless Pod'ами | Автопилот | Веб-приложения, микросервисы |
| **StatefulSet** | Stateful приложения с хранилищем | Deployment + диск | Базы данных, очереди |
| **Service** | Доступ к Pod'ам | Load Balancer | Любое приложение с >1 Pod |
| **Ingress** | Роутинг HTTP трафика | Nginx/Apache | Несколько доменов, SSL |
| **ConfigMap** | Конфигурация приложения | Файлы .conf | Настройки, переменные |
| **Secret** | Секретные данные | Vault/Keychain | Пароли, токены, ключи |
| **PV/PVC** | Постоянное хранилище | Диск/том | Базы данных, файловые хранилища |

## 🔄 Взаимодействие ресурсов

```
Пользователь
    ↓
[ Ingress ]  ← HTTPS → [ Secret (TLS) ]
    ↓
[ Service ]  ← конфиг → [ ConfigMap ]
    ↓
[ Deployment ]  или  [ StatefulSet ]
    ↓                    ↓
[ Pod's ]            [ Pod's ]
    ↓                    ↓
[ Контейнеры ]       [ Контейнеры ]
    ↙                   ↘
[ Secret ] ← пароли   [ PVC ] ← запрос на диск
                      ↓
                   [ PV ] ← реальный диск
```

## 🎯 Практическое правило выбора

1. **Нужно запустить контейнер?** → `Pod` (для отладки) или `Deployment` (для прода)
2. **Приложение stateful?** → `StatefulSet` вместо `Deployment`
3. **Нужен доступ к Pod'ам?** → `Service`
4. **Нужен роутинг по доменам?** → `Ingress`
5. **Есть конфигурационные файлы?** → `ConfigMap`
6. **Есть пароли/токены?** → `Secret`
7. **Нужно постоянное хранилище?** → `PersistentVolumeClaim`

Каждый ресурс решает конкретную задачу в архитектуре Kubernetes, позволяя декларативно описывать сложные распределенные системы.