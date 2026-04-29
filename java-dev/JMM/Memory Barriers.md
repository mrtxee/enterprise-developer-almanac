---
aliases:
  - Memory Fence
  - Memory Barrier
---
### **Java Memory Barrier (Барьеры памяти в Java)**

**Memory Barrier** (также known as **Memory Fence**) — это механизм, который обеспечивает порядок выполнения операций с памятью в многопоточных программах. В Java они неявно используются в механизмах синхронизации.

---

## **1. Проблема: переупорядочивание операций (Reordering)**

Без барьеров памяти процессор и компилятор могут **переупорядочивать операции** для оптимизации:

```java
// Исходный код
int x = 1;
boolean ready = false;

// Поток 1
x = 42;
ready = true;

// Поток 2
while (!ready) {
    // spin
}
System.out.println(x);  // Может напечатать 1 или 42?
```

**Без memory barrier** операции могут быть переупорядочены:
```java
// Фактический порядок выполнения (возможен)
ready = true;  // Сначала!
x = 42;        // Потом
```

---

## **2. Виды Memory Barriers**

### **LoadLoad Barrier**
- Гарантирует, что все операции **чтения ДО барьера** завершатся до операций **чтения ПОСЛЕ барьера**
- Пример: `volatile read`

### **StoreStore Barrier**  
- Гарантирует, что все операции **записи ДО барьера** завершатся до операций **записи ПОСЛЕ барьера**
- Пример: `volatile write`

### **LoadStore Barrier**
- Гарантирует, что операции **чтения ДО барьера** завершатся до операций **записи ПОСЛЕ барьера**

### **StoreLoad Barrier** (самый строгий)
- Гарантирует, что все операции **записи ДО барьера** завершатся до операций **чтения ПОСЛЕ барьера**
- Самый дорогой барьер

---

## **3. Как создаются Memory Barriers в Java?**

### **Ключевое слово `volatile`**
```java
class Example {
    volatile boolean flag = false;
    int data = 0;
    
    void writer() {
        data = 42;           // обычная запись
        flag = true;         // volatile запись → StoreStore барьер
    }
    
    void reader() {
        if (flag) {          // volatile чтение → LoadLoad барьер
            System.out.println(data);  // гарантированно увидит 42
        }
    }
}
```

**Что гарантирует `volatile`:**
- **StoreStore барьер** перед записью в `volatile`
- **StoreLoad барьер** после записи в `volatile`  
- **LoadLoad барьер** после чтения `volatile`
- **LoadStore барьер** после чтения `volatile`

---

## **4. `synchronized` и Memory Barriers**

```java
class SynchronizedExample {
    private int x = 0;
    private boolean ready = false;
    
    public synchronized void write() {
        x = 42;
        ready = true;
        // При выходе из synchronized → StoreLoad барьер
    }
    
    public synchronized void read() {
        if (ready) {
            System.out.println(x);  // гарантированно 42
        }
        // При входе в synchronized → LoadLoad барьер
    }
}
```

**`synchronized` создает барьеры:**
- **Вход в монитор** → LoadLoad + LoadStore барьеры
- **Выход из монитора** → StoreStore + StoreLoad барьеры

---

## **5. Практические примеры**

### **Без барьеров (проблема видимости)**
```java
class NoVisibility {
    static boolean ready = false;  // НЕ volatile!
    static int number = 0;
    
    static class ReaderThread extends Thread {
        public void run() {
            while (!ready) {
                Thread.yield();
            }
            System.out.println(number);  // Может напечатать 0!
        }
    }
    
    public static void main(String[] args) {
        new ReaderThread().start();
        number = 42;
        ready = true;  // Может быть переупорядочено!
    }
}
```

### **С барьерами (правильно)**
```java
class Visibility {
    static volatile boolean ready = false;  // volatile!
    static int number = 0;
    
    static class ReaderThread extends Thread {
        public void run() {
            while (!ready) {
                Thread.yield();
            }
            System.out.println(number);  // Гарантированно 42
        }
    }
    
    public static void main(String[] args) {
        new ReaderThread().start();
        number = 42;
        ready = true;  // StoreStore барьер гарантирует, что number=42 выполнится до ready=true
    }
}
```

---

## **6. Happens-Before и Memory Barriers**

**Правило happens-before для `volatile`:**
- Запись в `volatile` happens-before последующее чтение того же `volatile`
- Это создает необходимые memory barriers автоматически

```java
// Thread 1
nonVolatileVar = 123;    // (1)
volatileVar = true;      // (2) StoreStore барьер между (1) и (2)

// Thread 2  
if (volatileVar) {       // (3) LoadLoad барьер после (3)
    print(nonVolatileVar); // (4) Гарантированно увидит 123
}
```

---

## **7. Атомарные классы (java.util.concurrent.atomic)**

```java
import java.util.concurrent.atomic.*;

class AtomicExample {
    private AtomicInteger atomicInt = new AtomicInteger(0);
    private int regularInt = 0;
    
    void update() {
        regularInt = 100;                    // обычная запись
        atomicInt.lazySet(42);              // StoreStore барьер
        // или
        atomicInt.set(42);                  // volatile запись (StoreStore + StoreLoad)
    }
    
    void read() {
        int value = atomicInt.get();        // volatile чтение (LoadLoad)
        System.out.println(regularInt);     // может не увидеть 100 без дополнительных барьеров
    }
}
```

---

### **Итог**

**Memory Barriers в Java:**
- **Невидимый механизм** обеспечения порядка операций с памятью
- **Автоматически создаются** при использовании `volatile`, `synchronized`, атомарных операций
- **Гарантируют видимость изменений** между потоками
- **Предотвращают переупорядочивание** операций компилятором/процессором

**Когда думать о memory barriers:**
- При разработке многопоточных структур данных
- При оптимизации производительности synchronized блоков
- При работе с lock-free алгоритмами

> **Важно:** В большинстве случаев достаточно использовать высокоуровневые механизмы (`volatile`, `synchronized`, `AtomicXXX`), и JVM автоматически вставит нужные барьеры памяти.