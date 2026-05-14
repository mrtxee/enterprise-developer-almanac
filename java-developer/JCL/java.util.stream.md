---
aliases:
  - Stream API
  - Stream
---
>[!info] `java.util.stream`
# Методы Java Stream API

#### Промежуточные (intermediate) методы

Возвращают новый поток (`Stream<T>`), позволяя выстраивать цепочки операций. Выполняются лениво — только при вызове терминальной операции.

1. `filter(Predicate<? super T> predicate)` — фильтрация элементов по условию.
2. `map(Function<? super T, ? extends R> mapper)` — преобразование каждого элемента.
3. `flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)` — преобразование и «разворачивание» вложенных потоков.
4. `distinct()` — удаление дубликатов (на основе `equals()`).
5. `sorted()` — сортировка элементов (по естественному порядку).
6. `sorted(Comparator<? super T> comparator)` — сортировка с использованием компаратора.
7. `limit(long maxSize)` — ограничение количества элементов.
8. `skip(long n)` — пропуск первых `n` элементов.
9. `peek(Consumer<? super T> action)` — выполнение действия для отладки/логирования (не изменяет элементы).
10. `parallel()` — преобразование в параллельный поток.
11. `sequential()` — преобразование в последовательный поток.
12. `unordered()` — снятие требования упорядоченности (может повысить производительность).

#### Терминальные (terminal) методы
Завершают работу с потоком, возвращают результат или выполняют действие. После вызова терминальной операции поток нельзя использовать повторно.

1. `forEach(Consumer<? super T> action)` — выполнение действия для каждого элемента.
2. `forEachOrdered(Consumer<? super T> action)` — выполнение действия с сохранением порядка элементов (даже в параллельном потоке).
3. `toArray()` — преобразование потока в массив `Object[]`.
4. `toArray(IntFunction<A[]> generator)` — преобразование потока в массив заданного типа.
5. `reduce(BinaryOperator<T> accumulator)` — свёртка элементов с использованием бинарного оператора (возвращает `Optional<T>`).
6. `reduce(T identity, BinaryOperator<T> accumulator)` — свёртка с начальным значением (возвращает `T`).
7. `reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)` — свёртка с преобразованием типа (для параллельных потоков).
8. `collect(Collector<? super T,A,R> collector)` — сбор элементов в коллекцию/структуру данных (например, `Collectors.toList()`, `Collectors.toSet()`).
9. `min(Comparator<? super T> comparator)` — поиск минимального элемента (возвращает `Optional<T>`).
10. `max(Comparator<? super T> comparator)` — поиск максимального элемента (возвращает `Optional<T>`).
11. `count()` — подсчёт количества элементов (возвращает `long`).
12. `anyMatch(Predicate<? super T> predicate)` — проверка, есть ли хотя бы один элемент, удовлетворяющий условию (возвращает `boolean`).
13. `allMatch(Predicate<? super T> predicate)` — проверка, удовлетворяют ли все элементы условию (возвращает `boolean`).
14. `noneMatch(Predicate<? super T> predicate)` — проверка, что ни один элемент не удовлетворяет условию (возвращает `boolean`).
15. `findFirst()` — получение первого элемента (возвращает `Optional<T>`; сохраняет порядок).
16. `findAny()` — получение любого элемента (возвращает `Optional<T>`; в параллельном потоке может быть быстрее).

---

**Важные замечания:**
* Промежуточные операции делятся на **stateless** (не зависят от других элементов: `filter`, `map`) и **stateful** (требуют информации о других элементах: `sorted`, `distinct`).
* Терминальная операция **обязательно** должна присутствовать в цепочке — без неё промежуточные операции не выполняются.
* После терминальной операции поток **закрывается** — попытка использовать его повторно вызовет исключение `IllegalStateException`.

Хотите, разберу какой‑то метод подробнее или приведу примеры использования?

# Java Stream API

