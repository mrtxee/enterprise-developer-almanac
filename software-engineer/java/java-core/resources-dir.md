---
aliases:
  - ClassLoader
  - Classpath resources
  - getResourceAsStream
  - Project resources
  - resources
  - resources dir
  - resources-dir
  - src/main/resources
  - Каталог ресурсов
  - Ресурсы проекта
---

## Каталог resources

**Суть**

`/src/main/resources` — стандартная директория в Java-проектах (особенно при использовании Maven/Gradle), предназначенная для некомпилируемых файлов, которые должны попасть в classpath и быть доступны во время выполнения (runtime). В отличие от `/src/main/java`, где лежат `.java`-файлы (они компилируются в `.class`), в `resources` кладут всё остальное: конфиги, шаблоны, тексты лицензий, картинки, SQL-скрипты, JSON/YAML и т. п.

**Ключевые особенности**

- **Становится частью classpath.** При сборке (Maven `mvn package`, Gradle `build`) содержимое `src/main/resources` копируется в выходную папку (обычно `target/classes` или `build/classes`) и затем включается в [[jdk-jls-jni|JAR]]/WAR. В рантайме это «корневая» часть classpath.
- **Путь внутри артефакта совпадает с относительным путём в проекте.** Файл `src/main/resources/spdx/references/file1.txt` окажется в JAR как `spdx/references/file1.txt`. Никаких префиксов `resources/` не будет.
- **Доступ только через [[classloader|ClassLoader]], не через `File`.** В собранном JAR это не настоящая файловая система, поэтому `new File("src/main/resources/...")` не работает.
- **Поддержка фильтрации (подстановки переменных).** В Maven можно включить `<filtering>true</filtering>` для ресурсов: `${project.version}`, `${build.timestamp}` и т. п. будут заменены на реальные значения при сборке. Полезно для генерации `version.properties`, манифестов и т. д.
- **Разделение на main и test.** Есть `src/main/resources` (для продакшн-кода) и `src/test/resources` (только для тестов). Это позволяет иметь разные конфиги и фикстуры для тестов и основного приложения.
- **Автоматическое поведение в сборщиках.** Maven и Gradle по умолчанию копируют всё из `src/main/resources`. Исключения и кастомные правила задаются явно.

---

## Схема структуры проекта

```text
my-project/
  src/
    main/
      java/
        com/example/...
      resources/
        application.yml
        spdx/
          references/
            file1.txt
        version.properties
    test/
      java/
      resources/
  pom.xml (или build.gradle)
```

---

## Различия между IDE и JAR

Потому что в IDE ресурсы лежат как обычные файлы на диске, а в JAR-файле упакованы внутрь архива, это принципиально разные типы хранилищ, и Java обращается к ним по-разному.

**В IDE (IntelliJ IDEA / Eclipse)**

- Сборка помещает ресурсы в папку вроде `target/classes` (Maven) или `build/classes/main` (Gradle).
- Это обычные файлы в файловой системе.
- Можно использовать `Path` / `Files.list` / `new File(...)`.
- URL ресурса выглядит как `file:/.../target/classes/spdx/references`.

**При запуске `java -jar my-app.jar`**

- Ресурсы находятся внутри JAR (ZIP-архива).
- Нет отдельных файлов на диске: есть только записи внутри архива.
- `new File(...)`, `Files.list`, `Paths.get` не работают для путей внутри JAR.
- URL ресурса выглядит как `jar:file:/.../my-app.jar!/spdx/references` — это не путь к файлу, а специальный URL протокола `jar`.

---

## Чтение одного файла

Для чтения одного конкретного файла различий почти нет: `getResourceAsStream` работает и в IDE, и в JAR, потому что `ClassLoader` сам знает, где искать ресурс.

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

**Важно:** если путь начинается с `/` — это абсолютный путь от корня classpath; без `/` — относительно пакета класса. Для переносимости лучше использовать относительный путь через `getContextClassLoader()`.

---

## Получение списка файлов в директории

Допустим, нужно получить список всех `.txt`-файлов в `spdx/references`.

**Для IDE (classpath — папка)**

```java
URL dirUrl = getClass().getResource("spdx/references");
Path path = Path.of(dirUrl.toURI()); // OK в IDE
List<String> files = Files.list(path)
    .filter(p -> p.getFileName().toString().endsWith(".txt"))
    .map(p -> p.getFileName().toString())
    .toList();
```

В JAR этот код выбросит `FileSystemNotFoundException`, потому что для `jar:...` URI нет зарегистрированной файловой системы по умолчанию.

**Для JAR (архив)**

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

Но этот код сломается в IDE, если пытаться открыть как JAR-файл то, что на самом деле является папкой.

**Почему нельзя просто «всегда через Files»**

`Files.list(Path)` требует, чтобы `Path` ссылался на реальную директорию в файловой системе. Внутри JAR такой директории не существует: есть только записи архива. Поэтому `Path.of(jarUrl.toURI())` + `Files.list` приведёт к ошибке. И наоборот: пытаться открывать JAR как архив в IDE, когда это папка `classes`, бессмысленно.

**Универсальный подход**

Нужно определить тип URL и выбрать соответствующую стратегию:

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

Этот метод работает и в IDE, и при запуске через `java -jar`.

---

## Частые ошибки

- Использовать `new File("src/main/resources/...")` — в JAR такой папки нет.
- Делать `Files.list(Path.of(url.toURI()))` без проверки протокола — ломается в JAR.
- Предполагать, что `dirUrl.getPath()` содержит путь к JAR без `!/` — формат URL зависит от реализации, лучше разбирать через `URI` и искать `!`.
- Не проверять `null` у `getResourceAsStream` — если опечататься в пути, получится `NullPointerException` при чтении.
- Путать относительные и абсолютные пути в `getResource` — лучше избегать абсолютных путей с `/` и использовать `ClassLoader`.
- Забывать про кодировку — всегда явно указывать `StandardCharsets.UTF_8` при чтении текстовых ресурсов.
