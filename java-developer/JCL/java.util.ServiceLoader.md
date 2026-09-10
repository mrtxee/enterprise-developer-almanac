---
aliases:
  - java.util.ServiceLoader
  - ServiceLoader
  - SPI
---

`java.util.ServiceLoader` — это встроенный в Java механизм для автоматического поиска и загрузки реализаций заданного интерфейса (или абстрактного класса). Он является фундаментальной основой паттерна **SPI (Service Provider Interface)** в Java.

Простыми словами: это способ сказать Java: *"Найди мне все реализации этого интерфейса, которые есть в classpath (или в модулях), и дай мне их экземпляры"*.

## Как это работает (3 простых шага)

1. **Интерфейс**: Вы определяете интерфейс сервиса.
   ```java
   package com.example;
   public interface GreetingService {
       String greet();
   }
   ```
2. **Реализация и конфигурация**: В библиотеке, которая предоставляет реализацию, создается специальный текстовый файл.
   * **Путь к файлу**: `META-INF/services/com.example.GreetingService` (полное имя интерфейса).
   * **Содержимое файла**: Полное имя класса реализации, например: `com.example.impl.EnglishGreeting`.
3. **Загрузка**: Приложение загружает реализации с помощью `ServiceLoader`.
   ```java
   ServiceLoader<GreetingService> loader = ServiceLoader.load(GreetingService.class);
   for (GreetingService service : loader) {
       System.out.println(service.greet());
   }
   ```

## Где это применяется на практике?
* **JDBC**: Когда вы вызываете `Class.forName("com.mysql.cj.jdbc.Driver")` (или просто полагаетесь на автозагрузку в Java 6+), `DriverManager` использует `ServiceLoader` для поиска всех доступных драйверов в classpath.
* **Логирование**: Библиотека SLF4J использует SPI, чтобы найти и подключить конкретную реализацию (Logback, Log4j2) без жесткой привязки к ней в коде.
* **Плагины**: Создание расширяемых приложений, куда можно просто "подкинуть" JAR-файл с новой реализацией интерфейса, и приложение подхватит её автоматически.

## Важные особенности и нюансы
1. **Ленивая загрузка (Lazy Loading)**: `ServiceLoader` не создает экземпляры всех классов сразу. Он создает их по одному по мере итерации (`iterator()` или `stream()`).
2. **Отсутствие приоритетов**: По умолчанию `ServiceLoader` не гарантирует порядок загрузки и не имеет встроенного механизма выбора "лучшей" реализации (хотя в Java 9+ через `Stream` API фильтрацию делать удобнее).
3. **Java 9+ (Модульная система JPMS)**: В современных версиях Java механизм стал строже. Вместо файлов в `META-INF/services` модуль-провайдер объявляет `provides com.example.GreetingService with com.example.impl.EnglishGreeting;` в файле `module-info.java`, а модуль-потребитель пишет `uses com.example.GreetingService;`. Загрузка происходит через `ServiceLoader.load(GreetingService.class, ModuleLayer.boot())`.

## Краткий итог

Используйте `ServiceLoader`, когда вам нужно реализовать **слабую связность (loose coupling)** и дать возможность подключать разные реализации одного и того же интерфейса без изменения кода основного приложения (принцип инверсии зависимостей).

---

## Собрать все имплементации, прмер
```java
public class PackageProcessor {  
  
  private final Map<String, AbstractPackageRepository> repositories;  
  
  /**  
   * Конструктор по умолчанию. Использует Java SPI (ServiceLoader) для автоматического   * обнаружения всех реализаций. Это гарантирует OCP: при добавлении нового формата   * не потребуется менять ни одной строки в этом классе.   */  public PackageProcessor() {  
    Map<String, AbstractPackageRepository> map = new HashMap<>();  
    ServiceLoader<AbstractPackageRepository> loader =  
        ServiceLoader.load(AbstractPackageRepository.class);  
  
    for (AbstractPackageRepository repository : loader) {  
      map.put(repository.getType(), repository);  
    }  
  
    // Создаем неизменяемую копию для безопасности  
    this.repositories = Map.copyOf(map);  
  }  
  
  /**  
   * Альтернативный конструктор. Полезен для Unit-тестов или если вы предпочитаете   * собирать реестр стратегий вручную в отдельном конфигурационном классе.   */  public PackageProcessor(List<AbstractPackageRepository> repositoryList) {  
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
