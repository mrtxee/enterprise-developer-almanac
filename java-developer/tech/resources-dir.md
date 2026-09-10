---
aliases:
  - resources
  - resources-dir
---
## resources-dir это

`/src/main/resources`

## Как читать пример в разных среда
```java
public static List<Path> listClasspathDir(String resourcePath) {  
  final URL dirUrl = Thread.currentThread()  
      .getContextClassLoader()  
      .getResource(resourcePath);  
  if (dirUrl == null) {  
    throw new IllegalStateException("Resource not found in classpath: " + resourcePath);  
  }  
  
  // IDE
  if (!"jar".equals(dirUrl.getProtocol())) {  
    try (Stream<Path> stream = Files.list(Paths.get(dirUrl.toURI()))) {  
      return stream  
          .filter(Files::isRegularFile)  
          .collect(Collectors.toList());  
    } catch (Throwable e) {  
      log.error("Error listing files in classpath directory: " + resourcePath, e);  
    }  
  }  
  
  // JAR
  try {  
    final String jarPath = dirUrl.getPath().substring(5, dirUrl.getPath().indexOf("!"));  
    try (var jarFile = new java.util.jar.JarFile(jarPath)) {  
      return jarFile.stream()  
          .filter(entry -> entry.getName().startsWith(resourcePath))  
          .map(jarEntry -> Path.of(jarEntry.getName()))  
          .toList();  
    }  
  } catch (IOException e) {  
    throw new RuntimeException("URI error: " + resourcePath, e);  
  }  
}
```

Потому что **в IDE ресурсы лежат как обычные файлы на диске**, а **в JAR‑файле они упакованы внутрь архива** — это принципиально разные типы хранилищ, и Java обращается к ним по‑разному.

---

### Что происходит в каждом случае

#### В IDE (IntelliJ IDEA / Eclipse)

- Сборка помещает ресурсы в папку вроде `target/classes` (Maven) или `build/classes/main` (Gradle).
- Это **обычные файлы в файловой системе**.
- Можно использовать `Path` / `Files.list` / `new File(...)`.
- URL ресурса выглядит как `file:/.../target/classes/spdx/references`.

#### При запуске `java -jar my-app.jar`

- Ресурсы находятся **внутри JAR** (ZIP‑архива).
- Нет отдельных файлов на диске: есть только записи внутри архива.
- `new File(...)`, `Files.list`, `Paths.get` **не работают** для путей внутри JAR.
- URL ресурса выглядит как `jar:file:/.../my-app.jar!/spdx/references` — это не путь к файлу, а специальный URL протокола `jar`.

---

### Практический пример

Допустим, нужно получить список всех `.txt` файлов в `spdx/references`.

#### Для IDE (работает только тут)

```java
URL dirUrl = getClass().getResource("spdx/references");
Path path = Path.of(dirUrl.toURI()); // OK в IDE
List<String> files = Files.list(path)
    .filter(p -> p.getFileName().toString().endsWith(".txt"))
    .map(p -> p.getFileName().toString())
    .toList();
```

В JAR это выбросит `FileSystemNotFoundException`, потому что для `jar:...` URI нет зарегистрированного файловой системы по умолчанию.

#### Для JAR (работает там, но не универсально само по себе)

```java
try (var jarFile = new JarFile(jarPath)) {
    String prefix = "spdx/references/";
    List<String> files = jarFile.stream()
        .filter(e -> e.getName().startsWith(prefix) && e.getName().endsWith(".txt"))
        .map(JarEntry::getName)
        .map(name -> name.substring(prefix.length()))
        .toList();
}
```

Но этот код сломается в IDE, если пытаться открыть как JAR файл то, что на самом деле — папка.

---

### Почему нельзя просто «всегда через Files»

`Files.list(Path)` требует, чтобы `Path` ссылался на реальную директорию в файловой системе. Внутри JAR такой директории **не существует**: есть только записи архива. Поэтому попытка сделать `Path.of(jarUrl.toURI())` и вызвать `Files.list` приведёт к ошибке.

И наоборот: пытаться открывать JAR как архив в IDE, когда у тебя на самом деле папка `classes`, бессмысленно и приведёт к ошибкам (файл не является валидным JAR).

---

### Как делать правильно (универсальный подход)

Нужно **определить тип URL** и выбрать соответствующую стратегию:

