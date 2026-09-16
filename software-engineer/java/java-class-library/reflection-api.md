---
aliases:
  - Class
  - Class object
  - Class.forName
  - getDeclaredMethods
  - getMethods
  - Java Reflection
  - java.lang.Class
  - java.lang.reflect
  - Method
  - Reflection
  - Reflection API
  - Класс
  - Рефлексия
---
> `java.lang.reflect`

## Рефлексия

Рефлексия Java — это механизм, который позволяет получать информацию о классах, интерфейсах, полях и методах во время выполнения, не зная их имён на этапе компиляции, и вносить в них изменения.

Reflection API также помогает создавать новые экземпляры классов, вызывать методы и получать или устанавливать значения полей.

**Особенности**

- ❌ Низкая производительность
- ❌ Прореха в безопасности

**Пример вызова метода через Reflection**

```java
import java.lang.reflect.InvocationTargetException;

Class<?> clazz = null;
try {
    clazz = Class.forName("myClass");
    clazz.getMethod("myMethod", Integer.class, String.class).invoke(null, 1, "input2");
} catch (ClassNotFoundException | NoSuchMethodException | IllegalAccessException |
         InvocationTargetException e) {
    throw new RuntimeException(e);
}
```

## Класс java.lang.Class

> [!info] `import java.lang.Class`
> `implements java.io.Serializable, GenericDeclaration, Type, AnnotatedElement, TypeDescriptor.OfField<Class<?>>, Constable`

Экземпляры класса `Class` представляют классы и интерфейсы в работающем Java-приложении. Enum-класс и [[record-type|record-класс]] — виды классов; annotation-интерфейс — вид интерфейса. Каждый массив также принадлежит классу, представленному объектом `Class`, общим для всех массивов с одинаковым типом элемента и количеством измерений. [[computer-memory|Примитивные типы Java]] (boolean, byte, char, short, int, long, float, double) и ключевое слово `void` также представлены объектами `Class`.

Объект `Class` создаётся автоматически [[jvm|виртуальной машиной Java]] при загрузке класса из байт class-файла.

**Основные методы класса Class**

```java
String getName(); // Полное название класса
int getModifiers(); // Модификаторы доступа
Package getPackage(); // Информация о пакете
Class getSuperclass(); // Класс-родитель
Class[] getInterfaces(); // Массив интерфейсов
Constructor[] getConstructors(); // Конструкторы класса
Field[] getFields(); // Поля класса
Field getField(String fieldName); // Поле класса по имени
Method[] getMethods(); // Массив методов
getSimpleName(); // Простое имя класса
getCanonicalName(); // Каноническое имя класса
getTypeName(); // Информативная строка с именем типа
```

**Получение класса объекта**

```java
// .forName()
try {
    Class<?> aClass = Class.forName("com.company.Person");
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}

// статически
Class aClass = Person.class;

// object.getClass()
Class aClass = person.getClass();
```

**Получение методов класса**

Методы `java.lang.reflect.Method`:

`Method[] getDeclaredMethods()` возвращает массив всех объявленных методов класса.

`Method[] getMethods()` возвращает массив всех объявленных методов класса, а также методы, унаследованные от суперклассов и суперинтерфейсов.

```java
final Method[] declaredMethods = Number.class.getDeclaredMethods();
List<String> actualMethodNames = getMethodNames(declaredMethods);
actualMethodNames.forEach(System.out::println);

private static List<String> getMethodNames(Method[] methods) {
    return Arrays.stream(methods)
        .map(Method::getName)
        .collect(Collectors.toList());
}
```

**Получение полей класса**

Методы `Field[] getFields()`, `Field[] getDeclaredFields()` и `Field getField(String name)` используются для получения полей класса.
