---
aliases:
  - CompletableFuture
  - Future
  - ListenableFuture
---
Вот подробное сравнение `Future`, `ListenableFuture` и `CompletableFuture` в Java — с акцентом на **назначение, возможности, ограничения и практическое применение**.

---

## 🔷 1. `java.util.concurrent.Future`  
**Появился**: Java 5  
**Пакет**: `java.util.concurrent`  
**Цель**: Представление результата асинхронной операции (базовый интерфейс).

### 📌 Интерфейс:
```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

### ✅ Плюсы:
- Стандартная часть JDK.
- Простой API для получения результата (`get()`).
- Поддержка отмены и проверки статуса.

### ❌ Минусы:
- **Блокирующий** вызов `get()` — нельзя обрабатывать результат асинхронно.
- Нет обратных вызовов (callbacks), нет возможности цепочки операций.
- Нет встроенной поддержки комбинирования нескольких будущих задач.

### 🧪 Пример:
```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(() -> 42);
Integer result = future.get(); // БЛОКИРУЕТ поток!
```

> ⚠️ Используется в старых фреймворках (например, `ThreadPoolExecutor`, `FutureTask`), но не рекомендуется для современных асинхронных приложений.

---

## 🔷 2. `com.google.common.util.concurrent.ListenableFuture`  
**Появился**: Google Guava (v10+, ~2011)  
**Пакет**: `com.google.common.util.concurrent`  
**Цель**: Добавить **callback-возможности** к `Future` без блокировки.

### 📌 Интерфейс (расширяет `Future`):
```java
public interface ListenableFuture<V> extends Future<V> {
    void addListener(Runnable listener, Executor executor);
}
```

### ✅ Плюсы:
- Поддержка **асинхронных слушателей** через `addListener`.
- Легко интегрируется с Guava-утилитами (`Futures.transform`, `Futures.allAsList`, `Futures.whenAllComplete`).
- Не требует Java 8+ (работает даже на Java 6/7).
- Часто используется в старых системах (например, Apache Beam, Hadoop, старые версии gRPC).

### ❌ Минусы:
- Нет встроенного API для **цепочек операций** (`.thenApply`, `.thenCompose` и т.д.).
- Нет встроенной поддержки **обработки исключений** в цепочке.
- Требует сторонней зависимости (Guava).
- Не является частью JDK.

### 🧪 Пример:
```java
ListenableFuture<Integer> future = executor.submit(() -> 42);
future.addListener(() -> {
    try {
        System.out.println("Result: " + future.get()); // Внимание: get() всё ещё блокирует!
    } catch (Exception e) {
        e.printStackTrace();
    }
}, executor);
```

> 💡 Обратите внимание: `get()` внутри `listener` всё ещё блокирует поток исполнителя! Чтобы избежать этого, Guava предлагает `Futures.transform` с `AsyncFunction`.

### 🔁 Улучшенный пример с `Futures.transform`:
```java
ListenableFuture<String> transformed = Futures.transform(
    future,
    input -> "Result: " + input,
    executor
);
```

---

## 🔷 3. `java.util.concurrent.CompletableFuture`  
**Появился**: Java 8  
**Пакет**: `java.util.concurrent`  
**Цель**: Полноценная реализация **асинхронной реактивной модели** с поддержкой цепочек, комбинирования, обработки ошибок и неблокирующих операций.

### 📌 Ключевые особенности:
- Реализует **`Future`** и **`CompletionStage`**.
- Поддерживает **цепочки операций**:
  - `thenApply`, `thenAccept`, `thenRun`
  - `thenCompose` (для flatMap)
  - `exceptionally`, `handle`, `whenComplete`
- Комбинирование:
  - `allOf`, `anyOf`
- Асинхронное выполнение с указанием `Executor`.
- Возможность **явно завершить** будущее: `complete()`, `completeExceptionally()`.
- Поддержка **timeout**, `orTimeout()`, `completeOnTimeout()` (Java 9+).

### ✅ Плюсы:
- Встроен в JDK — никаких зависимостей.
- Мощный, выразительный API для асинхронного программирования.
- Неблокирующий по умолчанию (если использовать `*Async` методы).
- Хорошо интегрируется с реактивными фреймворками (Project Reactor, RxJava через адаптеры).

### ❌ Минусы:
- Сложность для новичков (глубокие цепочки могут быть трудны для отладки).
- Нет встроенной поддержки backpressure (в отличие от Reactive Streams).
- `CompletableFuture` — не `Publisher`, поэтому не подходит напрямую для реактивных пайплайнов.

### 🧪 Пример (современный):
```java
CompletableFuture.supplyAsync(() -> 42, executor)
    .thenApply(x -> x * 2)
    .thenAccept(System.out::println)  // выводит 84
    .exceptionally(ex -> {
        System.err.println("Error: " + ex.getMessage());
        return null;
    });
