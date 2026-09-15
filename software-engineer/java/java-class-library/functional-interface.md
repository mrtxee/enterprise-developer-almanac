---
aliases:
  - Consumer
  - function
  - Function
  - function interface
  - functional interface
  - functional interfaces
  - Predicate
  - Runnable
  - SAM
  - Supplier
  - Поставщик
  - Потребитель
  - Предикат
  - функциональные интерфейсы
  - Функциональные интерфейсы
---

## Пакет java.util.function
> `java.util.function`

Функциональные интерфейсы предоставляют **целевые типы для лямбда-выражений и ссылок на методы**. Каждый функциональный интерфейс имеет один абстрактный метод, называемый **функциональным методом**, которому сопоставляются или адаптируются параметры и возвращаемые типы лямбда-выражения. Функциональные интерфейсы могут предоставлять целевой тип в нескольких контекстах: контекст назначения, вызов метода или контекст приведения.

**Пример использования**

```java
// Assignment context
Predicate<String> p = String::isEmpty;
// Method invocation context
stream.filter(e -> e.getSize() > 10)...
// Cast context
stream.map((ToIntFunction) e -> e.getSize())...
```

### Consumer — потребитель

Потребитель принимает на вход 1 параметр и ничего не возвращает.

```java
public void whenNamesPresentConsumeAll() {
  Consumer<String> printConsumer = t -> {
    if (Objects.equals(t, "New York")) {
      System.out.println("some usa city");
    } else {
      System.out.println(t);
    }
  };
  Stream<String> cities = Stream.of("Sydney", "Dhaka", "New York", "London");
  cities.forEach(printConsumer);
}
```

### Supplier — поставщик

У поставщика есть только метод `get()`. Он служит для возврата результата значений.

```java
public static void supplierWithOptional() {
  Supplier<Double> doubleSupplier = () -> Math.random();
  System.out.println(doubleSupplier.get());
}
```

### Predicate — предикат

Предикат — это утверждение, высказанное о субъекте. Более всего подходит для фильтра данных.

```java
public void testPredicate() {
  List<String> names = Arrays.asList("Smith", "Samueal", "Catley", "Sie");
  Predicate<String> nameStartsWithS = str -> str.startsWith("S");
  names.stream().filter(nameStartsWithS).forEach(System.out::println);
}
```

### Function — функция

Интерфейс `Function` применяет единый абстрактный метод **SAM**, который принимает аргумент типа `T` и выдаёт результат типа `R`. Один из распространённых вариантов использования — метод `Stream.map`.

```java
public void testFunctions() {
  List<String> names = Arrays.asList("Smith", "Gourav", "John", "Catania");
  Function<String, Integer> nameMappingFunction = String::length;
  // Function<String, Integer> nameMappingFunction = (str) -> str.length();
  List<Integer> nameLength = names.stream()
    .map(nameMappingFunction).collect(Collectors.toList());
  System.out.println(nameLength);
}
```

### Runnable — исполняемый

Интерфейс `Runnable` представляет любую лямбда-функцию как объект.

```java
final Runnable kzvlfn = () -> System.out.println("kzvlfn");
kzvlfn.run();
```

[[optional]]
