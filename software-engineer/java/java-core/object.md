---
aliases:
  - equals
  - equals()
  - hashCode
  - hashCode()
  - hashCode contract
  - java.lang.Object
  - monitor
  - notify
  - notify()
  - notifyAll
  - notifyAll()
  - Object
  - Object class
  - object equality
  - synchronized
  - toString
  - toString()
  - wait
  - wait()
  - wait/notify
  - контракт hashCode
  - контракт хэшкода
  - монитор
  - равенство объектов
---

## Object в Java

Все классы Java являются наследниками данного класса.

Ключевые публичные методы, которые он определяет:

### equals()

Равенство объектов Java соответствует математическому отношению эквивалентности:

- рефлексивно: объект равен сам себе
- симметрично
- транзитивно
- консистентно: можно вызывать метод `equals()` сколько угодно раз для одних и тех же объектов, результат меняться не будет

Пример реализации по умолчанию:

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

### hashCode()

`public native int hashCode();`

Возвращает хэшкод объекта, который нужен прежде всего при хешировании. Разные реализации JVM предлагают разные методы генерации хэшкода. Часто он бывает привязан к адресу объекта в памяти.

#### Контракт хэшкода

- Если для двух объектов `equals()` == `true`, у них должен быть одинаковый хэшкод
- Если для двух объектов `equals()` == `false`, для них допускается одинаковый хэшкод
- Значение `hashCode()` частично консистентно, т. е. хэшкод может измениться только в том случае, если изменился результат `equals()` для этих объектов

### toString()

Возвращает строковое представление объекта.

Пример реализации по умолчанию:

```java
return getClass().getName() + "@" + Integer.toHexString(hashCode());
```

Формат вида `MyObject{field1='data', field2='other'}` возвращаемой строки называется декларативным строковым представлением; также встречается название human-readable representation.

Если `toString()` не объявлен, вернётся строка вида `ClassName@hashCode`, например `java.lang.Object@1b6d3586`. Такая форма называется каноническим строковым представлением.

### wait()

Заставляет текущий поток не выполнять никаких действий, пока он не будет пробуждён методами `notify*()`. Работает только внутри `synchronized` блоков.

Пример:

```java
public final void wait() throws InterruptedException {
    wait(0L);
}
```

### notify() и notifyAll()

`notify()` пробуждает один поток, который ожидает на мониторе этого объекта. Пробуждённый поток не сможет продолжить работу до тех пор, пока текущий поток не снимет блокировку с этого объекта. `notifyAll()` пробуждает все потоки, ожидающие на мониторе этого объекта. Оба метода работают только внутри `synchronized` блоков.

Пример:

```java
public final native void notify();
public final native void notifyAll();
```

#### Синхронизация потоков

Методы `Object` позволяют потокам приостанавливать и продолжать выполнение по требованию параллельных потоков:

- `wait()` освобождает монитор и переводит вызывающий поток в состояние ожидания до тех пор, пока другой поток не вызовет `notify()`
- `notify()` продолжает работу потока, у которого ранее был вызван `wait()`
- `notifyAll()` возобновляет работу всех потоков, у которых ранее был вызван `wait()`
- Все методы работают только внутри мониторов — `synchronized` блоков кода

Пример (Producer-Consumer):

```java
public class WaitNotifyEx {
    public static void main(String[] args) {
        Store store = new Store();
        Producer producer = new Producer(store);
        Consumer consumer = new Consumer(store);
        new Thread(producer).start();
        new Thread(consumer).start();
    }
}

class Store {
    private int product = 0;

    public synchronized void get() {
        while (product < 1) {
            try {
                wait();
            } catch (InterruptedException e) {
            }
        }
        product--;
        System.out.println("Покупатель купил 1 товар");
        System.out.println("Товаров на складе: " + product);
        notify();
    }

    public synchronized void put() {
        while (product >= 3) {
            try {
                wait();
            } catch (InterruptedException e) {
            }
        }
        product++;
        System.out.println("Производитель добавил 1 товар");
        System.out.println("Товаров на складе: " + product);
        notify();
    }
}

class Producer implements Runnable {
    Store store;

    Producer(Store store) {
        this.store = store;
    }

    public void run() {
        for (int i = 1; i < 6; i++) {
            store.put();
        }
    }
}

class Consumer implements Runnable {
    Store store;

    Consumer(Store store) {
        this.store = store;
    }

    public void run() {
        for (int i = 1; i < 6; i++) {
            store.get();
        }
    }
}
