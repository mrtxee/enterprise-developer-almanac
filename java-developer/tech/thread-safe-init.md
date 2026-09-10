---
aliases:
  - DCL
  - Double-Checked Locking
  - Eager Initialization
  - Enum Singleton
  - EnumSingleton
  - Initialization-on-Demand
  - Initialization-on-Demand Holder
  - Initialization-on-Demand Holder Idiom
  - Thread safe initialization
  - Thread safe initialization patterns
---
## Потокобезопасная инициализации единственного экземпляра
> способы потокобезопасной инициализации единственного экземпляра класса

Все эти подходы — это **реализации паттерна Singleton («Одиночка»)**, а точнее — **способы потокобезопасной ленивой (или неленивой) инициализации единственного экземпляра класса**.

Иногда их ещё называют **стратегиями реализации Singleton** или **паттернами создания единственного экземпляра**.

---

### Для чего нужны эти паттерны

Их общая цель — **гарантировать, что в приложении будет существовать ровно один экземпляр класса**, и сделать это корректно в многопоточной среде.

#### Конкретные задачи, которые они решают

- **Контроль количества экземпляров.** Запретить создание нескольких объектов там, где по смыслу должен быть один (конфигурация, пул соединений, кэш, логгер).
- **Потокобезопасность.** В многопоточном приложении (особенно в вебе и CI‑пайплайнах) несколько потоков могут одновременно попытаться создать экземпляр. Эти паттерны исключают появление двух объектов и гонки данных при инициализации.
- **Управление моментом создания (lazy vs eager).** Одни варианты создают объект сразу при загрузке класса (eager), другие — только при первом обращении (lazy). Это важно, если объект тяжёлый: инициализация может требовать ресурсов, чтения файлов, подключения к БД и т. п.
- **Защита от «неполного» объекта.** Некоторые реализации (например, DCL без `volatile` или с ошибками) могут вернуть ссылку на объект, конструктор которого ещё не до конца отработал. Проверенные паттерны гарантируют, что любой поток получит полностью инициализированный экземпляр.
- **Устойчивость к сериализации и рефлексии.** Например, `Enum Singleton` автоматически закрывает уязвимости, связанные с десериализацией и созданием объектов через рефлексию — частые проблемы «обычных» синглтонов.

### Что такое Double-Checked Locking

