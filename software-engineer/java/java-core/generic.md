---
aliases:
  - contravariance
  - contravariant
  - covariance
  - covariant
  - generic
  - generic type
  - Generic Types
  - generics
  - invariance
  - invariant
  - Java Generics
  - multiple bounds
  - parameterized type
  - PECS
  - raw type
  - type parameter
  - unbounded wildcard
  - wildcard
  - Wildcard Capture
  - вайлдкарта
  - дженерик
  - дженерики
  - инвариантность
  - ковариантность
  - контравариантность
  - Общий тип данных
  - сырой тип
---

## Generics — общий тип данных

Дженерики (общие типы) развивают принцип строгой типизации Java, позволяя задать допустимый тип объектов коллекции. Для типа коллекции дженериков соблюдается принцип подстановки Барбары Лисков #👨.

**Пример дженериков**

```java
Number n = Integer.valueOf(42);
List<Number> aList = new ArrayList<>();
Collection<Number> aCollection = aList;
Iterable<Number> iterable = aCollection;
// Generic version of the Box class.
// @param <T> the type of the value being boxed
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
public interface Pair<K, V> {
    public K getKey();
    public V getValue();
}
public class OrderedPair<K, V> implements Pair<K, V> {
    private K key;
    private V value;
    public OrderedPair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    public K getKey() { return key; }
    public V getValue() { return value; }
}
Pair<String, Integer> p1 = new OrderedPair<String, Integer>("Even", 8);
Pair<String, String> p2 = new OrderedPair<String, String>("hello", "world");
// Parameterized Types
OrderedPair<String, Box<Integer>> p = new OrderedPair<>("primes", new Box<Integer>(...));
```

**Элементы дженерика являются инвариантными**, то есть не допускают приведение типов к родительскому либо дочернему.

Дженерики могут включать только ссылочные типы. Если дженерик задан типом `<?>`, он допускает подстановку любого ссылочного типа — наследника класса `Object`. Это короткая запись для вайлдкарты `<? extends Object>`.

## Неограниченная вайлдкарта

**`<? extends Object>` = `<?>`**

Принимает любые объекты ссылочного типа, так как все объекты наследуются от класса `Object`.

**Пример эквивалентности**

```java
Collection<?> // равносильно Collection<? extends Object>
Collection<? extends Object> // равносильно Collection<?>
```

## Сырой тип — Raw Type

`Raw Type` позволяет объявить дженерик с параметром неопределенного типа.

Сырые типы плохи тем, что отключают проверку типа переменной на этапе компиляции и повышают риск `ClassCastException` во время выполнения. Если есть возможность, сырые типы использовать не стоит.

**Пример сырого типа**

```java
List<String> fruits = new ArrayList<>(); // список параметризован типом <String>
List fruits = new ArrayList<>(); // список с объектами Raw type
fruits.add("apple");
fruits.add("pear");
fruits.add(1);
// для Raw type компилятор использует меньше проверок, поэтому повышается риск
// поймать ошибку не на компиляции, а на выполнении, поэтому лучше вместо сырого
// типа использовать параметризованный тип <Object>
List<Object> fruits2 = new ArrayList<>();
```

Сырые типы в Java есть только для обеспечения совместимости с Java 5, в новом коде их использовать не следует.

## Конвенция об именах

Наиболее часто используемые имена параметров типа:
- **E** — Element (используется в [[java-collection-framework|Java Collections Framework]]);
- **K** — Key;
- **N** — Number;
- **T** — Type;
- **V** — Value;
- **S, U, V** и т. д. — 2-й, 3-й, 4-й типы.

## Вайлдкарты

Вайлдкарты — запись, которая позволяет сделать тип элементов дженерика ковариантным либо контравариантным. Для этого используются вайлдкарты `extends`, `super`.

**Пример ковариантности и контравариантности**

```java
// Ковариантный дженерик
List<Integer> ints = new ArrayList<Integer>();
List<? extends Number> nums = ints;
// Контравариантный дженерик
List<Number> nums = new ArrayList<Number>();
List<? super Integer> ints = nums;
```

Вайлдкарты используют, когда точный тип добавляемых в контейнер объектов неизвестен и нужно избежать применения сырого типа: сырой тип всегда плох, так как не происходит проверки типов на этапе компиляции.

### Ковариантность, контравариантность и инвариантность

Термины ковариантность, контравариантность и инвариантность описывают, сохраняется ли иерархия наследования исходных типов в производных типах.

- **Ковариантность** — wildcard `? extends`, верхняя граница. Иерархия исходных типов сохраняется в том же порядке: `Множество<Кошки>` — подтип `Множество<Животные>`.
  - используется, когда необходимо читать из контейнера — `producer`;
    - метод `get()`.
- **Контравариантность** — wildcard `? super`, нижняя граница. Иерархия исходных типов в производных типах обращается: `Множество<Животные>` — подтип `Множество<Кошки>`.
  - используется, когда необходимо писать в контейнер — `consumer`;
    - метод `set()`.
- **Инвариантность** — без wildcard. Наследование между производными типами отсутствует: `Множество<Животные>` и `Множество<Кошки>` не связаны отношением подтипов.
  - wildcard не используется, если нужно производить и запись, и чтение.

Если wildcard объявлен с `extends` — это `producer`: он только предоставляет элемент из контейнера, а сам ничего не принимает. Если wildcard объявлен с `super` — это `consumer`: он только принимает, а предоставить ничего не может.

**PECS — Producer Extends, Consumer Super** — запоминалка принципа.

**Пример PECS**

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    // ...
}
```

- Если необходимо читать из контейнера — используйте wildcard с верхней границей `? extends T`.
- Если необходимо писать в контейнер — используйте wildcard с нижней границей `? super T`.
- Если нужно производить чтение и запись — не используйте wildcard `<>`.

### Множественные ограничения — Multiple Bounds

`Multiple Bounds` — множественные ограничения. Записываются через символ `&`: тип, представленный переменной типа `T`, должен быть ограничен сверху классом `Object` и интерфейсом `Comparable`.

**Пример множественных ограничений**

```java
<T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll)
```

#### Символ подстановки

Служит, чтобы передать в метод аргумент с дженериком произвольного типа. Не применяется при создании объектов, только при передаче объекта в метод. `<?>` означает неопределенный или любой тип, а значит, компилятор не поймет, какого типа переменная передается в структуру. Так как структуры работают со ссылочными типами, любая переменная ссылочного типа является наследником `Object`.

**Пример символа подстановки**

```java
public static void wildAdder(List<?> list) {
    Object o = list.get(0);
    list.add("another string"); // Ошибка компиляции
}
// Эквивалентно:
public static void wildAdder(List<? extends Object> list)
```

### Паттерн Wildcard Capture

Это способ записывать данные в продюсер без нарушения принципа PECS.

Здесь происходит захват символа подстановки (wildcard capture). При вызове метода `reverse(List<?> list)` в качестве аргумента передается список объектов. Если захватить тип этих объектов и присвоить его переменной типа `X`, то можно заключить, что `T` является `X`.

**Пример Wildcard Capture**

```java
public static void reverse(List<?> list) {
  rev(list);
}
private static <T> void rev(List<T> list) {
  List<T> tmp = new ArrayList<T>(list);
  for (int i = 0; i < list.size(); i++) {
    list.set(i, tmp.get(list.size() - i - 1));
  }
}
```

## Часто задаваемые вопросы

- Можно ли кастить к дженерику?
  - Да.
