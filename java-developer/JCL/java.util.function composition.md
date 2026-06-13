---
aliases:
  - java.util.function composition
  - function composition
  - functional composition
  - compose
  - andThen
---
## 🔗 Композиция функций в Java: `andThen` vs `compose`

### 📌 Краткий ответ

> **Композиция функций** — объединение нескольких функций в одну цепочку, где выход одной функции становится входом другой.  
> **`andThen`** — выполняет сначала **текущую** функцию, потом **переданную**.  
> **`compose`** — выполняет сначала **переданную** функцию, потом **текущую**.

---

## 🧩 Как это работает

```
f.andThen(g)  →  g(f(x))   ← сначала f, потом g
f.compose(g)  →  f(g(x))   ← сначала g, потом f
```

### Визуализация:

```
┌─────────────────────────────────────────────────────────────┐
│  andThen:  input → [f] → [g] → output                       │
│            (слева направо, естественный порядок)            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  compose:  input → [g] → [f] → output                       │
│            (справа налево, математический порядок)          │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Сравнение `andThen` vs `compose`

| Характеристика | **`andThen`** | **`compose`** |
|----------------|---------------|---------------|
| **Порядок выполнения** | Сначала `this`, потом `other` | Сначала `other`, потом `this` |
| **Читаемость** | ✅ Более интуитивно (слева→направо) | ⚠️ Математическое (справа←налево) |
| **Математическая запись** | `g ∘ f` | `f ∘ g` |
| **Рекомендация** | ✅ Используйте по умолчанию | ⚠️ Когда нужна математическая нотация |

---

## 🧪 Примеры с `Function<T, R>`

### 1. **Базовое использование**

```java
Function<Integer, Integer> addOne = x -> x + 1;
Function<Integer, Integer> multiplyByTwo = x -> x * 2;

// ✅ andThen: сначала addOne, потом multiplyByTwo
Function<Integer, Integer> f1 = addOne.andThen(multiplyByTwo);
System.out.println(f1.apply(5));  // (5 + 1) * 2 = 12

// ✅ compose: сначала multiplyByTwo, потом addOne
Function<Integer, Integer> f2 = addOne.compose(multiplyByTwo);
System.out.println(f2.apply(5));  // (5 * 2) + 1 = 11
```

### 2. **Цепочка трансформаций (andThen)**

```java
// ✅ Читаемая цепочка обработки данных
Function<String, Integer> parse = Integer::parseInt;
Function<Integer, Integer> square = x -> x * x;
Function<Integer, String> toString = String::valueOf;

Function<String, String> pipeline = 
    parse.andThen(square).andThen(toString);

System.out.println(pipeline.apply("5"));  // "25"
```

### 3. **Валидация + Трансформация**

```java
Function<String, String> trim = String::trim;
Function<String, String> toUpperCase = String::toUpperCase;
Function<String, Optional<String>> validate = 
    s -> s.isEmpty() ? Optional.empty() : Optional.of(s);

// ✅ Цепочка: trim → toUpperCase → validate
Function<String, Optional<String>> pipeline = 
    trim.andThen(toUpperCase).andThen(validate);

System.out.println(pipeline.apply("  hello  "));  // Optional[HELLO]
System.out.println(pipeline.apply("     "));      // Optional.empty
```

---

## 🔄 Композиция с другими функциональными интерфейсами

### 1. **`Consumer<T>` (возвращает void)**

```java
Consumer<String> print = System.out::print;
Consumer<String> println = System.out::println;

// ✅ andThen работает (выполняются по порядку)
Consumer<String> consumer = print.andThen(println);
consumer.accept("Hello");  // "Hello\n"

// ❌ compose тоже работает, но менее интуитивно
Consumer<String> consumer2 = print.compose(println);
consumer2.accept("Hello");  // "\nHello"
```

### 2. **`Predicate<T>` (возвращает boolean)**

```java
Predicate<String> notNull = Objects::nonNull;
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> longerThan3 = s -> s.length() > 3;

// ✅ andThen для Predicates = logical AND
Predicate<String> valid = notNull.and(notEmpty).and(longerThan3);
System.out.println(valid.test("Hello"));  // true
System.out.println(valid.test(""));       // false

// ⚠️ compose для Predicates тоже = logical AND (но порядок обратный)
Predicate<String> valid2 = longerThan3.compose(notEmpty).compose(notNull);
System.out.println(valid2.test("Hello"));  // true
```

### 3. **`UnaryOperator<T>` (Function с одинаковыми типами)**

```java
UnaryOperator<Integer> addOne = x -> x + 1;
UnaryOperator<Integer> square = x -> x * x;

