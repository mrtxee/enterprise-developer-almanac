## топики собеса

- [x] Отличия между слабыми, мягкими, фантомными и обычными ссылками в Java
- [x] Эпсилон гарбадж коллектор
- [x] Корневые множества
- Ссылки из рутов
- Main

Мутабельный

- ThreadLocal
- Кэши в хибернете

Можем поймать эррор

- [x] трайКэтч
	- [ ] В кэтче ловишь исключение

паттерны
- Сhain-of-responsibility вспомнить паттерн
- Событие
- Декоратор

В базе 3 значения

Спринг

10 элементов

Почему peek() может не вызваться?

Reduce?

Паралельные стримы

Когда надо
- 16-ядерн
- FixedThreadPool

Реордеринг операций на примере
```java
private int x;
private int y;

public void Thread1() {
	int r1 = y;
	x = 1;
	sout(r1);
}

public void Thread2() {
	int r2 = x;
	y = 1;
	sout(r2);
}
```

synhronizedMap

N+1 проблема

Не оптимальное построение запросов

Утечки памяти

Кластерные и некластерные индекс в базе данных

EXPLAIN SELECT * FROM categories

- Из чего состоит?

Контример когда наличие индекса не даст прирост производительности

Блокировки транзакций
- Оптимистическая
- Пессимистическая

этапы инициации контекста спринга.

beanDefinition

Spring MVC как устроен

Паттерны микросервисной архитектуры, кафка

- ALLBOX

Почему кафка быстрее реббита?

Высоконагржуенные приложения клептона

ALLBOX
## вопросы с собесов

1. Напиши алгоритм
    - Напиши алгоритм quicksort
        ```Java
        private static void swap(int a, int b, int[] arr) {
            int tmp = arr[a];
            arr[a] = arr[b];
            arr[b] = tmp;
        }
        
        private static int getNewPivot(int left, int right, int[] arr) {
            int pivot = right;
            right--;
        
            while (left < right) {
                while (arr[left] < arr[pivot] && left < right) {
                    left++;
                }
                while (arr[right] > arr[pivot] && left < right) {
                    right--;
                }
                if (right != left) {
                    swap(left, right, arr);
                }
            }
            if (arr[left] > arr[pivot]) {
                swap(left, pivot, arr);
            } else {
                left = pivot;
            }
            return left;
        }
        
        private static void quickSort(int[] arr, int left, int right) {
            if (left < right) {
                int pivot = getNewPivot(left, right, arr);
                quickSort(arr, 0, pivot - 1);
                quickSort(arr, pivot + 1, right);
            }
        }
        
        public static void sort() {
            int[] arr = new int[]{3, 5, 8, 1, 2, 9, 4, 7, 6};
            System.out.println(Arrays.toString(arr));
            quickSort(arr, 0, arr.length - 1);
            System.out.println(Arrays.toString(arr));
        
        }
        ```
        
    
    - [x] обход бинарного дерева поиска в ширину
    - [x] расчет факториала в 2 строчки
2. Расскажи про типы индексов в БД (кластерные / некластерные)
    1. ???
- [x] Что такое **идемпотентность**?
    - идемпотентность
        
        Идемпотентная операция — это операция, которая при многократном вызове возвращает один и тот же результат.
        
        Пример: `Delete where id=3` - идемпотентная. `Insert name=”вася” into table` - не идемпотентная.
        
        Идемпотентные операции - хорошо, т.к. не могут нарушить консистентность данных, а не идемпотентные - плохо.
        
        Чтобы сделать операцию идемпотентной, обычно, стоит использовать `Idempotency-Key: <UUID>` в api-запросах. Сервер выполнит операцию, только если данный ключе не был зарегистрирован ранее, но в любом случае вернет результат, будто операция была выполнена.
        
1. Что такое **api gateway — API-шлюз**?
    - **API Gateway — API-шлюз**
2. [x] Привыкни к запросам вида `SELECT id_department, MAX(salary) FROM employee GROUP BY id_department HAVING MAX(salary)<1000`
3. [x] Расскажи про GC
4. Как упорядочивается куча в JMM по мере того, как объекты в ней отрабатывают
    - [x] Как удаляются объекты из кучи при чистке муссора
5. В джава бывает 4 типа ссылок
    1. [x] стронг референст
    2. [x] вик реф
        1. разберись с примером, как исползуеют в кэшах
    3. [x] софт
        1. !!
    4. [x] фантом
