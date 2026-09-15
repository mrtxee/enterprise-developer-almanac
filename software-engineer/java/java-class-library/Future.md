---
aliases:
  - Asynchronous programming
  - CompletableFuture
  - CompletionStage
  - ExecutorService
  - Flux
  - Future
  - Futures
  - FutureTask
  - Google Guava
  - Guava
  - ListenableFuture
  - Mono
  - Non-blocking operations
  - Project Reactor
  - Reactive Streams
  - RxJava
  - Virtual threads
  - Асинхронное программирование
  - Виртуальные потоки
  - Комбинирование будущих задач
  - Неблокирующие операции
  - Обработка ошибок
  - Цепочки операций
---

Сравнение `Future`, `ListenableFuture` и `CompletableFuture` в Java: назначение, возможности, ограничения и практическое применение.

## Future

`java.util.concurrent.Future` — представление результата асинхронной операции (базовый интерфейс).

- Появился: Java 5.
- Пакет: `java.util.concurrent`.

**Интерфейс**

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

**Особенности**

- ✅ Стандартная часть JDK.
- ✅ Простой API для получения результата (`get()`).
- ✅ Поддержка отмены и проверки статуса.
- ❌ Блокирующий вызов `get()` — нельзя обрабатывать результат асинхронно.
- ❌ Нет обратных вызовов (callbacks) и возможности цепочки операций.
- ❌ Нет встроенной поддержки комбинирования нескольких будущих задач.

**Пример**

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(() -> 42);
Integer result = future.get(); // БЛОКИРУЕТ поток!
```

Используется в старых фреймворках (например, `ThreadPoolExecutor`, `FutureTask`), но не рекомендуется для современных асинхронных приложений.

## ListenableFuture

`com.google.common.util.concurrent.ListenableFuture` — добавляет callback-возможности к `Future` без блокировки.

- Появился: Google Guava (v10+, примерно 2011).
- Пакет: `com.google.common.util.concurrent`.

**Интерфейс**

```java
public interface ListenableFuture<V> extends Future<V> {
    void addListener(Runnable listener, Executor executor);
}
```

**Особенности**

- ✅ Поддержка асинхронных слушателей через `addListener`.
- ✅ Лёгкая интеграция с Guava-утилитами (`Futures.transform`, `Futures.allAsList`, `Futures.whenAllComplete`).
- ✅ Не требует Java 8+ (работает даже на Java 6/7).
- ✅ Часто используется в старых системах (например, Apache Beam, Hadoop, старые версии gRPC).
- ❌ Нет встроенного API для цепочек операций (`.thenApply`, `.thenCompose` и т. д.).
- ❌ Нет встроенной поддержки обработки исключений в цепочке.
- ❌ Требует сторонней зависимости (Guava), не является частью JDK.

**Пример**

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

`get()` внутри `listener` всё ещё блокирует поток исполнителя. Чтобы избежать этого, Guava предлагает `Futures.transform` с `AsyncFunction`.

**Улучшенный пример с `Futures.transform`**

```java
ListenableFuture<String> transformed = Futures.transform(
    future,
    input -> "Result: " + input,
    executor
);
```

## CompletableFuture

`java.util.concurrent.CompletableFuture` — полноценная реализация асинхронной модели с поддержкой цепочек, комбинирования, обработки ошибок и неблокирующих операций.

- Появился: Java 8.
- Пакет: `java.util.concurrent`.

**Ключевые особенности**

- Реализует `Future` и `CompletionStage`.
- Цепочки операций: `thenApply`, `thenAccept`, `thenRun`, `thenCompose`.
- Обработка ошибок: `exceptionally`, `handle`, `whenComplete`.
- Комбинирование: `allOf`, `anyOf`.
- Асинхронное выполнение с указанием `Executor`.
- Возможность явно завершить будущее: `complete()`, `completeExceptionally()`.
- Поддержка timeout: `orTimeout()`, `completeOnTimeout()` (Java 9+).

**Особенности**

- ✅ Встроен в JDK — никаких зависимостей.
- ✅ Мощный, выразительный API для асинхронного программирования.
- ✅ Неблокирующий по умолчанию (если использовать `*Async` методы).
- ✅ Хорошо интегрируется с реактивными фреймворками (Project Reactor, RxJava через адаптеры).
- ❌ Сложность для новичков (глубокие цепочки трудно отлаживать).
- ❌ Нет встроенной поддержки backpressure (в отличие от Reactive Streams).
- ❌ `CompletableFuture` — не `Publisher`, поэтому не подходит напрямую для реактивных пайплайнов.

**Пример**

```java
CompletableFuture.supplyAsync(() -> 42, executor)
    .thenApply(x -> x * 2)
    .thenAccept(System.out::println)  // выводит 84
    .exceptionally(ex -> {
        System.err.println("Error: " + ex.getMessage());
        return null;
    });
