---
aliases:
<<<<<<< HEAD
  - Blocking queue
  - BlockingDeque
  - BlockingQueue
  - Concurrent collections
  - ConcurrentHashMap
  - ConcurrentLinkedDeque
  - ConcurrentLinkedQueue
  - ConcurrentMap
=======
  - ArrayBlockingQueue
  - Blocking queue
  - BlockingDeque
  - BlockingQueue
  - CAS
  - Compare-And-Swap
  - Compare and Swap
  - Collection
  - Concurrent collections
  - ConcurrentHashMap
  - ConcurrentHashMap KeySetView
  - ConcurrentLinkedDeque
  - ConcurrentLinkedQueue
  - ConcurrentMap
  - ConcurrentModificationException
>>>>>>> 8667d92 (stashing)
  - ConcurrentNavigableMap
  - ConcurrentSkipListMap
  - ConcurrentSkipListSet
  - Copy On Write
  - Copy-on-write
  - Copy-on-write collections
  - CopyOnWrite
  - CopyOnWriteArrayList
  - CopyOnWriteArraySet
<<<<<<< HEAD
  - Java Concurrency Collections
  - Java Concurrency Utilities
  - java-concurrent
  - java-concurrent-collections
---
## Java's Concurrent Collections
=======
  - Deque
  - DelayQueue
  - Delayed
  - FIFO
  - HashMap
  - Hashtable
  - Java Concurrency Collections
  - Java Concurrency Utilities
  - LIFO
  - LinkedBlockingDeque
  - LinkedBlockingQueue
  - LinkedTransferQueue
  - List
  - Map
  - NavigableMap
  - NavigableSet
  - PriorityBlockingQueue
  - Queue
  - Serializable
  - Set
  - SynchronousQueue
  - Thread-safe
  - TransferQueue
  - UnsupportedOperationException
  - AbstractQueue
  - ArrayList
  - java-concurrent
  - java-concurrent-collections
  - java.util
  - java.util.concurrent
  - wait
  - wait-free
  - notify
  - weakly consistent
  - consistent iterator
  - блокирующая очередь
  - блокирующие очереди
  - безопасность потоков
  - копирование при записи
  - копия при записи
  - консистентный итератор
  - многопоточность
  - мультипоточность
  - потокобезопасные коллекции
  - потоки
  - сегмент
  - сегменты
  - слабоконсистентный итератор
  - итератор
---

## Java's Concurrent Collections

>>>>>>> 8667d92 (stashing)
> `java.util.concurrent`

