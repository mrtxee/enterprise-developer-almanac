---
aliases:
  - lombok
---
## Lombok
Lombok — это библиотека для Java, которая с помощью аннотаций автоматически генерирует шаблонный код (геттеры, сеттеры, конструкторы, `equals()`, `hashCode()`, `toString()` и т. д.) на этапе компиляции — это сокращает объём рутинных записей и делает код чище и компактнее.

---
## Lombok Annotaion Types

1. *constructors*
	1. AllArgsConstructor
	2. RequiredArgsConstructor
	3. NoArgsConstructor
	4. Builder
		1. Builder.Default
		2. Builder.ObtainVia
		3. Singular
2. *fields*
	1. Cleanup
	2. EqualsAndHashCode
		1. EqualsAndHashCode.Exclude
		2. EqualsAndHashCode.Include
	3. ToString
		1. ToString.Exclude
		2. ToString.Include
	4. Setter
	5. Getter
	6. NonNull
	7. Data
	8. Value
	9. val
	10. var
3. *concurrency*
	1. Synchronized
	2. Locked
		1. Locked.Read
		2. Locked.Write
4. Log
	1. CommonsLog
	2. Flogger
	3. JBossLog
	4. Log
	5. Log4j
	6. Log4j2
	7. Slf4j
	8. XSlf4j
	9. CustomLog 
5. Generated
6. SneakyThrows
7. With
## @Value

Аннотация `@Value` — это «неизменяемая» версия `@Data`. Она создаёт **иммутабельные** (неизменяемые) классы.

**Что генерирует:**
* финальный класс (`final class`);
* приватные финальные поля (`private final`);
* конструктор со всеми полями;
* геттеры (но не сеттеры);
* `equals()`, `hashCode()` и `toString()`.

**Пример:**

```java
import lombok.Value;

@Value
public class User {
    String name;
    int age;
    String email;
}
```

**Сгенерированный код (эквивалент):**

```java
public final class User {
    private final String name;
    private final int age;
    private final String email;

    public User(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
    public String getEmail() { return email; }

    @Override
    public boolean equals(Object o) { /* реализация */ }
    @Override
    public int hashCode() { /* реализация */ }
    @Override
    public String toString() { /* реализация */ }
}
```

**Как использовать:**

```java
User user = new User("Анна", 25, "anna@mail.com");
System.out.println(user); // User(name=Анна, age=25, email=anna@mail.com)

// user.setName("Мария"); // Ошибка компиляции — нет сеттера
```

**Когда применять:**
* DTO (Data Transfer Objects);
* value‑объекты;
* конфигурации;
* везде, где нужна неизменяемость для потокобезопасности.
### record vs @Value

**Lombok `@Value`** — аннотация, которая генерирует код на этапе компиляции: создаёт неизменяемый класс с геттерами, `equals()`, `hashCode()` и `toString()`.

**Java `record`** (с Java 14) — языковая конструкция для объявления неизменяемых классов‑данных. Компилятор автоматически генерирует поля, конструктор, геттеры и переопределённые методы `equals()`, `hashCode()`, `toString()`.

| Характеристика    | `@Value`                                                                   | `record`                                                                                                      |
| ----------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Версия Java**   | Любая (с поддержкой Lombok)                                                | Java 14+                                                                                                      |
| **Изменяемость**  | Полностью неизменяемый (`final` класс и поля)                              | Полностью неизменяемый (неявный `final`)                                                                      |
| **Наследование**  | Может наследоваться от других классов                                      | Не может наследоваться; не может быть расширен                                                                |
| **Кастомизация**  | Высокая (можно добавлять методы, конструкторы, логику)                     | Ограниченная (можно добавлять статические методы, статические поля, компакт‑конструкторы)                     |
| **Методы**        | Автоматически генерируются `equals()`, `hashCode()`, `toString()`, геттеры | Автоматически генерируются `equals()`, `hashCode()`, `toString()`, «геттеры» (методы‑акцессоры с именем поля) |
| **Конструкторы**  | Генерируется один конструктор со всеми полями                              | Автоматически создаются канонический конструктор и компактный конструктор                                     |
| **Поля**          | Приватные финальные (`private final`)                                      | Приватные финальные, неявные                                                                                  |
| **Геттеры**       | Методы вида `getName()`, `getAge()`                                        | Методы с именем поля: `name()`, `age()`                                                                       |
| **Сериализация**  | Поддерживается (можно добавить `implements Serializable`)                  | Поддерживается                                                                                                |
| **IDE‑поддержка** | Требуется плагин Lombok                                                    | Встроенная поддержка                                                                                          |
| **Зависимости**   | Требуется зависимость Lombok                                               | Нет дополнительных зависимостей                                                                               |

