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

## Kubectl

**kubectl** — утилита командной строки (CLI) для управления [[kubernetes|Kubernetes]].

### Поды

```bash
kubectl -n cinemaabyss get pods                                  # список подов
kubectl -n cinemaabyss describe pod events-service-5756fd6fb-lr8ph   # подробности о поде
kubectl -n cinemaabyss delete pod events-service-5756fd6fb-lr8ph     # удалить под (Deployment пересоздаст)
kubectl -n cinemaabyss rollout restart deployment/events-service     # перезапуск подов деплоймента
```

### Деплойменты и репликасеты

```bash
kubectl apply -f src/kubernetes/events-service.yaml   # создать/обновить ресурсы из манифеста

kubectl -n cinemaabyss get replicaset                               # список репликасетов
kubectl -n cinemaabyss scale deployment/events-service --replicas=5  # изменить число реплик
kubectl -n cinemaabyss delete replicaset events-service-67b57b756b   # удалить репликасет

kubectl -n cinemaabyss delete deployment events-service proxy-service movies-service monolith  # удалить несколько деплойментов сразу
kubectl -n cinemaabyss get pods,deploy,rs                           # посмотреть несколько типов ресурсов
```

Деплойменты управляют подами, репликасет — частный случай деплоймента. Требуемое число реплик задаётся в секции `spec`:

```yaml
replicas: 3
```

### Неймспейсы и доступы

```bash
kubectl -n cinemaabyss get secrets                # секреты проекта
kubectl delete namespace cinemaabyss              # удалить неймспейс (все поды, реплики и т.д.)
kubectl create namespace cinemaabyss              # пересоздать неймспейс
kubectl cluster-info                              # адрес кластера
```

### Ingress

```bash
# Установка официального NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml  # применить манифест контроллера
kubectl get pods -n ingress-nginx                 # проверить наличие контроллера
kubectl apply -f src/kubernetes/ingress.yaml      # применить ingress
kubectl -n cinemaabyss get ingress cinemaabyss-ingress   # проверить статус ingress
```

Пример вывода после применения:

```text
NAME                  CLASS   HOSTS                      ADDRESS   PORTS   AGE
cinemaabyss-ingress   nginx   cinemaabyss.example.com    localhost  80      20m
```

Создать неймспейс из Helm-чартов:

```bash
helm install cinemaabyss src/kubernetes/helm --namespace cinemaabyss --create-namespace  # установить чарт и создать неймспейс
```

### Скачивание образов из частного репозитория GitHub

```bash
docker pull ghcr.io/mrtxee/architecture-cinemaabyss/proxy-service     # скачать образ proxy-сервиса
docker pull ghcr.io/mrtxee/architecture-cinemaabyss/events-service    # скачать образ events-сервиса
```

### Удаление всех ресурсов проекта

```bash
kubectl delete all --all -n cinemaabyss   # удалить все ресурсы в неймспейсе
kubectl delete namespace cinemaabyss      # удалить неймспейс целиком
```