```mermaid
---
title: Java's Concurrent Collections
---
classDiagram
<<<<<<< HEAD
    %% Основные интерфейсы Java Collections
=======
>>>>>>> 8667d92 (stashing)
    class Collection {
        <<interface>>
        java.util
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class Queue {
        <<interface>>
        java.util
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class List {
        <<interface>>
        java.util
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class Set {
        <<interface>>
        java.util
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class NavigableSet {
        <<interface>>
        java.util
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class Map {
        <<interface>>
        java.util
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class NavigableMap {
        <<interface>>
        java.util
    }
<<<<<<< HEAD

    %% java.util.concurrent - Blocking Queues
=======
>>>>>>> 8667d92 (stashing)
    class BlockingQueue {
        <<interface>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class TransferQueue {
        <<interface>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class BlockingDeque {
        <<interface>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class ArrayBlockingQueue {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class DelayQueue {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class LinkedBlockingQueue {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class PriorityBlockingQueue {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class SynchronousQueue {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class LinkedTransferQueue {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class LinkedBlockingDeque {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
    %% java.util.concurrent - Copy-on-write Collections
=======
>>>>>>> 8667d92 (stashing)
    class CopyOnWriteArrayList {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class CopyOnWriteArraySet {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
    %% java.util.concurrent - Concurrent Collections
=======
>>>>>>> 8667d92 (stashing)
    class ConcurrentMap {
        <<interface>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class ConcurrentNavigableMap {
        <<interface>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class ConcurrentHashMap {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class ConcurrentHashMapKeySetView["ConcurrentHashMap.KeySetView"] {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class ConcurrentSkipListMap {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class ConcurrentSkipListSet {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class ConcurrentLinkedQueue {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class ConcurrentLinkedDeque {
        <<class>>
        java.util.concurrent
    }
<<<<<<< HEAD

    %% Наследование основных интерфейсов
    Collection <|-- Queue
    Collection <|-- List
    Collection <|-- Set

=======
    Collection <|-- Queue
    Collection <|-- List
    Collection <|-- Set
>>>>>>> 8667d92 (stashing)
    Queue <|-- BlockingQueue
    BlockingQueue <|-- TransferQueue
    BlockingQueue <|-- BlockingDeque
    BlockingQueue <|.. ArrayBlockingQueue
    BlockingQueue <|.. DelayQueue
    BlockingQueue <|.. LinkedBlockingQueue
    BlockingQueue <|.. PriorityBlockingQueue
    BlockingQueue <|.. SynchronousQueue
    TransferQueue <|.. LinkedTransferQueue
    BlockingDeque <|.. LinkedBlockingDeque
    BlockingDeque <|.. LinkedTransferQueue
    Queue <|.. ConcurrentLinkedQueue
    Queue <|.. ConcurrentLinkedDeque
<<<<<<< HEAD

    List <|.. CopyOnWriteArrayList

=======
    List <|.. CopyOnWriteArrayList
>>>>>>> 8667d92 (stashing)
    Set <|.. CopyOnWriteArraySet
    Set <|-- NavigableSet
    Set <|.. ConcurrentHashMapKeySetView
    NavigableSet <|.. ConcurrentSkipListSet
<<<<<<< HEAD

=======
>>>>>>> 8667d92 (stashing)
    Map <|-- ConcurrentMap
    Map <|-- NavigableMap
    ConcurrentMap <|.. ConcurrentHashMap
    ConcurrentMap <|.. ConcurrentNavigableMap
    ConcurrentHashMap <-- ConcurrentHashMapKeySetView
    NavigableMap <|-- ConcurrentNavigableMap
    ConcurrentNavigableMap <|.. ConcurrentSkipListMap
<<<<<<< HEAD
    
=======
>>>>>>> 8667d92 (stashing)
    class ConcurrentLinkedDeque:::Concurrent
    class ConcurrentLinkedQueue:::Concurrent
    class ConcurrentSkipListSet:::Concurrent
    class ConcurrentSkipListMap:::Concurrent
    class ConcurrentHashMap:::Concurrent
    class ConcurrentNavigableMap:::Concurrent
    class ConcurrentMap:::Concurrent
    class ConcurrentHashMapKeySetView:::Concurrent
    class CopyOnWriteArraySet:::Copy
    class CopyOnWriteArrayList:::Copy
    class LinkedTransferQueue:::Blocking
    class LinkedBlockingDeque:::Blocking
    class SynchronousQueue:::Blocking
    class PriorityBlockingQueue:::Blocking
    class LinkedBlockingQueue:::Blocking
    class DelayQueue:::Blocking
    class ArrayBlockingQueue:::Blocking
    class BlockingQueue:::Blocking
<<<<<<< HEAD
	class BlockingDeque:::Ash
	class TransferQueue:::Ash
	class BlockingDeque:::Ash
	class NavigableMap:::Ash
	class Map:::Ash
	class NavigableSet:::Ash
	class Set:::Ash
	class List:::Ash
	class Queue:::Ash
	class Collection:::Ash
    
    classDef Ash :,stroke-width:1px,stroke-dasharray:none,stroke:#999999,fill:#EEEEEE,color:#000000
	classDef Concurrent :,stroke-width:1px,stroke-dasharray:none,stroke:#FF5978,fill:#FFDFE5,color:#8E2236
	classDef Copy :,stroke-width:1px,stroke-dasharray:none,stroke:#46EDC8,fill:#DEFFF8,color:#378E7A
	classDef Blocking :,stroke-width:1px,stroke-dasharray:none,stroke:#374D7C,fill:#E2EBFF,color:#374D7Cv
	
=======
    class BlockingDeque:::Ash
    class TransferQueue:::Ash
    class NavigableMap:::Ash
    class Map:::Ash
    class NavigableSet:::Ash
    class Set:::Ash
    class List:::Ash
    class Queue:::Ash
    class Collection:::Ash
    classDef Ash :,stroke-width:1px,stroke-dasharray:none,stroke:#999999,fill:#EEEEEE,color:#000000
    classDef Concurrent :,stroke-width:1px,stroke-dasharray:none,stroke:#FF5978,fill:#FFDFE5,color:#8E2236
    classDef Copy :,stroke-width:1px,stroke-dasharray:none,stroke:#46EDC8,fill:#DEFFF8,color:#378E7A
    classDef Blocking :,stroke-width:1px,stroke-dasharray:none,stroke:#374D7C,fill:#E2EBFF,color:#374D7C
>>>>>>> 8667d92 (stashing)
```