**Производительность**

* **`record`**: немного быстрее из‑за отсутствия overhead от обработки аннотаций.
* **`@Value`**: есть небольшой overhead на этапе компиляции из‑за генерации кода Lombok.

На практике разница в производительности обычно незначительна и не должна быть решающим фактором при выборе.

**Ограничения**

**Ограничения `@Value`:**
* зависимость от Lombok;
* необходимость настройки IDE (плагин);
* возможная несовместимость с некоторыми инструментами анализа кода.

**Ограничения `record`:**
* только Java 14+;
* нельзя наследоваться или расширять;
* ограниченная кастомизация (нельзя добавить нестатические поля);
* геттеры с именами полей (`name()`), а не `getName()`.

---

## val

Ключевое слово `val` позволяет объявлять **локальные финальные переменные** с выводом типа.

**До Lombok:**

```java
final HashMap<String, List<String>> map = new HashMap<>();
final String name = "Иван";
final List<User> users = Arrays.asList(new User("Петр", 30, "p@mail.com"));
```

**С Lombok `val`:**

```java
import lombok.val;

val map = new HashMap<String, List<String>>();
val name = "Иван";
val users = Arrays.asList(new User("Петр", 30, "p@mail.com"));
```

**Что происходит:**
* переменная автоматически становится `final`;
* тип выводится из правой части выражения;
* попытка изменить переменную вызовет ошибку компиляции.

**Пример ошибки:**

```java
val x = 10;
x = 20; // Ошибка компиляции: cannot assign a value to final variable x
```

---

## var

`var` похож на `val`, но **не делает переменную финальной** — её можно переназначать.

Доступен  версии Lombok 1.18.16+

**Пример:**

```java
import lombok.var;

var counter = 0;
counter++; // Работает

var list = new ArrayList<String>();
list.add("item1"); // Работает
list = new ArrayList<>(); // Можно переназначить
```

**Сравнение `val` и `var`:**

| Характеристика | `val` | `var` |
|-------------|-------|-------|
| Финалность | Да (`final`) | Нет |
| Переприсвоение | Запрещено | Разрешено |
| Аналог в Java | `final Type name` | `Type name` |
| Вывод типа | Да | Да |

**Примеры использования:**

```java
// val — для неизменяемых значений
val connection = Database.getConnection();
val config = loadConfiguration();

// var — когда нужно менять значение
var attempts = 3;
while (attempts > 0) {
    try {
        processData();
        break;
    } catch (Exception e) {
        attempts--;
    }
}
```

---
**Важные замечания**
1. **Поддержка версий:**
   * `val` доступен с ранних версий Lombok;
   * `var` добавлен в Lombok 1.18.16 (для совместимости с Java 10+).
2. **Область применения:**
   * `@Value` — на уровне класса;
   * `val`/`var` — только для локальных переменных внутри методов.

---

Разберу подробно использование аннотаций Lombok `@Builder`, `@Builder.Default`, `@Builder.ObtainVia` и `@Singular` — с примерами и объяснениями.

## @Builder

Аннотация `@Builder` генерирует шаблон «Строитель» (Builder) для удобного создания объектов.

**Пример:**

```java
import lombok.Builder;
import lombok.ToString;

@Builder
@ToString
public class User {
    private String name;
    private int age;
    private String email;
}
```

**Как использовать:**

```java
User user = User.builder()
    .name("Анна")
    .age(25)
    .email("anna@mail.com")
    .build();

System.out.println(user);
// User(name=Анна, age=25, email=anna@mail.com)
```

**Что генерируется:**
* статический метод `builder()`;
* внутренний класс‑строитель `UserBuilder`;
* методы‑сеттеры в строителе для каждого поля;
* метод `build()` для создания объекта.

---

### @Builder.Default

Позволяет задать **значения по умолчанию** для полей, которые не были указаны в строителе.

**Пример:**

```java
import lombok.Builder;
import lombok.Builder.Default;
import lombok.ToString;

@Builder
@ToString
public class Product {
    private String name;

    @Default
    private int quantity = 10;

    @Default
    private boolean available = true;
}
```

**Использование:**