- [x] Для чего нужна аннотация `@Qualifier` в Spring ?
- Если я сделаю volatile переменную — счетчик и вызову инкремент счетчика в 2 потока по 10 раз, будет у меня результат 20 или нет?
    
    _Корректный результат в даном случае не гарантирован. Т.к. потоки могут работать с разной скоростью, а операция инкремента не является атомарной. Поэтому может получится что оба потока прочитают одинаковое значение, а после поочередно запишут инрементированное значение. Таким образом часть результата будет потеряна. Для того, чтобы гарантировать поочередный доступ можено сделать операцию инкремента synchronized, но это будет работать медленно. Мы практически лишимся от преимуществ паралельного выполнения. Поэтому можно рассмотреть операции из Atomic типов, которые построе по аналогии с CAS-инструкцией процессора. Ключевая операция Atomic типов — compareAndWrite._
    
1. Напиши в многопоточную операцию чтобы посчитать сумму из 10 списков в 10 потоков
- Научись пользоваться GROUP BY
    
    ```SQL
    SELECT COUNT(CustomerID), Country
    FROM Customers
    GROUP BY Country
    ORDER BY COUNT(CustomerID) DESC;
    
    -- SELECT column_name(s)
    -- FROM table_name
    -- WHERE condition
    -- GROUP BY column_name(s)
    -- ORDER BY column_name(s); 
    ```
    
- Что делать, если я складывал много Long-ов и вышел за диапазон?
    
    _Диапазон значений как бы прокрутится по кругу и мы продолжим подсчет с наименьшего (отрицательного) числа в диапазоне_
    
- Как BigInteger и BigDecimal поможет с этим?
    1. _Чтобы не обрабатывать вручную свойство JVM “прокручивать диапазон” при превышении значений, можно использовать существующие классы для работы с большими знечениями —_ BigInteger и BigDecimal. _Это классы, которые умеют под капотом считать большие. Суть в том, что на вход они принимают число как строку, и выдают строку. Корректный подсчет больших значений реализован под капотом._
- SQL: WHERE vs HAVING
    
    _WHERE выполняется до выполнения агрегатных функций (например GROUP BY), a HAVING после. Это важно понимать для экономии ресурсов при написании фильтров запроса. Будет разумно вначале уменьшить диапазон строк для агрегации при помощи WHERE, а после отфильтровать по значению агрегатов при помощи HAVING._
    
1. Что такое ==@EntityGraph==
2. Что такое ==@EmbededClass==
3. Что такое ==Иерархия запросов==?
4. Чем @Controller отличается от ==@RestController==
    1. @RestController является комбинацией из `@Controller` и `@ResponceBody`
5. Расскажи про Jenkins и TeamCity
6. Расскажи про CI/CD
7. По каким критериям принимать решение о способе расширения возможностей класса: Когда через наследование IS A, а когда через композицию HAS A
- Почему в Spring рекомендуется внедрять зависимости через конструктор, а не через сеттеры?
    - _внедрение зависимостей через конструктор способствует созданию неизменяемых (immutable) объектов, что повышает безопасность, упрощает управление зависимостями, читабельность кода_
    - _более простой и надежный способ инициализации объектов, поскольку все необходимые зависимости передаются при создании объекта. Это делает код более чистым и понятным_
    - _упрощает тестирование объектов за счет возможности передавать (mocks) зависимостей при их создании._
    - _помогает избежать ситуаций, когда объект находится в недостаточно инициализированном состоянии_
- Как профилировать выполнение SQL запросов?
    
    `EXPLAIN ANALYZE SELECT ...`
    