**Основные различия**

| Характеристика            | Copy-on-write              | Concurrent                |
| ------------------------- | -------------------------- | ------------------------- |
| Модификация               | Копирование всей коллекции | Прямая модификация        |
| Производительность записи | Низкая                     | Высокая                   |
| Производительность чтения | Высокая                    | Средняя                   |
| Итератор                  | Consistent                 | Weakly consistent         |
| Thread-safe               | Да (immutable)             | Да (lock-free algorithms) |

<<<<<<< HEAD
### 	🔵 Blocking Queues

**Интерфейсы:**

=======
---

### BlockingQueues

**Интерфейсы:**
>>>>>>> 8667d92 (stashing)
- `java.util.concurrent.BlockingQueue`
  - `java.util.concurrent.TransferQueue`
    - `LinkedTransferQueue`
  - `java.util.concurrent.BlockingDeque`
    - `LinkedBlockingDeque`

**Классы:**
<<<<<<< HEAD

=======
>>>>>>> 8667d92 (stashing)
- `ArrayBlockingQueue`
- `DelayQueue`
- `LinkedBlockingQueue`
- `PriorityBlockingQueue`
- `SynchronousQueue`

**Характеристики:**
<<<<<<< HEAD

- Shared collection used to exchange data between two threads
- Provide methods that block until a point of time when data can be exchanged (e.g. when the queue is not empty)
- Useful for producer consumer. Better than wait/notify
- `take()` - blocks until object is available
- `put(E e)` - blocks until space is available in queue
- `LinkedTransferQueue` is new in Java 7 and is basically the best Blocking Queue you will ever need. It is generally more efficient than all the others
- `SynchronousQueue` is a fixed size (bounded) blocking queue, with zero capacity and will block until there is both a producer and a consumer ready to write/read from the queue

---

### 🟢 Copy-on-write Collections

**Классы:**

=======
- Shared collection used to exchange data between two threads.
- Provide methods that block until a point of time when data can be exchanged (e.g. when the queue is not empty).
- Useful for producer-consumer patterns. Better than wait/notify.
- `take()` — blocks until object is available.
- `put(E e)` — blocks until space is available in queue.
- Методы блокировки и снятия блокировки работают быстрее, чем `wait()`, `notify()` класса `Object`.
- `LinkedTransferQueue` is new in Java 7 and is generally more efficient than all the others.
- `SynchronousQueue` is a fixed-size (bounded) blocking queue with zero capacity; blocks until both a producer and a consumer are ready to write/read from the queue.