```java
public static List<String> listSpdxReferenceFiles() throws IOException {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    URL dirUrl = cl.getResource("spdx/references");
    if (dirUrl == null) {
        throw new IllegalStateException("Resource directory not found");
    }

    // Сценарий 1: обычный file:// (IDE, classpath — папка)
    if ("file".equals(dirUrl.getProtocol())) {
        Path dirPath = Path.of(dirUrl.toURI());
        return Files.list(dirPath)
            .filter(p -> p.getFileName().toString().endsWith(".txt"))
            .map(p -> p.getFileName().toString())
            .toList();
    }

    // Сценарий 2: jar:// (собранный JAR)
    if ("jar".equals(dirUrl.getProtocol())) {
        URI uri = dirUrl.toURI();
        String spec = uri.getSchemeSpecificPart(); // например: "file:/path/to/app.jar!/spdx/references"
        int bangIndex = spec.indexOf('!');
        if (bangIndex == -1) {
            throw new IllegalArgumentException("Invalid jar URL");
        }
        String jarPart = spec.substring(0, bangIndex); // "file:/path/to/app.jar"

        try (var jarFile = new java.util.jar.JarFile(new URI(jarPart).getPath())) {
            String prefix = "spdx/references/";
            return jarFile.stream()
                .filter(e -> e.getName().startsWith(prefix) && e.getName().endsWith(".txt"))
                .map(java.util.zip.ZipEntry::getName)
                .map(name -> name.substring(prefix.length()))
                .toList();
        }
    }

    throw new UnsupportedOperationException("Unsupported protocol: " + dirUrl.getProtocol());
}
```

Этот метод будет работать и в IDE, и при запуске через `java -jar`.

---

### Чтение одного файла — проще

Для чтения **одного конкретного файла** различий почти нет: всегда используй `getResourceAsStream`:

```java
try (InputStream is = Thread.currentThread().getContextClassLoader()
        .getResourceAsStream("spdx/references/file1.txt")) {
    if (is == null) throw new FileNotFoundException();
    String content = new String(is.readAllBytes(), StandardCharsets.UTF_8);
}
```

Это работает и в IDE, и в JAR, потому что `ClassLoader` сам знает, где искать ресурс.

---

### Связь с твоими задачами (лицензии, SPDX, утилиты)

Учитывая, что ты пишешь утилиты для анализа лицензий:

- **Чтение конкретных файлов** (шаблоны лицензий, конфиги) — всегда через `getResourceAsStream`. Это самый простой и надёжный вариант.
- **Перечисление файлов** (все `.txt` в папке с эталонными лицензиями) — нужен универсальный метод с проверкой протокола, как выше.
- **Тестирование**: в `src/test/resources` можно класть тестовые данные и читать их аналогично — в тестах тоже будет работать `getResourceAsStream`, а для перечисления файлов — аналогичная логика.

---

### Частые ошибки, которые приводят к проблемам

- Использовать `new File("src/main/resources/...")` — в JAR такой папки нет.
- Пытаться сделать `Files.list(Path.of(url.toURI()))` без проверки протокола — ломается в JAR.
- Предполагать, что `dirUrl.getPath()` содержит путь к JAR без `!/` — формат URL зависит от реализации, лучше разбирать через `URI` и искать `!`.
- Не проверять `null` у `getResourceAsStream` — если опечатался в пути, получишь `NullPointerException` при чтении.

Если скажешь, какой именно сценарий тебе важнее (например, «перечислить все файлы лицензий», «прочитать один шаблон», «сделать это и в тестах, и в продакшене»), могу дать готовый, заточенный под этот кейс метод.

## Что такое `/src/main/resources`

Это стандартная директория в Java‑проектах (особенно при использовании Maven/Gradle), предназначенная для **некомпилируемых файлов, которые должны попасть в classpath и быть доступны во время выполнения (runtime)**.

В отличие от `/src/main/java`, где лежат `.java` файлы (они компилируются в `.class`), в `resources` кладут всё остальное: конфиги, шаблоны, тексты лицензий, картинки, SQL‑скрипты, JSON/YAML и т. п.

---

### Ключевые особенности

1. **Становится частью classpath.**
   При сборке (Maven `mvn package`, Gradle `build`) содержимое `src/main/resources` копируется в выходную папку (обычно `target/classes` или `build/classes`) и затем включается в JAR/WAR. В рантайме это «корневая» часть classpath.

2. **Путь внутри артефакта совпадает с относительным путём в проекте.**
   Файл `src/main/resources/spdx/references/file1.txt` окажется в JAR как `spdx/references/file1.txt`. Никаких префиксов `resources/` не будет.

3. **Доступ только через ClassLoader, не через `File`.**
   В собранном JAR это не настоящая файловая система. Правильно:
   ```java
   InputStream is = getClass().getResourceAsStream("/spdx/references/file1.txt");
   // или
   InputStream is = Thread.currentThread().getContextClassLoader()
       .getResourceAsStream("spdx/references/file1.txt");
   ```

   Неправильно пытаться открыть как `new File("src/main/resources/...")` или по абсолютному пути — в JAR такого файла не существует.

4. **Поддержка фильтрации (подстановки переменных).**
   В Maven можно включить `<filtering>true</filtering>` для ресурсов: `${project.version}`, `${build.timestamp}` и т. п. будут заменены на реальные значения при сборке. Полезно для генерации `version.properties`, манифестов и т. д.

