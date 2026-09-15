---
aliases:
  - ++i
  - boxing
  - cglib
  - CLASSPATH
  - Cloneable
  - Copy Constructor
  - DAO
  - Data Access Object
  - Data Transfer Object
  - deep copy
  - deepCopy
  - default methods
  - DTO
  - dynamic dispatch
  - i++
  - immutable
  - J2EE
  - JAR
  - Java Servlet API
  - JMX
  - JSR
  - Object.clone
  - PID
  - Plain Old Java Object
  - POJO
  - primitive types
  - Process ID
  - Serialization
  - StringBuffer
  - StringBuilder
  - unboxing
  - Автораспаковка
  - Автоупаковка
  - Глубокая копия
  - Дефолтные методы
  - Динамическая диспетчеризация
  - Инкремент
  - Клонирование
  - Контейнер сервлетов
  - Копи-конструктор
  - Приватные методы
  - Примитивные типы
  - Сервлет
  - Сериализация
---

## Тестирование приватных методов

1. Подумать, нужно ли это делать: тестировать надо то, что код делает, а не как он это делает. Второй сценарий дорог и не проверяет бизнес-значимое поведение. Приватный метод скорее всего носит служебный характер и может быть заменён на другой.
   - Юнит-тесты должны проверять только контракты публичных методов.
2. Изменить видимость метода через рефлексию.

```java
private Method getDoubleIntegerMethod() throws NoSuchMethodException {
    Method method = Utils.class.getDeclaredMethod("doubleInteger", Integer.class);
    method.setAccessible(true);
    return method;
}

@Test
void givenNull_WhenDoubleInteger_ThenNull() throws InvocationTargetException, IllegalAccessException, NoSuchMethodException {
    assertEquals(null, getDoubleIntegerMethod().invoke(null, new Integer[] { null }));
}
```

## StringBuilder vs StringBuffer

| Свойство | `StringBuilder` | `StringBuffer` |
| --- | --- | --- |
| Скорость | Быстрее | Медленнее |
| Потокобезопасность | Нет | Да (потокобезопасная реализация) |

## Конфликт дефолтных методов интерфейсов

При коллизии дефолтных реализаций метод необходимо переопределить. Можно использовать конструкцию `Interface.super.method();`, чтобы выбрать конкретную дефолтную реализацию метода.

```java
interface A {
    default void foo() {
        System.out.println("Foo A");
    }
}

interface B {
    default void foo() {
        System.out.println("Foo B");
    }
}

public class Impl implements B, A {
    @Override // обязательно!
    public void foo() {
        B.super.foo();
    }
}
```

## Динамическая диспетчеризация методов

Динамическая диспетчеризация — механизм, который позволяет вызвать переопределённый метод в процессе выполнения программы, а не во время компиляции. Динамическая диспетчеризация важна при реализации полиморфизма.

**Класс A**

```java
class A {
    void method() {
        // ...
    }
}
```

**Класс B (наследует A)**

```java
class B extends A {
    void method() {
        // ...
    }
}
```

**Класс C (наследует B)**

```java
class C extends B {
    void method() {
        // ...
    }
}
```

**Создание объектов и работа со ссылкой `refA`**

```java
A objA = new A();
B objB = new B();
C objC = new C();

A refA; // refA => A <- B <- C

// Пример 1: refA ссылается на objA
refA = objA; // refA -> objA
refA.method(); // выполняется objA.method()

// Пример 2: refA ссылается на objB
refA = objB; // refA -> objB
refA.method(); // выполняется objB.method()

// Пример 3: refA ссылается на objC
refA = objC; // refA -> objC
refA.method(); // выполняется objC.method()
```

**Описание**

На схеме показана иерархия наследования: `A` — базовый класс, `B` наследуется от `A`, `C` наследуется от `B`. Все классы реализуют метод `method()`.

Переменная `refA` типа `A` может ссылаться на объекты классов `A`, `B` и `C` благодаря полиморфизму. При вызове `refA.method()` выполняется метод того класса, на объект которого в данный момент ссылается `refA`.

