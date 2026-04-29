---
aliases:
  - Dynamic Binding
  - Динамическое связывание
---
## Динамическое связывание в Java (Dynamic Binding)

**Динамическое связывание** (позднее связывание) — это механизм в Java, при котором вызов метода разрешается **во время выполнения программы**, а не на этапе компиляции. Это основа полиморфизма в объектно-ориентированном программировании.

---

## 🔍 Основные понятия

### Статическое vs Динамическое связывание

| Характеристика         | Статическое связывание      | Динамическое связывание      |
| ---------------------- | --------------------------- | ---------------------------- |
| **Время разрешения**   | Компиляция                  | Выполнение                   |
| **Какие методы**       | static, private, final      | Обычные (не final) методы    |
| **На основе чего**     | Тип ссылки (тип переменной) | Тип объекта (реальный класс) |
| **Производительность** | Быстрее                     | Немного медленнее            |
| **Другое название**    | Раннее связывание           | Позднее связывание           |

---

## 📚 Как это работает

### Базовый пример

```java
class Animal {
    void sound() {
        System.out.println("Animal makes sound");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog barks");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("Cat meows");
    }
}

public class TestBinding {
    public static void main(String[] args) {
        Animal myAnimal;  // Тип ссылки - Animal
        
        myAnimal = new Dog();
        myAnimal.sound();  // Выводит: "Dog barks"
        
        myAnimal = new Cat();
        myAnimal.sound();  // Выводит: "Cat meows"
        
        myAnimal = new Animal();
        myAnimal.sound();  // Выводит: "Animal makes sound"
    }
}
```

**Объяснение:**
- Переменная `myAnimal` имеет тип `Animal` (объявлена как Animal)
- Но во время выполнения она ссылается на разные объекты: Dog, Cat, Animal
- JVM определяет **реальный тип объекта** и вызывает соответствующую версию метода `sound()`

---

## 🎯 Детальный разбор механизма

### 1. Как JVM определяет какой метод вызвать?

```java
class Parent {
    void show() {
        System.out.println("Parent show");
    }
    
    static void display() {
        System.out.println("Parent static display");
    }
}

class Child extends Parent {
    @Override
    void show() {
        System.out.println("Child show");
    }
    
    static void display() {
        System.out.println("Child static display");
    }
}

public class BindingDemo {
    public static void main(String[] args) {
        Parent obj = new Child();
        
        // Динамическое связывание - по объекту
        obj.show();  // "Child show"
        
        // Статическое связывание - по типу ссылки
        obj.display();  // "Parent static display"
    }
}
```

### 2. Внутренний механизм (упрощённо)

```
Компиляция:
┌─────────────────────┐
│ Parent obj = new    │
│ Child();            │
│ obj.show();         │← Компилятор проверяет, есть ли show() в Parent
└─────────────────────┘

Выполнение:
┌─────────────────────┐
│ 1. JVM смотрит:     │
│    obj → [Child]    │
│ 2. Ищет show() в    │
│    таблице методов  │
│    класса Child     │
│ 3. Вызывает метод   │
│    из Child         │
└─────────────────────┘
```

---

## 🌟 Реальные примеры использования

### Пример 1: Полиморфные коллекции

```java
import java.util.*;

interface Payment {
    void pay(double amount);
}

class CreditCard implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with Credit Card");
    }
}

class PayPal implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with PayPal");
    }
}

class Crypto implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with Cryptocurrency");
    }
}

public class PaymentSystem {
    public static void main(String[] args) {
        List<Payment> payments = Arrays.asList(
            new CreditCard(),
            new PayPal(),
            new Crypto()
        );
        
        // Динамическое связывание в действии
        for (Payment p : payments) {
            p.pay(100.0);  // Каждый раз вызывается правильная реализация
        }
    }
}
```

**Вывод:**
```
Paid $100.0 with Credit Card
Paid $100.0 with PayPal
Paid $100.0 with Cryptocurrency
```

### Пример 2: Фабричный метод с динамическим связыванием

```java
abstract class Shape {
    abstract void draw();
}

class Circle extends Shape {
    void draw() {
        System.out.println("  ⚪ Drawing Circle");
    }
}

class Rectangle extends Shape {
    void draw() {
        System.out.println("  ◼ Drawing Rectangle");
    }
}

class Triangle extends Shape {
    void draw() {
        System.out.println("  🔺 Drawing Triangle");
    }
}

class ShapeFactory {
    static Shape createShape(String type) {
        switch(type.toLowerCase()) {
            case "circle": return new Circle();
            case "rectangle": return new Rectangle();
            case "triangle": return new Triangle();
            default: throw new IllegalArgumentException();
        }
    }
}

public class DrawingApp {
    public static void main(String[] args) {
        String[] types = {"circle", "rectangle", "triangle"};
        
        for (String type : types) {
            Shape shape = ShapeFactory.createShape(type);
            // Динамическое связывание - правильный draw()
            shape.draw();
        }
    }
}
```

