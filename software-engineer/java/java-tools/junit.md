---
aliases:
  - @AfterAll
  - @AfterEach
  - @BeforeAll
  - @BeforeEach
  - @Disabled
  - @DisplayName
  - @Nested
  - @Order
  - @Tag
  - @Test
  - @TestFactory
  - @TestMethodOrder
  - Assertions
  - Assumptions
  - DynamicTest
  - JUnit
  - JUnit 5
  - JUnit Jupiter
  - junit-jupiter-api
  - jupiter
  - Lifecycle
  - maven-surefire-plugin
  - org.junit.jupiter
  - org.junit.platform
  - Test lifecycle
  - unit test
  - Unit testing
  - Жизненный цикл теста
  - Предположения
  - Утверждения
---

## JUnit 5 тест со всеми аннотациями

`JUnit` — фреймворк для тестирования.

**Пример: OrderServiceTest.java**

```java
package com.example.order;

import org.junit.jupiter.api.*;
import org.junit.jupiter.api.DynamicTest;

import java.math.BigDecimal;
import java.util.Collection;
import java.util.List;
import java.util.stream.Stream;

import static org.junit.jupiter.api.Assertions.*;
import static org.junit.jupiter.api.Assumptions.assumeTrue;

// Теги для категоризации тестов
@Tag("unit")
@Tag("order-service")
@DisplayName("Тесты сервиса заказов (OrderService)")
class OrderServiceTest {

    private OrderService sut; // system under test
    private static TestDatabase database;

    // @BeforeAll — один раз перед ВСЕМИ тестами класса
    // Должен быть static (если не @TestInstance(PER_CLASS))
    @BeforeAll
    static void init() {
        System.out.println("@BeforeAll: инициализация БД");
        database = new TestDatabase();
        database.connect();
    }

    // @AfterAll — один раз после ВСЕХ тестов класса
    // Должен быть static
    @AfterAll
    static void finalize() {
        System.out.println("@AfterAll: закрытие БД");
        if (database != null) {
            database.disconnect();
        }
    }

    // @BeforeEach — перед каждым тестом
    @BeforeEach
    void setUp() {
        System.out.println("@BeforeEach: создание сервиса");
        sut = new OrderService(database);
        database.clear();
    }

    // @AfterEach — после каждого теста
    @AfterEach
    void tearDown() {
        System.out.println("@AfterEach: очистка состояния");
        sut = null;
    }

    // @Test + @DisplayName — простой тест
    @Test
    @DisplayName("Создание заказа возвращает ненулевой ID")
    void createOrder_shouldReturnNonZeroId() {
        // Given
        Order order = new Order("user-1", new BigDecimal("99.99"));

        // When
        Order created = sut.create(order);

        // Then
        assertNotNull(created.getId(), "ID заказа не должен быть null");
        assertTrue(created.getId() > 0, "ID должен быть положительным");
    }

    // @Test с проверкой исключения
    @Test
    @DisplayName("Заказ с отрицательной суммой выбрасывает IllegalArgumentException")
    void createOrder_withNegativeAmount_shouldThrowException() {
        // Given
        Order invalidOrder = new Order("user-1", new BigDecimal("-10"));

        // When & Then
        IllegalArgumentException exception = assertThrows(
            IllegalArgumentException.class,
            () -> sut.create(invalidOrder),
            "Ожидалось IllegalArgumentException для отрицательной суммы"
        );

        assertEquals("Order amount cannot be negative", exception.getMessage());
    }

    // @Disabled — тест пропущен (например, ещё не готов)
    @Test
    @Disabled("TODO: реализовать после добавления платёжного шлюза")
    @DisplayName("Оплата заказа через платёжный шлюз")
    void payOrder_shouldProcessPayment() {
        // Этот тест не будет выполнен
        fail("Не реализовано");
    }

    // @Tag на уровне метода
    @Test
    @Tag("slow")
    @Tag("integration")
    @DisplayName("Массовое создание 1000 заказов")
    void createOrders_bulk_shouldSucceed() {
        for (int i = 0; i < 1000; i++) {
            Order order = new Order("user-" + i, BigDecimal.TEN);
            assertNotNull(sut.create(order).getId());
        }
    }

    // @Nested — вложенная группа тестов
    @Nested
    @DisplayName("Расчёт скидок")
    @Tag("discount")
    class DiscountTests {

        @BeforeEach
        void setUpDiscounts() {
            System.out.println("@BeforeEach (Nested): настройка скидок");
            sut.enableDiscounts();
        }

        @Test
        @DisplayName("Скидка 10% при сумме > 1000")
        void calculateDiscount_largeOrder_shouldApply10Percent() {
            // Given
            Order order = new Order("user-1", new BigDecimal("1500"));

            // When
            BigDecimal discount = sut.calculateDiscount(order);

            // Then
            assertEquals(new BigDecimal("150.00"), discount);
        }

        @Test
        @DisplayName("Нет скидки при сумме < 1000")
        void calculateDiscount_smallOrder_shouldReturnZero() {
            Order order = new Order("user-1", new BigDecimal("500"));
            assertEquals(BigDecimal.ZERO, sut.calculateDiscount(order));
        }

        // @TestFactory — динамические тесты
        @TestFactory
        @DisplayName("Динамические тесты скидок для разных сумм")
        Collection<DynamicTest> discountDynamicTests() {
            return List.of(
                DynamicTest.dynamicTest("Сумма 999 → скидка 0",
                    () -> assertEquals(BigDecimal.ZERO,
                        sut.calculateDiscount(
                            new Order("u", new BigDecimal("999"))))),

                DynamicTest.dynamicTest("Сумма 1000 → скидка 0",
                    () -> assertEquals(BigDecimal.ZERO,
                        sut.calculateDiscount(
                            new Order("u", new BigDecimal("1000"))))),

                DynamicTest.dynamicTest("Сумма 2000 → скидка 200",
                    () -> assertEquals(new BigDecimal("200.00"),
                        sut.calculateDiscount(
                            new Order("u", new BigDecimal("2000")))))
            );
        }

        // @TestFactory со стримом (больше сценариев)
        @TestFactory
        Stream<DynamicTest> boundaryValueTests() {
            record TestCase(BigDecimal amount, BigDecimal expectedDiscount) {}

            return Stream.of(
                new TestCase(new BigDecimal("0"),     BigDecimal.ZERO),
                new TestCase(new BigDecimal("100"),   BigDecimal.ZERO),
                new TestCase(new BigDecimal("1001"),  new BigDecimal("100.10")),
                new TestCase(new BigDecimal("9999"),  new BigDecimal("999.90"))
            ).map(tc -> DynamicTest.dynamicTest(
                String.format("Сумма %s → скидка %s", tc.amount(), tc.expectedDiscount()),
                () -> assertEquals(tc.expectedDiscount(),
                    sut.calculateDiscount(new Order("u", tc.amount())))
            ));
        }
    }

    // Второй @Nested — тесты валидации
    @Nested
    @DisplayName("Валидация заказов")
    class ValidationTests {

        @Test
        @DisplayName("Пустой userId отклоняется")
        void createOrder_emptyUserId_shouldFail() {
            assertThrows(IllegalArgumentException.class,
                () -> sut.create(new Order("", BigDecimal.TEN)));
        }

        @Test
        @DisplayName("Null userId отклоняется")
        void createOrder_nullUserId_shouldFail() {
            assertThrows(NullPointerException.class,
                () -> sut.create(new Order(null, BigDecimal.TEN)));
        }
    }

    // Вспомогательный класс для теста
    static class TestDatabase {
        void connect() { System.out.println("БД подключена"); }
        void disconnect() { System.out.println("БД отключена"); }
        void clear() { System.out.println("БД очищена"); }
    }
}
```

