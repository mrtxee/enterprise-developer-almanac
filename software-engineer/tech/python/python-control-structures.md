---
aliases:
  - Constructions
  - Constructions Python
  - List comprehension
  - Loop control
  - matplotlib
  - NumPy
  - Panda3D
  - PEP
  - PEP 8
  - pip
  - pip install
  - Py-операторы
  - pygame
  - PyPI
  - Python control structures
  - Python operators
  - Python Package Index
  - SciPy
  - Syntactic sugar
  - Walrus operator
  - Глоссарий Python
  - Конструкции Python
  - Культура программирования Python
  - Оператор assert
  - Оператор проверки утверждения
  - Тернарный оператор
  - Троичный оператор
  - Управляющие конструкции
  - Условные операторы
  - Условный оператор
  - Установка модулей pip
---

## Синтаксический сахар

- Групповое переназначение переменных

```python
  x, y = y, x
```

- Дзен Питона, **пасхалка**: прочитать философию питона

```python
  import this
  '''
  The Zen of Python, by Tim Peters

  Beautiful is better than ugly.
  Explicit is better than implicit.
  Simple is better than complex.
  Complex is better than complicated.
  Flat is better than nested.
  Sparse is better than dense.
  Readability counts.
  Special cases aren't special enough to break the rules.
  Although practicality beats purity.
  Errors should never pass silently.
  Unless explicitly silenced.
  In the face of ambiguity, refuse the temptation to guess.
  There should be one-- and preferably only one --obvious way to do it.
  Although that way may not be obvious at first unless you're Dutch.
  Now is better than never.
  Although never is often better than *right* now.
  If the implementation is hard to explain, it's a bad idea.
  If the implementation is easy to explain, it may be a good idea.
  Namespaces are one honking great idea -- let's do more of those!
  '''
```

- Моржовый оператор `:=`

```python
  print(walrus:=int(input()))
  # то же самое, что и
  not_walrus = int(input())
  print(not_walrus)
```

- List comprehension (генератор коллекции)

```python
  # Генератор коллекции: кубы чисел, делятся на 6 без остатка
  cubes = [i**3 for i in range(50) if i%6==0]  # [0, ...]
  print(cubes)
```

- Присваивание с оператором `+=`, `-=`, `*=`, `/=`, `%=`, `**=`, `//=`

```python
  x = 3
  for i in range(1, 9, 2):
      x *= i
  print(x)
```

- Троичный оператор, Ternary operator

```python
  # троичный оператор
  status = 1
  msg = "Logout" if status == 1 else "Login"
  print(msg)
```

## Вывод данных и комментарии

**Пример вывода данных**

```python
  print('this is how output')
  # this is a short comment
  '''
  this is a
  multyline comment
  '''
  print('''this is a
  multyline output''')
  name = 'Вася'  # this is variable declaration
  nums = [4, 5, 6]  # this is list of values
  print('Здесь был ' + name)  # string concatination classic style output
  print('Здесь был', name)  # string concatination with auto add spaces output
  print(f'Здесь был {name}')  # f-string style output
  print('Numbers: {0} {1} {2}'.format(name, nums[1], nums[2]))  # .format(*args, **kwargs) style output
```

## Ввод данных

**Пример ввода данных**

```python
  # общение с оператором в терминале
  print('your input is: ', input('Введите строку: '))
```

## Условный оператор

**Синтаксис условного оператора**

```python
  if [smth] :
      ...
  [elif [smth] :
      ...]
  [else:
      ...]
```

### Conditional statements all(), any()

**Пример функций all(), any() и enumerate**

```python
  nums = [55, 44, 33, 22, 11]
  if all([i > 5 for i in nums]):
      print("All larger than 5")
  if any([i % 2 == 0 for i in nums]):
      print("At least one is even")
  for v in enumerate(nums):
      print(v)
```

### Оператор match-case

**Пример оператора match-case**

```python
  match action:  # switch
      case "truncate_devices":
          result['data']['msg'] = f"do {action} for {action_id}"
      case "reload_devices":
          result['data']['msg'] = f"do {action} for {action_id}"
      case _:  # default
          result['success'] = False
          result['data']['msg'] = f"unknown action: {action} for {action_id}"
```

## Оператор проверки высказывания assert

