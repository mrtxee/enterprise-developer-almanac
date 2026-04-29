---
aliases:
  - probes
  - healthcheck
---

pom.xml
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-actuator</artifactId>  
</dependency>
```

application.yml
```yml
management:  
  endpoints:  
    web:  
      exposure:  
         include: health,info,env,beans,metrics,loggers,scheduledtasks,httptrace  
         # include: "*"  # ВКЛЮЧИТЬ ВСЁ (ОПАСНО В PROD!)  
  endpoint:  
    health:  
      show-details: never  
      probes:  
        enabled: true
```

toc: `http://localhost:8080/actuator`
ref: [[Kubernetes probes]]