```java
Product product1 = Product.builder()
    .name("Ноутбук")
    .build(); // quantity=10, available=true

Product product2 = Product.builder()
    .name("Мышь")
    .quantity(50)
    .available(false)
    .build(); // quantity=50, available=false

System.out.println(product1);
// Product(name=Ноутбук, quantity=10, available=true)
System.out.println(product2);
// Product(name=Мышь, quantity=50, available=false)
```

**Важно:**
* Поле должно быть инициализировано при объявлении.
* Если в строителе не вызвать метод для этого поля, будет использовано значение по умолчанию.

---

### @Builder.ObtainVia

Позволяет **кастомизировать способ получения значения** для поля — например, брать его из другого поля или вычислять.

**Пример:**

```java
import lombok.Builder;
import lombok.Builder.ObtainVia;
import lombok.ToString;
import java.time.LocalDate;

@Builder
@ToString
public class Order {
    private String orderId;
    private LocalDate creationDate;

    @ObtainVia(method = "calculateTotal")
    private double totalAmount;

    // Вспомогательный метод для расчёта
    private double calculateTotal() {
        return orderId.hashCode() % 1000; // упрощённый расчёт
    }
}
```

**Использование:**

```java
Order order = Order.builder()
    .orderId("ORD-001")
    .creationDate(LocalDate.now())
    .build();

System.out.println(order);
// Order(orderId=ORD-001, creationDate=2023-10-15, totalAmount=987.0)
```

**Варианты использования `@ObtainVia`:**
* `method = "methodName"` — вызов метода в текущем классе;
* `field = "fieldName"` — взять значение из другого поля;
* комбинация с другими параметрами.

---

### @Singular

Упрощает работу с **коллекциями** в строителе: позволяет добавлять элементы по одному или сразу всю коллекцию.

**Пример:**

```java
import lombok.Builder;
import lombok.Singular;
import lombok.ToString;
import java.util.List;

@Builder
@ToString
public class Team {
    private String teamName;

    @Singular
    private List<String> members;
}
```

**Способы использования:**

```java
// Способ 1: добавлять элементы по одному
Team team1 = Team.builder()
    .teamName("Разработчики")
    .member("Анна")
    .member("Иван")
    .member("Мария")
    .build();

// Способ 2: передать всю коллекцию сразу
Team team2 = Team.builder()
    .teamName("Дизайнеры")
    .members(List.of("Ольга", "Пётр"))
    .build();

// Способ 3: комбинация
Team team3 = Team.builder()
    .teamName("Тестировщики")
    .member("Сергей")
    .members(List.of("Елена", "Дмитрий"))
    .build();

System.out.println(team1);
// Team(teamName=Разработчики, members=[Анна, Иван, Мария])
System.out.println(team2);
// Team(teamName=Дизайнеры, members=[Ольга, Пётр])
System.out.println(team3);
// Team(teamName=Тестировщики, members=[Сергей, Елена, Дмитрий])
```

**Особенности:**
* Для списка генерируются два метода: `member(String)` и `members(Collection<String>)`.
* Lombok автоматически выбирает подходящее имя метода (для `List<String>` — `member` и `members`).
* Работает с `List`, `Set`, `Map` и другими коллекциями.

---

**Важные замечания**
1. **Порядок инициализации:** `@Builder.Default` выполняется до `@Builder.ObtainVia`.
2. **Версии:** `@Builder.ObtainVia` доступен в Lombok 1.18.12 и выше.
3. **Коллекции:** `@Singular` автоматически инициализирует коллекцию (не будет `null`).

---
## @With

Аннотация `@With` генерирует **методы для создания копии объекта с изменённым значением одного поля**. Это позволяет реализовать **паттерн «с копированием» (with-copy)** для неизменяемых классов.

Ключевые особенности:
* создаёт методы вида `withFieldName(newValue)`;
* возвращает новый экземпляр объекта с обновлённым полем;
* оставляет исходный объект неизменным;
* идеально сочетается с `@Value` и `@Data`.

**Базовый пример**

**Без Lombok** (ручное написание):

```java
public class User {
    private final String name;
    private final int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Метод для создания копии с новым именем
    public User withName(String newName) {
        return new User(newName, this.age);
    }

    // Метод для создания копии с новым возрастом
    public User withAge(int newAge) {
        return new User(this.name, newAge);
    }

    // Геттеры...
    public String getName() { return name; }
    public int getAge() { return age; }
}
```

**С Lombok `@With`** (гораздо короче):

```java
import lombok.Value;
import lombok.With;

@Value
@With
public class User {
    String name;
    int age;
}
```