Пакет `java.util.stream` содержит классы для поддержки операций с потоками элементов в функциональном стиле. Ключевой абстракцией, введенной в этом пакете, является Поток.
`Stream API` – по сути это поток данных и последовательные операции над ними.
Интерфейсы Stream, IntStream, LongStream и DoubleStream – это потоки объектов и примитивных типов int, long и double.
## Создание потока
```Java
List<Integer> numbers = new ArrayList<>();
numbers.stream().mapToInt(value -> value);
numbers.stream().mapToDouble(value -> value);     
numbers.stream().mapToLong(value -> value*196500);
List<String> list = new ArrayList<>();
list.stream();
list.parallelStream();           // параллельный поток
Map<String, String> map = new HashMap<>();
map.entrySet().stream();
map.values().stream();
String[] array = new String[10];
Arrays.stream(array);
Stream.of("a", "b", "c");        // поток из элементов
Stream.of(array);                // поток из элементов массива
Stream.of(list);                 // поток из элементов списка List
Stream.generate(Math::random);   // генерация потока рандомных чисел
Stream.concat(stream1, stream2); // объединяет два потока в один
IntStream.range(1, 10);          // поток диапазона чисел от 1 до 9
IntStream.rangeClosed(1, 10);    // поток диапазона чисел от 1 до 10
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
bufferedReader.lines();
Path path = Path.of("/root/test.txt");
Files.lines(path);
Random random = new Random();
random.ints();
random.longs();
random.doubles();
```
## Intermediate Промежуточные операции потока
Они же **отложенные** операции. Выполнятся когда их запустит конечная терминальная операция над стримом
```Java
//привести данные к потоковому типу, map()
Stream.of(1, 2, 3).map((x) -> String.valueOf(x));
Stream.of(1, 2, 3).map(String::valueOf);// лямбда выражение
Stream.of("1", "2", "3").map(Integer::parseInt);
Arrays.stream(int[] arr);
//Отфильтровать элементы filter()
Stream.of(1, 2, 3, 4, 5).filter(n -> n < 4);    // [1, 2, 3]
//Удалить дублирующиеся элементы distinct()
Stream.of(1, 2, 3, 2, 4, 2, 5).distinct();      // [1, 2, 3, 4, 5]
//Лимит количества элементов limit()
Stream.of(1, 2, 3, 4, 5, 6).limit(3);            // [1, 2, 3]
//Сортировка и обратная сортировка элементов sorted()
Stream.of(4, 2, 3, 5, 1).sorted();              // [1, 2, 3, 4, 5]
Stream.of(4, 2, 3, 5, 1).sorted(Comparator.reverseOrder())
//Пропустить первые элементы skip()
Stream.of(1, 2, 3, 4, 5).skip(2);                // [3, 4, 5]
// привязать функцию к каждому элементу, которая будет исполнена во вр обхода списка
// Как foreach, но не терминальная
Stream.of("a", "b", "c").peek(System.out::println);
// слить 2мерный массив в 1
Integer[][] array2d = new Integer[][] { {1, 2, 3},{4, 5} };
Arrays.stream(array2d).flatMap(Arrays::stream);    // [1, 2, 3, 4, 5]
// Stream over int[] array Ex
  int[] nums3 = new int[]{1,2,3,4,5,6,7,8,9};
  int[] coll3 = Arrays.stream(nums3)
          .skip(2)
          .limit(4)
          .filter(v->v<8&&v>0)
          .map(v->v*12)
          .toArray();
  System.out.println(Arrays.toString(coll3));
// вывести в строку через запятую
String str = transitions.keySet().stream().map(ch -> ch + "->" + transitions.get(ch)).collect(Collectors.joining("; "));
String str = outputStack.stream().map(Lexem::getValue).collect(Collectors.joining(","));
```
## Terminal Конечные операции
Запускают всю цепь промежуточных операций и возвращают конечный результат, закрывают поток.
```Java
//Собрать элементы потока и преобразовать их к нужному типу collect(Collector с)
List<String> collect = Stream.of("a", "b", "c").collect(Collectors.toList());
String collect = Stream.of("a", "b", "c").collect(Collectors.joining());
//Итерация по каждому элементу forEach()
Stream.of("a", "b", "c").forEach(System.out::println);
Stream.of("a", "b", "c").count();
Optional<Integer> max = Stream.of(4, 2, 3, 5, 1)
        .max(Comparator.naturalOrder());
Integer maximum = max.get();
Integer minimum = Stream.of(4, 2, 3, 5, 1)
        .min(Comparator.naturalOrder())
        .get();
//
Stream.of("a", "bb", "ccc")
        .min((s1, s2) -> s1.length() - s2.length())
        .get();
Stream.of("a", "bb", "ccc")
        .max(Comparator.comparingInt(String::length))
        .get();
```
### Сборка потока
Терминальный метод `collect()` интерфейса Stream служит для того, чтобы перейти от потоков к привычным коллекциям — `List<T>`, `Set<T>`, `Map<T, R>` и другим.
В метод `collect()` нужно передать специальный объект — `collector`.
### Класс Collectors
`import java.util.stream.Collectors;`
Коллектор имплиментирет интерфейс `public interface Collector<T,A,R>`. Последний тип `R` — это обычно и есть тип вроде `List<T>`. Поэтому компилятор может по этому типу подставить правильный тип результата самого метода `collect()`.
Обычно можно воспользоваться уже готовыми объектами, которые возвращают статические методы класса `Collectors`.

|   |   |
|---|---|
|`toList()`|Объект, который преобразует поток в список — `List<T>`|
|`toSet()`|Объект, который преобразует поток во множество — `Set<T>`|
|`toMap()`|Объект, который преобразует поток в мэп — `Map<K, V>`|
|`joining()`|Склеивает элементы потока в одну строку|
|`mapping()`|Преобразует элементы потока в `Map<K, V>`|
|`groupingBy()`|Группирует элементы, возвращает `Map <K, V>`|
```Java
ArrayList<String> list = new ArrayList<String>();
Collections.addAll(list, "a=2", "b=3", "c=4", "d==3");
Map<String, String> result = list.stream()
   .map( e -> e.split("=") )
   .filter( e -> e.length == 2 )
   .collect( Collectors.toMap(e -> e[0], e -> e[1]) );
```

---
