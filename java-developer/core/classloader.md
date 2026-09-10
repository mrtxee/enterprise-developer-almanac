---
aliases:
  - Application ClassLoader
  - Bootstrap ClassLoader
  - ClassLoader
  - CLASSPATH
  - delegation model
  - Platform ClassLoader
  - загрузчик классов
---

">[!info] `java.lang.ClassLoader`

## Загрузчики классов

`ClassLoader` обеспечивает загрузку классов Java.Точнее его наследники т.к. java.lang.ClassLoader – абстрактный класс. Каждый раз, когда загружается какой-либо `.class` файл, например, вследствие обращения к конструктору или статическому методу класса это действие выполняет один из наследников ClassLoader.

Системный загрузчик классов используется при запуске приложения командой: `java <имя_класса>`

- системный загрузчик классов загружает все JAR из CLASSPATH
  - переменной среды либо параметре «-cp» утилиты «java»
- можно реализовать наследник **ClassLoader** – и использовать его вместо системного.*

Загрузчик классов выполняет две основные функции:

1. Загрузка классов – различные встроенные и пользовательские загрузчики классов загружают классы. Мы можем расширить `abstract class java.lang.ClassLoader` для реализации собственного загрузчика
2. Поиск ресурсов, где ресурс это некоторые данные, такие как файл .class, информация о конфигурации, изображение и т.п.

### Динамическая загрузка классов

ClassLoader загружает классы динамически, то есть в процессе выполнения программы. С помощью ClassLoader можно загрузить класс, которого не было в момент компиляции. Виртуальная машина загрузит только файлы классов, необходимые для выполнения программы.

### Иерархия загрузчиков классов

Основные загрузчик в порядке наследования:

1. **Bootstrap ClassLoader**
  - загружает основные библиотеки из папки `<JAVA_HOME>/jre/lib`.
  - т.н. Primordial ClassLoader
1. **Platform ClassLoader**
  - loads platform-specific extensions from the JDK's module system
  - loads modules system property java.platform or –module-path
  - загружает расширения из `<JAVA_HOME>/jre/lib/ext`.
  - т.н. Extension ClassLoader, название до java 9
1. **Application ClassLoader** – загружает классы приложения, зависящие от classpath.
  - It loads files found in the classpath environment variable, _-classpath,_ or _-cp_ command line option
  - т.н. System ClassLoader
1. _\[UserDefined ClassLoader\]_ может быть унаследован от системного
> [!info] CLASSPATH
Параметр в виртуальной машине Java или компиляторе Java, который определяет расположение пользовательских классов и пакетов. Параметр может быть задан либо в командной строке, либо через переменную среды.

### Модель делегирования загрузки класса

Когда происходит загрузка класса, Системный загрузчик делигирует задачу Платформенному, тот делигирует Бутстрап загрузчику. Только если класс не был найден Бутстрап загрузчиком и после платформенным, случится попытка загрузки класса системным загрузчиком.

Следствие модели делегирования в том, что любой загрузчик знает классы, которые были загружены им и его родителем.

```mermaid
---
title: JVM ClassLoader Hierarchy
---
flowchart TD
 subgraph subGraph0["JVM ClassLoader Hierarchy"]
        Bootstrap["Bootstrap ClassLoader (native)"]
        Extension["Extension ClassLoader"]
        App["Application ClassLoader"]
        Custom["Custom ClassLoader (user-defined)"]
  end
 subgraph subGraph1["Bootstrap ClassLoader"]
        BootstrapCore["java.lang.*<br>java.util.*<br>java.io.*<br>javax.*<br>sun.*<br>etc."]
  end
 subgraph subGraph2["Extension ClassLoader"]
        ExtLib["jre/lib/ext/<br>javax.crypto.*<br>javax.security.*<br>etc."]
  end
 subgraph subGraph3["Application ClassLoader"]
        Classpath["classpath<br>-cp<br>-jar<br>java.class.path"]
  end
 subgraph subGraph4["Custom ClassLoader"]
        CustomCode["Custom jars<br>Modules<br>Web apps<br>Plugins<br>etc."]
  end
    Bootstrap -- parent of --> Extension
    Extension -- parent of --> App
    App -- parent of --> Custom
    Bootstrap --> BootstrapCore
    Extension --> ExtLib
    App --> Classpath
    Custom --> CustomCode

     Bootstrap:::Pine
     Extension:::Aqua
     App:::Sky
     Custom:::Rose
     BootstrapCore:::Pine
     ExtLib:::Aqua
     Classpath:::Sky
     CustomCode:::Rose
    classDef Rose stroke-width:1px, stroke-dasharray:none, stroke:#FF5978, fill:#FFDFE5, color:#8E2236
    classDef Sky stroke-width:1px, stroke-dasharray:none, stroke:#374D7C, fill:#E2EBFF, color:#374D7C
    classDef Aqua stroke-width:1px, stroke-dasharray:none, stroke:#46EDC8, fill:#DEFFF8, color:#378E7A
    classDef Pine stroke-width:1px, stroke-dasharray:none, stroke:#254336, fill:#27654A, color:#FFFFFF
```

### Принципы работы загрузчиков класса
- Видимость (_Visibility_): Дочерний ClassLoader может видеть классы, загруженные его родителем, но не наоборот, что обеспечивает инкапсуляцию;
- Уникальность (_Uniqueness):_ Класс, загруженный родителем, не будет повторно загружен его дочерним классом, что повышает эффективность;
- Иерархия делегирования (_Delegation Hierarchy_): Application ClassLoader (дочерний) передает запрос на загрузку класса родителям, загрузчикам Platform и Bootstrap. Если они не могут найти класс, то запрос передается обратно по цепочке, пока класс не будет найден или не выкинут `ClassNotFoundException`