---

## Порядок выполнения

```mermaid
---
title: Порядок выполнения тестов JUnit 5
---
flowchart TD
    BeforeAll["@BeforeAll — один раз"] --> BeforeEach["@BeforeEach — перед каждым тестом"]
    BeforeEach --> Test["@Test — выполнение теста"]
    Test --> AfterEach["@AfterEach — после каждого теста"]
    AfterEach --> BeforeEach
    BeforeAll --> Disabled["@Disabled — тест пропущен"]
    BeforeEach --> Nested["@Nested — вложенные группы"]
    Nested --> Test
    AfterEach --> AfterAll["@AfterAll — один раз в конце"]
```

---

## Сводная таблица аннотаций

| Аннотация | Когда выполняется | Метод | Назначение |
| --------- | ----------------- | ----- | ---------- |
| **`@BeforeAll`** | 1 раз перед всеми тестами | `static` | Инициализация дорогих ресурсов |
| **`@AfterAll`** | 1 раз после всех тестов | `static` | Закрытие ресурсов |
| **`@BeforeEach`** | Перед каждым тестом | экземпляр | Подготовка состояния |
| **`@AfterEach`** | После каждого теста | экземпляр | Очистка состояния |
| **`@Test`** | Сам тест | экземпляр | Проверка поведения |
| **`@DisplayName`** | Метаданные | любое | Человекочитаемое имя |
| **`@Disabled`** | Пропускается | `@Test` | Временное отключение |
| **`@Nested`** | Вложенный класс | класс | Группировка тестов |
| **`@Tag`** | Метаданные | любое | Категоризация для запуска |
| **`@TestFactory`** | Генерация тестов | возвращает `DynamicNode` | Параметризация в рантайме |

