> [!question]
> изучи далее
> https://javarush.com/groups/posts/2592-top-50-java-core-voprosov-iotvetov-na-sobesedovanii-chastjh-2
  
Может ли быть проблема с тем, что в эдеме закончилось место и мы не смогли создать новый объект по этой причине?
# mutable vs `immutable` объекты
для чего нужны?
как сделать немутабельным?
# Какие бывают стратегии ретраев?
ретраев
# Как протестировать приватный метод?
1. Подумать, надо ли это делать, т.к. надо тестировать что код делает, а не как он это делает. Так второй сценарий дорог и не проверяет бизнес значимое поведение. Приватный метод скорее всего носит служебный характер и может быть заменен на другой.
	- Юнит-тесты должны проверять только контракты публичных методов
2. Можем изменить видимость метода, через рефлексию
	* data
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

# StringBuilder vs StringBuffer
- StringBuilder быстрее чем StringBuffer
- StringBuffer - потокобезопасная реализация StringBuilder-а
# Диспетчеризация более 1 дефолтной реализации метода
- Если случилась коллизия, то метод необходимо переопределить. Можно использовать конструкцию вида `Interface.super.method();` чтобы выбрать конкретную дефолтнуют реализацию метода.
    
    ```Java
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
    
    public class Impl implements B, A{
        @Override // -- ОБЯЗАТЕЛЬНО !
        public void foo() {
            B.super.foo();
        }
    }
    ```
    
# Динамическая диспетчеризация методов
- Динамическая диспетчеризация методов – это специальный механизм, который позволяет вызвать переопределенный метод в процессе выполнения программы а не во время компиляции. Динамическая диспетчеризация методов важна при реализации полиморфизма.

**Класс A:**
```markdown
```java
class A {
    void method() {
        // ...
    }
}
```

**Класс B (наследует A):**
```java
class B extends A {
    void method() {
        // ...
    }
}
```

**Класс C (наследует B):**
```java
class C extends B {
    void method() {
        // ...
    }
}
```

**Создание объектов и работа с ссылкой `refA`:**
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

**Описание:**
На схеме показана иерархия наследования: `A` — базовый класс, `B` наследуется от `A`, `C` наследуется от `B`. Все классы реализуют метод `method()`.

Переменная `refA` типа `A` может ссылаться на объекты классов `A`, `B` и `C` благодаря полиморфизму. При вызове `refA.method()` выполняется метод того класса, на объект которого в данный момент ссылается `refA`.
# Многослойная архитектура
см. [[Layered Architecture]]
# cglib — Code generation library
Динамическое генерирование прокси-классов
```Shell
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
User user = new User("Вася");
MethodInterceptor handler = (obj, method ,  args,  proxy) -> {
    if(method.getName().equals("getName")){
        return ((String)proxy.invoke(user, args)).toUpperCase() ;
    }
    return proxy.invoke(user, args);
};
User userProxy = (User) Enhancer.create(User.class, handler);
assertEquals("ВАСЯ", userProxy.getName());
```
# Java Servlet API
Java Servlet API — стандартизированный API, предназначенный для реализации на сервере и работе с клиентом по схеме запрос-ответ.
## Сервлет класс
Сервлет — это класс, который умеет получать запросы от клиента и возвращать ему ответы. Да, сервлеты в Java — именно те элементы, с помощью которых строится клиент-серверная архитектура.
## Контейнер сервелетов
Это программа, которая запускается на сервере и умеет взаимодействовать с созданными нами сервлетами. Иными словами, если мы хотим запустить наше веб-приложение на сервере, мы сначала разворачиваем контейнер сервлетов, а потом помещаем в него сервлеты.
## Tomcat
Распространенный веб-сервер, который используется чтобы публиковать контейнеры сервлетов
# Object copy — cloning
Когда надо скопировать объект `java.lang` предоставляет метод `Object.clone()`. Метод прекрасно работает в области копирования полей примитивных типов и ссылок на поля сложных объектов.
# Глубокая копия — deepCopy
Если мы подразумевает, что для поля каждого сложного типа надлежит создать копию соответствующего объекта, появляется необходимость делать глубокую копию объекта `deepCopy`
Есть 4 способа решить сделать глубокую копию объекта:
3. **Copy Constructor**
    
    - суть в том, что в каждом включенном в объект классе и ниже должен быть конструктор, который создает свою копию от экземпляра своего же типа.
        
```Java
public class Order{
	private String orderNumber;
	private double orderAmount;
	private String orderStatus;
	//constructors, getters and setters
	//Copy Constructor
	public Order(Order order){
	   this(order.getOrderNumber(),order.getOrderAmount(),order.getOrderStatus());
	}
}

