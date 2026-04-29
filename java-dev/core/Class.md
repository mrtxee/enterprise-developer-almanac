---
aliases:
  - Class,
  - классы
---
# Java классы
## Декларация класса
включает в себя сокрытие внутренних полей и методов (инкапсуляция), описание сеттеров, геттеров и конструктора.
Хороший тон описания класса
- Класс должен отражать ровно одну сущность бизнес-логики SRP
- Все изменения полей должны происходить только через публичные методы OCP
## Структура класса
- Класс включает в себя атрибуты и методы
- Класс должен содержать модификатор доступа:
    - `public` — методы и атрибуты класса доступны всем другим объектам
    - `default` / `package-protected` — доступно только внутри пакета
    - `protected` — доступно только наследуемым классам
    - `protected` — доступно только внутри этого класса
- Геттеры и сеттеры
    - позволяют контролировать ввод данных
    - позволяют сокрыть внутреннее устройство класса
    - позволяют выдавать данные клиенту в заданном представлении
    - имплементируют принцип инкапсуляции
- Конструктор
    - совпадает с названием класса, не имеет явного типа
    - метод, который выполняется при инициализации класса
        - конструктор выполняется при наличии директивы `new`
    - у всех классов есть конструкторы, даже если не были описаны в явном виде, их генерирует java
    - следует использовать поинтер `this`.
- Статические методы и атрибуты
    - `static` атрибут означает что будет только 1 общий для всех экземпляров класса. Можно применить для подсчета количества экземпляров итп.
    - к статическим методам и атрибутам можно обратиться без инициализации экземпляра класса
- Указатель `this.*` — не обязательный, но стоит использовать, чтобы не было путаницы с переменными полями сигнатуры
- Указатель `super.*` для обращения к полям родительского класса
- модификатор `final`
    - `final` метода запрещает переопределение метода потомками класса
    - `final` класс запрещает переопределение наследование класса
    - `final` поле запрещает изменение переменной
- Аннотация `@Override` используется для переопределение метода в наследнике
- модификатор метода `native` означает, что сущность реализована на платформозависимом языке, _напр. cpp._
```Java
public class MyClass {
    protected int wheels;
    private String color;
    String name;
    public static int COUNT=0;
    public static final double PI = 3.14;
    @Override
    public boolean equals(Object obj) {
        return this.color==((MyClass)obj).color;
    }
    MyClass() {
        this.color = "Red";
        COUNT++;
    }
    MyClass(String color) {
        this.color = color;
        COUNT++;
    }
    // Getter
    public String getColor() {
        return this.color;
    }
    // Setter
    public void setColor(String c) {
        this.color = c;
    }
    static void sayHello() {
        System.out.println("Hello World!");
    }
    public static void main(String[] args) {
        sayHello();
        MyClass mc = new MyClass("green");
        mc.setColor("gray");
        System.out.println(mc.PI);
        System.out.println(mc.getColor());
        System.out.println(mc.hashCode());
        MyClass mc2 = new MyClass("gray");
        System.out.println(mc2.hashCode());
        System.out.println(mc2.equals(mc));
    }
}
```
## Класс Object
Все классы являются потомками класса `Object`. Поэтому они наследуют его стандартные методы, которые можно переопределить по необходимости: `hashCode()`, `equals()`, `clone()`, `toString()`, `_finalize()_` _…_
[[Object]]
### Пакетирование
Классы можно пакетировать, а пакеты можно импортировать.
```Java
package samples;
...
import samples.Vehicle;
import samples.*
class MyClass {
  public static void main(String[ ] args) {
    Vehicle v1 = new Vehicle();
    v1.horn();
  }
}
```
### Инкапсуляция
Сокрытие реализации. Приватный параметр + сеттер, геттер
### Наследование
- подкласс наследуется от суперкласса при помощи ключевого слова `extends`
    
    ```Java
    class Animal{
     // ...
    }
    class Dog extends Animal {
     // ...
    }
    class Cat extends Animal {
     // ...
    }
    ```
    
- при наследовании наследуются все методы и поля, кроме `private`
    - из наследника можно прочитать поле родителя через поинтер `super.*`
