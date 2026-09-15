---
aliases:
  - build.gradle
  - Gradle
  - gradle.properties
  - Groovy
  - java-library
  - settings.gradle
---

**Gradle** — система автоматической сборки для JVM.

## Файлы конфигурации

### settings.gradle vs build.gradle

Оба файла — скрипты Groovy. `settings.gradle` — более общий скрипт, чем `build.gradle`.

В каждой сборке будет выполняться только один скрипт `settings.gradle` (по сравнению с несколькими скриптами `build.gradle` в сборках с несколькими проектами). Скрипт `settings.gradle` будет выполнен перед любым скриптом `build.gradle` и даже перед созданием экземпляров проекта.

**`settings.gradle`:**

Основная роль `settings.gradle` — определить все включаемые подмодули и отметить корень дерева модулей. Поэтому в мультимодульном проекте может быть только один такой файл.

```groovy
rootProject.name = 'project-x'
include 'sub-a', 'sub-b'
```

**`build.gradle`:**

Для каждого модуля существует один такой файл, содержащий логику сборки этого модуля.

В `build.gradle` **главного модуля** можно использовать `allprojects {}` или `subprojects {}`, чтобы задать настройки для всех остальных модулей.

В `build.gradle` подмодулей можно использовать `compile project(':sub-a')`, чтобы связать один подмодуль с другим.

### gradle.properties

Файл `gradle.properties` — это обычный Java [`Properties`](https://docs.oracle.com/javase/7/docs/api/java/util/Properties.html) файл, который получает особую роль благодаря автоматическому включению в область видимости объекта `Project` (как «project properties»). Это простой key-value store, который допускает только строковые значения (списки и массивы приходится разбивать самостоятельно).

Расположение файла `gradle.properties`:

- непосредственно в каталоге проекта (для значений, связанных с проектом);
- в домашнем каталоге пользователя `.gradle` (для значений, связанных с пользователем или окружением);
- опционально — для стартовых параметров самого Gradle, например:

```groovy
org.gradle.jvmargs=-Xmx=... -Dfile.encoding=UTF-8 ...
org.gradle.configureondemand=true
```

## Жизненный цикл проекта

Скрипт сборки приложения и прочие этапы жизненного цикла проекта находятся в плагине Gradle — `java-library`, который поставляется вместе с Gradle.

**Подключение плагина:**

```groovy
plugins {
    id('java-library')
}
```

**Доступные задачи:**

```groovy
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
check - Runs all checks.
classes - Assembles main classes.
clean - Deletes the build directory.
compileJava - Compiles main Java source.
compileTestJava - Compiles test Java source.
jar - Assembles a jar archive containing the main classes.
javadoc - Generates Javadoc API documentation for the main source code.
processResources - Processes main resources.
processTestResources - Processes test resources.
test - Runs the test suite.
testClasses - Assembles test classes.
```

## Плагины

Все плагины Gradle наследуют интерфейс `Plugin<Project>`.

**Пример простого плагина:**

```groovy
class HelloPlugin implements Plugin<Project> {
    void apply(Project project) {
        project.task('hello') {
            doLast {
                println 'Hello from the HelloPlugin'
            }
        }
    }
}
```