**Описание классов:**
- `extends Queue<E>` — `java.util.concurrent.BlockingQueue` — очередь, которая дополнительно поддерживает операции, ожидающие, пока очередь станет пустой или пока в очереди освободится место для элемента.
- `extends AbstractQueue<E> implements BlockingQueue<E>, Serializable` — `LinkedBlockingQueue` — реализация очереди с блокировками на основе связанных элементов. FIFO.
- `extends AbstractQueue<E> implements BlockingQueue<E>, Serializable` — `ArrayBlockingQueue` — FIFO.
- `extends AbstractQueue<E> implements BlockingQueue<E>, Serializable` — `PriorityBlockingQueue` — массив-based binary heap: добавление и удаление за логарифмическое время, все элементы comparable.
- `class DelayQueue<E extends Delayed> extends AbstractQueue<E> implements BlockingQueue<E>` — очередь, в которой можно взаимодействовать с объектами только по истечении периода задержки (`interface Delayed extends Comparable<Delayed>`).
- `extends AbstractQueue<E> implements BlockingQueue<E>, Serializable` — `SynchronousQueue` — каждая операция вставки должна дождаться соответствующей операции удаления другим потоком, и наоборот. Не имеет внутренней ёмкости, даже равной единице. Нельзя выполнить peek, итерировать или вставить элемент, пока другой поток не пытается его удалить.
- `extends BlockingQueue<E>, Deque<E>` — `BlockingDeque` — двусторонняя очередь, которая дополнительно поддерживает операции, ожидающие, пока очередь не станет непустой или пока в очереди освободится место для элемента.
- `extends AbstractQueue<E> implements BlockingDeque<E>, Serializable` — `LinkedBlockingDeque` — двунаправленная очередь с блокировками ожидания освобождения места, на основе связанных элементов.

---

### CopyOnWrite-коллекции

**Классы:**
>>>>>>> 8667d92 (stashing)
- `CopyOnWriteArrayList`
- `CopyOnWriteArraySet`

**Характеристики:**
<<<<<<< HEAD

- During modification (add/set/remove/etc), entire contents is copied to a new collection which replaces original
- Being immutable means they are thread-safe
- Modifications to collection are expensive
- Reads are inexpensive
- **Consistent iterator** - means iterator always represents what is in the collection (unlike ConcurrentHashMap which uses weakly consistent iterators)
- Any mutating methods called on the copy-on-write-based iterator (add/set, remove, etc) result in an `UnsupportedOperationException`
- Use with enhanced for loop, not traditional, to make use of Consistent iterator (or use `iterator()`)

---

### 🔴 Concurrent Collections

**Интерфейсы:**

=======
- During modification (add/set/remove/etc), entire contents is copied to a new collection which replaces the original.
- Being immutable means they are thread-safe.
- Modifications to collection are expensive. Reads are inexpensive.
- Consistent iterator — means iterator always represents what is in the collection (unlike ConcurrentHashMap which uses weakly consistent iterators).
- Any mutating methods called on the copy-on-write-based iterator (add/set, remove, etc) result in an `UnsupportedOperationException`.
- Use with enhanced for loop, not traditional, to make use of consistent iterator (or use `iterator()`).

**Описание классов:**
- `CopyOnWriteArrayList` реализует алгоритм CopyOnWrite и является потокобезопасным аналогом `ArrayList`. Содержит изменяемую ссылку на неизменяемый массив, обеспечивая потокобезопасность без блокировок. При операции модификации создаётся новая копия списка, и его итераторы возвращают состояние на момент создания итератора и не вызывают `ConcurrentModificationException`.
- `CopyOnWriteArraySet` выполнен на основе `CopyOnWriteArrayList` с реализацией интерфейса Set.

---

### Concurrent-коллекции

**Интерфейсы:**
>>>>>>> 8667d92 (stashing)
- `java.util.concurrent.ConcurrentMap`
  - `java.util.concurrent.ConcurrentNavigableMap`

**Классы:**
<<<<<<< HEAD

=======
>>>>>>> 8667d92 (stashing)
- `ConcurrentHashMap`
  - `ConcurrentHashMap.KeySetView`
- `ConcurrentSkipListMap`
- `ConcurrentSkipListSet`
- `ConcurrentLinkedQueue`
- `ConcurrentLinkedDeque`

**Характеристики:**
<<<<<<< HEAD