### Пример 3: Сложная иерархия

```java
class Employee {
    String getName() { return "Employee"; }
    double calculateBonus() { return 1000; }
}

class Manager extends Employee {
    @Override
    String getName() { return "Manager"; }
    
    @Override
    double calculateBonus() { 
        return super.calculateBonus() * 2;  // 2000
    }
    
    void manage() {
        System.out.println("Managing team");
    }
}

class Director extends Manager {
    @Override
    String getName() { return "Director"; }
    
    @Override
    double calculateBonus() { 
        return super.calculateBonus() * 3;  // 6000
    }
    
    void strategize() {
        System.out.println("Setting strategy");
    }
}

public class Company {
    public static void processEmployee(Employee emp) {
        // Динамическое связывание определяет, 
        // чей getName() и calculateBonus() вызовется
        System.out.println(emp.getName() + " bonus: $" + emp.calculateBonus());
        
        // instanceof всё ещё нужен для специфичных методов
        if (emp instanceof Director) {
            ((Director) emp).strategize();
        } else if (emp instanceof Manager) {
            ((Manager) emp).manage();
        }
    }
    
    public static void main(String[] args) {
        processEmployee(new Employee());   // Employee bonus: $1000
        processEmployee(new Manager());    // Manager bonus: $2000
        processEmployee(new Director());   // Director bonus: $6000
                                          // Setting strategy
    }
}
```

---

## 🧠 Как JVM реализует динамическое связывание

### Таблица виртуальных методов (VMT)

Для каждого класса JVM создаёт **таблицу виртуальных методов**:

```
Класс Animal:
┌─────────────────┐
│ sound() → Animal│
└─────────────────┘

Класс Dog (наследует Animal):
┌─────────────────┐
│ sound() → Dog   │← переопределён
│ toString() → Object
└─────────────────┘

Класс Cat:
┌─────────────────┐
│ sound() → Cat   │← переопределён
│ toString() → Object
└─────────────────┘
```

### Процесс вызова:
1. JVM получает ссылку на объект
2. По ссылке находит класс объекта
3. В классе находит таблицу методов
4. По индексу (определённому на этапе компиляции) получает адрес метода
5. Вызывает метод по этому адресу

---

## ⚠️ Важные ограничения и нюансы

### 1. Динамическое связывание не работает с полями

```java
class Parent {
    String value = "parent";
}

class Child extends Parent {
    String value = "child";
}

public class FieldBinding {
    public static void main(String[] args) {
        Parent obj = new Child();
        System.out.println(obj.value);  // "parent" - поля связываются статически!
    }
}
```

### 2. Работает только для обычных методов

```java
class Demo {
    void normal() { System.out.println("normal"); }
    static void staticMethod() { System.out.println("static"); }
    private void privateMethod() { System.out.println("private"); }
    final void finalMethod() { System.out.println("final"); }
}

class DemoChild extends Demo {
    @Override
    void normal() { System.out.println("overridden"); }  // динамическое
    
    // static - скрытие, а не переопределение
    static void staticMethod() { System.out.println("child static"); }
    
    // Нельзя переопределить private и final
}
```

### 3. Производительность

- Динамическое связывание немного медленнее статического
- JIT-компилятор оптимизирует частые вызовы (инлайнит)
- В 99.9% случаев разница незаметна

---

## 📊 Диаграмма принятия решений

```
                    Вызов метода
                         │
                         ▼
            ┌─────────────────────┐
            │ Является ли метод   │
            │ static, private,    │
            │ или final?          │
            └─────────┬───────────┘
                      │
            ┌─────────┴─────────┐
            ▼                   ▼
    ┌───────────────┐   ┌───────────────┐
    │      Да       │   │      Нет      │
    └───────┬───────┘   └───────┬───────┘
            ▼                   ▼
    ┌───────────────┐   ┌───────────────┐
    │ Статическое   │   │ Динамическое  │
    │ связывание    │   │ связывание    │
    │ (compile-time)│   │ (runtime)     │
    └───────────────┘   └───────────────┘
            │                   │
            ▼                   ▼
    ┌───────────────┐   ┌───────────────┐
    │ По типу       │   │ По реальному  │
    │ ссылки        │   │ типу объекта  │
    └───────────────┘   └───────────────┘
```

---

## 🎯 Резюме

**Динамическое связывание** — это механизм Java, который:
1. **Позволяет реализовать полиморфизм**
2. **Разрешает вызов методов во время выполнения** на основе реального типа объекта
3. **Работает для всех обычных (не static, non-final) методов**
4. **Не работает для полей, static и private методов**

Это один из ключевых механизмов ООП в Java, делающий код гибким и расширяемым без изменения существующего кода (принцип открытости/закрытости).