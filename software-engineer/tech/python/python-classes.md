---
aliases:
  - __eq__
  - __init__
  - __repr__
  - __str__
  - class
  - Classes
  - Python
  - Python classes
  - self
  - Волшебные методы
  - Классы Python
  - Магические методы
  - Посылка сообщений
---
## Парадигма ООП

- **Наследование** — возможность создания новых абстракций на основе существующих.
- **[[oop|Инкапсуляция]]** — скрытие внутреннего состояния и функций объекта и предоставление доступа только через открытый набор функций.
- **Полиморфизм** — возможность реализации наследуемых свойств или методов отличающимися способами в рамках множества абстракций.
- **Абстракция** — моделирование требуемых атрибутов и взаимодействий сущностей в виде классов для определения абстрактного представления системы.
- **Посылка сообщений** (добавлено в Java) — форма связи, взаимодействия между сущностями.
- **Переиспользование** (добавлено в Java) — всё, что перечислено выше, работает на повторное использование кода.

### Классы в Python

Классы нужны, так как имплементируют логику объектно-ориентированного подхода программирования.

При описании всех функций во всех объявлениях функций класса, кроме **_статического метода_** и **_метода класса_**, первым аргументом идёт `self`.

Пример класса с конструктором и методом:

```python
  class Student:
      def __init__(self, name):
          self.name = name

      def greet(self):
          print(self.name+" says hi")
  obj = Student("John")
  obj.greet()
```

#### Инкапсуляция

- Инкапсуляция ~ сокрытие данных.
- Выделение параметра или метода символом `_param` говорит о том, что он protected, но не закрывает доступ к нему.
- Выделение параметра или метода символом `__method` делает параметр private, то есть недоступным, но к такому методу всё равно можно обратиться через `_Class__privatemethod`.

Пример скрытия реализации:

```python
  class Queue:
      def __init__(self, contents):
          # маркируем параметры скрытым
          self._hiddenlist = list(contents)
          # делаем параметр private
          self.__privatelist = list(contents)

      def push(self, value):
          self._hiddenlist.insert(0, value)

      def pop(self):
          return self._hiddenlist.pop(-1)

      def __repr__(self):
          return "Queue({})".format(self._hiddenlist)
  queue = Queue([1, 2, 3])
  print(queue)
  queue.push(0)
  print(queue)
  queue.pop()
  print(queue)
  print(queue._hiddenlist)
  # print(queue.__privatelist) # сбой в работе транслятора
```

Можно ограничить доступ, а также описать setter и getter свойства при помощи декоратора `@property`:

```python
  class Pizza:
      def __init__(self, toppings):
          self.toppings = toppings
          self._pineapple_allowed = False
      @property
      def pineapple_allowed(self):
          return self._pineapple_allowed
      @pineapple_allowed.setter
      def pineapple_allowed(self, value):
          if value:
              password = input("Enter the password: ")
              if password == "Sw0rdf1sh!":
                  self._pineapple_allowed = value
              else:
                  raise ValueError("Alert! Intruder!")
          # @pineapple_allowed.getter

  pizza = Pizza(["cheese", "tomato"])
  print(pizza.pineapple_allowed)
  pizza.pineapple_allowed = True
  print(pizza.pineapple_allowed)
```

#### Наследование

Наследуемый класс объявляется через структуру `class My_Class(Parent_Class):`

Пример наследования:

```python
  class Wolf:
      def __init__(self, name, color):
          self.name = name
          self.color = color
      def bark(self):
          print("Grr...")
  class Dog(Wolf):
      def bark(self):
          print("Woof")
  husky = Dog("Max", "grey")
  husky.bark()
```

##### Субкласс и суперкласс

Класс-наследник называется **субкласс, subclass**. Класс-родитель называют **суперкласс, superclass**.

- `super()` — оператор обращения к родительскому классу.

Пример обращения к родительскому классу через `super()`:

```python
  class A:
      def spam(self):
          print('1 - parent')
  class B(A):
      def spam(self):
          print('2 - son')
          super().spam()
  B().spam()
```

#### Полиморфизм

##### Свойства класса

Свойства (**properties**) служат для организации кастомного доступа к параметрам класса с сокрытием внутренних структур класса. Можно описать параметр класса с режимом доступа **read-only**. Декларируется через декоратор `@property`. Для чтения и записи значений такого параметра можно создать `@[название свойства].getter` и `@[название свойства].setter`.