5. **Разделение на main и test.**
   Есть `src/main/resources` (для продакшн‑кода) и `src/test/resources` (только для тестов). Это позволяет иметь разные конфиги/фикстуры для тестов и основного приложения.

6. **Автоматическое поведение в сборщиках.**
   Maven и Gradle по умолчанию копируют всё из `src/main/resources`. Исключения и кастомные правила задаются явно.

---

### Зачем нужна эта директория (на примерах из твоих задач)

Учитывая твои предыдущие вопросы, вот где она критически важна:

- **Конфигурации, которые не хочется хардкодить.**
  Например, паттерны поиска файлов, правила нормализации текста, маппинги типов лицензий — всё это можно вынести в YAML/JSON/properties и подгружать при старте.

- **Тестовые данные и фикстуры.**
  В `src/test/resources` хранят тестовые архивы, примеры SPDX‑документов, «грязные» тексты лицензий для проверки детекторов.

- **Статические данные для утилит.**
  Если пишешь утилиту для анализа лицензий, туда можно положить словари, стоп‑слова, регулярные выражения в виде текстовых файлов.

- **Локализация и сообщения.**
  `.properties` файлы для i18n (например, сообщения об ошибках на разных языках).

- **Шаблоны отчётов.**
  Шаблоны для генерации отчётов (текстовые, Markdown, HTML) удобно хранить как ресурсы и заполнять в рантайме.

---

### Как пользователь должен работать с этой директорией

#### 1. Куда класть файлы

Структура типичного проекта:

```text
my-project/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/...
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── spdx/
│   │       │   └── references/
│   │       │       └── file1.txt
│   │       └── version.properties
│   └── test/
│       ├── java/
│       └── resources/
└── pom.xml (или build.gradle)
```

#### 2. Как читать ресурсы в коде (JDK 17)

**Вариант A: через ClassLoader (универсально, работает в IDE и в JAR)**

```java
try (var is = Thread.currentThread().getContextClassLoader()
        .getResourceAsStream("spdx/references/file1.txt")) {
    if (is == null) {
        throw new FileNotFoundException("Resource not found");
    }
    String content = new String(is.readAllBytes(), StandardCharsets.UTF_8);
}
```

**Вариант B: через конкретный класс (если ресурс «привязан» к классу)**

```java
try (var is = MyClass.class.getResourceAsStream("/spdx/references/file1.txt")) {
   // ...
}
```

> Важно: если путь начинается с `/` — это абсолютный путь от корня classpath; без `/` — относительно пакета класса. Для переносимости лучше использовать относительный путь через `getContextClassLoader()`.

#### 3. Как перечислить файлы в папке ресурсов (в том числе в JAR)

Как мы обсуждали ранее, `Files.list` не работает для `jar:...` URL. Нужно либо:

- В IDE (classpath как папка) использовать `Files.list`.
- В JAR использовать `JarFile` или `ZipFile` и перебирать записи по префиксу.

Пример (ранее уже показывал, но он прямо про эту директорию):

```java
public static List<String> listSpdxReferenceFiles() throws IOException {
    var cl = Thread.currentThread().getContextClassLoader();
    URL dirUrl = cl.getResource("spdx/references");
    if (dirUrl == null) throw new IllegalStateException("Directory not found");

    if (!"jar".equals(dirUrl.getProtocol())) {
        // IDE: classpath — папка
        var dirPath = Path.of(dirUrl.toURI());
        return Files.list(dirPath)
            .filter(p -> p.getFileName().toString().endsWith(".txt"))
            .map(p -> p.getFileName().toString())
            .toList();
    }

    // JAR: читаем через JarFile
    var uri = dirUrl.toURI();
    int bang = uri.getSchemeSpecificPart().indexOf('!');
    var jarPath = Paths.get(URI.create(uri.getSchemeSpecificPart().substring(0, bang)));

    try (var jarFile = new JarFile(jarPath.toFile())) {
        String prefix = "spdx/references/";
        return jarFile.stream()
            .filter(e -> e.getName().startsWith(prefix))
            .map(JarEntry::getName)
            .map(name -> name.substring(prefix.length()))
            .toList();
    }
}
```

#### 4. Частые ошибки

- **Пытаться открыть ресурс как `File` по пути `src/main/resources/...`.** В JAR такой папки нет.
- **Путать относительные и абсолютные пути в `getResource`.** Лучше избегать абсолютных путей с `/` и использовать `ClassLoader`.
- **Забывать про кодировку.** Всегда явно указывайте `StandardCharsets.UTF_8` при чтении текстовых ресурсов.
- **Не проверять `null` у `getResourceAsStream`.** Если опечатались в пути — будет `null`, а не исключение.

---
