---
aliases:
  - функциональные интерфейсы
  - function
  - functional interface
  - function interface
---
>[!info] `java.util.function`
# функциональные интерфейсы
Функциональные интерфейсы предоставляют ==целевые типы для лямбда-выражений и ссылок на методы==. Каждый функциональный интерфейс имеет один абстрактный метод, называемый **функциональным методом** для этого функционального интерфейса, которому сопоставляются или адаптируются параметры и возвращаемые типы лямбда-выражения. Функциональные интерфейсы могут предоставлять целевой тип в нескольких контекстах, таких как контекст назначения, вызов метода или контекст приведения.
```Java
// Assignment context
Predicate<String> p = String::isEmpty;
// Method invocation context
stream.filter(e -> e.getSize() > 10)...
// Cast context
stream.map((ToIntFunction) e -> e.getSize())...
```
## Consumer — потребитель
Потребитель принимает на вход 1 параметр, ничего не в возвращает
```Java
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
## Supplier — поставщик
У поставщика есть только метод get(). Он служит для возврата результата занчений.
```Java
public static void supplierWithOptional() {
    Supplier<Double> doubleSupplier = () -> Math.random();
    System.out.println(doubleSupplier.get());
}
```
## Predicate — предикат
Предикат – это утверждение , высказанное о субъекте. Более всего подходит для фильтра данных.
```Java
public void testPredicate() {
    List<String> names = Arrays.asList("Smith", "Samueal", "Catley", "Sie");
    Predicate<String> nameStartsWithS = str -> str.startsWith("S");
    names.stream().filter(nameStartsWithS).forEach(System.out::println);
}
```
## Function — функция
Интерфейс `Function` — применяется единый абстрактный метод **SAM**, который принимает аргумент типа T и выдает результат типа R. Одним из распространенных вариантов использования этого интерфейса является метод Stream.map.
```Java
public void testFunctions() {
    List<String> names = Arrays.asList("Smith", "Gourav", "John", "Catania");
    Function<String, Integer> nameMappingFunction = String::length;
		//Function<String, Integer> nameMappingFunction = (str) -> str.length();
    List<Integer> nameLength = names.stream()
      .map(nameMappingFunction).collect(Collectors.toList());
    System.out.println(nameLength);
}
```
## Runnable — исполняемй
Интерфейс `Runnable` — представляет любую лямбда функцию, как объект.
```Java
final Runnable kzvlfn = () -> System.out.println("kzvlfn");
kzvlfn.run();
```
[[java.util.Optional]]