Конструктор класса:

```python
  class Player:
      def __init__(self, name, level):
          self.name = name
          self.level = level
      def intro(self):
          print(self.name + " (Level " + str(self.level) + ")")
  new_player = Player('Tony', 12)
  new_player.intro()
```

##### Магические методы класса

Так же называют **magic method** или **dunders**. Выделяются двойным нижним подчёркиванием в начале и конце. Вызываются методы без `__`.

Переопределение волшебных методов называется **перегрузка операторов, operator overloading**.

Примеры волшебных методов:
  - `__init__()` — конструктор класса.
  - `__add__()` — операция `+`, описывает, как складывать 2 объекта одного класса.
  - `__sub__()` — операция `-`.
  - `__mul__()` — операция `*`.
  - `__truediv__()` — операция `/`.
  - `__floordiv__()` — операция `//`.
  - `__mod__()` — операция `%`.
  - `__pow__()` — операция `**`.
  - `__and__()` — операция `&`.
  - `__xor__()` — операция `^`.
  - `__or__()` — операция `|` ...
  - `__lt__()` for `<`
  - `__le__()` for `<=`
  - `__eq__()` for `==`
  - `__ne__()` for `!=`
  - `__gt__()` for `>`
  - `__ge__()` for `>=`
  - `__len__()` for `len()`
  - `__getitem__()` for indexing
  - `__setitem__()` for assigning to indexed values
  - `__delitem__()` for deleting indexed values
  - `__iter__()` for iteration over objects (e.g., in for loops)
  - `__contains__()` for `in`
  - `__call__()` for calling objects as functions
  - `__repr__()` — magic method is used for string representation of the instance.
  - `__str__()`

##### Статический метод

Метод, к которому можно обращаться, не создавая экземпляр класса. Можно пользоваться как методом, как обычной функцией.

Статический метод декларируется через декоратор `@staticmethod`.

##### Метод класса

Декоратор `@classmethod` объявляет **метод класса**, позволяет классу обращаться к параметрам и методам класса. Этот метод можно вызвать, не создавая экземпляр класса. Метод класса обязан содержать `cls` в качестве первого аргумента, к которому не будет доступа извне.

Пример класса Vector2D:

```python
  class Vector2D:
      def __init__(self, x, y):
          self.x = x
          self.y = y
      def area(self):
          return self.x*self.y
      def __add__(self, other):
          return Vector2D(self.x + other.x, self.y + other.y)
      def __sub__(self):
          return self.x
      def __getitem__(self, index=1):
          if 1==index:
              return self.x
          else:
              return self.y
      def __len__(self):
          return self.x+self.y
      def __gt__(self, other):
          return self.area() > other.area()
      @classmethod
      def new_square(cls, side_length):
          return cls(side_length, side_length)
      # статические методы можно вызывать без экземпляра класса
      @staticmethod
      def area_for_any(h,w):
          return h*w
  first = Vector2D(5, 7)
  second = Vector2D(3, 9)
  result = first + second
  print(f'result.x \t {result.x}')
  print(f'second[2] \t {second[2]}')
  print(f'len(second) \t {len(second)}')
  print(f'first > first \t {first > first}')
  square = Vector2D.new_square(3)
  print(f'square .x, .y \t {square.x}, {square.y}')
  print(f'static_method_call \t {Vector2D.area_for_any(12, 17)}')
```

### Жизненный цикл объекта класса

Жизненный цикл объекта включает стадии **создания → использования → уничтожения** (на английском: creation → manipulation → destruction, definition → instantiation → garbage collection). В течение этого цикла с объектом происходят следующие магические функции:

`__init__(self)` → `__new__(self)` → `__del__(self)`

Объект может быть уничтожен только тогда, когда на него ровно ноль ссылок. Этим занимается автомат среды, который называется **garbage collector** (сборщик мусора). Для очистки памяти можно вручную вызвать метод `del`.

Пример подсчёта ссылок на объект:

```python
  a = 42  # Create object <42>
  b = a  # Increase ref. count  of <42>
  c = [a]  # Increase ref. count  of <42>
  del a  # Decrease ref. count  of <42>
  b = 100  # Decrease ref. count  of <42>
  c[0] = -1  # Decrease ref. count  of <42>
  # probably garbage collector frees memory at this point
```
