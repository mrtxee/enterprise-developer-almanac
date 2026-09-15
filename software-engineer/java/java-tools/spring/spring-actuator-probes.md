---
aliases:
  - Actuator
  - health
  - healthcheck
  - Kubernetes probe
  - Kubernetes probes
  - liveness
  - Management endpoints
  - probes
  - readiness
  - Spring Boot
  - Spring Boot Actuator
  - spring-boot-starter-actuator
---

## Spring Boot Actuator Probes

**Зависимость (pom.xml)**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**Настройка (application.yml)**

```yml
management:
  endpoints:
    web:
      exposure:
        include: health, info, env, beans, metrics, loggers, scheduledtasks, httptrace
        # include: "*"  # ВКЛЮЧИТЬ ВСЁ (ОПАСНО В PROD!)
  endpoint:
    health:
      show-details: never
      probes:
        enabled: true
```

**Индекс эндпоинтов:** `http://localhost:8080/actuator`

**См. также** [[Kubernetes-probe]]