- Can be read and modified by multiple threads
- Does **NOT** copy on writes. So the iterator is **weakly consistent**
- As such the iterator may return objects from the time the iterator was created or later, you may experience elements being added or removed
- Similarly the `size()` method may produce inaccurate results
- More efficient writes than the copy-on-write collections, but at the cost that the reading is less predictable

---

### CopyOnWrite-коллекции (Set, List)

Набор контейнеров, основанный на Copy-On-Write-алгоритме со следующими особенностями:

- при операциях модификации создаётся копия коллекции, которая затем заменяет оригинал
  - операции модификации медленные
- итератор консистентный
  - данные в итераторе всегда актуальные
  - операции чтения быстрые

#### class CopyOnWriteArrayList

`CopyOnWriteArrayList` реализует алгоритм CopyOnWrite и является потокобезопасным аналогом ArrayList. Содержит изменяемую ссылку на неизменяемый массив, обеспечивая потокобезопасность без блокировок. При операции модификации создаётся новая копия списка, и его итераторы возвращают состояние на момент создания итератора и не вызывают `ConcurrentModificationException`.

#### class CopyOnWriteArraySet

`CopyOnWriteArraySet` выполнен на основе `CopyOnWriteArrayList` с реализацией интерфейса Set.

### Concurrent-коллекции (Queue, Set, Map)

- При операциях модификации копия коллекции не создаётся.
  - операции модификации выполняются быстрее, чем у `CopyOnWrite`
- Слабоконсистентный итератор
  - данные в итераторе могут быть неактуальными, в состоянии на момент создания итератора. `size()` может быть неактуальным
  - возможны неточные данные при чтении

#### class ConcurrentLinkedQueue

`extends AbstractQueue` `implements Queue<E>, Serializable`

Размер очереди не ограничен. Имплементация использует **wait-free алгоритм**, адаптированный для работы с garbage collector'ом. Алгоритм эффективен и быстр, так как построен на операции CAS (Compare-And-Swap).

#### class ConcurrentHashMap

`ConcurrentHashMap<K, V>` реализует интерфейс `java.util.concurrent.ConcurrentMap` и отличается от `HashMap` и `Hashtable` внутренней структурой хранения пар key-value. Использует несколько сегментов, поэтому класс можно рассматривать как группу HashMap'ов. По умолчанию количество сегментов равно 16. Доступ к данным определяется по сегментам, а не по объекту. Итераторы фиксируют структуру данных на момент начала его использования.

#### class ConcurrentLinkedDeque

`ConcurrentLinkedDeque` следует использовать, когда необходимо реализовать LIFO, поскольку за счёт двунаправленности класс проигрывает по производительности очереди `ConcurrentLinkedQueue`.

#### interface ConcurrentNavigableMap

Интерфейс расширяет возможности `NavigableMap` для использования в многопоточных приложениях; итераторы декларируются как потокобезопасные и не вызывают `ConcurrentModificationException`.

#### class ConcurrentSkipListMap

Аналог коллекции `TreeMap` с сортировкой данных по ключу и поддержкой многопоточности.

#### class ConcurrentSkipListSet

Выполнен на основе `ConcurrentSkipListMap` с реализацией интерфейса Set.

### BlockingQueue-коллекции (Queue)

- Бывают только блокирующие очереди.
- Методы блокировки и снятия блокировки работают быстрее, чем `wait()`, `notify()` класса `Object`.
- `LinkedTransferQueue`

#### interface BlockingQueue

`extends Queue<E>` `import java.util.concurrent.BlockingQueue`

Очередь, которая дополнительно поддерживает операции, ожидающие, пока очередь станет пустой или пока в очереди освободится место для элемента.

#### class LinkedBlockingQueue

`extends AbstractQueue<E>` `implements BlockingQueue<E>, java.io.Serializable`

`import java.util.concurrent.LinkedBlockingQueue`

Реализация очереди с блокировками на основе связанных элементов. FIFO.

#### class ArrayBlockingQueue

`extends AbstractQueue<E>` `implements BlockingQueue<E>, Serializable`

`import java.util.concurrent.ArrayBlockingQueue`

FIFO.

#### class PriorityBlockingQueue

