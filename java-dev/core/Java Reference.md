---
aliases:
  - Reference
  - Java Reference
---
# Java Reference
>[`java.lang.ref`](https://docs.oracle.com/javase/9/docs/api/java/lang/ref/package-summary.html)

Тип ссылки в Java влияет на **жизненный цикл объекта** и то, **как Garbage Collector (GC) решает, когда можно удалить объект из памяти**.

Java Reference бывают типов
1. `Strong` → объект жив, пока на него есть ссылки
2. `Soft` → объект жив, пока есть память в куче
	1. подходит для логов
3. `Weak` → объект удаляется при следующей сборке, если у него нет strong-ссылок.
	1. подходит для мета информации об объекте
4. `Phantom` → Post-mortem: объект уже мёртв, ссылка нужна для уведомления об очистке

### 📊 Сравнение типов ссылок

| Тип ссылки              | Класс              | Когда GC удаляет объект                                   | Типичное использование                  |
| ----------------------- | ------------------ | --------------------------------------------------------- | --------------------------------------- |
| **Strong** (Сильная)    | `Object ref`       | **Никогда**, пока ссылка существует                       | Обычные объекты (по умолчанию)          |
| **Soft** (Мягкая)       | `SoftReference`    | Только при **нехватке памяти** (перед `OutOfMemoryError`) | Кэширование данных                      |
| **Weak** (Слабая)       | `WeakReference`    | При **следующей сборке мусора**, если нет сильных ссылок  | `WeakHashMap`, канонические отображения |
| **Phantom** (Фантомная) | `PhantomReference` | После **финализации**, но перед освобождением памяти      | Очистка ресурсов, логирование удаления  |

---

2.  **Метод `get()`**
    *   `Strong`, `Soft`, `Weak` → Возвращают объект (или `null`, если собран).
    *   `Phantom` → **Всегда возвращает `null`** (используется только с `ReferenceQueue`).

3.  **Управление памятью**
    *   Позволяет создавать **кэши**, которые не приводят к `OutOfMemoryError` (`Soft`).
    *   Позволяет избегать **утечек памяти** в мапах (`WeakHashMap`).

---


```java
// Strong
Object object = new Object(); //создал обьект 
object = null; //теперь может быть собран сборщиком мусора

// Weak Reference
Counter counter = new Counter(); 
WeakReference weakCounter = new WeakReference(counter); 
counter = null; 
// теперь weakCounter будет удален сборщиком муссора, т.к. не содержит в себе сильной ссылки

// Soft Reference
Counter counter = new Counter(); 
SoftReference softCounter = new SoftReference(counter); 
counter = null; 
// теперь softCounter будет удален сборщиком муссора, но не сразу, а в отличие от weak ссылки удаление не случится пока не появится острая нехватка памяти

// Можно отслеживать цикл жизни при помощи `ReferenceQueue`
ReferenceQueue refQueue = new ReferenceQueue(); 
DigitalCounter digit = new DigitalCounter();
PhantomReference phantom = new PhantomReference(digit, refQueue);
```

### Цикл жизни ссылочного объекта


```mermaid
---
title: Reference object lyfecycle
---
flowchart TB
    Created["Created"] --> Initialized["Initialized"]
    Initialized --> StronglyReachable["StronglyReachable"]
    StronglyReachable --> SoftlyReachable["SoftlyReachable"] & WeaklyReachable["WeaklyReachable"] & Finalized["Finalized"]
    SoftlyReachable --> Finalized & WeaklyReachable
    WeaklyReachable --> Finalized
    Finalized --> PhantomReachable["PhantomReachable"]

     StronglyReachable:::Pine
     SoftlyReachable:::Aqua
     WeaklyReachable:::Aqua
     PhantomReachable:::Rose
    classDef Pine stroke-width:1px, stroke-dasharray:none, stroke:#254336, fill:#27654A, color:#FFFFFF
    classDef Aqua stroke-width:1px, stroke-dasharray:none, stroke:#46EDC8, fill:#DEFFF8, color:#378E7A
    classDef Rose stroke-width:1px, stroke-dasharray:none, stroke:#FF5978, fill:#FFDFE5, color:#8E2236
```