1. Как резолвить циклические зависимости в Спринг?
2. Что такое нормальная форма в БД?
3. Как имплементить роли пользователя на методы?
4. Актуаторы в к8с
5. Как устроены аннотации
**ОБСУДИТЬ С ХАРПСЕРГОМ:**
- [https://cloud-mts.notion.site/385d2ea65f7f4b83a48df64ac2dda1a2](https://www.notion.so/385d2ea65f7f4b83a48df64ac2dda1a2?pvs=21)
- Когда анализируешь SQL запрос, на что обращаешь внимание в первую очередь?
- [https://yandex.ru/jobs/vacancies/java-разработчик-в-кинопоиск-16733](https://yandex.ru/jobs/vacancies/java-%D1%80%D0%B0%D0%B7%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%87%D0%B8%D0%BA-%D0%B2-%D0%BA%D0%B8%D0%BD%D0%BE%D0%BF%D0%BE%D0%B8%D1%81%D0%BA-16733)

## что изучить

### Индексы на примере PostgreSQL
PostgreSQL поддерживает несколько типов индексов: **B-дерево, хеш, GiST, SP-GiST, GIN и BRIN**. Для разных типов индексов применяются разные алгоритмы, ориентированные на определённые типы запросов. По умолчанию команда CREATE INDEX создаёт индексы типа B-дерево, эффективные в большинстве случаев… 
### Можем авторвайрить через аннотацию, через конструктор, через метод. Как лучше?
### Как избежать дедлока?
```java
synchronized (a1) {
	synchronized (a2) {
	// Проверки
		a1.money = a1.money - summa;
		a2.money = a2.money + summa;
	}
}
```

### Будет ли увеличитьваться счетчик при ошибки транзакции?
```java
@Service
class Bean {
    private int count = 0;
    @Transactional
    public void method() {
        count++;
        // что-то делаем
        // получаем exception
    }

    public int getCount() {
        return count;
    }
}

```
## Базы данных
### индексы
### блокировки
оптимистическая / пессимистическая
### SQL Explain, Analyze
как устроены
## Java virtual machine
[[JDK JVM JLS JNI]]
### сборка мусора
### корневые множества
#### поколения
#### циклы сборки
поколения, минорный мажорный цикл сборки
### типы сборщиков мусора
с пониманием деталей алгоритмов
[[JMM]]
#### Parallel
#### Serial
#### Epsilon
#### G1
#### CMS
#### ZGC
#### Shadara
### типы ссылок java
[[Java Core]]

## Параллелизм
[[java.util.concurrent]] [[Многопоточность]]
#### Проблемы паралелизма
#### Reordering
#### Happens-before
### Потоки

## Java Core
### Stream API
почему peek может не выполниться?
[[java.util.stream]]
#### Parallel Stream
Устройство под капотом: ФоркДжойнПуул, н-1
## Hibernate
[[Java Persistence API]] [[hibernate]]
Как устроен?
**Н+1 проблема**, как бороться? Почему называется Н+1? Как создать утечку памяти? Какие еще проблемы есть в хибернейте?
## Spring
[[Spring]]
### Spring MVC
как устроен?
## Web
### Rest
### SOAP
[[SOAP]]
## Брокеры сообщений
### Kafka
[[Kafka]]
### AMQP
[[Java faq]]
[[faq]]
### SOLID
Хороший пример для каждой буквы
[[ООП]]
### Паттерны
Лучше окунуться в знание основных
[[gof]]


# ~ учебный план
- Reverse Polish notation
    - [x] сделать калькулятор обратно польской записи
- Детерминированный конечный автомат
    - [x] DFM в java Maven + JUnit
    - [x] StatesForEmailFSM в JSON resources folder
    - [x] JSON -> JavaClass при помощи Gson
- [x] TypeToken
- [x] record
- [x] что значит троеточие в параметрах сигнатуры метода? (Type... typeArguments)
- [x] класс Optional
- [x] Strteam API
    - [x] класс Collectors
- [x] Java Collection Frameworks
    - [x] ArrayList, LinkedList отличие реализации?
    - [x] interface Iterrator
    - [x] interface Spliterator
- [x] `java.util.concurrent`
    - [x] надо бы тебе про многопоточку посмотреть уроки
    - [x] Thread        
    - [x] почитай про Lock-free алгоритмы, про CAS
        - [x] про Atomic типы
- [x] Java Memory Model
    - [x] happens-before
    - [x] Reordering
    - [x] JMM [https://www.youtube.com/watch?v=iB2N8aqwtxc](https://www.youtube.com/watch?v=iB2N8aqwtxc)
        
        - [ ] InputStram, OutputStream
            - [ ] [https://javarush.com/quests/lectures/questcore.level08.lecture01](https://javarush.com/quests/lectures/questcore.level08.lecture01)
        
        13:00 - что на слайде?  
        19:20 - что такое живые векторные конструкции?  
        
- [x] диспетчерезация методов Java
- [ ] `join()` interrupt(), final*
    - [ ] ForkJoin framework
- [x] reflection apiВ
- [x] java.io.Serializable
- [x] Внутренняя работа HashMap в Java
    - [x] как устроен HashMap, HashSet?
- [x] иерархия исключений
- [x] Контракт equals(), hashCode()
    - [x] Все методы Object
- [x] [https://javarush.com/groups/posts/646-kak-proiskhodit-zagruzka-klassov-v-jvm](https://javarush.com/groups/posts/646-kak-proiskhodit-zagruzka-klassov-v-jvm)
    - [x] понимание что такое classloader и classpath и как это все используется
- [x] как работает память ПК [https://www.youtube.com/watch?v=Wh22_O8jXVQ](https://www.youtube.com/watch?v=Wh22_O8jXVQ)
- [x] общие уроки по java
    - [x] [https://www.youtube.com/playlist?list=PLAma_mKffTOSUkXp26rgdnC0PicnmnDak](https://www.youtube.com/playlist?list=PLAma_mKffTOSUkXp26rgdnC0PicnmnDak)
- [ ] [https://habr.com/ru/company/tinkoff/blog/490738/](https://habr.com/ru/company/tinkoff/blog/490738/)
- [x] Lombok
- [ ] BitSet

> [!info] Отличия между слабыми, мягкими, фантомными и обычными ссылками в Java  
> «Слабые» ссылки и «мягкие» ссылки (WeakReference, SoftReference) были добавлены в Java API давно, но не каждый программист знаком с ними.  
> [https://javarush.com/groups/posts/1267-otlichija-mezhdu-slabihmi-mjagkimi-fantomnihmi-i-obihchnihmi-ssihlkami-v-java](https://javarush.com/groups/posts/1267-otlichija-mezhdu-slabihmi-mjagkimi-fantomnihmi-i-obihchnihmi-ssihlkami-v-java)  
- [x] осмысли это
    - приведение типов при расчетах и погрешность
        
        ```Plain
        System.out.println(9.1 + 0.1);
        System.out.println(9.100001 + 0.100001);
        System.out.println((float)(9.1 + 0.1));
        System.out.println((float)(9.100001 + 0.100001));
        System.out.println((int)(9.1 + 0.1));
        System.out.println((int)(9.100001 + 0.100001));
        ```
        
- [x] Spring [https://www.youtube.com/watch?v=cIYUAKDnFo4](https://www.youtube.com/watch?v=cIYUAKDnFo4)
- [x] Spring [https://www.youtube.com/watch?v=rd6wxPzXQvo](https://www.youtube.com/watch?v=rd6wxPzXQvo)
- [x] log4j
- [ ] [https://habr.com/ru/post/188010/](https://habr.com/ru/post/188010/)
- [ ] [https://www.baeldung.com/cs/verify-if-power-of-two](https://www.baeldung.com/cs/verify-if-power-of-two)
- [ ] [https://habr.com/ru/company/tinkoff/blog/490738/](https://habr.com/ru/company/tinkoff/blog/490738/)
- [ ] static [https://javarush.com/groups/posts/modifikator-static-java](https://javarush.com/groups/posts/modifikator-static-java)
- [x] класс System
- [x] Уровни изоляции в транзакциях
- [x] `@ControllerAdvice`

> [!info] Java quest - Что такое сырые типы (raw types) в Java  
> Если в среде разработки набрать что-то вроде: List list = new ArrayList(); то среда выдаст предупреждение, наподобие: "Raw use of parametrized class 'List'", что переводится как "Сырое использование параметризованного класса 'List'".  
> [https://www.sites.google.com/view/javaquest/книга-рецептов-java/обобщения-рецепты/что-такое-сырые-типы-raw-types-в-java](https://www.sites.google.com/view/javaquest/книга-рецептов-java/обобщения-рецепты/что-такое-сырые-типы-raw-types-в-java)  

> [!info] Online Java Compiler - java  
> Online Java Compiler - The best online Java programming compiler and editor to provide an easy to use and simple Integrated Development Environment (IDE) for the students and working professionals to Edit, Save, Compile, Execute and Share Java Code with in your browser itself.  
> [https://www.tutorialspoint.com/compile_java_online.php](https://www.tutorialspoint.com/compile_java_online.php)  
  
Вопросы на СОБЕСЕ
- **Каков контракт на equals() и hashCode()? Чем он обусловлен?**
- **Как устроены HashSet и HashMap? Как осуществляются основные**  
    **операциии в данных коллекциях? Как применяются методы equals() и**  
    **hashCode()?**

> [!info] Вопросы по Java на собеседовании (1)  
> Вопросы по языку программирования Java для собеседования, часть 1  
> [https://java-online.ru/java-interview-01.xhtml](https://java-online.ru/java-interview-01.xhtml)  

> [!info] Java Core. Вопросы к собеседованию, ч. 3  
> В предыдущих двух статьях мы обсудили некоторые важные вопросы, которые Вам чаще всего задают на собеседованиях.  
> [https://javarush.com/groups/posts/780-java-core-voprosih-k-sobesedovaniju-ch-3](https://javarush.com/groups/posts/780-java-core-voprosih-k-sobesedovaniju-ch-3)  

