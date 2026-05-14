---
aliases:
  - Generics
  - Generic Types
  - Общий тип данных
---
_Java Generics, Generic Types, Общий тип данных_
## Generics — общий тип данных
Дженерики (общие типы) развивают принцип строгой типизации Java, позволяя задать допустимый тип объектов коллекции. Для типа коллекции дженериков соблюдается принцип подстановок Барбары Лисков.
```Java
Number n = Integer.valueOf(42);
List<Number> aList = new ArrayList<>();
Collection<Number> aCollection = aList;
Iterable<Number> iterable = aCollection;
/**
 * Generic version of the Box class.
 * @param <T> the type of the value being boxed
 */
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
    public K getKey()	{ return key; }
    public V getValue() { return value; }
}
Pair<String, Integer> p1 = new OrderedPair<String, Integer>("Even", 8);
Pair<String, String>  p2 = new OrderedPair<String, String>("hello", "world");
//Parameterized Types
OrderedPair<String, Box<Integer>> p = new OrderedPair<>("primes", new Box<Integer>(...));
```
**Элементы дженерика являются инвариантными**. Т.е. не допускают приведение типов к родительскому либо дочернему.
Дженерики бывают могут включать только ссылочные типы. Если дженерик задан типа `<?>`, то он допускает подстановку любого ссылочного типа, наследника класса Object. Это короткая запись для вайлдкарты `<? extends Object>`.

## `<?>` — unbounded wildcard — неограниченный тип

**`<? extends Object> **= <?>`**
Принимает любые объекты ссылочного типа, так как все объекты наследуются от класса `Object`
```Java
Collection<?> // равносильно Collection<? extends Object>
Collection<? extends Object> // Collection<?>
```
## Сырой тип Raw type `<*>`
`Raw type` позволяет объявить с параметром неопределённого типа.  
**Сырые типы плохи, тем, что отключают проверку типа переменной на этапе компиляции, повышают риск** **ClassCastException at runtime**. Если есть возможность — не используй сырые типы.**
```Java
List<String> fruits = new ArrayList<>(); // список параметризован типом <String>
List fruits = new ArrayList<>(); // список с объектами Raw type
fruits.add("apple");
fruits.add("pear");
fruits.add(1);
// для Raw type компилятор использует меньше проверок, поэтому повышается риск поймать ошибку не на компиляции, а на выполнении, 
// поэтому лучше вместо сырого типа использовать параметризованный тип <Object>
List<Object> fruits2 = new ArrayList<>();
```
Сырые типы использовать в java не надо они там есть, только для обеспечения совместимости с java 5.
## Конвенция об именах
The most commonly used type parameter names are:  
**E** - Element (used extensively by the Java Collections Framework)  
**K** - Key  
**N** - Number  
**T** - Type  
**V** - Value  
**S,U,V** etc. - 2nd, 3rd, 4th types
# Wildcards
Wildcards - запись, которая позволяет сделать тип элементов дженерика ковариантным, либо контравариантными. Для этого используются вайлдкарты `extends`, `super`.
```Java
// Ковариантный дженерик
List<Integer> ints = new ArrayList<Integer>();
List<? extends Number> nums = ints;
// Контрвариантный дженерик
List<Number> nums = new ArrayList<Number>();
List<? super Integer> ints = nums;
```
Вайлдкарты дженериков надо использовать, когда мы не знаем, какого типа будем добавлять объекты в контейнер и хотим избежать применения сырого типа. Сырой тип всегда плохо, т.к. не происходит проверки типов на этапе компиляции.
## Ковариантность, контравариантность и инвариантность
_—_ иерархии наследования
- **Ковариантность** `**wildcard: ? extends**` верхняя граница
    - если необходимо читать из контейнера, **`producer`**
        - метод `set()`
    - _**Множество<Животные> ⊂ Множество<Кошки>**_
        - это сохранение иерархии наследования исходных типов в производных типах в том же порядке  
            _Множество<Кошки> — это подтип Множество<Животные>_
- **Контравариантность** `**wildcard: ? super**` нижняя граница
    - если необходимо писать в контейнер, **`consumer`**
        - метод `get()`
    - **_Множество<Кошки> ⊂ Множество<Животные>_**
        - это обращение иерархии исходных типов на противоположную в производных типах.
- **Инвариантность** `**no wildcard**`
    - не используйте wildcard, если нужно производить и запись, и чтение
    - _Множество<Животные> ⊄ Множество<Кошки>_
        - отсутствие наследования между производными типами.
Если мы объявили _wildcard с_ `_extends_`, то это `_producer_`. Он только предоставляет элемент из контейнера, а сам ничего не принимает.
Если же мы объявили _wildcard с_ `_super_` — то это `_consumer_`. Он только принимает, а предоставить ничего не может.
**PECS —** _**Producer Extends Consumer Super**,_ запоминалка принципа, _Производитель расширяет возможности потребителя_.
```Java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
	…
}
```
- Если необходимо **читать из контейнера**, то используйте wildcard с верхней границей `**? extends T**`
- Если необходимо **писать в контейнер**, то используйте wildcard с нижней границей `**? super T**`
- Если нужно производить **чтение и запись** — не используйте wildcard **`<>`**
## Multiple bounds — множественные ограничения
`Multiple Bounds` – множественные ограничения. Записывается через символ "`&`", то есть мы говорим, что тип, представленный переменной типа `T`, должен быть ограничен сверху классом `Object` и интерфейсом `Comparable`.
```Java
<T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll)
```
### Cимвол подстановки <?>
Служит, чтобы сообщить в метод аргумент с дженериком произвольного типа. Не применяется при создании объектов, только при передачи объекта в метод. `<?>` означает неопределённый или любой тип, а значит компилятор не поймет какого типа переменная передается в структуру. Так, как структуры работают с ссылочными типами, любя переменная ссылочного типа является наследником `Object`.
```Java
public static void wildAdder(List<?> list) {
    Object o = list.get(0);
    list.add("another string");//Ошибка компиляции
}
// Эквивалентно :
public static void wildAdder(List<? extends Object> list)
```
## Паттерн Wildcard Capture
Это способ записывать данные в продюсер, без нарушения принципа PECS
Здесь произошел захват символа подстановки (wildcard capture). При вызове метода reverse(List<?> list) в качестве аргумента передается список объектов. Если мы можем захватить тип этих объектов и присвоить его переменной типа X, то можем заключить, что T является Xы
```Java
public static void reverse(List<?> list) { 
  rev(list); 
}
private static <T> void rev(List<T> list) {
  List<T> tmp = new ArrayList<T>(list);
  for (int i = 0; i < list.size(); i++) {
    list.set(i, tmp.get(list.size()-i-1));
  }
}
```
# faq
- Можно ли кастить к дженерику?
	- да