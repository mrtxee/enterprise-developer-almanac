---
aliases:
  - OAS
  - OpenAPI
  - OpenAPI Specification
  - REST
  - REST API
  - RESTful API
  - Swagger
  - Swagger Codegen
  - Swagger UI
  - Swagger-Codegen
  - springdoc
  - springdoc-openapi
  - springdoc-openapi-starter-webmvc-ui
  - Сваггер
  - Спецификация OpenAPI
---

## Swagger

Swagger — это спецификация и набор инструментов для проектирования, документирования, тестирования и развертывания RESTful API, позволяющий разработчикам и машинам легко понимать возможности и структуру API.

Swagger позволяет разработчикам описывать структуру своих API и генерировать интерактивную документацию, клиентские библиотеки и серверные модули для реализации API на разных языках.

Swagger предоставляет спецификацию для документирования [[REST]] API, которая называется [[REST#OpenAPI|OpenAPI Specification]] (**OAS**). Эта спецификация дает четкий и лаконичный способ описания эндпойнтов, их параметров, моделей запросов и ответов и других аспектов API.

Swagger позволяет сгенерировать код клиента или сервера по имеющейся OAS — для этого нужен генератор кода Swagger-Codegen.

## Быстрый старт

**Подключение зависимости (springdoc-openapi)**

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.2</version>
</dependency>
```

После добавления зависимости документация доступна по ссылкам:

- `http://localhost:8080/swagger-ui.html` — GUI для тестов
- `http://localhost:8080/v3/api-docs` — OAS документация
