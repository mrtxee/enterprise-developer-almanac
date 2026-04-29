Текущее задание: https://practicum.yandex.ru/learn/software-architect/courses/958ef1df-e5b3-4055-b01e-540e479ca32f/sprints/590668/topics/7a581a1a-47fe-43ad-a0ae-c1b7ce1adfc7/lessons/160c1e58-688e-4d31-8a3d-a59d342ac492/

Типовые инструменты визуализации архитектуру: с помощью модели С4, разработаете ER-диаграмма
[[4C]], [[ER-diagram]]
# intro

# Архитектура программного обеспечения

[software-architect course index](https://practicum.yandex.ru/profile/software-architect/)

> «_Архитектура — это всё, что важно, чем бы оно ни являлось_».
> _— Ральф Джонсон_

* Важно держать решаемую проблему как «компас» перед глазами. Для этого проблему нужно однозначно понимать.

>Any run-of-the-mill engineer can design something which is elegant. A good engineer designs systems to be efficient. A great engineer designs them to be effective.
>Любой заурядный инженер может спроектировать что-то элегантное. Хороший инженер создаёт эффективные системы. Великий инженер проектирует их так, чтобы они были результативными.
> — [Akin's Laws of Spacecraft Design](https://spacecraft.ssl.umd.edu/akins_laws.html)

* Кажется, одна из главных характеристик сильного инженера — умение бороться со сложностью, а не увеличивать её.

> _A project is shipped when the important people at your company believe it is shipped._
> Проект считается завершённым, когда важные люди в вашей компании считают его завершённым.
> _Источник: [https://www.seangoedecke.com/how-to-ship/](https://www.seangoedecke.com/how-to-ship/)_

> _(Larrabee's Law) Half of everything you hear in a classroom is crap. Education is figuring out which half is which._
> (Закон Ларраби) Половина того, что вы слышите на занятиях, — чушь. Учёба же — это умение понять, какая именно половина.
> _Источник: [https://spacecraft.ssl.umd.edu/akins_laws.html](https://spacecraft.ssl.umd.edu/akins_laws.html)_

# о курсе
* В курсе будет 11 спринтов
* В конце каждого спринта вы будете сдавать проектную работу, то есть с завершением курса в вашей копилке появится 11 реализованных проектов. Выполнение и проверка задания могут проходить в несколько итераций (циклов). Выглядеть это будет примерно так: отправка ревьюеру → проверка → возвращение работы и доработка с комментариями.
* Кроме теории и индивидуальных заданий, на курсе вас ждут воркшопы. Это групповые встречи примерно на 2 часа. Всего вас ждёт 8 воркшопов на протяжении обучения. 
	* Два формата воркшопов:
		- **Воркшопы в формате Q&A — вопрос/ответ** Вы ==сможете заранее отправить свои вопросы по изученным материалам наставнику==
		- **Воркшопы в формате SDI — System Design Interview** Помогут подготовиться к реальным собеседованиям с помощью решения комплексных задач в условиях ограниченного времени — как на настоящем интервью.
- Комьюнити Практикума базируется в мессенджере «Пачка».
**роли сопровождения**
* **Куратор** отвечает за организационные вопросы.
* **Наставник** — ваш проводник по практической работе.
* **Ревьюер** — опытный специалист в архитектурных решениях, который будет проверять ваши проектные работы.
* Служба поддержки — 24/7

# почитай контент

### введение в архитектуру
- Статья "[Are you solving the right problem?](https://hbr.org/2012/09/are-you-solving-the-right-problem)” на HBR учит правильно формулировать проблемы перед их решением.
- Отрывок из книги [“Вы, конечно, шутите, мистер Фейнман!”](https://www.asc.ohio-state.edu/kilcup.1//262/feynman.html) — Ричард Фейнман о важности игры и любопытства в его работе
- [“How to Ship projects at big tech companies”](https://www.seangoedecke.com/how-to-ship/) — напоминание о том, почему проект считается запущенным только тогда, когда руководство компании считает, что он был запущен
- “[50 лаконичных правил инженера Джима Акинса о проектировании космических аппаратов](https://spacecraft.ssl.umd.edu/akins_laws.html)” — классный свод принципов, актуальных в любой инженерной работе
- Статья в журнале Nature [Adding is favoured over subtracting in problem solving](https://www.nature.com/articles/d41586-021-00592-0).

### документирование
- https://github.com/plantuml-stdlib/C4-PlantUML шамблон c4-plantUML
- https://plantuml.com/ [[PlantUML]]

### [[микрофронтенды]]
- [Статья Майкла Гирса, автора книги «Микрофронтенды в действии»](https://micro-frontends.org/).
- [Пример онлайн-магазина — идеи из книги со ссылками на GitHub](https://the-tractor.store/).

### [[Docker]]
- [Здесь можно посмотреть все команды Docker.](https://docs.docker.com/reference/cli/docker/)
- [Это полный список инструкций Dockerfile.](https://docs.docker.com/reference/dockerfile/)
- [Это официальные рекомендации, как сделать Dockerfile легче.](https://docs.docker.com/build/building/best-practices/)

### [[Anti-Corruption Layer]]
- [Статья про Facade на сайте Refactoring.Guru с примерами кода.](https://refactoring.guru/ru/design-patterns/facade)
- [Статья про Adapter там же.](https://refactoring.guru/ru/design-patterns/adapter)

### [[CQRS]]
* [Command Query Responsibility Segregation](https://microservices.io/patterns/data/cqrs.html
* [A case for CQRS: How CQRS helps a SaaS product process thousands of images asynchronously](https://kreuzwerker.de/en/post/a-case-for-cqrs)
### [[Saga]]
* [Saga и Event Sourcing с Axon. Первое знакомство](https://habr.com/ru/articles/744460/)
* [SAGA на golang](https://habr.com/ru/companies/karuna/articles/582808/)
* [Паттерн Saga в микросервисной архитектуре](https://habr.com/ru/companies/otus/articles/751134/)
* [Pattern: Saga](https://microservices.io/patterns/data/saga.html)

### [[API Gateway]]
* [Разработка архитектуры для чайников. Часть 2](https://habr.com/ru/articles/658151/)
* [Простой API gateway на базе PHP и Lumen](https://habr.com/ru/articles/315128/)
* [Use API gateways in microservices](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/gateway?source=post_page-----facba5d0ae3b--------------------------------)
* [Building an API Gateway to Get Out of the Monoliths](https://www.digitalocean.com/community/tech-talks/building-an-api-gateway-to-get-out-of-the-monoliths?source=post_page-----facba5d0ae3b--------------------------------)
* [Pattern: API Gateway](https://microservices.io/patterns/apigateway.html)

### [[Kafka]]
* [Kafka Downloads](https://kafka.apache.org/downloads)
* [Kafka 3.8 Documentation](https://kafka.apache.org/documentation/)

### [[Helm]]-чарты
- [Документация про чарты в Helm](https://helm.sh/docs/topics/charts/).
- [Документация про файл конфигурации](https://www.notion.so/bcaa6942ac814246b74605bdf32fcad7?pvs=21).
- [Раздел документации о директории templates](https://www.notion.so/4-Helm-Kubernetes-ff3c9e1e45b34f858262a12255dcb0eb?pvs=21).
- [Раздел документации про директорию для зависимых чартов](https://helm.sh/docs/topics/chart_repository/).
- [Раздел документации о файле requirements.yaml](https://helm.sh/docs/faq/changes_since_helm2/#consolidation-of-requirementsyaml-into-chartyaml).

### [[Service Mesh]]:
- [Большой обзор Service Mesh: часть первая.](https://habr.com/ru/companies/oleg-bunin/articles/719394/)
- [Большой обзор Service Mesh: часть вторая.](https://habr.com/ru/companies/oleg-bunin/articles/723092/)

### [[Istio]]
- [Архитектура Istio](https://istio.io/latest/docs/ops/deployment/architecture/)
- [Jaeger](https://www.jaegertracing.io/)

### [[цифровая трансформация]]
- [Официальный сайт инструмента Business Model Canvas](https://www.strategyzer.com/library/the-business-model-canvas)
- [Примеры цифровой трансформации](https://www.plerdy.com/ru/blog/digital-transformation/)
- [Статья про построение Business Model Canvas](https://agilemasters.ru/2023/06/09/shablon-biznes-modeli-business-model-canvas/)
- [Статья на сайте «Ростелеком. Ведомости» о цифровой трансформации бизнеса](https://rostelecom.vedomosti.ru/) рассматривает, почему компании увеличивают инвестиции в цифровизацию даже в условиях кризиса.
- [Статья об инновациях в бизнес-моделях и использовании Business Model Canvas](https://www.strategyzer.com/business-models-the-toolkit-to-design-a-disruptive-company)
- [Разборы бизнес-кейсов](https://www.strategyzer.com/library/essential-strategyzer-video-cases-to-help-you-with-business-model-innovation)
- [Статья про потоки создания ценности в компании](https://dialog.guide/potoki-sozdaniia-tsiennosti/)
- [Описание Business Capabilities в стандарте TOGAF](https://pubs.opengroup.org/togaf-standard/business-architecture/business-capabilities.html#_Toc95135878) рассказывает о том, что это, как строить и как связано с организационной структурой компании и потоками создания ценности.
- [Статья на Хабре про корпоративную архитектуру на базе TOGAF](https://habr.com/ru/companies/otus/articles/756986/)
- [Обзор различных фреймворков корпоративной архитектуры (enterprise architecture)](https://www.leanix.net/en/wiki/ea/enterprise-architecture-frameworks)
- [Материал о концепции бизнес-архитектуры, её значении и принципах построения](https://dasreda.ru/media/for-managers/biznes-arhitektura/)
- [Статья-навигатор по Scaled Agile Framework](https://scrumtrek.ru/blog/enterprise-agility/8180/safe-russia/)
- [Статья о методах оптимизации организационной структуры с использованием Scaled Agile Framework](https://scrumtrek.ru/blog/enterprise-agility/2208/bezzhalostnoe-nepreryvnoe-uluchshenie-orgstruktury-po-safe/)
- [Статья о бизнес-архитектуре](https://habr.com/ru/companies/vsk_insurance/articles/713668/)
- [Статья с последовательным описанием IT-ландшафта](https://habr.com/ru/articles/745348/)
- [Основные типы IT-систем](https://www.chelidze-d.com/post/it-systems#viewer-98pav)
- [Статья про Data Warehouse](https://www.oracle.com/cis/database/what-is-a-data-warehouse/)
- [Database, Data Warehouse и Data Lake: что это и когда следует использовать каждое?](https://habr.com/ru/companies/smartup_tech/articles/807379/) — статья на Хабре про разные виды работы с данными в BI.
- [Официальный сайт OrbusSoftware, описывающий диаграмму интеграции приложений](https://www.orbussoftware.com/resources/blog/detail/landscape-diagrams-pt6-application-communication-diagram)
- [Статья о диаграммах коммуникации в UML](https://www.geeksforgeeks.org/communication-diagram-unified-modeling-languageuml/) поможет понять, как визуализировать взаимодействие объектов в системе через обмен сообщениями.
- [Статья на Хабре о главном технологическом тренде — Platform Engineering](https://habr.com/ru/companies/vk/articles/792368/)
- - [Статья о роадмапе цифровой трансформации](https://productschool.com/blog/digital-transformation/how-to-create-a-digital-transformation-roadmap)
- [Статья о продуктовых роадмапах](http://www.productplan.com/learn/example-roadmaps-for-product-managers)
- [Статья о проектах и портфельном управлении](https://5pmbok.blogspot.com/2017/09/blog-post_25.html)
- [Статья о развитии продукта проектами](https://blog.bitobe.ru/article/raznica-mezhdu-proektom-i-produktom/)
- [Статья о Lean-модели](https://habr.com/ru/companies/makeright/articles/299560/) и [статья о Lean Canvas](https://scrumtrek.ru/blog/product-management/9113/lean-canvas-chto-eto/)
- [Статья о SAFe](https://kaiten.ru/blog/safe/)
- [Статья о метриках CAPEX и OPEX](https://bcs-express.ru/novosti-i-analitika/chto-takoe-capex-i-pochemu-on-vazhen-dlia-investora)
- [Статья о метриках MAU, WAU и DAU](https://www.unisender.com/ru/glossary/mau-dau-ili-osnovnye-metriki-mobilnyh-prilozhenij/#anchor-1)
- [Статья о метриках выручки в продуктах](https://gopractice.ru/product/arpu_vs_ltv/)
- [Статья о CustDev](https://practicum.yandex.ru/blog/chto-takoe-custdev/)
- [Статья о CJM](https://practicum.yandex.ru/blog/customer-journey-map/)

### [[управление изменениями]]
- Материал в блоге Практикума о том, [кто такие стейкхолдеры и как выгодно взаимодействовать с ними.](https://practicum.yandex.ru/blog/kto-takie-steykkholdery/)
- Развёрнутый разбор требований [FURPS+ c дополнительными примерами.](https://sysana.wordpress.com/2010/09/16/furps/)
- Исчерпывающая статья в блоге Практикума [с примерами пользовательских сценариев.](https://practicum.yandex.ru/blog/chto-takoe-use-case-kak-ih-napisat/)
- Про [agile-подход при построении пользовательских сценариев.](https://scrumtrek.ru/blog/product-management/3498/user-story-mapping-guide/)
- Англоязычный материал о том, [что такое основные и неосновные потоки в Use Cases.](https://www.batimes.com/articles/use-case-goals-scenarios-and-flows/)
- Про нотации бизнес-процессов:
    - [в блоге Практикума;](https://practicum.yandex.ru/blog/notacii-modelirovaniya-biznes-processov/)
    - [отдельно про EPC;](https://www.businessstudio.ru/help/docs/current/doku.php/ru/csdesign/bpmodeling/epc_notation)
    - [отдельно про Idef.](https://www.comindware.ru/blog/азы-моделирования-в-idef0/)
- [Официальный ресурс, посвящённый концепции Architecture Decision Records (ADR)](https://adr.github.io/).
- [Реестр архитектурных решений](https://pragmatic-km.guide/practices/knowledge-registration/registration/architecture.html).
- [Статья о дизайн-мышлении, о том, как задавать правильные вопросы](https://tilda.education/articles-design-thinking-great-questions).
- [Статья об этапах дизайн-мышления](https://blog.ikraikra.ru/chto-takoe-dizajn-myshlenie/).
- [Репозиторий на github с примерами ADR](https://github.com/joelparkerhenderson/architecture-decision-record).
- [Репозиторий с макросами, стереотипами и другими средствами для создания диаграмм C4 с использованием PlantUML](https://github.com/plantuml-stdlib/C4-PlantUML).
- [Репозиторий на github с шаблонами MADR](https://github.com/adr/madr).
- [Статья на Хабре про архитектурный комитет и его работу](https://habr.com/ru/companies/simbirsoft/articles/506154/).
### [[Redis]]
* [Подробная статья на Хабре с архитектурой Redis и принципами его работы](https://habr.com/ru/companies/wunderfund/articles/685894/)
### [[sharding]]
* https://habr.com/ru/companies/oleg-bunin/articles/313366/
* [Готовая реализация примера шардирования Mongo с использованием docker-compose на 15 инстансов.](https://git.jl-k.com/minhhungit/mongodb-cluster-docker-compose)
### [[application scaling]]
* [Подробная статья про балансировку и проксирование на Хабре](https://habr.com/ru/companies/vk/articles/347026/)
* [Статья про балансировщики в Yandex Cloud и их выбор в случае использования Облака](https://education.yandex.ru/knowledge/balansirovshchiki-yandex-cloud-kakoi-vibrat)
* [Статья на английском про подходы stateful и stateless и их реализацию](https://www.freecodecamp.org/news/stateful-vs-stateless-architectures-explained/)
### [[Leaderless-архитектура]]
- [Хорошая документация архитектуры leaderless репликации в ScyllaDB](https://opensource.docs.scylladb.com/stable/architecture/index.html)
- [«Высоконагруженные приложения. Программирование, масштабирование, поддержка» Мартина Клеппмана (глава 5 «Репликация», раздел 5.4 «Репликация без ведущего узла)](https://www.piter.com/collection/all/product/vysokonagruzhennye-prilozheniya-programmirovanie-masshtabirovanie-podderzhka-2)
- [Невредные советы по Cassandra — как избежать ошибок?](https://habr.com/ru/companies/digitalleague/articles/738908/)
- [Как Netflix использует Cassandra, чтобы эффективно обрабатывать и хранить высокие QPS с низкой задержкой](https://netflixtechblog.com/introducing-netflix-timeseries-data-abstraction-layer-31552f6326f8)
### распределенное кэширование
* [Статья на Хабре с опытом реализации кеширования на базе другой технологии — Hazelcast](https://habr.com/ru/companies/yoomoney/articles/332462/)
* [Подробное описание устройства распределённого кеша](https://telegra.ph/Raspredelennyj-kehsh-02-13)
* [Сравнение скорости разных дисков.](https://medium.com/ctrlaltdelux/i-o-wars-ram-disk-vs-hdd-and-ssd-aa6502a26ed3) 
	* Статья 2018 года, но с тех пор мало что изменилось.
### [[CDN]]
* [Статья о поиске авторитетного сервера для имён для DNS-зоны](https://www.baeldung.com/cs/dns-authoritative-server-ip)
* [Документация про бакеты в Yandex Cloud](https://yandex.cloud/ru/docs/storage/concepts/bucket)
* [Документация про хостинг статических сайтов в Yandex](https://yandex.cloud/ru/docs/storage/concepts/hosting)
* [L7-балансировщик нагрузки из Yandex Application Load Balancer](https://yandex.cloud/ru/docs/application-load-balancer/concepts/application-load-balancer)
* [Документация о CDN в Yandex Cloud](https://yandex.cloud/ru/docs/glossary/cdn)
* [Статья на Хабре с подробностями об устройстве CDN](https://habr.com/ru/companies/ruvds/articles/503800/)

### [[RAG]]
* [Осваиваем RAG: 4 показателя, чтобы повысить производительность](https://www.galileo.ai/blog/mastering-rag-improve-performance-with-4-powerful-metrics)
* [Осваиваем RAG: передовые технологии разбивки на чанки для приложений LLM](https://www.galileo.ai/blog/mastering-rag-advanced-chunking-techniques-for-llm-applications)
* [Long chain Chat with Your Data](https://learn.deeplearning.ai/courses/langchain-chat-with-your-data/lesson/3/document-splitting)
* [Как извлекать данные каждого документа с помощью нескольких векторов](https://python.langchain.com/docs/how_to/multi_vector/)
* [Обзор техник RAG](https://habr.com/ru/articles/904032/)
* [Обзор RAG для больших языковых моделей](https://arxiv.org/abs/2312.10997)
* [Осваиваем RAG: 8 сценариев, которые нужно оценить, перед началом продакшена](https://www.galileo.ai/blog/mastering-rag-8-scenarios-to-test-before-going-to-production)
* [Создание синтетических данных для RAG всего за 10 долларов](https://www.galileo.ai/blog/synthetic-data-rag)
* [Осваиваем RAG: выбираем идеальную векторную базу данных](https://www.galileo.ai/blog/mastering-rag-choosing-the-perfect-vector-database)
* [Осваиваем RAG: как выбрать векторную модель](https://www.galileo.ai/blog/mastering-rag-how-to-select-an-embedding-model)
* [Осваиваем RAG: как выбрать модель повторного ранжирования](https://www.galileo.ai/blog/mastering-rag-how-to-select-a-reranking-model)