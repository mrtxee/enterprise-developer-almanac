# result
```text

Форма запроса сертификата и гранта (Инженер облачных сервисов)

https://forms.yandex.ru/surveys/10034488.ad81b65a7990f170d616903fb18add30cf13003e/success/?akey=8d250034fbccdfd053ad0457d74a1197abda826e

Спасибо! Мы приняли вашу заявку.

Рассмотрение заявки и подпись сертификата о повышении квалификации требуют времени.  
Мы ответим на вашу заявку в течение трех недель.
```
# прохождение теста

## Создание ВМ и балансировка нагрузки

1. Создайте две виртуальные машины (платформа Intel Ice Lake, 2 vCPU, 4ГБ RAM, 20ГБ SDD, ОС Ubuntu 22.04) c именами `test-vm1` и `test-vm2`. Установите на эти ВМ веб-серверы NGINX и измените информационные страницы веб-серверов так, чтобы на них в качестве приветствия выводилась фраза “Welcome to the first test!”.
2. Добавьте созданные ВМ в целевую группу `test1`.
3. Создайте балансировщик нагрузки `test1-balancer` и настройте проверку состояния ВМ в целевой группе.
4. Остановите одну из ВМ.
5. Убедитесь, что вторая ВМ доступна.
6. Когда ресурсы будут созданы, запустите следующую команду для проверки задания: `docker run --rm cr.yandex/sol/edu-checker validate balancer --token <TOKEN> --folder-id <FOLDER_ID>` где `<TOKEN>` — ваш IAM-токен, а `<FOLDER ID>` — идентификатор каталога, в котором вы работаете.
7. Если проверка пройдена успешно, то скопируйте ключ проверки, вставьте его в поле ниже и нажмите кнопку **Проверить**.
## Работа с объектным хранилищем

Это задание следует выполнять с помощью AWS CLI

1. Создайте в объектном хранилище S3 бакет и загрузите в него два объекта: `image01.dat` и `image02.dat`.
2. Добавьте к обоим объектам следующие метаданные:
    а) Метаданные с именем patient и значением ivanov. 
    б) Метаданные с именем status и значением ok.
3. Измените метаданные объекта `image02.dat`, заменив значение status с ok на ill.
4. Создайте временную ссылку на объект `image02.dat`.
5. Создайте временную ссылку на объект `image02.dat`.
6. Для проверки задания запустите команду :
    
    `docker run --rm -v ~/.aws:/root/.aws cr.yandex/sol/edu-checker validate s3 --bucket <BUCKET_NAME>`
    
    где `<BUCKET_NAME>` — имя созданного бакета.
7. Если проверка пройдена успешно, то скопируйте ключ проверки, вставьте его в поле ниже и нажмите кнопку **Проверить**.
## Добавление данных в ClickHouse