Идея в том, чтобы **снизить накладные расходы** на синхронизацию при ленивой инициализации (например, в паттерне Singleton). Обычно, если делать синхронизацию на каждом вызове метода, это тормозит приложение. DCL пытается оптимизировать: сначала проверяет условие без блокировки, и только если ресурс ещё не создан — заходит в синхронизированный блок. [```2```](https://javarush.com/quests/lectures/questservlets.level17.lecture02)[```4```](https://ru.wikipedia.org/wiki/%D0%91%D0%BB%D0%BE%D0%BA%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0_%D1%81_%D0%B4%D0%B2%D0%BE%D0%B9%D0%BD%D0%BE%D0%B9_%D0%BF%D1%80%D0%BE%D0%B2%D0%B5%D1%80%D0%BA%D0%BE%D0%B9)[```5```](https://java-design-patterns.com/patterns/double-checked-locking/)

Пример реализации (для Java 5+):

```java
public class Singleton {
    private static volatile Singleton instance; // Ключевое слово volatile критически важно

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) { // Первая проверка (без блокировки)
            synchronized (Singleton.class) { // Вторая проверка (внутри блокировки)
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Зачем две проверки?** Первая — чтобы в большинстве случаев (когда объект уже создан) не тратить время на блокировку. Вторая внутри `synchronized` нужна, потому что между первой проверкой и входом в блок другой поток мог уже создать объект. [```1```](https://web.archive.org/web/20171027162134/https://www.ibm.com/developerworks/java/library/j-dcl/index.html)[```4```](https://ru.wikipedia.org/wiki/%D0%91%D0%BB%D0%BE%D0%BA%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0_%D1%81_%D0%B4%D0%B2%D0%BE%D0%B9%D0%BD%D0%BE%D0%B9_%D0%BF%D1%80%D0%BE%D0%B2%D0%B5%D1%80%D0%BA%D0%BE%D0%B9)

### В чём проблема

Главная ловушка кроется в **модели памяти Java**. До Java 5 реализация DCL была **небезопасной**. Проблема в том, что компилятор или JIT-компилятор могут переупорядочивать инструкции. Теоретически возможно такое поведение:

- Поток 1 начинает инициализацию.
- Он выделяет память под объект и записывает ссылку в поле `instance` (но конструктор ещё не выполнился полностью).
- Поток 2 видит, что `instance` уже не `null`, и возвращает ссылку на объект, который **ещё не полностью инициализирован**. [```1```](https://web.archive.org/web/20171027162134/https://www.ibm.com/developerworks/java/library/j-dcl/index.html)[```12```](https://alxkm.github.io/posts/multithreaded_programming_anti_patterns_in_java_pt2/)

**Решение появилось с Java 5:** добавление модификатора `volatile` к полю `instance` гарантирует видимость изменений для всех потоков и предотвращает нежелательное переупорядочивание операций. С этим DCL становится безопасным. [```12```](https://alxkm.github.io/posts/multithreaded_programming_anti_patterns_in_java_pt2/)[```13```](https://www.java67.com/2016/04/why-double-checked-locking-was-broken-before-java5.html)[```3```](https://habr.com/ru/companies/pvs-studio/articles/819625/)

Но даже с `volatile` у подхода есть минусы: код сложнее для чтения, а в некоторых сценариях накладные расходы на синхронизацию всё равно могут быть ощутимы. Поэтому часто рекомендуют обходиться без DCL. [```6```](https://www.baeldung.com/java-singleton-double-checked-locking)

### Альтернативы

Я подобрала несколько подходов, которые в разных ситуациях могут быть предпочтительнее. [```6```](https://www.baeldung.com/java-singleton-double-checked-locking)[```7```](https://www.initgrep.com/posts/design-patterns/thread-safety-in-java-singleton-pattern)[```13```](https://www.java67.com/2016/04/why-double-checked-locking-was-broken-before-java5.html)

- **Eager Initialization (ленивая инициализация при загрузке класса).** Объект создаётся сразу при загрузке класса. Это потокобезопасно по определению, так как инициализация статических полей в Java потокобезопасна. Подходит, если объект не слишком тяжёлый и его использование вероятно. [```6```](https://www.baeldung.com/java-singleton-double-checked-locking)[```7```](https://www.initgrep.com/posts/design-patterns/thread-safety-in-java-singleton-pattern)
  ```java
  public class EagerSingleton {
      private static final EagerSingleton instance = new EagerSingleton();

      private EagerSingleton() {}

      public static EagerSingleton getInstance() {
          return instance;
      }
  }
  ```

- **Initialization-on-Demand Holder Idiom (идиома с внутренним классом).** Используется вложенный статический класс. Инициализация этого класса (и создание экземпляра) происходит только при первом обращении к нему — то есть при первом вызове `getInstance()`. Это гарантирует потокобезопасность без явной синхронизации. [```6```](https://www.baeldung.com/java-singleton-double-checked-locking)[```7```](https://www.initgrep.com/posts/design-patterns/thread-safety-in-java-singleton-pattern)
  ```java
  public class HolderSingleton {
      private HolderSingleton() {}

      private static class InstanceHolder {
          private static final InstanceHolder INSTANCE = new InstanceHolder();
      }

      public static InstanceHolder getInstance() {
          return InstanceHolder.INSTANCE;
      }
  }
  ```

- **Enum Singleton.** В Java перечисление гарантирует, что экземпляр создаётся ровно один раз, инициализация потокобезопасна «из коробки», а также есть защита от сериализации. Это часто считается самым лаконичным и безопасным решением. [```6```](https://www.baeldung.com/java-singleton-double-checked-locking)[```8```](https://dev.to/abh1navv/ways-to-create-singletons-and-the-tradeoffs-between-them-ofd)
  ```java
  public enum EnumSingleton {
      INSTANCE;
      // методы и поля
  }
  ```

### Какой путь выбрать?

- Если объект тяжёлый и его создание действительно нужно откладывать до первого использования — можно рассмотреть DCL, но только с `volatile` и с пониманием всех нюансов. [```12```](https://alxkm.github.io/posts/multithreaded_programming_anti_patterns_in_java_pt2/)[```3```](https://habr.com/ru/companies/pvs-studio/articles/819625/)
- Если объект не такой ресурсоёмкий и его использование почти гарантировано — Eager Initialization проще и понятнее. [```6```](https://www.baeldung.com/java-singleton-double-checked-locking)[```7```](https://www.initgrep.com/posts/design-patterns/thread-safety-in-java-singleton-pattern)
- Если важна максимальная простота и гарантия безопасности (включая защиту от сериализации) — Enum Singleton. [```6```](https://www.baeldung.com/java-singleton-double-checked-locking)[```8```](https://dev.to/abh1navv/ways-to-create-singletons-and-the-tradeoffs-between-them-ofd)
- Initialization-on-Demand Holder — хороший компромисс: ленивая инициализация + потокобезопасность без явной синхронизации. [```6```](https://www.baeldung.com/java-singleton-double-checked-locking)[```7```](https://www.initgrep.com/posts/design-patterns/thread-safety-in-java-singleton-pattern)

В итоге выбор зависит от конкретного сценария: насколько тяжёлый объект, как часто к нему обращаются, какие есть дополнительные требования (например, защита от сериализации). Если расскажешь подробнее про твою задачу (например, что именно инициализируется лениво), подскажу, какой вариант подойдёт лучше!
