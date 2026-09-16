---
aliases:
  - CoreDNS
  - Fluent Bit
  - Ingress Controller
  - NGINX Ingress
  - Node Problem Detector
  - OpenTelemetry
  - Инфраструктурные поды
  - кластер
  - Кластерная инфраструктура
  - Поды kube-system
---

## Инфраструктурные поды

Поды в namespace `kube-system` (и в связанных: `kube-node-lease`, `ingress-nginx`, `monitoring`) обеспечивают жизненно важные функции кластера. **Не удаляйте и не модифицируйте их вручную.**

| Компонент | Назначение |
|-----------|-----------|
| **CoreDNS** | DNS-сервер кластера: преобразует имена сервисов в IP-адреса (`my-service.ns.svc.cluster.local`) |
| **node-local-dns** | Локальный DNS-кэш на каждой ноде — снижает задержку и нагрузку на CoreDNS |
| **kube-proxy** | Под на каждой ноде: управляет правилами маршрутизации трафика к Pod'ам (iptables/IPVS) |
| **everest-csi-controller / driver** | CSI-драйвер для управления хранилищами (монтирование PV/PVC) |
| **Node Problem Detector** | Мониторит ошибки ядра, Docker/containerd, файловой системы; переводит ноду в NotReady |
| **Fluent Bit** | Сбор логов контейнеров из `/var/log/containers`, доставка в агрегатор (Elasticsearch, Loki) |
| **OpenTelemetry Collector** | Сбор метрик и трейсов в едином формате, экспорт в систему мониторинга |
| **NGINX Ingress Controller** | Реализация Ingress: маршрутизация, TLS, rate limiting |
| **NGINX Ingress default backend** | Ответ на запросы без подходящего Ingress-правила (404) |
| **metrics-server** | Сбор метрик CPU/RAM по Pod'ам и нодам (источник для `kubectl top` и HPA) |
| **Cluster Autoscaler** | Добавляет/удаляет ноды облачного кластера по загрузке (см. [[kubernetes-scaling|Масштабирование]]) |
| **kube-state-metrics** | Экспорт состояния объектов кластера (Deployments, Pods, Services) в Prometheus |
| **cert-manager** | Автоматическое получение и продление TLS-сертификатов (Let's Encrypt) |

Компоненты Control Plane (`kube-apiserver`, `etcd`, `kube-scheduler`, `kube-controller-manager`) запускаются как статические поды на Control Plane нодах — их подробное описание см. в [[kubernetes|Архитектура кластера]].

### Как определить назначение пода

```bash
kubectl get pod <name> -n kube-system -o jsonpath="{.spec.containers[0].image}"
kubectl get pod <name> -n kube-system -L app,component
kubectl describe pod <name> -n kube-system
kubectl top pod -n kube-system
```

### Managed Kubernetes

Managed Kubernetes (EKS, GKE, AKS) — сервис автоматического управления кластером Kubernetes в облаке: масштабирование, обновления, безопасность. Рабочие поды (kube-proxy, logging, monitoring, CSI-драйверы, Ingress Controller) остаются, но контрольная плоскость (etcd, apiserver) управляется провайдером и не видна.