Как использовать сгенерированные методы

```java
// Создаём исходный объект
User original = new User("Анна", 25);

// Создаём копию с новым именем — исходный объект не меняется!
User updatedName = original.withName("Мария");

// Цепочка вызовов: меняем и имя, и возраст
User fullyUpdated = original.withName("Елена").withAge(28);

System.out.println("Оригинал: " + original);
// User(name=Анна, age=25)

System.out.println("Обновлённое имя: " + updatedName);
// User(name=Мария, age=25)

System.out.println("Полностью обновлённый: " + fullyUpdated);
// User(name=Елена, age=28)
```

Пример с несколькими полями

```java
import lombok.Value;
import lombok.With;

@Value
@With
public class Product {
    String name;
    double price;
    String category;
    boolean available;
}

// Использование
Product original = new Product("Ноутбук", 50000.0, "Электроника", true);

Product updatedPrice = original.withPrice(45000.0);
Product updatedCategory = original.withCategory("Ноутбуки");
Product outOfStock = original.withAvailable(false);

System.out.println(original);
// Product(name=Ноутбук, price=50000.0, category=Электроника, available=true)

System.out.println(updatedPrice);
// Product(name=Ноутбук, price=45000.0, category=Электроника, available=true)
```

Сочетание `@Value` + `@With` особенно полезно для создания **неизменяемых DTO** и value‑объектов:

```java
import lombok.Value;
import lombok.With;

@Value
@With
public class Address {
    String street;
    String city;
    String postalCode;
}

// Применение в бизнес‑логике
Address originalAddress = new Address("Ленина, 15", "Москва", "123456");
Address updatedCity = originalAddress.withCity("Санкт‑Петербург");
Address correctedStreet = originalAddress.withStreet("Ленина, 17");
```

Можно указать **пользовательское имя метода** с помощью параметра `withName`:

```java
import lombok.Value;
import lombok.With;

@Value
public class Order {
    @With(withName = "changeOrderId")
    String orderId;

    @With(withName = "updateTotal")
    double totalAmount;
}

// Использование кастомных имён
Order original = new Order("ORD-001", 1000.0);
Order updatedId = original.changeOrderId("ORD-002");
Order updatedTotal = original.updateTotal(1500.0);
```

Важные нюансы и ограничения

1. **Совместимость с другими аннотациями:**
   * отлично работает с `@Value`, `@Data`, `@AllArgsConstructor`;
   * может конфликтовать с пользовательскими конструкторами.

2. **Типы полей:**
   * работает со всеми типами данных;
   * для коллекций создаёт копию ссылки (не глубокую копию коллекции).

3. **Производительность:**
   * создаёт новый объект при каждом вызове — учитывайте это в высоконагруженных сценариях;
   * не подходит для очень больших объектов, если создаётся много копий.

4. **Цепочка вызовов:**
   * методы можно вызывать последовательно: `obj.withA(a).withB(b).withC(c)`;
   * каждый вызов создаёт новый промежуточный объект.

5. **Null‑значения:**
   * позволяет устанавливать `null` в поля;
   * поведение зависит от логики класса.

## Lombok annotaions explanation