public class Customer {
	private String firstName;
	private String lastName;
	private Order order;
	//constructors, getters and setters
	//Copy Constructor
	public Customer(Customer customer) {
		this(customer.getFirstName(),customer.getFirstName()
				,new Order(customer.getOrder()));
	}
}
```
        
          
        
    
    Недостаток такого решения в том, что надо обеспечивать наличие копи-конструктора для множества типов.
    
4. **Cloneable Interface**
    - имплементировать `Cloneable Interface` и переопределить `Object.clone()`
        
        ```Java
        public class Order implements Cloneable{
        //...
        	@Override
        	public Order clone() implements Cloneable{
        	  try {
        	        return (Order) super.clone();
        	      }catch (CloneNotSupportedException e) {
        	         return new Order(this.orderNumber,this.orderAmount,this.orderStatus);
        	     }
        	}
        }
        public class Customer  implements Cloneable{
        //...
        	@Override
        	public Customer clone() implements Cloneable{
        	Customer customer =null;
        	try {
        	      customer = (Customer) super.clone();
        	     }catch (CloneNotSupportedException e) {
        	       customer = new Customer(this.firstName,this.lastName,this.order);
        	     }
        	     customer.order=this.order.clone();
        	     return customer;
        	}
        }
        ```
        
    
    Недостаток в том, что такой метод писать может быть затруднительно.
    
5. **Deep Copy using Serialization**
    
    - Серилизуем и десерилизуем в новый объект
        
        ```Java
        public class JavaDeepCloneBySerialization {
        
        public static void main(String[] args) {
        
          Order order = new Order("12345", 100.45, "In Progress");
          Customer customer = new Customer("Test", "CUstomer", order);
        
          Customer cloneCustomer = deepClone(customer);
          order.setOrderStatus("Shipped");
          System.out.println(cloneCustomer.getOrder().getOrderStatus());
        
        }
        
        public static  T deepClone(T object){
          try {
                ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
                ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
                objectOutputStream.writeObject(object);
                ByteArrayInputStream bais = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
                ObjectInputStream objectInputStream = new ObjectInputStream(bais);
                  return (T) objectInputStream.readObject();
            }
            catch (Exception e) {
              e.printStackTrace();
              return null;
            }
          }
        }
        ```
        
    
    Недостаток — серилизация дорогая операция
    
6. **Deep Copy Using External Libraries, eg Apache Common Lang**
    - Статический метод сторонней библиотеки `SerializationUtils.clone(customer);`
        
        ```Java
        public class SerializationUtilsTest {
        
        	@Test
        	public void testDeepClone(){
        	
        	  Order order = new Order("12345", 100.45, "In Progress");
        	  Customer customer = new Customer("Test", "Customer", order);
        	  Customer cloneCustomer = SerializationUtils.clone(customer);
        	
        	  order.setOrderStatus("Shipped");
        	  assertNotEquals(customer.getOrder().getOrderStatus(), cloneCustomer.getOrder().getOrderStatus());
        	}
        }
        ```
        
# boxing / unboxing
`boxing` **—** Создание объекта-оболочки из переменной примитивного типа называется упаковкой , а получение значения примитивного типа из объекта-оболочки — распаковкой `unboxing`.
```Java
int a = 5; // Integer a = 5; -- упакованный вариант переменной boxed
ArrayList list = new ArrayList();
String s = a.toString(); // ERROR, когда unboxed
list.add(a))             // OK, но произойдет автоупаковка
                         // и в коллекцию будет помещен Integer
```
- Компилятор по необходимости делает автоупаковку и автораспаковку
    
    ```JavaScript
    int a = 5;
    Integer b = 10;
    a = b;             // OK, атораспаковка
    b = a * 123;       // OK, автоупаковка
    ```
    
Все объекты-оболочки - неизменяемые (`**immutable**`) типы, т.е. когда мы присваиваем им новое значение, фактически на замену прежнему объекту создается новый.
# DTO — Data Transfer Object
DTO объект - объект, который не содержит методы. Он не может содержать внешних связей. Он может содержать только поля, геттеры/сеттеры, и конструкторы.
Data Transfer Object - объект, передающий данные. Данные - это и есть поля в классе. Все внешние клиенты должны получать DTO
# POJO — Plain Old Java Object
POJO (англ. Plain Old Java Object) — «старый добрый Java-объект», простой Java-объект, не унаследованный от какого-то специфического объекта и не реализующий никаких служебных интерфейсов сверх тех, которые нужны для бизнес-модели.
Термин, придуманный Мартином Фаулером c сотоварищами в пику EJB (Enterprise JavaBeans), так как отсутствие звучного термина для простых объектов приводило к тому, что молодые Java-программисты пренебрежительно к ним относились, считая что только EJB «спасут мир».
# DAO — Data Access Object
- Промежуточный слой между данными и клиентом.
- Служит для того, чтобы предоставить интерфейс доступа к данным, который не будет зависеть от структуры данных и от механизмов доступа к данным.
- Относится к **Core J2EE Patterns — Data Access Object**
- Распространенный пример реализации паттерна DAO — **JpaRepository**
# JMX, JSR, J2EE, JAR
[[JDK JVM JLS JNI]]
# PID — Process ID
Идентификатор процесса в JVM, который выполеняет программу
# узнать текущее имя метода, класса, папки
- `this.getClass().getName()` || `EmailValidator.class.getName()` → имя класса
- `Thread.currentThread().getStackTrace()[1].getMethodName()` → текущий метод
- `System.getProperty("user.dir")` → текущая директория
# ручной импорт пакета в среду
Set the environment variable CLASSPATH to `%CLASSPATH%;%GSON_HOME%\gson-2.8.6.jar;:;`
# i++ vs ++i
`i++` — вначале вывод, потом инкремент  
  
`++i` — вначале инкремент, потом вывод
```Java
int i = 0;
System.out.println(i++); // 0
System.out.println(++i); // 2
```
# Пример SQLNativeQuery
```Java
List<Tuple> comments = entityManager.createNativeQuery("""
    SELECT
        pc.id AS id,
        pc.review AS review,
        pc.post_id AS postId
    FROM post_comment pc
    """, Tuple.class)
.getResultList();
```
# Сколько памяти используют примитивные типы java
- Сколько памяти используют примитивные типы java
- [[Java Core#Примитивные типы]]
[[Layered Architecture]]
