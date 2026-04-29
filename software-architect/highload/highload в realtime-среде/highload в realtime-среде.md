
# highload в realtime-среде

методы оценки масштабируемости систем и научитесь внедрять контейнеризацию для динамического масштабирования, используя Kubernetes и его механизмы HPA, VPA и Cluster Autoscaler


Apache Kafka, Flink и Storm
стратегии отказоустойчивости, такие как Active-Passive, Active-Active и Geo-Redundancy
Rate Limiting и BulkHead
проектировании front-to-back и back-to-back интеграций
технологии, как REST, GraphQL, WebSockets, gRPC, RabbitMQ, Apache ActiveMQ Artemis и Kafka


* [[Performance testing|Нагрузочное тестирование]]
* [[Kubernetes scaling]]
	* Vertical Pod Autoscaler – выделяем больше ресурсов поду
		* Существует три режима работы VPA: «Off», «Initial» и «Auto»
	* Horizontal Pod Autoscaler – наращивает число подов
	* Cluster Autoscaler – наращиваем число нодов
* [[Resilience Patterns|Fault Tolerance]] – отказоустойчивость системы
	* системы [[Observability|мониторинга и алертов]]
	* [[Resilience Patterns|Rate Limiting]]  – ограничение скорости
	* [[Resilience Patterns|Bulkhead]] – отсек
	* [[Retry Policy]]
	* [[Rate Limiting|Rate limiter]]
	* Устойчивость микросервисов
		* [[Saga]]
			* компенсационные транзакции, undo's
			* тайм-ауты и дедлайны
		* [[2PC]] – 2-фазные коммиты
* [[Event-Driven Architecture|EDA]] – событийно‑ориентированныая архитектура
	* [[Event Storming]]
	* [[CQRS]]
	* [[Event sourcing]]
		* специализированные БД событий
			* [[EventStoreDB]]
			* Axon Server
			* Apache [[Kafka]]
		* паттерн [[Snapshotting]] чтобы кропать длину лога
	* паттерны обеспечения консистентности событий между сервисом и ивент басом
		* [[Transaction log tailing]]
			* Есть внешний tailer, который напрямую читает WAL и выносит из него события в брокер
		* [[Transactional outbox]]
			* дополнительная таблица в которую выношу события которые появляются в 1 транзакции с событиями
			* отдельный демон, публикует события этой таблицы в брокер
	* событийно-ориентированная обработка ошибок
* [[stream processing|потоковая обработка]]
	* Существует 4 модели потоковой обработки:
		* синхронная микропакетная обработка
		* асинхронная микропакетная обработка
		* streaming-модель
		* модель непрерывного обновления.
	- В потоковой обработке используется метод оконной агрегации
