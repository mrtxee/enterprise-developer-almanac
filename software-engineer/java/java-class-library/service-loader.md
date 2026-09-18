---
aliases:
  - Dependency inversion
  - Java Platform Module System
  - java.util.ServiceLoader
  - JDBC
  - JPMS
  - Loose coupling
  - META-INF/services
  - Service Provider Interface
  - ServiceLoader
  - SLF4J
  - SPI
  - Инверсия зависимостей
  - Слабая связность
---
## java.util.ServiceLoader
> `java.util.ServiceLoader`

**ServiceLoader** — это встроенный в Java механизм для автоматического поиска и загрузки реализаций заданного интерфейса (или абстрактного класса). Он является фундаментальной основой паттерна **SPI (Service Provider Interface)**.

Простыми словами: это способ сказать Java: «Найди мне все реализации этого интерфейса, которые есть в classpath (или в модулях), и дай мне их экземпляры».

## Как работает ServiceLoader

**Определение интерфейса сервиса**

```java
package com.example;

public interface GreetingService {
  String greet();
}
```

**Реализация и конфигурация**

В библиотеке, которая предоставляет реализацию, создаётся специальный текстовый файл:

- **Путь к файлу**: `META-INF/services/com.example.GreetingService` (полное имя интерфейса).
- **Содержимое файла**: полное имя класса реализации, например `com.example.impl.EnglishGreeting`.

**Загрузка реализаций**

```java
ServiceLoader<GreetingService> loader = ServiceLoader.load(GreetingService.class);
for (GreetingService service : loader) {
  System.out.println(service.greet());
}
```

## Применение на практике

- **JDBC**: при вызове `Class.forName("com.mysql.cj.jdbc.Driver")` (или при автозагрузке в Java 6+) `DriverManager` использует `ServiceLoader` для поиска всех доступных драйверов в classpath.
- **Логирование**: библиотека [[Logger|SLF4J]] использует SPI, чтобы найти и подключить конкретную реализацию (Logback, Log4j2) без жёсткой привязки к ней в коде.
- **Плагины**: создание расширяемых приложений, куда можно просто добавить [[jdk-jls-jni|JAR]]-файл с новой реализацией интерфейса, и приложение подхватит её автоматически.

## Особенности и нюансы

1. **Ленивая загрузка (Lazy Loading)** — `ServiceLoader` не создаёт экземпляры всех классов сразу, они создаются по одному по мере итерации (`iterator()` или `stream()`).
2. **Отсутствие приоритетов** — по умолчанию `ServiceLoader` не гарантирует порядок загрузки и не имеет встроенного механизма выбора «лучшей» реализации (в Java 9+ удобнее фильтровать через `Stream` API).
3. **Java 9+ (модульная система [[module-info|JPMS]])** — вместо файлов в `META-INF/services` модуль-провайдер объявляет `provides com.example.GreetingService with com.example.impl.EnglishGreeting;` в `module-info.java`, а модуль-потребитель — `uses com.example.GreetingService;`. Загрузка происходит через `ServiceLoader.load(GreetingService.class, ModuleLayer.boot())`.

**Краткий итог**

Используйте `ServiceLoader`, когда нужно реализовать **слабую связность (loose coupling)** и дать возможность подключать разные реализации одного и того же интерфейса без изменения кода основного приложения (принцип [[clean-code|инверсии зависимостей]]).

---

## Сбор всех реализаций

Пример класса, собирающего все реализации `AbstractPackageRepository` через SPI:

```java
public class PackageProcessor {

  private final Map<String, AbstractPackageRepository> repositories;

  // Конструктор по умолчанию. Использует Java SPI (ServiceLoader) для автоматического
  // обнаружения всех реализаций. Это гарантирует OCP: при добавлении нового формата
  // не потребуется менять ни одной строки в этом классе.
  public PackageProcessor() {
    Map<String, AbstractPackageRepository> map = new HashMap<>();
    ServiceLoader<AbstractPackageRepository> loader =
        ServiceLoader.load(AbstractPackageRepository.class);

    for (AbstractPackageRepository repository : loader) {
      map.put(repository.getType(), repository);
    }

    // Создаём неизменяемую копию для безопасности
    this.repositories = Map.copyOf(map);
  }

  // Альтернативный конструктор. Полезен для юнит-тестов или если нужно собирать реестр
  // стратегий вручную в отдельном конфигурационном классе.
  public PackageProcessor(List<AbstractPackageRepository> repositoryList) {
    this.repositories = repositoryList.stream()
        .collect(Collectors.toMap(
            AbstractPackageRepository::getType,
            Function.identity(),
            (existing, replacement) -> existing, // защита от дубликатов
            HashMap::new
        ));
  }
}
```
