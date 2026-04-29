>[!info] `java.util.Optional`
# class Optional
Объявление оptional-объектов позволяет разрешить или запретить возможность записи NULL-объектов в переменную. Для этого служат cтатические методы `of()`, `ofNullable()`, `empty()`
```Java
/** Создание Optional объектов */
//Пустой Optional объект
Optional<Person> optionalPerson = Optional.empty();
//Optional объект с ненулевым значением
Optional<Person> optionalNonNull = Optional.of(somePerson);
//Optional объект с возможностью нулевого значения
Optional<Person> optionalNullable = Optional.ofNullable(somePerson);
```
`ifPresent()` проверка not NULL и выполнением лямбда выражения
```Java
if(person != null) {
	System.out.println(person);
}
person.ifPresent(System.out::println);
person.ifPresent( st -> System.out.println(st));
```
`isPresent()` - возвращает булево, как результат проверки, является ли объект NULL
```Java
if(person != null) {
	System.out.println(person)
}
if (person.isPresent()) {
	System.out.println(person.get());
}
```
`orElse()`, `orElseThrow()` - статические методы, которые позволяют сократить код путем применения лямбда выражений.
```Java
State st = Optional.ofNullable(getStateOrNull(str)).orElse(new State());
Optional<Person> person = Optional.ofNullable(p);
Person personNew = person.orElse(new Person());
Person personNewThrow = person.orElseThrow(Exception::new);
```