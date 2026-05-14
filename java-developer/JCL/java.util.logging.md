>java.util.logging
# Logger

`java.util.logging.Logger` считается устаревшим. Принято использовать Lombok SL4J, когда нет ограничений на подключение внешний пакетов, либо Logback

```Java
// Инициализация логгера
Logger log = Logger.getLogger(FSM.class.getName());
// Декларативная инициализация
private static final Logger LOGGER = Logger.getLogger( this.getClass().getName() );
// добавляение сообщений
LOGGER.logp(Level.INFO, this.getClass().getName(), Thread.currentThread().getStackTrace()[1].getMethodName(), "input is "+input);
log.info( String.format("%n\tthere is no transition provided when %s+%s%n",currentState,ch) );
// типовая настройка логгера
	// disable mandatory console output
	LogManager.getLogManager().reset();
	// set standard log message format
	System.setProperty("java.util.logging.SimpleFormatter.format", "[%1$tF %1$tT] [%4$-7s] %5$s %n");
public static void startLogger(String loggerName, String fileName) {
    Logger logger = Logger.getLogger(loggerName);
    try {
        // This block configure the logger with handler and formatter
        FileHandler fileHandler = new FileHandler(fileName, true);
        fileHandler.setFormatter(new SimpleFormatter());
        fileHandler.setLevel(Level.INFO);
        logger.addHandler(fileHandler);
        Handler consoleHandler = new ConsoleHandler();
        consoleHandler.setLevel(Level.WARNING);
        logger.addHandler(consoleHandler);
        // the following statement is used to log any messages
        logger.info(String.format(" - - %s start - - ", loggerName));
    } catch (SecurityException | IOException e) {
        e.printStackTrace();
    }
}
startLogger(FSM.class.getName(), System.getProperty("user.dir") +"/"+ Thread.currentThread().getStackTrace()[1].getMethodName() + ".log");
// ...
log = Logger.getLogger(this.getClass().getName());
log.info(lm);
```