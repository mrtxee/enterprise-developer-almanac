---
aliases:
  - Docker Desktop
  - Helm
  - Ingress
  - k3d
  - kind
  - kubectl
  - kubernetes
  - Kustomize
  - Minikube
  - namespace
  - Namespace
  - Ингресс
  - Неймспейс
---

## Структура проекта и применение

Практическое применение Kubernetes к проекту: как организовать файлы манифестов, в каком порядке их применять и как проверить развёртывание.

### Варианты API-сервера

- **Minikube**, **kind**, **k3d** — для локальной разработки.
- **Docker Desktop** — встроенный кластер в macOS/Windows/Linux.

### Типовая структура файлов

| Файл | Назначение |
|------|-----------|
| `namespace.yaml` | Создаёт пространство имён |
| `configmap.yaml` | Общие ConfigMap |
| `secret.yaml` / `dockerconfigsecret.yaml` | Секреты и доступ к registry |
| `postgres.yaml` / `kafka.yaml` | Базы данных и очереди |
| `*-service.yaml` | Deployment + Service для микросервиса |
| `ingress.yaml` | Ingress для внешнего доступа |

### Порядок применения файлов

```bash
# 1. Namespace
kubectl apply -f kubernetes/namespace.yaml

# 2. Секреты и ConfigMap
kubectl apply -f kubernetes/secret.yaml
kubectl apply -f kubernetes/configmap.yaml

# 3. Зависимости (БД, очереди)
kubectl apply -f kubernetes/postgres.yaml
kubectl apply -f kubernetes/kafka.yaml

# 4. Сервисы
kubectl apply -f kubernetes/events-service.yaml
kubectl apply -f kubernetes/proxy-service.yaml

# 5. Внешний доступ
kubectl apply -f kubernetes/ingress.yaml
```

> Приложения зависят от secret'ов, configmap'ов, баз данных — создавайте зависимости до приложений.

### Шаблон нового сервиса

```yaml
# notifications-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: notifications-service
  namespace: cinemaabyss
spec:
  selector: { app: notifications-service }
  ports: [{ port: 8081, targetPort: 8081 }]
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notifications-service
  namespace: cinemaabyss
spec:
  replicas: 1
  selector:
    matchLabels: { app: notifications-service }
  template:
    metadata:
      labels: { app: notifications-service }
    spec:
      imagePullSecrets:
      - name: dockerconfigjson
      containers:
      - name: notifications
        image: your-registry.com/notifications:latest
        ports: [{ containerPort: 8081 }]
        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef: { name: secret, key: db-host }
        resources:
          requests: { memory: "128Mi", cpu: "100m" }
```

### Проверка развертывания

```bash
kubectl get all -n <namespace>
kubectl logs -n <namespace> <pod-name>
kubectl run -it --rm debug --image=busybox -n <namespace> -- sh
# wget -qO- http://events-service:8082/api/events
```

> Управлять всеми ресурсами как единым приложением удобно через [[helm|Helm]] или Kustomize.