// ✅ Цепочка операций
UnaryOperator<Integer> op = addOne.andThen(square).andThen(addOne);
System.out.println(op.apply(3));  // ((3 + 1)²) + 1 = 17
```

---

## ⚠️ Распространённые ошибки

### 1. **Путаница порядка выполнения**

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;

// ❌ Ошибка: думают что compose = andThen
Function<Integer, Integer> h = f.compose(g);
System.out.println(h.apply(5));  // 11 (не 12!)
// compose: f(g(5)) = f(10) = 11
```

### 2. **`null` в цепочке**

```java
Function<String, Integer> parse = Integer::parseInt;
Function<Integer, Integer> square = x -> x * x;

// ❌ NullPointerException если parse вернёт null
Function<String, Integer> pipeline = parse.andThen(square);
pipeline.apply("abc");  // 💥 NumberFormatException

// ✅ Обработка ошибок
Function<String, Integer> safeParse = s -> {
    try { return Integer.parseInt(s); }
    catch (NumberFormatException e) { return 0; }
};
Function<String, Integer> safePipeline = safeParse.andThen(square);
safePipeline.apply("abc");  // 0
```

### 3. **Побочные эффекты в `compose`**

```java
// ⚠️ Сложно отладить порядок побочных эффектов
Function<Integer, Integer> log1 = x -> { System.out.println("log1: " + x); return x; };
Function<Integer, Integer> log2 = x -> { System.out.println("log2: " + x); return x; };

log1.compose(log2).apply(5);
// Вывод:
// log2: 5  ← выполняется первым!
// log1: 5
```

---

## 📋 Когда что использовать

| Сценарий | Рекомендация | Почему |
|----------|--------------|--------|
| **Цепочка трансформаций** | ✅ `andThen` | Читается слева направо |
| **Математическая композиция** | ✅ `compose` | Соответствует `f ∘ g` |
| **Валидация + обработка** | ✅ `andThen` | Логический порядок |
| **Преобразование входа** | ✅ `compose` | Сначала подготовь данные |
| **Логирование/метрики** | ✅ `andThen` | После основной логики |

---

## 🎯 Практические паттерны

### 1. **Pipeline обработки данных**

```java
// ✅ Чистый pipeline с andThen
Function<String, User> pipeline = 
    String::trim
        .andThen(String::toLowerCase)
        .andThen(this::findUser)
        .andThen(this::enrichUserData);
```

### 2. **Декоратор с логированием**

```java
// ✅ Логирование до и после
Function<Integer, Integer> logged = 
    Logging.logInput("Input")
        .andThen(businessLogic)
        .andThen(Logging.logOutput("Output"));
```

### 3. **Кэширование + вычисление**

```java
// ✅ Сначала кэш, потом вычисление
Function<String, Integer> cached = 
    cache::getOrDefault
        .compose(this::computeKey)  // сначала вычисляем ключ
        .andThen(this::computeValue); // потом вычисляем значение
```

---

## 📦 Stream API + Композиция

```java
// ✅ Композиция в Stream
List<Integer> result = numbers.stream()
    .map(String::valueOf)
    .map(String::length)
    .map(x -> x * 2)
    .toList();

// ✅ Или через композицию функций
Function<Integer, Integer> pipeline = 
    String::valueOf
        .andThen(String::length)
        .andThen(x -> x * 2);

List<Integer> result = numbers.stream()
    .map(pipeline)
    .toList();
```

---

## 📌 Памятка

```
┌─────────────────────────────────────────────────────────────┐
│  Композиция функций                                         │
│                                                             │
│  f.andThen(g)  →  g(f(x))  ←  сначала f, потом g           │
│  f.compose(g)  →  f(g(x))  ←  сначала g, потом f           │
│                                                             │
│  ✅ andThen:  читаемее, естественный порядок                │
│  ✅ compose:  математическая нотация                        │
│                                                             │
│  ⚠️ Осторожно: порядок выполнения, null, побочные эффекты  │
│  ✅ Рекомендуется: andThen по умолчанию                     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Итог

| Вопрос | Ответ |
|--------|-------|
| **Что такое композиция?** | Объединение функций в цепочку |
| **В чём разница `andThen` vs `compose`?** | Порядок выполнения (прямой vs обратный) |
| **Что использовать по умолчанию?** | ✅ `andThen` (читаемее) |
| **Работает ли с `Consumer`/`Predicate`?** | ✅ Да, но семантика отличается |
| **Можно ли комбинировать?** | ✅ Да, но аккуратно с порядком |

> 💡 **Совет:** Используйте **`andThen`** для большинства случаев — код читается как пайплайн слева направо. **`compose`** оставьте для математических задач или когда нужно сначала трансформировать вход.