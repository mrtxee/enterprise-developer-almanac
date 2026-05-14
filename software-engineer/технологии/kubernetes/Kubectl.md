# типовые команы
###### Какие подсы активны
 ```bash
 kubectl -n cinemaabyss get pod
 # cinemaabyss - неймспейс
 ```
###### Применить конфигурацию yaml артефакта проекта
```bash
kubectl apply -f src/kubernetes/proxy-service.yaml
```

###### Список подсов
```bash
kubectl -n cinemaabyss get pod
```
###### удалить под
``` bash
kubectl -n cinemaabyss delete pod events-service-5756fd6fb-lr8ph
```


# скачать образ из частной репы гитхаба  
docker pull ghcr.io/mrtxee/architecture-cinemaabyss/proxy-service  
docker pull ghcr.io/mrtxee/architecture-cinemaabyss/events-service  
# статус подов проекта  
kubectl -n cinemaabyss get pod  
# удалить лишний под  
kubectl -n cinemaabyss delete pod events-service-5756fd6fb-lr8ph  
# состояние пода  
kubectl -n cinemaabyss describe pod events-service-5756fd6fb-lr8ph  
# перезапуск пода  
kubectl -n cinemaabyss rollout restart deployment/events-service  
# создать под из манифеста деплоймента  
kubectl apply -f src/kubernetes/events-service.yaml  
kubectl apply -f src/kubernetes/ingress.yaml  
  
## деплойменты управляет подсами !!  
## репликасет - частный случай деплоймента !!  
## задать требуемое число репликасетов деплойента  
kubectl -n cinemaabyss get replicaset  
### 1) в файле деплоймента spec:  
  replicas: 3          # ← ← ← ЗДЕСЬ ЗАДАЁТСЯ КОЛИЧЕСТВО РЕПЛИК!kubectl apply -f kubernetes/events-service.yaml  
### рунтайме  
kubectl -n cinemaabyss scale deployment/events-service --replicas=5  
  
## удалить репликасет  
kubectl -n cinemaabyss delete replicaset events-service-67b57b756b  
## удалить депдлоймент  
kubectl -n cinemaabyss delete deployment events-service  
kubectl -n cinemaabyss delete deployment events-service proxy-service movies-service monolith  
kubectl -n cinemaabyss get pods,deploy,rs  
  
# узнать адрес ПУ кубера  
kubectl cluster-info  
  
# пересоздать неймспейс  
kubectl delete namespace cinemaabyss  
  
## перезапустить деплоймент  
kubectl -n cinemaabyss rollout restart deployment/events-service  
## удаление всех подсов, реплик,..  
kubectl create namespace cinemaabyss  
  
# список доступов проекта  
kubectl -n cinemaabyss get secrets  
  
## настройка ингресса (внеший доступ к неймспейсу)  
### Установите официальный NGINX Ingress Controller  
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml  
  
### проверить наличие ингресса  
kubectl get pods -n ingress-nginx  
  
## применить ингресс  
kubectl apply -f src/kubernetes/ingress.yaml  
###>ingress.networking.k8s.io/cinemaabyss-ingress configured  
## проверить что все ок  
kubectl -n cinemaabyss get ingress cinemaabyss-ingress  
###>NAME                  CLASS   HOSTS                     ADDRESS     PORTS   AGE  
###>cinemaabyss-ingress   nginx   cinemaabyss.example.com   localhost   80      20m  
  
## создать неймспейст из helm-чатов
helm install cinemaabyss src/kubernetes/helm --namespace cinemaabyss --create-namespace
## Удаляем все  
```bash  
kubectl delete all --all -n cinemaabyss  
kubectl delete namespace cinemaabyss