Оператор `assert` проверки утверждения. Выбрасывает исключение `AssertionError` — _ошибка утверждения_, если утверждение не проходит логическую проверку.

**Пример оператора assert**

```python
  # пример raise
  print(1)
  assert 2 + 2 == 4
  print(2)
  assert 1 + 1 == 3  # except raise AssertionError
  print(3)
  # raise AssertionError("Colder than zero!") if False
  temp = -10
  assert (temp >= 0), "Colder than zero!"
```

## Циклы

**Синтаксис цикла for**

```python
  for i in range(1, 3):
      [smth]
      [continue]
      [break]
  [else:
      ...]
```

**Синтаксис цикла while**

```python
  while i < 100:
      i += 1
      [continue]
      [break]
  [else:
      ...]
```

### Операторы прерывания цикла

- Оператор `continue` позволяет пропустить все инструкции ниже него и перейти к началу следующей итерации.
- Оператор `break` прерывает выполнение операций, при этом все инструкции ниже `break` в текущей итерации не будут выполнены.
- Блок `else` цикла будет выполнен один раз, если в цикле не было `break`.

**Пример цикла с блоком else**

```python
  # for ... else statement
  men = {18, 26, 15}
  ages = []
  i = 0
  while i < 3:
      age = men.pop()
      if age < 16:
          print("Too young!")
          break
      ages.append(age)
      i += 1
  else:
      print("Get ready!")
```

### Функция range(start, stop, step)

`range()` returns a sequence of numbers, in a given range. The most common use of it is to iterate a sequence of numbers using Python loops.

> start: optional start value of the sequence
> stop: value after the end value of the sequence
> step: optional integer value, denoting the difference between any two numbers in the sequence

**Пример использования range**

```python
  for i in range(3, 99, 3):
      print(i)
```

## Исключения

### Типы исключений

- `ImportError`: an import fails;
- `IndexError`: a list is indexed with an out-of-range number;
- `NameError`: an unknown variable is used;
- `SyntaxError`: the code can't be parsed properly;
- `TypeError`: a function is called on a value of an inappropriate type;
- `ValueError`: a function is called on a value of the correct type, but with an inappropriate value;
- `ZeroDivisionError`
- `OSError`
- …
- custom exceptions from third-party developers, etc.

### Ловить исключения

**Пример обработки исключений**

```python
  try:
      num1 = 7
      num2 = 0
      print(num1 / num2)
      print("Done calculation")
  except ZeroDivisionError:
      print("An error occurred")
      print("due to zero division")
  except (ValueError, TypeError):
      print("Value or type error")
  except:  # любое исключение
      print("Any other Error occurred")
  else:
      print("There was no any exception!")
  finally:
      print("This code will run no matter what")
  raise TypeError("bad settings provided")
  ...
  try:
      tcc = get_tcc(action_id)
  except (KeyError, TypeError) as e:
      print(f"Exception: {str(e)}")  # Exception: bad settings provided
```

### Отправить исключение

**Пример оператора raise**

```python
  num = 99
  if num > 100:
      raise ValueError("Число больше 100!")
```

## Файлы

### Режимы доступа к файлу

- `r` — opens a file for reading only. The file pointer is placed at the beginning of the file. This is the default mode.
- `rb` — opens a file for reading only in binary format. The file pointer is placed at the beginning of the file. This is the default mode.
- `r+` — opens a file for both reading and writing. The file pointer will be at the beginning of the file.
- `rb+` — opens a file for both reading and writing in binary format. The file pointer will be at the beginning of the file.
- `w` — opens a file for writing only. Overwrites the file if the file exists. If the file does not exist, creates a new file for writing.
- `wb` — opens a file for writing only in binary format. Overwrites the file if the file exists. If the file does not exist, creates a new file for writing.
- `w+` — opens a file for both writing and reading. Overwrites the existing file if the file exists. If the file does not exist, creates a new file for reading and writing.
- `wb+` — opens a file for both writing and reading in binary format. Overwrites the existing file if the file exists. If the file does not exist, creates a new file for reading and writing.
- `a` — opens a file for appending. The file pointer is at the end of the file if the file exists. That is, the file is in the append mode. If the file does not exist, it creates a new file for writing.
- `ab` — opens a file for appending in binary format. The file pointer is at the end of the file if the file exists. That is, the file is in the append mode. If the file does not exist, it creates a new file for writing.
- `a+` — opens a file for both appending and reading. The file pointer is at the end of the file if the file exists. The file opens in the append mode. If the file does not exist, it creates a new file for reading and writing.
- `ab+` — opens a file for both appending and reading in binary format. The file pointer is at the end of the file if the file exists. The file opens in the append mode. If the file does not exist, it creates a new file for reading and writing.
- Adding **`b`** to a mode opens it in **binary** mode.