`extends AbstractQueue<E>` `implements BlockingQueue<E>, Serializable`

`import java.util.concurrent.PriorityBlockingQueue`

Использует массив-based binary heap:

- Добавление и удаление за логарифмическое время, так как реализация основана на куче.
- Все элементы comparable.

#### class DelayQueue

`class DelayQueue<E extends Delayed>` `extends AbstractQueue<E> implements BlockingQueue<E>`

`import java.util.concurrent.DelayQueue`

Очередь, в которой можно взаимодействовать с объектами только по истечении периода задержки, заданного через специальный интерфейс.

`interface Delayed extends Comparable<Delayed>`

#### class SynchronousQueue

`extends AbstractQueue<E>` `implements BlockingQueue<E>, Serializable`

`import java.util.concurrent.SynchronousQueue`

Блокирующая очередь, в которой каждая операция вставки должна дождаться соответствующей операции удаления другим потоком, и наоборот. Синхронная очередь не имеет внутренней ёмкости, даже равной единице. Нельзя выполнить peek: элемент присутствует, только когда его пытаются удалить. Нельзя вставить элемент, пока другой поток не пытается его удалить. Нельзя итерировать, поскольку нечего перебирать.

#### interface BlockingDeque

`extends BlockingQueue<E>, Deque<E>`

`import java.util.concurrent.BlockingDeque`

Двусторонняя очередь, которая дополнительно поддерживает операции, ожидающие, пока очередь не станет непустой или пока в очереди освободится место для элемента.

#### class LinkedBlockingDeque

`extends AbstractQueue<E>` `implements BlockingDeque<E>, Serializable`

`import java.util.concurrent.LinkedBlockingDeque`

Двунаправленная очередь с блокировками ожидания освобождения места, на основе связанных элементов.
=======
- Can be read and modified by multiple threads.
- Does **NOT** copy on writes, so the iterator is weakly consistent.
- As such the iterator may return objects from the time the iterator was created or later; you may experience elements being added or removed.
- Similarly the `size()` method may produce inaccurate results.
- More efficient writes than the copy-on-write collections, but at the cost that the reading is less predictable.
- При операциях модификации копия коллекции не создаётся; операции модификации выполняются быстрее, чем у CopyOnWrite.
- Слабоконсистентный итератор — данные в итераторе могут быть неактуальными, в состоянии на момент создания итератора. `size()` может быть неактуальным; возможны неточные данные при чтении.
- Итераторы не вызывают `ConcurrentModificationException`.

**Описание классов:**
- `extends AbstractQueue implements Queue<E>, Serializable` — `ConcurrentLinkedQueue` — размер очереди не ограничен. Имплементация использует **wait-free алгоритм**, адаптированный для работы с garbage collector'ом. Алгоритм эффективен и быстр, так как построен на операции CAS (Compare-And-Swap).
- `ConcurrentHashMap<K, V>` реализует интерфейс `java.util.concurrent.ConcurrentMap` и отличается от `HashMap` и `Hashtable` внутренней структурой хранения пар key-value. Использует несколько сегментов, поэтому класс можно рассматривать как группу HashMap'ов. По умолчанию количество сегментов равно 16. Доступ к данным определяется по сегментам, а не по объекту. Итераторы фиксируют структуру данных на момент начала его использования.
- `ConcurrentLinkedDeque` — следует использовать, когда необходимо реализовать LIFO, поскольку за счёт двунаправленности класс проигрывает по производительности очереди `ConcurrentLinkedQueue`.
- Интерфейс `ConcurrentNavigableMap` расширяет возможности `NavigableMap` для использования в многопоточных приложениях; итераторы декларируются как потокобезопасные и не вызывают `ConcurrentModificationException`.
- `ConcurrentSkipListMap` — аналог коллекции `TreeMap` с сортировкой данных по ключу и поддержкой многопоточности.
- `ConcurrentSkipListSet` — выполнен на основе `ConcurrentSkipListMap` с реализацией интерфейса Set.
>>>>>>> 8667d92 (stashing)