## Многослойная архитектура

См. [[layered-architecture]].

## cglib — Code Generation Library

Динамическое генерирование прокси-классов.

```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
User user = new User("Вася");
MethodInterceptor handler = (obj, method, args, proxy) -> {
    if (method.getName().equals("getName")) {
        return ((String) proxy.invoke(user, args)).toUpperCase();
    }
    return proxy.invoke(user, args);
};
User userProxy = (User) Enhancer.create(User.class, handler);
assertEquals("ВАСЯ", userProxy.getName());
```

## Java Servlet API

Java Servlet API — стандартизированный API, предназначенный для реализации на сервере и работы с клиентом по схеме запрос-ответ.

### Сервлет

Сервлет — класс, который умеет получать запросы от клиента и возвращать ему ответы. В Java сервлеты — элементы, с помощью которых строится клиент-серверная архитектура.

### Контейнер сервлетов

Программа, которая запускается на сервере и умеет взаимодействовать с созданными сервлетами. Чтобы запустить веб-приложение на сервере, сначала разворачивается контейнер сервлетов, а затем в него помещаются сервлеты.

### Tomcat

Распространённый веб-сервер, который используется, чтобы публиковать контейнеры сервлетов.

## Object copy — cloning

Когда надо скопировать объект, `java.lang` предоставляет метод `Object.clone()`. Метод прекрасно работает в области копирования полей примитивных типов и ссылок на поля сложных объектов.

## Глубокая копия — deep copy

Если для поля каждого сложного типа нужно создать копию соответствующего объекта, возникает необходимость делать глубокую копию объекта (`deepCopy`).

Есть 4 способа сделать глубокую копию объекта:

**1. Copy Constructor**

Суть в том, что в каждом классе, включённом в объект, и ниже должен быть конструктор, который создаёт копию от экземпляра своего же типа.

```java
public class Order {
    private String orderNumber;
    private double orderAmount;
    private String orderStatus;
    // constructors, getters and setters
    // Copy Constructor
    public Order(Order order) {
        this(order.getOrderNumber(), order.getOrderAmount(), order.getOrderStatus());
    }
}

public class Customer {
    private String firstName;
    private String lastName;
    private Order order;
    // constructors, getters and setters
    // Copy Constructor
    public Customer(Customer customer) {
        this(customer.getFirstName(), customer.getFirstName(),
            new Order(customer.getOrder()));
    }
}
```

Недостаток такого решения: нужно обеспечивать наличие копи-конструктора для множества типов.

**2. Cloneable Interface**

Имплементировать `Cloneable` и переопределить `Object.clone()`.

```java
public class Order implements Cloneable {
    // ...
    @Override
    public Order clone() {
        try {
            return (Order) super.clone();
        } catch (CloneNotSupportedException e) {
            return new Order(this.orderNumber, this.orderAmount, this.orderStatus);
        }
    }
}

public class Customer implements Cloneable {
    // ...
    @Override
    public Customer clone() {
        Customer customer = null;
        try {
            customer = (Customer) super.clone();
        } catch (CloneNotSupportedException e) {
            customer = new Customer(this.firstName, this.lastName, this.order);
        }
        customer.order = this.order.clone();
        return customer;
    }
}
```

Недостаток: такой метод может быть затруднительно писать.

**3. Deep Copy using Serialization**

Сериализуем и десериализуем объект в новый.

```java
public class JavaDeepCloneBySerialization {

    public static void main(String[] args) {

        Order order = new Order("12345", 100.45, "In Progress");
        Customer customer = new Customer("Test", "CUstomer", order);

        Customer cloneCustomer = deepClone(customer);
        order.setOrderStatus("Shipped");
        System.out.println(cloneCustomer.getOrder().getOrderStatus());
    }

    public static <T> T deepClone(T object) {
        try {
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
            objectOutputStream.writeObject(object);
            ByteArrayInputStream bais = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
            ObjectInputStream objectInputStream = new ObjectInputStream(bais);
            return (T) objectInputStream.readObject();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

Недостаток: сериализация — дорогая операция.

**4. Deep Copy Using External Libraries (Apache Commons Lang)**

Статический метод сторонней библиотеки `SerializationUtils.clone(customer)`.

```java
public class SerializationUtilsTest {

