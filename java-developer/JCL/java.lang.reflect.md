>[!info] `java.lang.reflect`
# рефлесия
Рефлексия Java — это механизм, который позволяет разработчику вносить изменения и получать информацию о классах, интерфейсах, полях и методах во время выполнения, не зная их имен при этом.
Reflection API также помогает создавать новые экземпляры классов, вызывать методы и получать или устанавливать значения полей.
Недостатки
- низкая производительность
- прореха в безопасности
```Java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Type;
Class<?> clazz = null;
  try {
    clazz = Class.forName("myClass");
    clazz.getMethod("myMethod", Integer.class, String.class).invoke(null, 1, "input2");
  } catch (ClassNotFoundException | NoSuchMethodException | IllegalAccessException |
           InvocationTargetException e) {
    throw new RuntimeException(e);
  }
};
```
# class Class
>[!info] `import java.lang.Class`
>`implements java.io.Serializable, GenericDeclaration, Type, AnnotatedElement, TypeDescriptor.OfField<Class<?>>, Constable`

Instances of the class Class represent **classes** and **interfaces** in a running Java application. An **enum** class and a **record** class are kinds of class; an annotation interface is a kind of interface. Every **array** also belongs to a class that is reflected as a Class object that is shared by all arrays with the same element type and number of dimensions. The **primitive Java types** (boolean, byte, char, short, int, long, float, and double), and the keyword **void** are also represented as Class objects.
Class object is constructed automatically by the Java Virtual Machine when a class is derived from the bytes of a class file
Основные методы
```Java
String getName(); // Возвращает название класса
int getModifiers(); // Возвращает модификаторы доступа
Package getPackage(); // Возвращает информацию о пакете
Class getSuperclass(); // Возвращает информацию о классе-родителе
Class[] getInterfaces(); // Возвращает массив интерфейсов
Constructor[] getConstructors(); // Возвращает информацию о конструкторах класса
Fields[] getFields(); // Возвращает поля класса
Files getFiled(String fieldName); // Возвращает определенное поле класса по имени
Method[] getMethods(); // Возвращает массив методов
getModifiers() // Получение модификаторов нашего класса
getPackage() // Получение пакета, в котором лежит наш класс
getSuperclass() // 	Получение суперкласса
getInterfaces() // Получение массива интерфейсов, которые имплементируют класс
getName() // Получение полного имени класса
getSimpleName() //  	Получение названия класса
getName() //возвращает имя сущности.
getCanonicalName() // возвращает каноническое имя базового класса, как определено спецификацией языка Java. Возвращает null, если базовый класс не имеет канонического имени (то есть если это локальный или анонимный класс или массив, тип компонента которого не имеет канонического имени).
getSimpleName() // возвращает простое имя базового класса, как указано в исходном коде. Возвращает пустую строку, если базовый класс является анонимным.
getTypeName() // возвращает информативную строку для имени этого типа.
```
## Получение класса объекта
```Java
// .forName()
try {
    Class<?> aClass = Class.forName("com.company.Person");
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
// статически
Class aClass = Person.class;
//object.getClass().
Class aClass = person.getClass();
```
## Получение методов класса
`import java.lang.reflect.Method`
`Method[] getDeclaredMethods()` возвращает массив всех объявленных методов класса.
`Method[] getMethods()` возвращает массив всех объявленных методов класса а так же методы унаследованые от суперклассов и суперинтерфейсов.
```Java
final Method[] declaredMethods = Number.class.getDeclaredMethods();
        List<String> actualMethodNames = getMethodNames(declaredMethods);
        actualMethodNames.forEach(System.out::println);
private static List<String> getMethodNames(Method[] methods) {
        return Arrays.stream(methods)
                .map(Method::getName)
                .collect(Collectors.toList());
```
## Получение полей класса
`import java.lang.reflect.Field`
Методы `Field[] getFields()`, `Field[] getDeclaredFields()` и `Field getField(String name)`используются для получения полей класса.