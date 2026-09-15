---
aliases:
  - Deployment
  - Helm
  - Ingress
  - k8s
  - kubectl
  - Kubernetes
  - Namespace
  - Pod
  - ReplicaSet
  - Деплоймент
  - Ингресс
  - Неймспейс
  - Под
  - Репликасет
---
## Типовые команды kubectl

### Поды

Список активных подов в неймспейсе:

```bash
kubectl -n cinemaabyss get pod
```

Подробная информация о поде:

```bash
kubectl -n cinemaabyss describe pod events-service-5756fd6fb-lr8ph
```

Удалить под:

```bash
kubectl -n cinemaabyss delete pod events-service-5756fd6fb-lr8ph
```

Перезапуск пода (через деплоймент):

```bash
kubectl -n cinemaabyss rollout restart deployment/events-service
```

### Деплойменты и репликасеты

Применить конфигурацию YAML-артефакта проекта:

```bash
kubectl apply -f src/kubernetes/proxy-service.yaml
```

Создать под из манифеста деплоймента:

```bash
kubectl apply -f src/kubernetes/events-service.yaml
```

Деплойменты управляют подами, репликасет — частный случай деплоймента. Требуемое число реплик задаётся в секции `spec` файла деплоймента:

```yaml
replicas: 3
```

Список репликасетов:

```bash
kubectl -n cinemaabyss get replicaset
```

Задать число реплик в рантайме:

```bash
kubectl -n cinemaabyss scale deployment/events-service --replicas=5
```

Удалить репликасет:

```bash
kubectl -n cinemaabyss delete replicaset events-service-67b57b756b
```

Удалить деплойменты:

```bash
kubectl -n cinemaabyss delete deployment events-service
kubectl -n cinemaabyss delete deployment events-service proxy-service movies-service monolith
```

Получить список подов, деплойментов и репликасетов:

```bash
kubectl -n cinemaabyss get pods,deploy,rs
```

### Неймспейсы и доступы

Список секретов проекта:

```bash
kubectl -n cinemaabyss get secrets
```

Удалить неймспейс:

```bash
kubectl delete namespace cinemaabyss
```

Пересоздать неймспейс (удаляет все поды, реплики и т.д.):

```bash
kubectl create namespace cinemaabyss
```

Узнать адрес кластера:

```bash
kubectl cluster-info
```

### Ingress

Установка официального NGINX Ingress Controller:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml
```

Проверить наличие ингресс-контроллера:

```bash
kubectl get pods -n ingress-nginx
```

Применить ингресс:

```bash
kubectl apply -f src/kubernetes/ingress.yaml
```

Результат: `ingress.networking.k8s.io/cinemaabyss-ingress configured`.

Проверить состояние ингресса:

```bash
kubectl -n cinemaabyss get ingress cinemaabyss-ingress
```

Вывод:

```
NAME CLASS HOSTS ADDRESS PORTS AGE
cinemaabyss-ingress nginx cinemaabyss.example.com localhost 80 20m
```

Создать неймспейс из helm-чартов:

```bash
helm install cinemaabyss src/kubernetes/helm --namespace cinemaabyss --create-namespace
```

### Скачивание образов из частного репозитория GitHub

```bash
docker pull ghcr.io/mrtxee/architecture-cinemaabyss/proxy-service
docker pull ghcr.io/mrtxee/architecture-cinemaabyss/events-service
```

### Удаление всех ресурсов проекта

```bash
kubectl delete all --all -n cinemaabyss
kubectl delete namespace cinemaabyss
```