### Чтение файла

- Для чтения всего текста в переменную подойдёт метод `.read()`

```python
  # прочитаю файл, либо создам новый и прочитаю со второй попытки
  fname = "filename.txt"
  try:
      f = open(fname, "r")
      print(f.read())
      # второй вариант - получить список строк
      # lines = file.readlines()
  except FileNotFoundError:
      print("не удалось открыть файл " + fname)
      print("создаю новый")
      f = open(fname, "w")
      f.write("This has been written to a file start")
  except:
      print("другой сбой при работе с файлом")
  finally:
      f.close()
```

- Для построчного чтения можно использовать метод `.readline()`

> Применение конструкции `with ... as` позволяет исключить необходимость вызова `.close()` и ловить исключения.

```python
  with open(filename) as f:
      while (line := f.readline().rstrip()):
          print(line)
```

- Для автоматического закрытия файла подойдёт конструкция `with open(fname) as f:`

```python
  # не требует f.close()
  with open(fname) as f:
      print(f.read())
```

### Запись в файл

**Пример записи в файл**

```python
  # оптимальная запись без ловли исключений и ручного закрытия файла
  fname = 'somefile.txt'
  with open(fname, "w", encoding="utf-8") as f:
      bytes_written = f.write(html)
  print(f'в файл {fname} записано: {bytes_written}')
  # дозапись в конец файла в режиме append
  fname = 'somefile.txt'
  try:
      file = open(fname, "a", encoding="utf-8")
      bytes_written = file.write("\nThis has been written WITH APPEND to a file")
      print(f'в файл {fname} записано: {bytes_written}')
  except Exception as exception:
      print('поймал сбой при работе с файлом', type(exception).__name__)
  finally:
      file.close()
```

## Типы данных

- `bool`
- `int`
- `float`
- `str`
- …

### Приведение типов

**Пример приведения типа**

```python
  # приведение строки к булену
  str = '1'
  print(bool(int(str)))
```

### Строки str

#### Функции строк

- `.join()` — joins a list of strings with another string as a separator.
- `.split()` — the opposite of join, turns a string with a certain separator into a list.
- `.replace(old, new[, count])` — replaces one substring in a string with another.
- `.count(sub[, start[, end]])` — количество вхождений подстроки в строку.
- `.startswith()` and `.endswith()` — determine if there is a substring at the start and end of a string, respectively.
- `.lower()` and `.upper()` — changes the case of a string.
- …
- `if "blah" not in string_var:` — вариант поиска по подстроке.

**Пример методов строк**

```python
  print(", ".join(["spam", "eggs", "ham"]))
  # prints "spam, eggs, ham"
  print("Hello ME".replace("ME", "world"))
  # prints "Hello world"
  print("This is a sentence.".startswith("This"))
  # prints "True"
  print("This is a sentence.".endswith("sentence."))
  # prints "True"
  print("This is a sentence.".upper())
  # prints "THIS IS A SENTENCE."
  print("AN ALL CAPS SENTENCE".lower())
  # prints "an all caps sentence"
  print("spam, eggs, ham".split(", "))
  # prints "['spam', 'eggs', 'ham']"
```

#### Операции со строками

`+`, `*`

### Специальные объекты

- `None` object is used to represent the absence of a value.

## Глоссарий

### Установка модулей pip

**Пример установки модуля**

```bash
  pip install numpy
```

### Популярные внешние модули

- Модуль **`matplotlib`** позволяет создавать графики на основе данных в Python.
- Модуль **`NumPy`** позволяет использовать многомерные массивы, которые намного быстрее нативного Python-решения на вложенных списках.
- **`SciPy`** содержит многочисленные расширения функциональности NumPy.
- **`Panda3D`** — для 3D-игр.
- **`pygame`** — для игр.

### PEP

