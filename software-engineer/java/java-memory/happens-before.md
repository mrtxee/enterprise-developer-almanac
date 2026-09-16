---
aliases:
  - CountDownLatch
  - ExecutorService
  - Happens-Before
  - Happens-Before Order
  - Java Memory Model
  - java.util.concurrent
  - JMM
  - Monitor Rule
  - Program Order
  - Safe Publication
  - Synchronized
  - Thread Start Rule
  - Thread Termination Rule
  - Transitivity
  - Volatile
  - Volatile Rule
  - Модель памяти Java
  - Правило монитора
  - Программный порядок
  - Транзитивность
---

## Happens-Before в Java Memory Model

**Happens-Before** — это фундаментальное понятие в [[jmm|Java Memory Model]], которое определяет **гарантии порядка выполнения операций** между потоками. Если операция A "happens-before" операции B, то результаты A гарантированно видны для B.

---

## Основная идея

Без happens-before компилятор и процессор могут **переупорядочивать операции**, что приводит к неожиданному поведению в многопоточных программах.

**Пример проблемы**

```java
// Поток 1
x = 1;          // (1)
ready = true;   // (2) — может выполниться ДО (1)!

// Поток 2
if (ready) {    // (3)
    print(x);   // (4) — может напечатать 0 вместо 1!
}
```

Happens-before решает эту проблему, устанавливая **гарантии видимости изменений**.

---

## Правила Happens-Before

### Программный порядок (Program Order Rule)

В рамках одного потока операции выполняются в том порядке, в котором они записаны в коде.

```java
// В одном потоке:
x = 1;          // (1)
y = 2;          // (2) — гарантированно выполнится после (1)
```

### Правило монитора (Lock Rule)

Освобождение [[object|монитора]] happens-before последующее захват того же монитора.

```java
synchronized (lock) {
    x = 1;      // (1)
}               // (2) — освобождение монитора
// ...
synchronized (lock) {
    print(x);   // (3) — гарантированно увидит x = 1
}               // happens-before между (2) и (3)
```

### Volatile Rule

Запись в volatile переменную happens-before последующее чтение той же volatile переменной.

```java
// Поток 1
x = 42;             // (1)
volatileFlag = true; // (2) — volatile запись

// Поток 2
if (volatileFlag) {  // (3) — volatile чтение (happens-after (2))
    print(x);        // (4) — гарантированно увидит 42
}
```

### Старт потока (Thread Start Rule)

Вызов `start()` happens-before все операции в запущенном потоке.

```java
x = 1;              // (1)

Thread t = new Thread(() -> {
    print(x);       // (2) — гарантированно увидит x = 1
});
t.start();          // happens-before между (1) и (2)
```

### Завершение потока (Thread Termination Rule)

Все операции в потоке happen-before завершение потока (когда `join()` возвращает управление).

```java
Thread t = new Thread(() -> {
    x = 42;         // (1)
});
t.start();
t.join();           // (2) — ожидание завершения
print(x);           // (3) — гарантированно увидит 42
```

### Транзитивность (Transitivity)

Если A happens-before B, и B happens-before C, то A happens-before C.

---

## Практические примеры

### Без happens-before (проблема)

```java
class NoVisibility {
    static int data = 0;
    static boolean ready = false;  // НЕ volatile!

    public static void main(String[] args) {
        // Поток-читатель
        new Thread(() -> {
            while (!ready) { /* ждем */ }
            System.out.println(data);  // Может напечатать 0!
        }).start();

        // Поток-писатель
        data = 42;
        ready = true;  // Может быть переупорядочено с data = 42
    }
}
```

### С happens-before (решение)

```java
class Visibility {
    static int data = 0;
    static volatile boolean ready = false;  // volatile!

    public static void main(String[] args) {
        new Thread(() -> {
            while (!ready) { /* ждем */ }
            System.out.println(data);  // Гарантированно 42
        }).start();

        data = 42;
        ready = true;  // volatile запись → happens-before чтение ready
    }
}
```

---

## Happens-Before в synchronized

```java
class SynchronizedExample {
    private int x = 0;
    private boolean ready = false;
    private final Object lock = new Object();

    void writer() {
        synchronized (lock) {
            x = 42;           // (1)
            ready = true;     // (2)
        } // (3) — освобождение lock happens-before захват в reader
    }

    void reader() {
        synchronized (lock) { // (4) — захват lock
            if (ready) {
                System.out.println(x);  // (5) — гарантированно 42
            }
        }
    }
}
```

**Гарантии**

- (1) happens-before (2) (программный порядок)
- (3) happens-before (4) (правило монитора)
- → (1) happens-before (5) (транзитивность)

---

## Happens-Before в java.util.concurrent

### CountDownLatch

```java
CountDownLatch latch = new CountDownLatch(1);
int data = 0;

// Поток 1
data = 42;
latch.countDown();    // (1) — happens-before await()

// Поток 2
latch.await();        // (2) — happens-after (1)
System.out.println(data);  // Гарантированно 42
```

### ExecutorService

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
int[] data = {0};

Future<?> future = executor.submit(() -> {
    data[0] = 42; // (1)
});

future.get();         // (2) — happens-after (1)
System.out.println(data[0]);  // Гарантированно 42
```

---

## Важные нюансы

### Happens-Before и фактическое время выполнения

Happens-before определяет **порядок видимости**, а не фактическое время выполнения:

```java
// Может выполняться в любом порядке процессором,
// но гарантии видимости сохраняются
x = 1;
y = 2;
```

### Отсутствие Happens-Before

Если между операциями нет отношения happens-before, компилятор/процессор может свободно их переупорядочивать.

---

## Применение на практике

### Безопасная публикация объекта

```java
class SafePublication {
    private static volatile MyObject instance;

    public static MyObject getInstance() {
        if (instance == null) {
            synchronized (SafePublication.class) {
                if (instance == null) {
                    MyObject temp = new MyObject();
                    // Настройка объекта...
                    instance = temp;  // volatile write — happens-before последующие reads
                }
            }
        }
        return instance;  // volatile read
    }
}
```

---

**Итог**

**Happens-Before** — это система гарантий, которая:

- ✅ **Определяет порядок видимости** изменений между потоками.
- ✅ **Предотвращает переупорядочивание** операций там, где это важно.
- ✅ **Обеспечивает корректность** многопоточных программ.
- ✅ **Автоматически работает** при использовании `volatile`, `synchronized`, `java.util.concurrent`.

**Ключевые правила**

- **Программный порядок** в одном потоке.
- **Монитор** (`synchronized`).
- **Volatile** переменные.
- **Старт/завершение** потоков.
- **Транзитивность**.

> **Важно:** используйте высокоуровневые конструкции (`synchronized`, `volatile`, concurrent collections), и [[jvm|JVM]] автоматически обеспечит нужные happens-before гарантии.
