`Gradle` — Система автоматической сборки для JVM

> [!info] Не бойтесь использовать Gradle  
> Статья-туториал от ведущего Java-разработчика &quot;ITQ Group&quot; Константина Киселевича.  
> [https://habr.com/ru/companies/itq_group/articles/711712/](https://habr.com/ru/companies/itq_group/articles/711712/)  
## Файлы конфигурации
[https://stackoverflow.com/questions/45387971/when-to-use-gradle-properties-vs-settings-gradle](https://stackoverflow.com/questions/45387971/when-to-use-gradle-properties-vs-settings-gradle)
### `settings.gradle` vs `build.gradle`
Оба файла — скрипты Groovy. `settings.gradle` — более общий скрипт чем `build.gradle`
В каждой сборке будет выполняться только один скрипт `settings.gradle` (по сравнению с несколькими скриптами `build.gradle` в сборках с несколькими проектами). Скрипт `settings.gradle` будет выполнен перед любым скриптом `build.gradle` и даже перед созданием экземпляров проекта.
#### `settings.gradle`
- The main role of settings.gradle is to define all included submodules and to mark the directory root of a tree of modules, so you can only have one `settings.gradle` file in a multi-module project.
    
    ```Groovy
    rootProject.name = 'project-x'
    include 'sub-a', 'sub-b'
    ```
    
#### `build.gradle`
There is one such file per module, it contains the build logic for this module.
In the `build.gradle` file of the **main module**, you can use `allprojects {}` or `subprojects {}` to define settings for all other modules.
In the `build.gradle` file of the submodules, you can use `compile project(':sub-a')` to make one submodule depend on the other.
### `gradle.properties`
The `gradle.properties` file is a simple Java [`Properties`](https://docs.oracle.com/javase/7/docs/api/java/util/Properties.html) file that only gains a special role by being automatically included into the scope of the `Project` object (as so-called 'project properties'). It's a **==simple key-value store that==** only allows string values (so you need to split lists or arrays by yourself). You can put `gradle.properties` files to these locations:
- directly in the project directory (for project-related values)
- in the user home `.gradle` directory (for user- or environment-related values)
- This is optional, its main purpose is to provide startup options to use for running gradle itself, e.g.
    
    ```Groovy
    org.gradle.jvmargs=-Xmx=... -Dfile.encoding=UTF-8 ...
    org.gradle.configureondemand=true
    ```
    
## Жизненный цикл проекта
Скрипт сборки приложения, и прочие этапы жизненного цикла проекта расположены в плагине Gradle — `java-library`, которые поставляется вместе с Gradle.
```Groovy
plugins {
    id('java-library')
}
```
дает операции:
```Groovy
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
Все плагины Gradle наследуют интерфейс `Plugin<Project>`
```Groovy
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