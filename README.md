# enterprise-developer-almanac
> альманах корпоративного разработчика


## toc enterprise-developer-almanac

```plain
Структура папок тома: 744A-2683
¦   ¦           
¦   L---ti
¦       L   myTI.md
¦           
+---java-developer
¦   ¦   Singleton через Enum.md
¦   ¦   
¦   +---core
¦   ¦       Class.md
¦   ¦       Classloader.md
¦   ¦       Exception.md
¦   ¦       Generic.md
¦   ¦       immutable.md
¦   ¦       Instant.md
¦   ¦       Java Core.md
¦   ¦       Java Dynamic Binding.md
¦   ¦       Java Persistence API.md
¦   ¦       Java Reference.md
¦   ¦       JDK JVM JLS JNI.md
¦   ¦       JVM.md
¦   ¦       Object.md
¦   ¦       package-info.md
¦   ¦       record type.md
¦   ¦       Thread.md
¦   ¦       
¦   +---faq
¦   ¦       AST.md
¦   ¦       boilerplate.md
¦   ¦       faq.md
¦   ¦       Future.md
¦   ¦       Java faq.md
¦   ¦       javap class.md
¦   ¦       spaghetti.md
¦   ¦       хранение паролей.md
¦   ¦       
¦   +---JCL
¦   ¦       java.io.Serializable.md
¦   ¦       java.lang.Math.md
¦   ¦       java.lang.reflect.md
¦   ¦       java.util.Collection.md
¦   ¦       java.util.concurrent.md
¦   ¦       java.util.function.md
¦   ¦       java.util.logging.md
¦   ¦       java.util.Optional.md
¦   ¦       java.util.stream.md
¦   ¦       javax.servlet.md
¦   ¦       
¦   +---JMM
¦   ¦       Cache Line.md
¦   ¦       Happens-Before.md
¦   ¦       JMH Java Microbenchmark Harness.md
¦   ¦       JMM deeper.md
¦   ¦       JMM GCs.md
¦   ¦       JMM Reordering.md
¦   ¦       JMM.md
¦   ¦       Memory Barriers.md
¦   ¦       memory leak.md
¦   ¦       Performance Penalty.md
¦   ¦       
¦   +---related
¦   ¦       gson.md
¦   ¦       hibernate.md
¦   ¦       javax vs jakarta.md
¦   ¦       junit.md
¦   ¦       keybindings.md
¦   ¦       liquibase.md
¦   ¦       lombok под капотом.md
¦   ¦       lombok.md
¦   ¦       swagger.md
¦   ¦       
¦   L---spring
¦           Spring actuator probes.md
¦           Spring Assert.md
¦           Spring beans.md
¦           Spring boot application yaml.md
¦           Spring boot threads.md
¦           Spring paths.md
¦           Spring PostProcessor.md
¦           Spring ResponseEntity.md
¦           Spring retry.md
¦           Spring Scheduled.md
¦           Spring vars.md
¦           Spring WebMvcConfigurer.md
¦           Spring.md
¦           SpringValidatorAdapter.md
¦           
+---software-architect
¦   +---ai
¦   ¦   ¦   ai.md
¦   ¦   ¦   DM.md
¦   ¦   ¦   LLM.md
¦   ¦   ¦   ML.md
¦   ¦   ¦   MLM.md
¦   ¦   ¦   NLP.md
¦   ¦   ¦   NLU.md
¦   ¦   ¦   Tokenizators.md
¦   ¦   ¦   Vector Index.md
¦   ¦   ¦   Vector Store.md
¦   ¦   ¦   
¦   ¦   +---алгоритмы
¦   ¦   ¦       Embedding-models.md
¦   ¦   ¦       NLP Encoding.md
¦   ¦   ¦       RAGAS.md
¦   ¦   ¦       Word Embedding.md
¦   ¦   ¦       
¦   ¦   +---нотации
¦   ¦   ¦       CRISP-DM.md
¦   ¦   ¦       
¦   ¦   +---паттерны
¦   ¦   ¦       Fine-tuning.md
¦   ¦   ¦       ModelOps.md
¦   ¦   ¦       prompt.md
¦   ¦   ¦       RAG.md
¦   ¦   ¦       
¦   ¦   L---технологии
¦   ¦           Airflow.md
¦   ¦           Camunda.md
¦   ¦           DAG.md
¦   ¦           Hugging Face.md
¦   ¦           LangChain.md
¦   ¦           MLflow.md
¦   ¦           Rasa.md
¦   ¦           
¦   +---authentification
¦   ¦       AD.md
¦   ¦       authentication.md
¦   ¦       Federated Identity.md
¦   ¦       Kerberos.md
¦   ¦       Keycloak.md
¦   ¦       LDAP.md
¦   ¦       MS-RPC.md
¦   ¦       NTLM.md
¦   ¦       OAuth.md
¦   ¦       OIDC.md
¦   ¦       OLTP.md
¦   ¦       PKCE.md
¦   ¦       PWA.md
¦   ¦       SAML.md
¦   ¦       SPA.md
¦   ¦       SSO.md
¦   ¦       
¦   +---cloud
¦   ¦       Cloud Architecture.md
¦   ¦       Cloud Computing.md
¦   ¦       Cloud Native.md
¦   ¦       cloud.md
¦   ¦       Deployment Model.md
¦   ¦       
¦   +---data
¦   ¦   +---data architecture
¦   ¦   ¦       Big Data.md
¦   ¦   ¦       Data Architecture.md
¦   ¦   ¦       Data Catalog.md
¦   ¦   ¦       Data Mesh.md
¦   ¦   ¦       DataHub.md
¦   ¦   ¦       Distributed Data Architectures.md
¦   ¦   ¦       Dremio.md
¦   ¦   ¦       HiveQL.md
¦   ¦   ¦       Hudi.md
¦   ¦   ¦       Iceberg.md
¦   ¦   ¦       JDBC Catalog.md
¦   ¦   ¦       MapReduce.md
¦   ¦   ¦       MinIO.md
¦   ¦   ¦       NVMe.md
¦   ¦   ¦       Open Table Formats.md
¦   ¦   ¦       Raw Data Store.md
¦   ¦   ¦       S3.md
¦   ¦   ¦       Storage Models.md
¦   ¦   ¦       Storage System.md
¦   ¦   ¦       
¦   ¦   +---data management
¦   ¦   ¦       COM.md
¦   ¦   ¦       Data Flow Diagram.md
¦   ¦   ¦       Data Flow Management.md
¦   ¦   ¦       Data Lineage.md
¦   ¦   ¦       data management.md
¦   ¦   ¦       DPIA.md
¦   ¦   ¦       Glue.md
¦   ¦   ¦       Hive Metastore.md
¦   ¦   ¦       Nessie.md
¦   ¦   ¦       NiFi.md
¦   ¦   ¦       PbD.md
¦   ¦   ¦       PET.md
¦   ¦   ¦       tagging.md
¦   ¦   ¦       TEE.md
¦   ¦   ¦       миграция архитектуры.md
¦   ¦   ¦       обфускация.md
¦   ¦   ¦       Слои данных.md
¦   ¦   ¦       
¦   ¦   +---data pocessing
¦   ¦   ¦       Columnar Databases.md
¦   ¦   ¦       Data Lake.md
¦   ¦   ¦       Data Lakehouse.md
¦   ¦   ¦       Data Mart.md
¦   ¦   ¦       data pocessing.md
¦   ¦   ¦       Data Vault.md
¦   ¦   ¦       Data Warehouse.md
¦   ¦   ¦       Delta Lake.md
¦   ¦   ¦       DWH модели.md
¦   ¦   ¦       ELT.md
¦   ¦   ¦       ETL.md
¦   ¦   ¦       Parquet.md
¦   ¦   ¦       Semistructured Data.md
¦   ¦   ¦       ПДн.md
¦   ¦   ¦       
¦   ¦   +---databases
¦   ¦   ¦   ¦   BASE.md
¦   ¦   ¦   ¦   CAP-теорема.md
¦   ¦   ¦   ¦   databases.md
¦   ¦   ¦   ¦   Elasticsearch.md
¦   ¦   ¦   ¦   graph database.md
¦   ¦   ¦   ¦   MongoDB.md
¦   ¦   ¦   ¦   MPP database.md
¦   ¦   ¦   ¦   Prometheus.md
¦   ¦   ¦   ¦   Redis.md
¦   ¦   ¦   ¦   Replica Set.md
¦   ¦   ¦   ¦   TSDB.md
¦   ¦   ¦   ¦   Распределенные информационные системы.md
¦   ¦   ¦   ¦   
¦   ¦   ¦   L---postgres
¦   ¦   ¦           index.md
¦   ¦   ¦           PostgreSQL.md
¦   ¦   ¦           sequence.md
¦   ¦   ¦           
¦   ¦   L---rdbms
¦   ¦           ACID.md
¦   ¦           CDC.md
¦   ¦           database normalization.md
¦   ¦           DSL.md
¦   ¦           fsync.md
¦   ¦           MVCC.md
¦   ¦           rdbms.md
¦   ¦           SQL.md
¦   ¦           Transaction isolation.md
¦   ¦           Transaction.md
¦   ¦           
¦   +---highload
¦   ¦   ¦   Adaptive Concurrency.md
¦   ¦   ¦   Back-to-Back.md
¦   ¦   ¦   Business metrics.md
¦   ¦   ¦   Circuit Breaker.md
¦   ¦   ¦   Client pull.md
¦   ¦   ¦   CRC-ошибки.md
¦   ¦   ¦   ELK.md
¦   ¦   ¦   Grok.md
¦   ¦   ¦   Hibernate Cache.md
¦   ¦   ¦   highload.md
¦   ¦   ¦   Jaeger.md
¦   ¦   ¦   Kibana.md
¦   ¦   ¦   Load Shedding.md
¦   ¦   ¦   Locust.md
¦   ¦   ¦   Logging.md
¦   ¦   ¦   Logstash.md
¦   ¦   ¦   Metrics.md
¦   ¦   ¦   Monitoring.md
¦   ¦   ¦   Observability.md
¦   ¦   ¦   OpenTelemetry.md
¦   ¦   ¦   Request Collapsing.md
¦   ¦   ¦   Resilience Patterns.md
¦   ¦   ¦   RFC 5424.md
¦   ¦   ¦   Server push.md
¦   ¦   ¦   SRE.md
¦   ¦   ¦   Stand-in.md
¦   ¦   ¦   State Machine.md
¦   ¦   ¦   syslog.md
¦   ¦   ¦   Tracing.md
¦   ¦   ¦   
¦   ¦   +---caching
¦   ¦   ¦       cache patterns.md
¦   ¦   ¦       caching.md
¦   ¦   ¦       client caching.md
¦   ¦   ¦       
¦   ¦   L---highload в realtime-среде
¦   ¦           AB testing.md
¦   ¦           Axon Framework.md
¦   ¦           Bulkhead.md
¦   ¦           Event Storming.md
¦   ¦           Event-Driven Architecture.md
¦   ¦           EventStoreDB.md
¦   ¦           Failover Strategy.md
¦   ¦           highload в realtime-среде.md
¦   ¦           Performance testing.md
¦   ¦           Rate Limiting.md
¦   ¦           reliability metrics.md
¦   ¦           Snapshotting.md
¦   ¦           stream processing.md
¦   ¦           Transaction log tailing.md
¦   ¦           Transactional outbox.md
¦   ¦           
¦   +---security
¦   ¦       adaptive auth.md
¦   ¦       asymmetric cryptography.md
¦   ¦       blockchain.md
¦   ¦       CIA Triad.md
¦   ¦       differential privacy.md
¦   ¦       DLP.md
¦   ¦       DRP.md
¦   ¦       E2EE.md
¦   ¦       encryption.md
¦   ¦       GDPR.md
¦   ¦       HMAC.md
¦   ¦       HSM.md
¦   ¦       IAM.md
¦   ¦       KMS.md
¦   ¦       KYC.md
¦   ¦       OTP.md
¦   ¦       PGP.md
¦   ¦       Scope-based authorization.md
¦   ¦       security contexts.md
¦   ¦       SIEM.md
¦   ¦       Tokenization.md
¦   ¦       zero trust.md
¦   ¦       киберугрозы.md
¦   ¦       методы авторизации.md
¦   ¦       угрозы иб.md
¦   ¦       
¦   +---system design
¦   ¦       Base62.md
¦   ¦       Branch mispredict.md
¦   ¦       QPS.md
¦   ¦       Round-Trip Time.md
¦   ¦       system design 2.md
¦   ¦       system design.md
¦   ¦       Untitled.md
¦   ¦       Zippy.md
¦   ¦       
¦   L---technology management
¦           Architectural Katas.md
¦           SAST.md
¦           SCA.md
¦           SDLC.md
¦           Tableau.md
¦           TCO.md
¦           Technical Debt.md
¦           Technology Landscape Management.md
¦           technology management.md
¦           Technology Radar.md
¦           Technology Readiness Levels.md
¦           Technology Roadmap.md
¦           TIME.md
¦           
L---software-engineer
    +---алгоритмы
    ¦       KD-Tree.md
    ¦       KNN.md
    ¦       quadtree.md
    ¦       R-Tree.md
    ¦       radix tree.md
    ¦       trie.md
    ¦       Алгоритмы.md
    ¦       Метрика триграмм.md
    ¦       Фильтр Блума.md
    ¦       Хеш-таблица.md
    ¦       Хеш-функция.md
    ¦       
    +---матан
    ¦       Геометрия.md
    ¦       декартово произведение.md
    ¦       КИС.md
    ¦       Комбинаторика.md
    ¦       Многопоточность.md
    ¦       Память ЭВМ.md
    ¦       Принципы юнит-тестирования.md
    ¦       Формальная логика.md
    ¦       
    +---нотации
    ¦       4C.md
    ¦       abbreviations.md
    ¦       BCM.md
    ¦       BPMN.md
    ¦       COBIT.md
    ¦       Context Map.md
    ¦       EPC.md
    ¦       ER-diagram.md
    ¦       FURPSplus.md
    ¦       Gantt diagram.md
    ¦       IT Landscape Map.md
    ¦       ITIL.md
    ¦       PaC.md
    ¦       playgrnd.md
    ¦       Sankey.md
    ¦       Sequence Diagram.md
    ¦       Stakeholder Map.md
    ¦       TOGAF.md
    ¦       UML.md
    ¦       Use Case.md
    ¦       нотация архитектуры.md
    ¦       обзор нотаций.md
    ¦       приоритезация решений.md
    ¦       управление изменениями.md
    ¦       
    +---паттерны
    ¦   ¦   Pattern.md
    ¦   ¦   
    ¦   +---architecture
    ¦   ¦   ¦   Aggregate Root.md
    ¦   ¦   ¦   Clean Code.md
    ¦   ¦   ¦   Domain-Driven Design.md
    ¦   ¦   ¦   Enterprise Architecture.md
    ¦   ¦   ¦   Hexagonal Architecture.md
    ¦   ¦   ¦   Untitled.md
    ¦   ¦   ¦   цифровая трансформация.md
    ¦   ¦   ¦   
    ¦   ¦   L---paradigm
    ¦   ¦           Domain-Driven Development.md
    ¦   ¦           АОП.md
    ¦   ¦           ООП.md
    ¦   ¦           
    ¦   +---enterprise patterns
    ¦   ¦   ¦   Anti-Corruption Layer.md
    ¦   ¦   ¦   API Gateway.md
    ¦   ¦   ¦   Backend for Frontend.md
    ¦   ¦   ¦   Backpressure.md
    ¦   ¦   ¦   Blameless Culture.md
    ¦   ¦   ¦   Canary release.md
    ¦   ¦   ¦   CQRS.md
    ¦   ¦   ¦   Enterprise Integration Patterns.md
    ¦   ¦   ¦   Event Sourcing.md
    ¦   ¦   ¦   Feature Flagging.md
    ¦   ¦   ¦   Feature Toggling.md
    ¦   ¦   ¦   Fishbone Diagram.md
    ¦   ¦   ¦   Fluentd.md
    ¦   ¦   ¦   IaC.md
    ¦   ¦   ¦   Postmortem.md
    ¦   ¦   ¦   Retry Policy.md
    ¦   ¦   ¦   Service Bus.md
    ¦   ¦   ¦   Service Mesh.md
    ¦   ¦   ¦   SOLID.md
    ¦   ¦   ¦   Swiss Cheese Model.md
    ¦   ¦   ¦   
    ¦   ¦   L---PoEAA
    ¦   ¦           Active Record.md
    ¦   ¦           Data Mapper.md
    ¦   ¦           Enterprise patterns.md
    ¦   ¦           Inheritance Mappers.md
    ¦   ¦           Layered Architecture.md
    ¦   ¦           LOB.md
    ¦   ¦           Offline Concurrency.md
    ¦   ¦           Page Controller.md
    ¦   ¦           PoEAA.md
    ¦   ¦           Row Data Gateway.md
    ¦   ¦           Table Module.md
    ¦   ¦           Unit of Work.md
    ¦   ¦           
    ¦   +---general patterns
    ¦   ¦   ¦   Eventloop.md
    ¦   ¦   ¦   grasp.md
    ¦   ¦   ¦   Observable state.md
    ¦   ¦   ¦   Publish-Subscribe.md
    ¦   ¦   ¦   
    ¦   ¦   L---gof
    ¦   ¦           GoF Chain of Responsibility vs Command.md
    ¦   ¦           gof.md
    ¦   ¦           
    ¦   +---microservice
    ¦   ¦       2PC.md
    ¦   ¦       aсинхронное взаимодействие.md
    ¦   ¦       Build-Time Композиция.md
    ¦   ¦       Database Decomposition.md
    ¦   ¦       Event bus.md
    ¦   ¦       fan-out.md
    ¦   ¦       Message Broker.md
    ¦   ¦       Message Queueing.md
    ¦   ¦       microservice.md
    ¦   ¦       Orchestration.md
    ¦   ¦       Reverse proxy.md
    ¦   ¦       Saga.md
    ¦   ¦       Service Discovery.md
    ¦   ¦       Sidecar.md
    ¦   ¦       Strangler Fig.md
    ¦   ¦       микрофронтенды.md
    ¦   ¦       
    ¦   L---scaling
    ¦           Anycast.md
    ¦           application scaling.md
    ¦           BGP.md
    ¦           caching.md
    ¦           CDN.md
    ¦           Consistent Hashing.md
    ¦           data scaling.md
    ¦           DNS-based failover.md
    ¦           gateaway scaling.md
    ¦           Leaderless-архитектура.md
    ¦           Load Balancer scaling.md
    ¦           merkle tree.md
    ¦           read-replica.md
    ¦           replication.md
    ¦           sharding.md
    ¦           
    +---протоколы
    ¦       ARP.md
    ¦       DNS.md
    ¦       ICMP.md
    ¦       Internet Protocol.md
    ¦       IP TCP UDP.md
    ¦       MIME.md
    ¦       PPP.md
    ¦       socket.md
    ¦       TLS.md
    ¦       VLAN.md
    ¦       webRTC webSocket.md
    ¦       WebRTC.md
    ¦       XRay.md
    ¦       
    +---стандарты
    ¦       DevOps.md
    ¦       i18n.md
    ¦       IETF RFC.md
    ¦       IETF.md
    ¦       ISO 8601 Time format.md
    ¦       OASIS.md
    ¦       RFC 2782 DNS SRV Records.md
    ¦       RFC 3339 Date and Time on the Internet.md
    ¦       RFC 4122 UUID.md
    ¦       RFC 4648 Base16 Encodings.md
    ¦       RFC 5424 Syslog Protocol.md
    ¦       RFC 8089 The file URI Scheme.md
    ¦       RFC 9110 HTTP Caching.md
    ¦       RFC 9457 Problem Details.md
    ¦       КИС.md
    ¦       классификация алгоритмов.md
    ¦       Память ЭВМ.md
    ¦       
    L---технологии
        ¦   Agile.md
        ¦   AMQP.md
        ¦   APISIX Gateway.md
        ¦   Avro.md
        ¦   Camel.md
        ¦   CI-CD Tools.md
        ¦   CI-CD.md
        ¦   CNCF.md
        ¦   config formats.md
        ¦   CPU vs GPU.md
        ¦   Distributed Tracing.md
        ¦   EC2.md
        ¦   Envoy.md
        ¦   Ethernet frame.md
        ¦   Gradle.md
        ¦   Grafana.md
        ¦   GraphQL.md
        ¦   Greenplum.md
        ¦   Groovy.md
        ¦   Istio.md
        ¦   JWT.md
        ¦   Maven.md
        ¦   MEAN.md
        ¦   MkDocs.md
        ¦   Module Federation.md
        ¦   MTU.md
        ¦   OLAP.md
        ¦   OpenAPI.md
        ¦   OSI.md
        ¦   PCI-DSS.md
        ¦   PlantUML.md
        ¦   Protobuf.md
        ¦   RabbitMQ.md
        ¦   Reactive Streams.md
        ¦   Reactor.md
        ¦   REST.md
        ¦   RPC.md
        ¦   RqUID.md
        ¦   Selenium.md
        ¦   Service Model.md
        ¦   SLI.md
        ¦   SOAP.md
        ¦   SPA.md
        ¦   SSI.md
        ¦   subnet mask.md
        ¦   UUID.md
        ¦   WebFlux.md
        ¦   КАП.md
        ¦   
        +---cli
        ¦       Ansible.md
        ¦       bash.md
        ¦       batch where.md
        ¦       cat.md
        ¦       choco.md
        ¦       cli.md
        ¦       curl.md
        ¦       git.md
        ¦       MobaXterm.md
        ¦       nohup.md
        ¦       nslookup.md
        ¦       openssl.md
        ¦       telnet.md
        ¦       vim.md
        ¦       winget.md
        ¦       
        +---docker
        ¦       Docker practice.md
        ¦       docker-compose.md
        ¦       Docker.md
        ¦       Raft.md
        ¦       Wildfly vs Docker.md
        ¦       
        +---kafka
        ¦       kafka listeners.md
        ¦       Kafka Serde.md
        ¦       Kafka.md
        ¦       
        +---kubernetes
        ¦       Gateway API.md
        ¦       Helm.md
        ¦       k8s rbac.md
        ¦       K8s ReplicaSet.md
        ¦       k8s иерархия сущностей.md
        ¦       k8s развертывание кластера.md
        ¦       Kubectl.md
        ¦       Kubernetes pod vs node.md
        ¦       Kubernetes probes.md
        ¦       Kubernetes scaling.md
        ¦       Kubernetes манифесты.md
        ¦       Kubernetes.md
        ¦       OpenShift.md
        ¦       
        L---nginx
                Синтаксис конфигураций Nginx.md
```


----

[[batch tree]]