Все класса Java являются наследниками данного класса.
Ключевые публичные методы, которые он определяет:
## `equals()`
Равенство объектов Java соответствует математическому отношения эквивалентности
- **рефлексивно**: объект равен сам себе
- **симметрично**
- **транзитивно**
- **консистентно:** можно вызывать метод `equals()` сколько угодно раз для одних и тех же объектов, результат меняться не будет.
```Java
public boolean equals(Object obj) {
    return (this == obj);
}
```
## `hashCode()`
`public native int hashCode();`
Возвращает хэшкод объекта. Который нужен прежде всего при хешировании. Разные реализации JVM предлагаю разные используют разные методы генерации хэшкода. Часто он бывает приввязак к адресу в объекта в памяти.

### контракт хэшкода
- Если для 2 объектов ==`equals()==true`== у них ==должен быть одинаковых хэш-код==
- Если для 2 объектов `equals()==false` для них ==допускается одинаковых хэш-код==
- значение `hashCode()` частично консистентно, т.е. ==хешкод может измениться только в том, случае, если изменился== результат `equals()` для 2 объектов
## `toString()`
Returns a string representation of the object.
```Java
return getClass().getName() + "@" + Integer.toHexString(hashCode());
```


* toString возвращает **строковое представление объекта**

Формат вида `MyObject{field1='data', field2='other'}`, возвращаемой строки этим методом называется
* **декларативное строковое представление**
* так же встрачается названия
	* human-readable representation

Если toString не объявлен, вернтся строка вида `ClassName@hashCode`, напр. `java.lang.Object@1b6d3586`. Такая форма называется **каноническое строковое представление**

## `wait()`
- Заставляет текущий поток не выполнять никаких действий, пока он не будет пробужден методами `notify*()`. Работает только внутри `syncronized` блоков.
    
    ```Java
    public final void wait() throws InterruptedException {
      wait(0L);
    }
    ```
    
## `notify()`, `notifyAll()`
`notify()` Пробуждает один поток, который ожидает на мониторе этого объекта. Пробужденный поток не сможет продолжить работу до тех пор, пока текущий поток не снимет блокировку с этого объекта. Работает только внутри `syncronized` блоков.
- `notifyAll()` пробуждает все потоки этого объекта
    
    ```Java
    public final native void notify();
    public final native void notifyAll();
    ```
    
### `wait()`, `notify()` синхронизация потоков
Тип `Object` поставлят методы, которые позволяют потокам выполнения приостанавливать и и продлжать выполнение по требованию паралельных потоков для объектов текущего типа. Для этого служат методы корневого класса `Object`:
- `wait()` освобождает монитор и переводит вызывающий поток в состояние ожидания до тех пор, пока другой поток не вызовет метод `notify()`
- `notify()` продолжает работу потока, у которого ранее был вызван метод `wait()`
- `notifyAll()` возобновляет работу всех потоков, у которых ранее был вызван метод `wait()`
- Эти методы работают только внутри мониторов — `synchronized` блоков кода
    
    ```Java
    public class WaitNotifyEx {
        public static void main(String[] args) {
            Store store = new Store();
            Producer producer = new Producer(store);
            Consumer consumer = new Consumer(store);
            new Thread(producer).start();
            new Thread(consumer).start();
        }
    }
    // Класс Магазин, хранящий произведенные товары
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
    // класс Производитель
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
    // Класс Потребитель
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
    ```