```

### 🔄 Явное завершение (полезно в callback-ориентированных API):
```java
CompletableFuture<String> future = new CompletableFuture<>();
// где-то позже, например в колбэке от внешнего сервиса:
future.complete("done");
// или при ошибке:
future.completeExceptionally(new RuntimeException("timeout"));
```

---

## 📊 Сравнительная таблица

| Фича | `Future` | `ListenableFuture` (Guava) | `CompletableFuture` (Java 8+) |
|------|----------|----------------------------|-------------------------------|
| **JDK** | ✅ Да (с Java 5) | ❌ Нет (требует Guava) | ✅ Да (с Java 8) |
| **Асинхронные callbacks** | ❌ Нет | ✅ `addListener` | ✅ `thenApply`, `thenAccept`, `handle` и др. |
| **Цепочка операций** | ❌ | ⚠️ Через `Futures.transform` (ограниченно) | ✅ Полная поддержка (`thenCompose`, `allOf`) |
| **Обработка ошибок** | Только через `get()` → `ExecutionException` | `Futures.catching`, `transform` с `AsyncFunction` | ✅ `exceptionally`, `handle`, `whenComplete` |
| **Комбинирование** | ❌ | ✅ `Futures.allAsList`, `whenAllComplete` | ✅ `CompletableFuture.allOf`, `anyOf` |
| **Явное завершение** | ❌ | ✅ `SettableFuture` (наследник) | ✅ `complete()`, `completeExceptionally()` |
| **Неблокирующий** | ❌ (`get()` блокирует) | ⚠️ `addListener` асинхр., но `get()` внутри — блокирует | ✅ Все `*Async` методы неблокирующие |
| **Поддержка `Executor`** | При создании `FutureTask` / `submit` | В `addListener`, `transform` | В `supplyAsync`, `thenApplyAsync` и др. |
| **Рекомендуется в 2025+** | ❌ Только для совместимости | ⚠️ В legacy-проектах (Guava) | ✅ **Да — стандарт де-факто** |

---

## 🎯 Когда что использовать?

| Сценарий | Рекомендуемый тип |
|---------|------------------|
| Старый код, Java 6/7, уже используется Guava | `ListenableFuture` |
| Современный Java 8+ проект, нужна асинхронная логика | `CompletableFuture` |
| Просто нужно получить результат одной задачи и подождать | `Future` (но лучше `CompletableFuture.get()` — он тоже блокирует, но даёт больше гибкости) |
| Интеграция с Reactor / RxJava | `CompletableFuture` → преобразовать через `Mono.fromFuture()` или `Observable.fromFuture()` |

---

## 💡 Bonus: CompletableFuture vs Reactive Streams

| Критерий                           | `CompletableFuture`              | `Mono`/`Flux` (Reactor)                               |
| ---------------------------------- | -------------------------------- | ----------------------------------------------------- |
| Backpressure                       | ❌ Нет                            | ✅ Да                                                  |
| Поток данных (множество элементов) | ❌ Только 1 значение              | ✅ Поддержка потоков                                   |
| Операторы                          | Базовые (`map`, `flatMap`)       | Расширенные (`retry`, `timeout`, `window`, `groupBy`) |
| Использование                      | Однократные асинхронные операции | Стримы событий, WebSocket, HTTP-стримы                |

> 📌 Совет: Для простых асинхронныховов (API-запросы, DB) — `CompletableFuture`.  
> Для сложных потоковых пайплайнов — переходите на Reactive Streams (Project Reactor, RxJava).

---

## ✅ Итог

- **`Future`** — база, но устарела для асинхронного программирования.
- **`ListenableFuture`** — мост между старым и новым, полезен в Guava-экосистеме.
- **`CompletableFuture`** — **современный стандарт** для асинхронных операций в Java.  
  → Используйте его по умолчанию в новых проектах.

Если вы пишете на Java 11+ — `CompletableFuture` + `ExecutorService` (или `virtual threads` в Java 21+) — это золотой стандарт.