---

## Запуск с фильтрацией по тегам

### Через Maven

```xml
<!-- pom.xml -->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.2</version>
            <configuration>
                <!-- Запуск только тестов с тегом "unit" -->
                <groups>unit</groups>
                <!-- Исключение медленных тестов -->
                <excludedGroups>slow</excludedGroups>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Через командную строку

```bash
# Только тесты с тегом "discount"
mvn test -Dgroups=discount

# Все, кроме "slow"
mvn test -DexcludedGroups=slow

# Комбинация тегов
mvn test -Dgroups="unit & !slow"
```

---

## Важные нюансы

**`@BeforeAll` / `@AfterAll` должны быть static**

```java
// Ошибка: @BeforeAll должен быть static
@BeforeAll
void init() { }  // Compilation Error

// Правильно
@BeforeAll
static void init() { }

// Альтернатива: @TestInstance(PER_CLASS)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyTest {
    @BeforeAll
    void init() { }  // Теперь не нужен static
}
```

**`@Nested` классы не могут быть static**

```java
@Nested
class ValidTests { }  // Правильно

@Nested
static class InvalidTests { }  // Ошибка
```

**`@TestFactory` возвращает типы**

```java
// Поддерживаемые возвращаемые типы:
DynamicTest              // один тест
DynamicContainer         // контейнер с тестами
Stream<DynamicTest>      // стрим тестов
Collection<DynamicTest>  // коллекция
Iterable<DynamicTest>    // итерируемый
Iterator<DynamicTest>    // итератор
```

**`@Disabled` с причиной**

```java
@Disabled("TODO: JIRA-1234 — реализовать после миграции")  // Всегда указывайте причину
@Test
void futureFeature() { }
```

---

## Итог

| Аннотация | Назначение | Ключевой момент |
| --------- | ---------- | --------------- |
| **`@BeforeAll`** | Инициализация 1 раз | Должен быть `static` |
| **`@AfterAll`** | Очистка 1 раз | Должен быть `static` |
| **`@BeforeEach`** | Подготовка перед каждым | Для изоляции тестов |
| **`@AfterEach`** | Очистка после каждого | Для изоляции тестов |
| **`@Test`** | Сам тест | Основной строительный блок |
| **`@DisplayName`** | Читаемое имя | Для отчётов |
| **`@Disabled`** | Пропуск | Всегда указывайте причину |
| **`@Nested`** | Группировка | Не может быть `static` |
| **`@Tag`** | Категории | Для выборочного запуска |
| **`@TestFactory`** | Динамические тесты | Возвращает `DynamicNode` |

**Совет:** используйте `@DisplayName` для всех тестов — это делает отчёты читаемыми. Группируйте связанные тесты через `@Nested`. Используйте `@Tag` для разделения быстрых и медленных тестов в CI/CD.

---

## Подключение в Maven

**Файл pom.xml**

```xml
<properties>
    <junit.jupiter.version>5.8.1</junit.jupiter.version>
    <junit.platform.version>1.8.1</junit.platform.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>${junit.jupiter.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>${junit.jupiter.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-params</artifactId>
        <version>${junit.jupiter.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-suite</artifactId>
        <version>${junit.platform.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## Аннотации JUnit 5

**@TestMethodOrder**

```java
@Order(5)
@Tag("model")
```

Аннотации задают порядок и категоризацию тестов, а также жизненный цикл тестирования.

```mermaid
---
title: JUnit Test Lifecycle
---
flowchart LR
    subgraph s1["Test run"]
        n1(["@BeforeEach<br>setup"])
        n2(["@Test<br>execution"])
        n3(["@AfterEach<br>cleanup"])
    end
    A(["@BeforeAll<br>class level setup"]) --> n1
    n1 --> n2
    n2 --> n3
    n3 -- repeat --> n1
    n3 --> n4(["@AfterAll<br>class level cleanup"])

    n1:::Aqua
    n2:::Aqua
    n3:::Aqua
    A:::Pine
    n4:::Pine
    classDef Pine stroke-width:1px, stroke-dasharray:none, stroke:#254336, fill:#27654A, color:#FFFFFF
    classDef Aqua stroke-width:1px, stroke-dasharray:none, stroke:#46EDC8, fill:#DEFFF8, color:#378E7A
```

---

## Assertions (утверждения)

Assertions (утверждения) позволяют сравнить ожидаемый результат с фактическим результатом теста.

Все утверждения JUnit Jupiter — `static` методы класса `org.junit.jupiter.Assertions`, например `assertEquals()`, `assertNotEquals()`.

```java
void testCase() {
    // Test will pass
    Assertions.assertNotEquals(3, Calculator.add(2, 2));
    // Test will fail
    Assertions.assertNotEquals(4, Calculator.add(2, 2), "Calculator.add(2, 2) test failed");
    // Test will fail
    Supplier<String> messageSupplier = () -> "Calculator.add(2, 2) test failed";
    Assertions.assertNotEquals(4, Calculator.add(2, 2), messageSupplier);
}
```

**Виды ассертов**

- `assertEquals()` и `assertNotEquals()`
- `assertArrayEquals()`
- `assertIterableEquals()`
- `assertLinesMatch()`
- `assertNotNull()` и `assertNull()`
- `assertNotSame()` и `assertSame()`
- `assertTimeout()` и `assertTimeoutPreemptively()`
- `assertTrue()` и `assertFalse()`
- `assertThrows()`
- `fail()`

---

## Assumptions (предположения)

Класс `Assumptions` (предположения) предоставляет `static` методы для поддержки условного выполнения теста на основе предположений. Неуспешное предположение приводит к прерыванию теста.

Предположения обычно используются, когда нет смысла продолжать выполнение данного метода тестирования. В отчёте о тестировании такие тесты отмечаются как пройденные.

Класс `Assumptions` имеет три метода: `assumeFalse()`, `assumeTrue()` и `assumingThat()`.

```java
public class AppTest {
    @Test
    void testOnDev() {
        System.setProperty("ENV", "DEV");
        Assumptions.assumeTrue("DEV".equals(System.getProperty("ENV")), AppTest::message);
    }

    @Test
    void testOnProd() {
        System.setProperty("ENV", "PROD");
        Assumptions.assumeFalse("DEV".equals(System.getProperty("ENV")));
    }

    private static String message() {
        return "TEST Execution Failed :: ";
    }
}
```

---
