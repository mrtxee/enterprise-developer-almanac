---
aliases:
  - BigDecimal
  - Currency
  - moneta
  - monetary-type-java
  - monetary-type
---
## Как правильно хранить деньги в Java

**Лучший вариант:** использовать **`BigDecimal`** для всех операций с деньгами.

---

### 📦 Основные варианты

| Тип                        | Применимость                  | Риски                                          |
| -------------------------- | ----------------------------- | ---------------------------------------------- |
| **`BigDecimal`**           | ✅ **Лучший выбор**            | Нет (при использовании `String` конструктора)  |
| `long` (в копейках/центах) | ✅ Приемлемо для простых валют | Ограничение точности (нет половинчатых копеек) |
| `double` / `float`         | ❌ **НЕЛЬЗЯ**                  | Ошибки округления (0.1 + 0.2 != 0.3)           |
| `int` (в копейках)         | ❌ Слишком малый диапазон      | Переполнение для больших сумм                  |

---

### 🏆 Правильный подход: `BigDecimal`

```java
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.Currency;

public class Money {
    private final BigDecimal amount;
    private final Currency currency;
    
    public Money(String amount, Currency currency) {
        // ВСЕГДА используйте конструктор из String, а не double!
        this.amount = new BigDecimal(amount);
        this.currency = currency;
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Currency mismatch");
        }
        return new Money(
            this.amount.add(other.amount).toString(),
            this.currency
        );
    }
    
    public Money multiply(int multiplier) {
        return new Money(
            this.amount.multiply(BigDecimal.valueOf(multiplier)).toString(),
            this.currency
        );
    }
    
    public Money roundToCents() {
        return new Money(
            this.amount.setScale(2, RoundingMode.HALF_UP).toString(),
            this.currency
        );
    }
    
    // Геттеры
    public BigDecimal getAmount() { return amount; }
    public Currency getCurrency() { return currency; }
    
    @Override
    public String toString() {
        return currency.getSymbol() + " " + amount;
    }
}
```

---

### ❌ Почему нельзя использовать `double`?

```java
double a = 0.1;
double b = 0.2;
System.out.println(a + b); // 0.30000000000000004 ❌
```

**Причина:** числа с плавающей точкой хранятся в двоичном формате, и не все десятичные дроби могут быть точно представлены.

---

### 🔧 Альтернатива: `long` в копейках

```java
public class MoneyInCents {
    private final long amountInCents; // 100 = 1.00 руб
    private final Currency currency;
    
    public MoneyInCents(long amountInCents, Currency currency) {
        this.amountInCents = amountInCents;
        this.currency = currency;
    }
    
    public MoneyInCents add(MoneyInCents other) {
        return new MoneyInCents(
            this.amountInCents + other.amountInCents,
            this.currency
        );
    }
}
```

**Плюсы:** быстродействие, компактность.  
**Минусы:** нет половинчатых копеек (например, при делении на 3).

---

### 📦 Готовая библиотека: JSR 354 / [[java-monetary-type-jsr354|Moneta]]
pom.xml
```xml
<dependency>
    <groupId>org.javamoney</groupId>
    <artifactId>moneta</artifactId>
    <version>1.4.4</version>
</dependency>
```
Main.java
```java
import javax.money.Monetary;
import javax.money.MonetaryAmount;
import org.javamoney.moneta.Money;

MonetaryAmount amount = Money.of(100.50, "USD");
MonetaryAmount result = amount.multiply(2);
System.out.println(result); // USD 201.00
```

---

### 🎯 Рекомендации по хранению в БД

| БД | Тип колонки |
|----|-------------|
| PostgreSQL | `NUMERIC(19,4)` |
| MySQL | `DECIMAL(19,4)` |
| Oracle | `NUMBER(19,4)` |
| H2 | `DECIMAL` |

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    total_amount DECIMAL(19,4) NOT NULL,
    currency_code CHAR(3) NOT NULL  -- "USD", "EUR", "RUB"
);
```

---

### 📌 Итог

| Аспект               | Рекомендация                                          |
| -------------------- | ----------------------------------------------------- |
| **Тип в Java**       | `BigDecimal` (с `String` конструктором)               |
| **Операции**         | `add()`, `subtract()`, `multiply()`                   |
| **Округление**       | `setScale(2, RoundingMode.HALF_UP)`                   |
| **Валюта**           | `java.util.Currency` или `javax.money.MonetaryAmount` |
| **Хранение в БД**    | `DECIMAL(19,4)` или `NUMERIC(19,4)`                   |
| **Передача по сети** | Строка (например, `"123.45"`) или `long` в копейках   |
## Почему BigDecimal лучше для хранения денег?

### 🎯 Главная причина: Точность

`BigDecimal` хранит числа в десятичном виде (масштабируемое целое + масштаб), а не в двоичном, как `double` / `float`. Это гарантирует **математически точные вычисления** с десятичными дробями.

```java
// ❌ double — ошибка округления
System.out.println(0.1 + 0.2); // 0.30000000000000004

// ✅ BigDecimal — точный результат
BigDecimal a = new BigDecimal("0.1");
BigDecimal b = new BigDecimal("0.2");
System.out.println(a.add(b)); // 0.3
```

---

### ⚖️ Контроль над округлением

`BigDecimal` позволяет явно задавать **правила округления** и **количество знаков**:

```java
BigDecimal price = new BigDecimal("19.999");
price = price.setScale(2, RoundingMode.HALF_UP);
System.out.println(price); // 20.00
```

Это критично для финансовых расчётов, где даже малейшая погрешность недопустима.

---

### 📏 Произвольная точность

В отличие от `double` (максимум ~15 знаков), `BigDecimal` может хранить **любое количество цифр**. Это нужно для:
- Работы с большими суммами (миллиарды).
- Высокой точности (например, 10 знаков после запятой для курсов валют).

---

### 🧮 Операции с дробными копейками

`long` в копейках не поддерживает дробные доли. С `BigDecimal` можно оперировать долями копеек:

```java
BigDecimal price = new BigDecimal("0.005"); // полкопейки
BigDecimal total = price.multiply(BigDecimal.valueOf(100)); // 0.50
```

В `long` это невозможно без потери точности.

---

### 🎯 Итог: почему именно BigDecimal

| Критерий | double/float | long (копейки) | BigDecimal |
|----------|--------------|----------------|------------|
| Точность десятичных дробей | ❌ Нет | ❌ Нет | ✅ Да |
| Контроль округления | ❌ Нет | ✅ Да (только целые) | ✅ Да |
| Произвольная точность | ❌ Нет | ❌ Нет | ✅ Да |
| Дробные доли копеек | ✅ Да (с ошибками) | ❌ Нет | ✅ Да |
| Простота использования | ✅ Да | ✅ Да | ⚠️ Сложнее |
| Производительность | ✅ Высокая | ✅ Высокая | ⚠️ Медленнее |

**Вывод:** для финансовых расчётов, где важна **точность** и **предсказуемость** результатов, `BigDecimal` является единственным правильным выбором. `double` и `long` подходят только для очень простых случаев с полным контролем над округлением (например, хранение целых копеек без операций деления).