**Python Enhancement Proposals (PEP)** — предложения по улучшению языка, сделанные опытными разработчиками Python.

### PIP (Package Installer for Python)

**Package Installer for Python (pip)** — система управления пакетами, которая используется для установки и управления программными пакетами, написанными на Python. Начиная с версии Python 2.7.9 и Python 3.4, они содержат пакет pip (или pip3 для Python 3) по умолчанию.

### PyPi (Python Package Index)

The Python Package Index (PyPI) is a repository of software for the Python programming language. Сайт, где описаны все пакеты, доступные в pip.

### Traceback

`traceback` — печать или получение обратной трассировки стека. Трассировка ошибок в консоли.

### Культура программирования Python

**PEP 8 is a style guide on the subject of writing readable code.** It contains a number of guidelines in reference to variable names, which are summarized here:

- **modules** should have short, **all-lowercase names**;
- **class** names should be in the **CapWords** style;
- most **variables and function** names should be **lowercase_with_underscores**;
- **constants** should be **CAPS_WITH_UNDERSCORES**;
- names that would clash with Python keywords should have a **trailing underscore**, eg. `for_`, `if_`;
- lines shouldn't be longer than 80 characters;
- `~~from module import *~~` should be avoided;
- there should only be one statement per line.

## Операторы Python

### Приоритет операторов

- Операторы сравнения (`==`, `>=`, `<`, ...) имеют более высокий приоритет, чем `or`, `xor`, `and`.
- Скобки служат для выделения приоритетов логики.

**Пример приоритета операторов**

```python
  print(False == False or True)
  # True
  print(False == (False or True))
  # False
  print((False == False) or True)
  # True
```

| Операторы | Описание |
|---|---|
| `**` | Возведение в степень |
| `~` | Комплиментарный оператор |
| `*`, `/`, `%`, `//` | Умножение, деление, деление по модулю, целочисленное деление |
| `+`, `-` | Сложение и вычитание |
| `>>`, `<<` | Побитовый сдвиг вправо и побитовый сдвиг влево |
| `&` | Бинарный "И" |
| `^`, `\|` | Бинарный "Исключительное ИЛИ" и бинарный "ИЛИ" |
| `<=`, `<`, `>`, `>=` | Операторы сравнения |
| `<>`, `==`, `!=` | Операторы равенства |
| `=`, `%=`, `/=`, `//=`, `-=`, `+=`, `*=`, `**=` | Операторы присваивания |
| `is`, `is not` | Тождественные операторы |
| `in`, `not in` | Операторы членства |
| `not`, `or`, `and` | Логические операторы |

### Арифметические операторы

`+`, `-`, `*`, `/`, `%`, `**`, `//`

- `%` — возвращает остаток от деления.
- `//` — возвращает целочисленную часть от деления, без дробной.
- `**` — возведение в степень.

### Операторы присваивания

`=`, `+=`, `-=`, `*=`, `/=`, `%=`, `**=`, `//=`

### Логические операторы

`<`, `>`, `==`, `<=`, `>=`, `!=`, `not`

### Операторы членства

`in`, `not in`

### Операторы тождественности

`is`, `is not`

### Побитовые операторы

- `&` — бинарный "И": копирует бит в результат, только если бит присутствует в обоих операндах.
- `|` — бинарный "ИЛИ": копирует бит, если тот присутствует хотя бы в одном операнде.
- `^` — бинарный "Исключительное ИЛИ": копирует бит, только если бит присутствует в одном из операндов, но не в обоих сразу.
- `~` — бинарный комплиментарный оператор. Является унарным (то есть ему нужен только один операнд), меняет биты на обратные: там, где была единица, становится ноль, и наоборот.
- `<<` — побитовый сдвиг влево. Значение левого операнда "сдвигается" влево на количество бит, указанных в правом операнде.
- `>>` — побитовый сдвиг вправо. Значение левого операнда "сдвигается" вправо на количество бит, указанных в правом операнде.

## Парадигмы программирования

1. **Императивное** программирование
   - линейные высказывания, циклы, функции как подпрограммы
2. **Функциональное** программирование
   - чистые функции, функции высшего порядка, рекурсии, λ-исчисление
3. **Объектно-ориентированное** программирование
   - классы, объекты, наследование, инкапсуляция, полиморфизм