- указатель `super` можно использовать чтобы обратиться к методам и полям родительского класса
### Полиморфизм
Могут быть разные реализации для одной сущности
```Java
Animal cat = new Cat();
Animal dog = new Dog();
```
Это может быть полезно для создания списка одного типа с разными реализациями.
### overriding — переопределение методов
- переопределенный метод маркируется аннотацией `@override`
- методы должны быть одинаковый тип и поля
- нельзя понижать модификаторы доступа
- нельзя переопределять методы `final`, `static`, ==конструктор==
### overloading — перегрузка методов
Описание метода с таким же названием, но с другой сигнатурой
- Сигнатура метода

    Сигнатура метода используется для однозначной идентификации метода.
    
    Сигнатура метода включает в себя:
    
    - имя метода
    - плюс параметры (причем порядок параметров имеет значение)
    - возвращаемый тип данных
## Абстракция
### Абстрактный класс
- определяется директивой `abstract`
- нельзя создать экземпляр, но можно наследовать
### Абстрактный метод
- определяется директивой `abstract`
- не требует имплементации
- может быть только внутри абстрактного класcа
    ```Java
    abstract void walk();
    ```
## Интерфейсы
- Как абстрактный класс, только не может содержать имплементированных методов.
    - можно добавить реализацию метода по-умолчанию при помощи ключевого слова `default`
- Предназначен для установление контракта между клиентам класса
- Defined using the `interface` keyword.
- May contain only `static final variables`
- Cannot contain a constructor because interfaces cannot be instantiated
- Can extend other interfaces
- A class can implement any number of interfaces
- An interface is implicitly abstract. You do not need to use the abstract keyword while declaring an interface.
- Each method in an interface is also implicitly abstract, so the abstract keyword is not needed.
- Methods in an interface are implicitly public.
- директива `implements` чтобы добавить интерфейс к классу
    ```Java
    interface Animal {
        public void eat();
        public void makeSound();
    }
    
    class Cat implements Animal, Wild {
        public void makeSound() {
            System.out.println("Meow");
        }
        public void eat() {
            System.out.println("omnomnom");
        }
    }
    ```
## Casting — приведение типов
```Java
int a = (int)3.14;
```
### Upcasting, Downcasting
`Upcasting` - когда субкласс приводится к типу суперкласса. `Downcasting` - когда наоборот.
```Java
Animal a = new Cat();  //Upcasting
((Cat)a).makeSound(); // Downcasting
```
Why is upcasting automatic, downcasting manual? Well, upcasting can never fail.s
## Проверка принадлежности к классу
```Java
if (account instanceof Account){...}
```



## top-level class — класс верхнего уровня
## local class — класс верхнего уровня
## inner Class — вложенный класс
`Inner Classes` позволяют создавать классы внутри другого класса. 
* пример inner class
	```Java
	class Robot {
		int id;
		Robot(int i) {
			id = i;
			Brain b = new Brain();
			b.think();
		}
		private class Brain {
			public void think() {
				System.out.println(id + " is thinking");
			}
		}
	}
	public class Program {
		public static void main(String[] args) {
			Robot r = new Robot(1);
		}
	}
	```
### отличия вложенных классов от обычных:
1. **способ декларации**
	1. Обычный класс на верхнем уровне в отдельном файле
	2. Вложенный класс внутри другого класса или метода
		1. LOCAL INNER CLASS
2. **Внутренний класс имеет доступ к `private` полям** экземпляра внешнего класса. Неявное использование ссылки на экземпляр `<OuterClassName>.this.*`
3. **Создание экземпляра** через экземпляр внешнего класса
	1. `Outer.inner innerInstance = outer.new inner()`
4. **не может быть статическим**, т.к. будет уже `static inner` 
5. **не может иметь статический полей**, кроме `static final`

## static inner class

## Анонимные классы
Анонимный класс это вложенный класс, у которого нет имени, так как за основу берется существующие класс, который меняется разработчиком “на лету”.
Нотация `@Override` используется, чтобы заменить метод сущности на лету в момент инициализации, создавая анонимный класс.
```Java
class Machine {
    public void start() {
        System.out.println("Starting...");
    }
}
class Program {
    public static void main(String[ ] args) {
        Machine m = new Machine() {
            @Override 
            public void start() {
                System.out.println("Wooooo");
            }
        };
        m.start();
    }
}
```