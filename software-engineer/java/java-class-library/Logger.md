---
aliases:
  - Java Logging
  - java.util.logging
  - JUL
  - Logback
  - Logger
  - Logging
  - SLF4J
  - Логгер
  - Логирование
---

## Пакет java.util.logging

`java.util.logging.Logger` считается устаревшим. Принято использовать Lombok SLF4J, когда нет ограничений на подключение внешних пакетов, либо Logback.

**Инициализация логгера**

```java
Logger log = Logger.getLogger(FSM.class.getName());
private static final Logger LOGGER = Logger.getLogger(this.getClass().getName());
```

**Добавление сообщений**

```java
LOGGER.logp(Level.INFO, this.getClass().getName(),
    Thread.currentThread().getStackTrace()[1].getMethodName(), "input is " + input);
log.info(String.format("%n\tthere is no transition provided when %s+%s%n", currentState, ch));
```

**Типовая настройка логгера**

```java
// disable mandatory console output
LogManager.getLogManager().reset();
// set standard log message format
System.setProperty("java.util.logging.SimpleFormatter.format", "[%1$tF %1$tT] [%4$-7s] %5$s %n");
```

**Настройка вывода в файл**

```java
public static void startLogger(String loggerName, String fileName) {
  Logger logger = Logger.getLogger(loggerName);
  try {
    // configure the logger with handler and formatter
    FileHandler fileHandler = new FileHandler(fileName, true);
    fileHandler.setFormatter(new SimpleFormatter());
    fileHandler.setLevel(Level.INFO);
    logger.addHandler(fileHandler);
    Handler consoleHandler = new ConsoleHandler();
    consoleHandler.setLevel(Level.WARNING);
    logger.addHandler(consoleHandler);
    logger.info(String.format(" - - %s start - - ", loggerName));
  } catch (SecurityException | IOException e) {
    e.printStackTrace();
  }
}
```

**Пример вызова**

```java
startLogger(FSM.class.getName(), System.getProperty("user.dir") + "/" +
    Thread.currentThread().getStackTrace()[1].getMethodName() + ".log");
log = Logger.getLogger(this.getClass().getName());
log.info(lm);
```