[AllArgsConstructor](https://projectlombok.org/api/lombok/AllArgsConstructor.html "annotation in lombok")
Generates an all-args constructor.
[RequiredArgsConstructor](https://projectlombok.org/api/lombok/RequiredArgsConstructor.html "annotation in lombok")
Generates a constructor with required arguments.

[Builder](https://projectlombok.org/api/lombok/Builder.html "annotation in lombok")
The builder annotation creates a so-called 'builder' aspect to the class that is annotated or the class that contains a member which is annotated with `@Builder`.
[Builder.Default](https://projectlombok.org/api/lombok/Builder.Default.html "annotation in lombok")
The field annotated with `@Default` must have an initializing expression; that expression is taken as the default to be used if not explicitly set during building.
[Builder.ObtainVia](https://projectlombok.org/api/lombok/Builder.ObtainVia.html "annotation in lombok")
Put on a field (in case of `@Builder` on a type) or a parameter (for `@Builder` on a constructor or static method) to indicate how lombok should obtain a value for this field or parameter given an instance; this is only relevant if `toBuilder` is `true`.

[Singular](https://projectlombok.org/api/lombok/Singular.html "annotation in lombok")
The singular annotation is used together with `@Builder` to create single element 'add' methods in the builder for collections.

[Cleanup](https://projectlombok.org/api/lombok/Cleanup.html "annotation in lombok")
Ensures the variable declaration that you annotate will be cleaned up by calling its close method, regardless of what happens.

[Data](https://projectlombok.org/api/lombok/Data.html "annotation in lombok")
Generates getters for all fields, a useful toString method, and hashCode and equals implementations that check all non-transient fields.

[EqualsAndHashCode](https://projectlombok.org/api/lombok/EqualsAndHashCode.html "annotation in lombok")
Generates implementations for the `equals` and `hashCode` methods inherited by all objects, based on relevant fields.
[EqualsAndHashCode.Exclude](https://projectlombok.org/api/lombok/EqualsAndHashCode.Exclude.html "annotation in lombok")
If present, do not include this field in the generated `equals` and `hashCode` methods.
[EqualsAndHashCode.Include](https://projectlombok.org/api/lombok/EqualsAndHashCode.Include.html "annotation in lombok")
Configure the behaviour of how this member is treated in the `equals` and `hashCode` implementation; if on a method, include the method's return value as part of calculating hashCode/equality.

[Generated](https://projectlombok.org/api/lombok/Generated.html "annotation in lombok")
Lombok automatically adds this annotation to all generated constructors, methods, fields, and types.

[Getter](https://projectlombok.org/api/lombok/Getter.html "annotation in lombok")
Put on any field to make lombok build a standard getter.

[Locked](https://projectlombok.org/api/lombok/Locked.html "annotation in lombok")
Guards all statements in an annotation method with a [`Lock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Lock.html "class or interface in java.util.concurrent.locks").
[Locked.Read](https://projectlombok.org/api/lombok/Locked.Read.html "annotation in lombok")
Locks using a [`ReadWriteLock.readLock()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReadWriteLock.html#readLock-- "class or interface in java.util.concurrent.locks").
[Locked.Write](https://projectlombok.org/api/lombok/Locked.Write.html "annotation in lombok")
Locks using a [`ReadWriteLock.writeLock()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReadWriteLock.html#writeLock-- "class or interface in java.util.concurrent.locks").

[NoArgsConstructor](https://projectlombok.org/api/lombok/NoArgsConstructor.html "annotation in lombok")
Generates a no-args constructor.

[NonNull](https://projectlombok.org/api/lombok/NonNull.html "annotation in lombok")
If put on a parameter, lombok will insert a null-check at the start of the method / constructor's body, throwing a `NullPointerException` with the parameter's name as message.

[Setter](https://projectlombok.org/api/lombok/Setter.html "annotation in lombok")
Put on any field to make lombok build a standard setter.

[SneakyThrows](https://projectlombok.org/api/lombok/SneakyThrows.html "annotation in lombok")
@SneakyThrows will avoid javac's insistence that you either catch or throw onward any checked exceptions that statements in your method body declare they generate.

[Synchronized](https://projectlombok.org/api/lombok/Synchronized.html "annotation in lombok")
Almost exactly like putting the 'synchronized' keyword on a method, except will synchronize on a private internal Object, so that other code not under your control doesn't meddle with your thread management by locking on your own instance.

[ToString](https://projectlombok.org/api/lombok/ToString.html "annotation in lombok")
Generates an implementation for the `toString` method inherited by all objects, consisting of printing the values of relevant fields.
[ToString.Exclude](https://projectlombok.org/api/lombok/ToString.Exclude.html "annotation in lombok")
If present, do not include this field in the generated `toString`.
[ToString.Include](https://projectlombok.org/api/lombok/ToString.Include.html "annotation in lombok")
Configure the behaviour of how this member is rendered in the `toString`; if on a method, include the method's return value in the output.

[Value](https://projectlombok.org/api/lombok/Value.html "annotation in lombok")
Generates a lot of code which fits with a class that is a representation of an immutable entity.

[val](https://projectlombok.org/api/lombok/val.html "annotation in lombok")
Use `val` as the type of any local variable declaration (even in a for-each statement), and the type will be inferred from the initializing expression.

[var](https://projectlombok.org/api/lombok/var.html "annotation in lombok")
Use `var` as the type of any local variable declaration (even in a `for` statement), and the type will be inferred from the initializing expression (any further assignments to the variable are not involved in this type inference).

[With](https://projectlombok.org/api/lombok/With.html "annotation in lombok")
Put on any field to make lombok build a 'with' - a withX method which produces a clone of this object (except for 1 field which gets a new value).