```

**Явное завершение**

```java
CompletableFuture<String> future = new CompletableFuture<>();
// где-то позже, например в колбэке от внешнего сервиса:
future.complete("done");
// или при ошибке:
future.completeExceptionally(new RuntimeException("timeout"));
```

## Сравнительная таблица

| Фича | `Future` | `ListenableFuture` (Guava) | `CompletableFuture` (Java 8+) |
| --- | --- | --- | --- |
| **JDK** | ✅ Да (с Java 5) | ❌ Нет (требует Guava) | ✅ Да (с Java 8) |
| **Асинхронные callbacks** | ❌ Нет | ✅ `addListener` | ✅ `thenApply`, `thenAccept`, `handle` и др. |
| **Цепочка операций** | ❌ | ⚠️ Через `Futures.transform` (ограниченно) | ✅ Полная поддержка (`thenCompose`, `allOf`) |
| **Обработка ошибок** | Только через `get()` → `ExecutionException` | `Futures.catching`, `transform` с `AsyncFunction` | ✅ `exceptionally`, `handle`, `whenComplete` |
| **Комбинирование** | ❌ | ✅ `Futures.allAsList`, `whenAllComplete` | ✅ `CompletableFuture.allOf`, `anyOf` |
| **Явное завершение** | ❌ | ✅ `SettableFuture` (наследник) | ✅ `complete()`, `completeExceptionally()` |
| **Неблокирующий** | ❌ (`get()` блокирует) | ⚠️ `addListener` асинхр., но `get()` внутри — блокирует | ✅ Все `*Async` методы неблокирующие |
| **Поддержка `Executor`** | При создании `FutureTask` или `submit` | В `addListener`, `transform` | В `supplyAsync`, `thenApplyAsync` и др. |
| **Рекомендуется в 2025+** | ❌ Только для совместимости | ⚠️ В legacy-проектах (Guava) | ✅ Стандарт де-факто |

## Выбор типа Future

| Сценарий | Рекомендуемый тип |
| --- | --- |
| Старый код, Java 6/7, уже используется Guava | `ListenableFuture` |
| Современный Java 8+ проект, нужна асинхронная логика | `CompletableFuture` |
| Нужно просто получить результат одной задачи и подождать | `Future` (но лучше `CompletableFuture.get()` — он тоже блокирует, но даёт больше гибкости) |
| Интеграция с Reactor или RxJava | `CompletableFuture` → преобразовать через `Mono.fromFuture()` или `Observable.fromFuture()` |

## CompletableFuture vs Reactive Streams

| Критерий | `CompletableFuture` | `Mono`/`Flux` (Reactor) |
| --- | --- | --- |
| Backpressure | ❌ Нет | ✅ Да |
| Поток данных (множество элементов) | ❌ Только 1 значение | ✅ Поддержка потоков |
| Операторы | Базовые (`map`, `flatMap`) | Расширенные (`retry`, `timeout`, `window`, `groupBy`) |
| Использование | Однократные асинхронные операции | Стримы событий, WebSocket, HTTP-стримы |

Для простых асинхронных операций (API-запросы, БД) достаточно `CompletableFuture`. Для сложных потоковых пайплайнов используются Reactive Streams (Project Reactor, RxJava).

## Итог

- `Future` — база, но устарела для асинхронного программирования.
- `ListenableFuture` — мост между старым и новым, полезен в Guava-экосистеме.
- `CompletableFuture` — современный стандарт для асинхронных операций в Java.

Для Java 11+ `CompletableFuture` + `ExecutorService` (или virtual threads в Java 21+) — стандарт асинхронного программирования.