    @Test
    public void testDeepClone() {

        Order order = new Order("12345", 100.45, "In Progress");
        Customer customer = new Customer("Test", "Customer", order);
        Customer cloneCustomer = SerializationUtils.clone(customer);

        order.setOrderStatus("Shipped");
        assertNotEquals(customer.getOrder().getOrderStatus(), cloneCustomer.getOrder().getOrderStatus());
    }
}
```

## Boxing / unboxing

`boxing` — создание объекта-оболочки из переменной примитивного типа (упаковка). Получение значения примитивного типа из объекта-оболочки — распаковка `unboxing`.

```java
int a = 5; // Integer a = 5; -- упакованный вариант переменной boxed
ArrayList list = new ArrayList();
String s = a.toString(); // ERROR, когда unboxed
list.add(a)              // OK, но произойдёт автоупаковка
                         // и в коллекцию будет помещён Integer
```

- Компилятор по необходимости делает автоупаковку и автораспаковку.

```java
int a = 5;
Integer b = 10;
a = b;             // OK, автораспаковка
b = a * 123;       // OK, автоупаковка
```

Все объекты-оболочки — неизменяемые (`immutable`) типы: когда им присваивается новое значение, вместо прежнего объекта создаётся новый.

## DTO — Data Transfer Object

DTO — объект, который не содержит методов и внешних связей. Он может содержать только поля, геттеры/сеттеры и конструкторы.

Data Transfer Object — объект, передающий данные. Данные — это поля в классе. Все внешние клиенты должны получать DTO.

## POJO — Plain Old Java Object

POJO (англ. Plain Old Java Object) — «старый добрый Java-объект», простой Java-объект, не унаследованный от специфического объекта и не реализующий служебных интерфейсов сверх тех, которые нужны для бизнес-модели.

Термин придуман Мартином Фаулером #👨 с сотоварищами в пику EJB (Enterprise JavaBeans): отсутствие звучного термина для простых объектов приводило к тому, что молодые Java-программисты пренебрежительно к ним относились, считая, что только EJB «спасут мир».

## DAO — Data Access Object

- Промежуточный слой между данными и клиентом.
- Служит для предоставления интерфейса доступа к данным, который не зависит от структуры данных и механизмов доступа к ним.
- Относится к **Core J2EE Patterns — Data Access Object**.
- Распространённый пример реализации паттерна DAO — **JpaRepository**.

## JMX, JSR, J2EE, JAR

[[jdk-jls-jni]]

## PID — Process ID

Идентификатор процесса в JVM, который выполняет программу.

## Получение имени метода, класса и директории

- `this.getClass().getName()` или `EmailValidator.class.getName()` → имя класса.
- `Thread.currentThread().getStackTrace()[1].getMethodName()` → текущий метод.
- `System.getProperty("user.dir")` → текущая директория.

## Ручной импорт пакета в среду

Установите переменную окружения CLASSPATH: `%CLASSPATH%;%GSON_HOME%\gson-2.8.6.jar;:;`.

## i++ vs ++i

| Оператор | Поведение |
| --- | --- |
| `i++` | вначале вывод, потом инкремент |
| `++i` | вначале инкремент, потом вывод |

```java
int i = 0;
System.out.println(i++); // 0
System.out.println(++i); // 2
```

## Пример SQLNativeQuery

```java
List<Tuple> comments = entityManager.createNativeQuery("""
    SELECT
        pc.id AS id,
        pc.review AS review,
        pc.post_id AS postId
    FROM post_comment pc
    """, Tuple.class)
.getResultList();
```

## Объём памяти примитивных типов Java

- [[java-core#Примитивные типы]]

**См. также**

- [[layered-architecture]]
