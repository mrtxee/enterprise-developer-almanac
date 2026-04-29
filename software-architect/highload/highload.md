---
aliases:
  - highload
  - high-load
  - высоконагруженные системы
  - нагрузка
  - load metrics
---

## Повышение отказоустойчивости системы

* Повышай отказоустойчивость системы [[Resilience Patterns|Fault Tolerance]]
	* Используй [[Resilience Patterns|паттерны устойчивости]] при проектировании системы
* Используй событийно‑ориентированную архитектуру [[Event-Driven Architecture|EDA]] для декуплинга распределенных систем
* Используй [[software-architect/highload/caching/caching|кэширование]] для кратного снижения нагрузки
* Проводи [[Performance testing|Нагрузочное тестирование]]
* Автоматизируй управление масштабированием, 
	* eg. [[Kubernetes scaling]]
* Улучшай наблюдаемость системы
	* [[Monitoring]] – *Что сломалось в системе?*
		* **Реактивных подход** к анализу состояния системы
		* [[Metrics|Метрики мониторинга]], [[SLI|SLI]]
		* [[Prometheus]] + [[Grafana]]
	* [[Observability]] – *Почему это сломалось?*
		* **Проактивный подход** к анализу состояния системы
		1. [[Metrics]] – метрики системы
			* Агрегация метрик по [[Metrics|USE]], [[Metrics|RED]], [[Metrics|4-Golden-Signals]]
			* [[reliability metrics|метрики доступности системы]]
		2. [[Logging]] – события
		3. [[Tracing]] – трасировка запросов
		4. [[ELK]] – стэк для работы с распределенными логами и трейсами
* Используй [[stream processing|потоковую обработку]] в среде реального времени, где это требуется.


# highload в realtime-среде


plan
* проектировании front-to-back и back-to-back интеграций
* технологии, как REST, GraphQL, WebSockets, gRPC, RabbitMQ, Apache ActiveMQ Artemis и Kafka

data
* front-to-back
	* [[Client pull]]
		* паттерн Polling
			* short polling
			* long polling
		* композиция API
	* [[Server push]]
		* [[webRTC webSocket|webSocket]]
		* [[Server push|SSE]] (Server sent events)
		* [[Server push|GraphQL Subscriptions]]
* [[Back-to-Back|back-to-back]]
	* Характер взаимодействия
		* p2p – Точка-точка
		* [[Publish-Subscribe|Publish/Subscribe]] – Публикация-подписка
	* Способ взаимодействия
		* синхронный
		* асинхронный

> pull – тянуть
> polling – опрос




## Композиция API
* с реализацией через 
	* [[REST]] для общего случая
	* [[GraphQL]] чтобы избежать избыточности

# [[Kubernetes scaling]]
* VPA – Vertical Pod Autoscaler – выделяем больше ресурсов поду
	* Существует три режима работы VPA: «Off», «Initial» и «Auto»
* HPA – Horizontal Pod Autoscaler – наращивает число подов
* CA – Cluster Autoscaler – наращиваем число нодов

# [[Event-Driven Architecture]]
* [[Event-Driven Architecture|EDA]] – событийно‑ориентированная архитектура
	* [[Event Storming]]
	* [[CQRS]]
	* [[Event sourcing]]
		* специализированные БД событий
			* [[EventStoreDB]]
			* Axon Server
			* Apache Kafka
		* паттерн [[Snapshotting]] чтобы кропать длину лога
	* паттерны обеспечения консистентности событий между сервисом и ивент басом
		* [[Transaction log tailing]]
			* Есть внешний tailer, который напрямую читает WAL и выносит из него события в брокер
		* [[Transactional outbox]]
			* дополнительная таблица в которую выношу события которые появляются в 1 транзакции с событиями
			* отдельный демон, публикует события этой таблицы в брокер
	* событийно-ориентированная обработка ошибок

# [[stream processing]]

* Существует 4 модели потоковой обработки:
	* синхронная микропакетная обработка
	* асинхронная микропакетная обработка
	* streaming-модель
	* модель непрерывного обновления.
- В потоковой обработке используется метод оконной агрегации
- Apache Kafka, Flink и Storm