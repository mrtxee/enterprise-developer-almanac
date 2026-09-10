## Maven Dependency management

> [!info] Maven – Introduction to the Dependency Mechanism
> Dependency management is a core feature of Maven.
> [https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Transitive_Dependencies](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Transitive_Dependencies)
## Консольные команды
```Bash
# version
mvn -v
# все этапы цикла жизни проекта до упаквоки включительно
mvn package
# build lifecycle and installs the build phase in the default build cycleо
mvn clean install
mvn compile
mvn test
mvn deploy
...
mvn dependency tree
# skip checks
mvn package -DskipTests -Dcheckstyle.skip=true
```
## Archetypes — Архетипы

Archetype — skeleton for projects | _Архетип —_ **_скелет проекта_**

|                             |                                                                                                                                                                                                                                                                                                                                                                                                  |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| maven-archetype-archetype   | An archetype to generate a sample archetype project.                                                                                                                                                                                                                                                                                                                                             |
| maven-archetype-j2ee-simple | An archetype to generate a simplifed sample J2EE application.  <br>Java EE (Enterprise Edition) - набор спецификаций и соответствующей документации для языка Java, описывающей архитектуру серверной платформы для задач средних и крупных предприятий.  <br>[https://ru.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition](https://ru.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition) |
| maven-archetype-mojo        | An archetype to generate a sample a sample Maven plugin.                                                                                                                                                                                                                                                                                                                                         |
| maven-archetype-plugin      | An archetype to generate a sample Maven plugin.                                                                                                                                                                                                                                                                                                                                                  |
| maven-archetype-plugin-site | An archetype to generate a sample Maven plugin site.                                                                                                                                                                                                                                                                                                                                             |
| maven-archetype-portlet     | An archetype to generate a sample JSR-268 Portlet.                                                                                                                                                                                                                                                                                                                                               |
| maven-archetype-quickstart  | An archetype to generate a sample Maven project.                                                                                                                                                                                                                                                                                                                                                 |
| maven-archetype-simple      | An archetype to generate a simple Maven project.                                                                                                                                                                                                                                                                                                                                                 |
| maven-archetype-site        | An archetype to generate a sample Maven site which demonstrates some of the supported document types like APT, XDoc, and FML and demonstrates how to i18n your site.                                                                                                                                                                                                                             |
| maven-archetype-site-simple | An archetype to generate a sample Maven site.                                                                                                                                                                                                                                                                                                                                                    |
| maven-archetype-webapp      | An archetype to generate a sample Maven Webapp project.                                                                                                                                                                                                                                                                                                                                          |

## Жизненный цикл Maven-проекта

Процесс построения приложения — `Maven Lifecycle`. Жизненный цикл разделен на 8 фаз:

1. clean — удаляются все скомпилированные файлы из каталога target (место, в котором сохраняются готовые артефакты);
2. validate — идет проверка, вся ли информация доступна для сборки проекта;
3. compile — компилируются файлы с исходным кодом;
4. test — запускаются тесты;
5. package — упаковываются скомпилированные файлы (в jar, war и т.д. архив);
6. verify — выполняются проверки для подтверждения готовности упакованного файла;
7. install — пакет помещается в локальный репозиторий. Теперь он может использоваться другими проектами как внешняя библиотека;
8. site — создается документация проекта;
9. deploy — собранный архив копируется в удаленный репозиторий.
Все фазы выполняются последовательно.
## pom.xml

Главный файл конфугирации Мавен-проекта. Главные фичи мавен - управление звисимостями на внешние библиотеки и возмождность запускать плагины на различных этапах жизенного цикла проеката

### `<parent>` — наследование

Нужно для включения несколько модулей в один проект.

```XML
<!-- РОДИТЕЛЬСКИЙ POM: -->
<groupId>com.example</groupId>
<artifactId>project</artifactId>
<version>0.0.1</version>
<packaging>pom</packaging>
. . .
<modules>
    <module>project1</module>
    <module>project2</module>
</modules>
<!-- ДОЧЕРНИЙ POM (напр. внутри папки project1): -->
<parent>
    <groupId>com.example</groupId>
    <artifactId>project</artifactId>
    <version>0.0.1</version>
</parent>
```
### dependencies — управление зависимостями

```xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.7</version>
    </dependency>
  </dependencies>
```
### plugins — плагины

Можно писать свои либо использовать существующие плагины, для контроля на этапха жизеннного цикла построения приложения Maven-проекта

```XML
<build>
     <plugins>
         <plugin>
             <groupId>com.soebes.maven.plugins</groupId>
             <artifactId>uptodate-maven-plugin</artifactId>
             <version>0.2.0</version>
             <executions>
                 <execution>
                     <goals>
                         <goal>dependency</goal>
                     </goals>
                     <phase>validate</phase>
                 </execution>
             </executions>
         </plugin>
     </plugins>
 </build>
```



## faq
### настройка maven

как посмотреть какой файл настроек управляет сейчас maven в windows

```bash
mvn -X
...
[DEBUG] Reading global settings from C:\Program Files\Maven\conf\settings.xml
[DEBUG] Reading user settings from C:\Users\Mironov.A.Al\.m2\settings.xml

mvn help:effective-settings
```
### структура Maven-проекта

`src/main/java` — содержатся java-классы;

`src/main/resources` — ресурсы, которые использует приложение (картинки, стили, конфигурации);

`src/test` — тесты

`pom.xml` — главный файл для управления Мавеном

### установка

Надо скачать архив и прописать в переменные среды path путь к bin-дирктории. `sysdm.cpl`

### фреймворк

`Apache Maven` — фреймворк для автоматизации сборки проектов на основе описания их структуры в файлах на языке POM (Project Object Model), являющемся подмножеством XML. Проект Maven является частью Jakarta Project.

Maven, a Yiddish word meaning _**accumulator of knowledge.**_

Maven используется для построения и управления проектами, написанными на Java, C#, Ruby, Scala, и других языках

### maven dependency scopes

**Maven dependency scopes** определяют, в каких контекстах зависимости будут использоваться — и попадут ли они в финальный артефакт (jar/war).

**Основные скоупы**

| Scope | Когда используется | Попадает в финальный артефакт? | Пример |
|-------|--------------------|----------------------------------|--------|
| `compile` (по умолчанию) | Для компиляции и запуска | ✅ Да | Библиотеки ядра приложения |
| `provided` | Нужна для компиляции, но предоставляется средой (сервером, контейнером) | ❌ Нет | `servlet-api`, `jakarta.servlet` |
| `runtime` | Только во время выполнения (не нужна для компиляции) | ✅ Да | JDBC‑драйверы, реализации SPI |
| `test` | Только для тестов | ❌ Нет | JUnit, Mockito, AssertJ |
| `system` | Зависимость вне репозитория (локальный файл) | ✅ Да (но считается антипаттерном) | Устаревшие/проприетарные jar‑файлы |

**применение**

- **Для CI‑пайплайнов**: скоуп `test` гарантирует, что тестовые библиотеки не улетят в продакшн‑образ — это уменьшает размер контейнера и снижает риски уязвимостей.
- **В Docker/Kubernetes**: `provided` помогает не дублировать библиотеки, которые уже есть в базовом образе (например, сервлет‑контейнер в Tomcat), чтобы итоговый WAR был компактнее.
- **Комплаенс/аудит**: разные скоупы дают чёткое разделение: «это нужно в рантайме», «это только для тестов», «это предоставляет среда». Это упрощает проверку состава зависимостей и соответствие политикам безопасности.
