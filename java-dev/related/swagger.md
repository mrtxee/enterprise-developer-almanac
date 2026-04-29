Swagger – это спецификация и набор инструментов для проектирования, документирования, тестирования и развертывания RESTful API, позволяющий разработчикам и машинам легко понимать возможности и структуру API

**Swagger** позволяет разработчикам описывать структуру своих API и генерировать интерактивную документацию, клиентские библиотеки и серверные модули для реализации API на разных языках.
Swagger предоставляет спецификацию для ==документирования [[REST]] API==, которая называется ==[[REST#OpenAPI]] OpenAPI Specification —== **==OAS==**. Эта спецификация предоставляет четкий и лаконичный способ описания эндпойнтов, их параметров, моделей запросов и ответов и других аспектов API.
Также стоит упомянуть, что Swagger позволяет сгенерировать  
непосредственно код клиента или сервера по имеющейся OAS, для этого  
нужен генератор кода  
[Swagger-Codegen](https://swagger.io/tools/swagger-codegen/).
# QuickStart
```XML
<dependency>
    <groupId>org.springdoc</groupId>
	    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.2</version>
</dependency>
```
После добавления зависимостей у нас уже есть документация, доступная по ссылке:
- [http://localhost:8080/swagger-ui](http://localhost:8080/swagger-ui.html) — гуи для тестов
- [http://localhost:8080/v3/api-docs](http://localhost:8080/v3/api-docs) — OAS документация
Далее >>

> [!info] Документирование SpringBoot API с помощью Swagger  
> Веб-приложение содержит API для работы.  
> [https://struchkov.dev/blog/ru/api-swagger/](https://struchkov.dev/blog/ru/api-swagger/)