1. Создайте кластер управляемой базы данных ClickHouse следующей конфигурации: тип хоста `burstable`, класс `b3-c1-m4` и стандартное сетевое хранилище размером 10 ГБ.
2. Сохраните на своей рабочей станции этот [файл](https://storage.yandexcloud.net/arhipov/weather_data.tsv). Это уже известный вам датасет с данными о погоде в Москве и Санкт-Петербурге.
3. Создайте в БД таблицу с именем `weather`, в которой есть следующие поля (в скобках указан тип данных): LocalDateTime (DateTime), LocalDate (Date), Month (Int8), Day (Int8), TempC (Float32), Pressure (Float32), RelHumidity (Int32), WindSpeed10MinAvg (Int32), VisibilityKm (Float32), City (String).
4. Загрузите данные из файла в БД.
5. Для проверки задания запустите команду : `docker run --rm cr.yandex/sol/edu-checker validate clickhouse --token <TOKEN> --host <CLUSTER_HOST> --database <DB_NAME> --username <DB_USERNAME> --password <DB_PASSWORD>`  
    где `<TOKEN>` — ваш IAM-токен, `<CLUSTER_HOST>` — идентификатор хоста кластера, `<DB_NAME>` — имя созданной БД, `<DB_USERNAME>` — имя пользователя БД, `<DB_PASSWORD>` — пароль пользователя.
6. Если проверка пройдена успешно, то скопируйте ключ проверки, вставьте его в поле ниже и нажмите кнопку **Проверить**.
## Развёртывание ресурсов с помощью Packer и Terraform

1. С помощью Packer создайте образ виртуальной машины с ОС Ubuntu 22.04 и установленным веб-сервером NGINX.
2. С помощью Terraform создайте:
    - сеть с именем `from-terraform-network`,
    - подсеть с именем `from-terraform-subnet` в зоне доступности `ru-central1-a`, и присвоить этой подсети диапазон адресов (CIDR) 10.2.0.0/16,
    - расположенную в этой подсети виртуальную машину с именем `from-terraform-vm` (два ядра vCPU, 2ГБ RAM, платформа Intel Broadwell) и присвоить ВМ публичный адрес;
    - кластер управляемой базы данных PostgresSQL с именем `test-vm` (класс `s1.micro`, платформа Intel Icelake, SSD диск объёмом 10ГБ).
        
        Для создания виртуальной машины используйте образ, созданный на предыдущем шаге.
        
3. Когда ресурсы будут созданы, запустите следующую команду для проверки задания: `docker run --rm cr.yandex/sol/edu-checker validate vm --token <TOKEN> --folder-id <FOLDER_ID>` где `<TOKEN>` — ваш IAM-токен, а `<FOLDER_ID>` — идентификатор каталога, в котором вы работаете.
4. Если проверка пройдена успешно, то скопируйте ключ, вставьте его в поле ниже и нажмите кнопку **Проверить**
## Сокращатель ссылок

1. Создайте сервис для конвертации ссылок из длинных в короткие. Воспользуйтесь инструкцией в уроке «[Практическая работа. Сокращатель ссылок](https://practicum.yandex.ru/trainer/ycloud/lesson/8c75809b-28a0-4898-ba8a-23fc2c3dc677/)».
2. Запустите сервис, чтобы сконвертировать ссылку на урок, упомянутый в предыдущем пункте.
3. Запустите следующую команду для проверки задания: `docker run --rm cr.yandex/sol/edu-checker validate shorter --url <SERVICE_URL>` где `<SERVICE_URL>` — адрес, по которому доступен ваш сервис.
4. Если проверка пройдена успешно, то скопируйте ключ проверки, вставьте его в поле ниже и нажмите кнопку **Проверить**.

## Права доступа и роли для сервисного аккаунта

Выполните ротацию ключа шифрования с использованием сервисного аккаунта.

1. Создайте ключ шифрования с именем `test-key`, выберите алгоритм шифрования AES-256.
2. Создайте сервисный аккаунт с именем `test-account` и присвойте ему необходимые роли для ротации ключа шифрования.
3. Ротируйте ключ шифрования `test-key` из-под сервисного аккаунта `test-account`.
4. Для проверки задания запустите команду: `docker run --rm cr.yandex/sol/edu-checker validate security --token <TOKEN> --folder-id <FOLDER_ID>` где `<TOKEN>` — ваш IAM-токен, а `<FOLDER_ID>` — идентификатор каталога, в котором вы работаете.
5. Если проверка пройдена успешно, то скопируйте ключ проверки, вставьте его в поле ниже и нажмите кнопку **Проверить**.
# туториалы

## ротация ключей 
## TLS
* https://practicum.yandex.ru/trainer/ycloud/lesson/32f3b7e7-722c-48b2-afa9-4603554d2c06/
* https://practicum.yandex.ru/trainer/ycloud/lesson/3d1ba67f-ceb8-4cc8-8aab-1f5b82d97a8c/ ротация ключей
## права доступа
* https://practicum.yandex.ru/trainer/ycloud/lesson/5ac0e905-b6d7-485e-976a-9f5ad9bc00d7/
* https://practicum.yandex.ru/trainer/ycloud/lesson/c360c1f9-2cda-477f-818f-2a4f6a5de988/
## очереди сообщений
* https://practicum.yandex.ru/trainer/ycloud/lesson/497f8e56-8d25-44c7-b109-3b3935bdd6df/
## serverless загрузка данных
* Практика. Загрузка данных, выполнение запросов AWS CLI
* https://practicum.yandex.ru/trainer/ycloud/lesson/0914346c-22d5-4180-b1c2-2d1134133de7/
* https://practicum.yandex.ru/trainer/ycloud/lesson/f5f6a261-55d3-4795-b46a-6edca0208f20/

## yandex API gateaway
* https://practicum.yandex.ru/trainer/ycloud/lesson/334b8e9b-7783-4294-a504-4dcf28aac5f3/
## Serverless функция
* https://practicum.yandex.ru/trainer/ycloud/lesson/9b81bf9d-21ff-47ab-9102-2080b9dd2e5c/
* https://practicum.yandex.ru/trainer/ycloud/lesson/1100c6c5-c4ce-476f-a75b-3c51c2668248/
* https://practicum.yandex.ru/trainer/ycloud/lesson/ce6e0d5c-c056-46ff-92e3-c93ca3014a9d/
* https://practicum.yandex.ru/trainer/ycloud/lesson/645cae42-b61c-4632-904e-a392e3c916ea/
* https://practicum.yandex.ru/trainer/ycloud/lesson/8e77c32d-662a-4981-9fb6-87dd197db527/
## Алерты
* https://practicum.yandex.ru/trainer/ycloud/lesson/da5762e1-7525-4fc9-b2de-56e739479c82/
## Мониторинг
* Настройка яндекс мониторинг
	* https://practicum.yandex.ru/trainer/ycloud/lesson/7be63965-74d4-4259-b2e2-083fea5643d2/
* Отправка собственных метрик
	* https://practicum.yandex.ru/trainer/ycloud/lesson/18d756e9-3f1a-4bb6-be9c-28e69b94d074/
## Отказоустойчивые системы
https://practicum.yandex.ru/trainer/ycloud/lesson/2f4feea9-e6ff-4e7a-a204-1b3994705fe0/
* распределение по зонам доступности
* обновление приложения
* восстановление после сбоя ВМ
## Создание докер-образа и загрузка его в Container Registry
https://practicum.yandex.ru/trainer/ycloud/lesson/a362f68c-9882-48b5-8c0e-68eb67c756a8/
- Создание докер-образа и загрузка его в Yandex Container Registry.
- Установка Docker и создание реестра в Yandex Container Registry.
- Аутентификация в Yandex Container Registry с помощью Docker Credential helper.
- Подготовка Dockerfile и сборка образа.
- Загрузка Docker-образа в реестр: 
	- `docker push cr.yandex/<идентификатор_реестра>/ubuntu-nginx:latest`
- Предоставление прав на использование образов всем пользователям реестра.
- Создание виртуальной машины с помощью Container Optimized Image и настройка загрузочного диска.
- Запуск виртуальной машины и проверка доступа к приветственной странице NGINX.
## Создание кластера k8s
https://practicum.yandex.ru/trainer/ycloud/lesson/166f2f98-e773-4e2a-949e-9c8f61459b24/
1. Создание кластера Kubernetes и группы узлов в нем. 
2. Выбор каталога для кластера и сервиса Managed Service for Kubernetes. 
3. Необходимость сервисного аккаунта для ресурсов и узлов Kubernetes. 
	1. Использование разных прав для сервисных аккаунтов. 
4. Выбор релиза Kubernetes (RAPID, REGULAR или STABLE). 
5. Конфигурация мастера: выбор версии Kubernetes, публичный IP-адрес, тип мастера и зоны доступности. 
	1. Настройки обновлений: четыре режима (Отключено, В любое время, Ежедневно или В выбранные дни). 
	2. Сетевые настройки: необязательные сетевые политики для кластера Kubernetes.
	3. Задание CIDR для внутренних IP-адресов подов и сервисов Kubernetes. 
	4. Создание группы узлов: имя, описание, версия Kubernetes, тип масштабирования, сетевые настройки, SSH-ключ и настройки обновления.
**Первое приложение в кластере**
* Развертывание приложения - веб-сервера NGINX в кластере Kubernetes с помощью командной строки.
- Основное средство взаимодействия с кластером - инструмент kubectl.
- Создание манифеста для описания настроек приложения в кластере.
- Выполнение манифеста с помощью команды kubectl apply.
- Получение подробной информации о развернутом приложении с помощью команд kubectl get pods и kubectl describe.
- Масштабирование приложения с помощью изменения файла манифеста или команды kubectl scale.
- Управление кластерами Kubernetes в концепции Infrastructure as Code и возможность развертывания с помощью Terraform.
**Балансировка нагрузки**
- Веб-приложения должны быть доступны из интернета.
- Сервис LoadBalancer используется для решения этой проблемы.
- Внутренний IP-адрес может меняться, поэтому нужен публичный IP-адрес балансировщика.
- Создается файл-манифест с описанием балансировщика.
- Выполняется манифест с помощью kubectl.
- В консоли управления можно увидеть созданный балансировщик.
- Копируется IP-адрес балансировщика в адресную строку браузера для доступа к приложению.
**Автоматическое масштабирование**
- Масштабирование позволяет распределить нагрузку между контейнерами и снизить риск сбоя.
- Ручное масштабирование - трудоёмкий и неэффективный процесс.
- Для автомасштабирования подходят инструменты Horizontal Pod Autoscaler и Cluster Autoscaler.
- Horizontal Pod Autoscaler масштабирует количество под-контейнеров, основываясь на нагрузке и запросах.
- Cluster Autoscaler автоматически изменяет количество узлов Kubernetes, основываясь на запросах под-контейнеров.
- В Yandex Managed Service for Kubernetes инструмент Cluster Autoscaler включён по умолчанию.
**Автомасштабирование в Yandex Managed Kubernetes**
- Создание манифеста load-balancer-hpa.yaml для горизонтального автомасштабирования в Kubernetes.
- Использование меток (labels) для контейнеров с меткой "nginx-hpa".
- Изменение образа контейнера с Yandex Container Registry на k8s.gcr.io/hpa-example для создания высокой нагрузки на процессор.
- Добавление настроек ресурсов (requests и limits) для контейнера nginx-hpa.
- Создание манифеста Horizontal Pod Autoscaler с настройками minReplicas, maxReplicas и targetCPUUtilizationPercentage.
- Применение манифеста и ожидание запуска компонентов.
- Создание рабочей нагрузки с помощью утилиты wget.
- Наблюдение за увеличением числа подов и узлов в кластере.
**Мониторинг Managed Kubernetes**
- Мониторинг состояния кластера с помощью дашбордов в Yandex Managed Kubernetes.
- Запуск цикла с утилитой wget для подачи нагрузки на кластер.
- Переход в веб-консоль, раздел Managed Service for Kubernetes, и переключение на вкладку Рабочая нагрузка.
- Просмотр состояния ресурсов и событий в кластере.
- Фильтрация событий по сообщению и использование выпадающих списков для быстрого поиска.
- Настройка мониторинга и выбор нужных данных для детализации.
- Исследование возможностей мониторинга для узлов, подов, балансировщиков и сервисов.
**Отказоустойчивость Managed Kubernetes**
- Yandex Managed Kubernetes обеспечивает определенный уровень отказоустойчивости.
- Автомасштабирование повышает отказоустойчивость.
- Managed Kubernetes интегрирован с инфраструктурой Yandex Cloud.
- Рассмотрите сценарии отказов: отказ ноды, обновление Kubernetes, сбой DNS.
- Общие рекомендации: обновляйте рабочую среду вручную, используйте региональный тип мастера, настройте параметры для LoadBalancer, настройте requests и limits, разверните сервисы в нескольких экземплярах, отслеживайте готовность приложения с помощью readiness-проб.
**Управление доступом**
- Надёжность Kubernetes зависит от уровней доступа к объектам и операциям над ними.
- Ролевая модель используется для разграничения доступа к объектам кластера.
- Квоты применяются для ограничения потребления ресурсов в облаке.
- Типичные роли в Kubernetes: администратор облака, сетевой администратор, специалист по безопасности, разработчик, DevOps/SRE, инструменты CI/CD.
- Роли для управления ресурсами внутри кластера и в Managed Kubernetes различаются.
- В Yandex Cloud используются роли для управления ресурсами и доступа к ним.
- Разделение рабочих и тестовых сред с помощью каталогов и настройка политик ролей и квот.
- Примеры ролей для DevOps/SRE, разработчика, кластера K8s и узлов K8s.
- Безопасность и продумывание ролевой матрицы для доступа к продуктивной и непродуктивной среде.
## Создаём виртуальную машину из образа и базу данных
https://practicum.yandex.ru/trainer/ycloud/lesson/e027d622-2926-4a4c-8231-3b6b44577c9b/
- Создание кластера PostgreSQL с помощью Terraform и Yandex Cloud.
- Установка и настройка Terraform для работы с Yandex Cloud.
- Создание виртуальной машины и базы данных с использованием образа, созданного с помощью Packer.
- Использование переменных в спецификации Terraform для создания разных конфигураций.
- Создание сети и подсети с использованием Yandex Cloud.
- Применение обновленной спецификации для создания кластера PostgreSQL.
- Удаление инфраструктуры с использованием Terraform.
## другие

- **резервное копирование** 
	- [https://practicum.yandex.ru/trainer/ycloud/lesson/0cf6ff0b-f476-49fb-b60c-b33032c17979/](https://practicum.yandex.ru/trainer/ycloud/lesson/0cf6ff0b-f476-49fb-b60c-b33032c17979/)
    
- **Кластер БД, Yandex Lens, Big Data**
    
    развернуть кластер [[MongoDB]]
    
    [https://practicum.yandex.ru/trainer/ycloud/lesson/9534f8eb-8585-4b3b-8393-b68d51c06898/](https://practicum.yandex.ru/trainer/ycloud/lesson/9534f8eb-8585-4b3b-8393-b68d51c06898/)
    
    развернуть кластер ClickHouse
    
    [https://practicum.yandex.ru/trainer/ycloud/lesson/f1b6a710-1fa1-4ee5-aa41-7c8ace3762a3/](https://practicum.yandex.ru/trainer/ycloud/lesson/f1b6a710-1fa1-4ee5-aa41-7c8ace3762a3/)
    
    развернуть кластер YDB
    
    [https://practicum.yandex.ru/trainer/ycloud/lesson/bb84c932-2cbd-4737-9d56-2f7952462493/](https://practicum.yandex.ru/trainer/ycloud/lesson/bb84c932-2cbd-4737-9d56-2f7952462493/)
    
    [https://practicum.yandex.ru/trainer/ycloud/lesson/40b92238-c508-482b-8687-ddb306345bd7/](https://practicum.yandex.ru/trainer/ycloud/lesson/40b92238-c508-482b-8687-ddb306345bd7/)
    
    Yandex Data Proc для настройки кластера Handoop & Spark, создание кластера
    
    [https://practicum.yandex.ru/trainer/ycloud/lesson/f2c70e28-1220-49e2-9791-9a2caf86dbe1/](https://practicum.yandex.ru/trainer/ycloud/lesson/f2c70e28-1220-49e2-9791-9a2caf86dbe1/)
    
    [https://practicum.yandex.ru/trainer/ycloud/lesson/d17c4bc1-7a72-4fcf-aeb1-003578d02795/](https://practicum.yandex.ru/trainer/ycloud/lesson/d17c4bc1-7a72-4fcf-aeb1-003578d02795/)
    
    Yandex Lens
    
    [https://practicum.yandex.ru/trainer/ycloud/lesson/2b7232e0-623d-4f7d-afc0-75bc37541855/](https://practicum.yandex.ru/trainer/ycloud/lesson/2b7232e0-623d-4f7d-afc0-75bc37541855/)
    
- **yandex cloud client**
    
    **CLI: quickstart yandex cloud client**
    
    [https://practicum.yandex.ru/trainer/ycloud/lesson/f419567a-ceb6-4f04-b1af-e9af6e217902/](https://practicum.yandex.ru/trainer/ycloud/lesson/f419567a-ceb6-4f04-b1af-e9af6e217902/)
    
    **CLI: Создание виртуальных машин с помощью CLI**
    [https://practicum.yandex.ru/trainer/ycloud/lesson/f5ef7735-66df-432c-ab99-2057874be107/](https://practicum.yandex.ru/trainer/ycloud/lesson/f5ef7735-66df-432c-ab99-2057874be107/
    **Использование файлов спецификаций**
    [https://practicum.yandex.ru/trainer/ycloud/lesson/373ca12b-4264-42b7-a987-9ea26628e6a4/](https://practicum.yandex.ru/trainer/ycloud/lesson/373ca12b-4264-42b7-a987-9ea26628e6a4/)
- **создаем обрам ВМ для Packer**
    [https://practicum.yandex.ru/trainer/ycloud/lesson/0e1aa1d8-3bb9-4154-a370-a7f645700923/](https://practicum.yandex.ru/trainer/ycloud/lesson/0e1aa1d8-3bb9-4154-a370-a7f645700923/)
    

---

- **флоу**
    
    ```
    -- <https://console.yandex.cloud/>
    -- vm-ecs-20240529 creds
    	ppk
    	mrtxee
    	qq
    	
    	vm-ecs-20240529 -- vm name
    	fhm5isqos1o9ur3nq9mt -- vm id
    
    **Последовательный порт** -- COM1
    ```
    


# intro

прхожу курс **Яндекс.Практикум Инженер облачный сервисов,** [https://practicum.yandex.ru/profile/ycloud/](https://practicum.yandex.ru/profile/ycloud/) c 20-05-2024 ПУ [https://console.yandex.cloud/](https://console.yandex.cloud/)
# creds
yc, yandex cloud cli token: y0_AgAAAAAT7u2sAATuwQAAAAEHaWeIAACc0TUyJzlDYr6aSIbtVhqWj0w6rw

---
 🚨 https://practicum.yandex.ru/software-architect/ проийти курс яндекс.практикум — **Архитектура программного обеспечения**

