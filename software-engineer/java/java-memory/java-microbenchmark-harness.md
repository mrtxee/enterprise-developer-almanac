---
aliases:
  - Benchmark
  - Java Microbenchmark Harness
  - JIT
  - JMH
  - Microbenchmark
  - бенчмарк
  - микробенчмарк
  - прогрев
---

## Общие сведения о JMH

**JMH** — это специализированный инструмент от компании Oracle (сейчас поддерживается OpenJDK) для написания, запуска и анализа **микробенчмарков** (точных замеров производительности небольших участков кода на Java).

### Назначение JMH

Обычные замеры времени через `System.currentTimeMillis()` или `System.nanoTime()` ненадежны из-за:

- оптимизаций JVM (dead code elimination, JIT-компиляция);
- влияния фоновых процессов ОС;
- неучтённых факторов (прогрев JVM, кэширование).

JMH решает эти проблемы, предоставляя **точные и воспроизводимые результаты**.

---

## Ключевые особенности JMH

1. **Контроль JVM-оптимизаций:** отключает нежелательные оптимизации (например, удаление "мёртвого" кода).
2. **Прогрев (warmup):** запускает код несколько раз перед замерами, чтобы JIT-компилятор оптимизировал его.
3. **Статистическая обработка:** вычисляет среднее время, погрешность, пропускную способность (ops/ms).
4. **Поддержка многопоточности:** тестирование в условиях конкуренции.
5. **Гибкость:** настройка режимов (например, `Mode.Throughput` или `Mode.AverageTime`).

---

## Пример простого бенчмарка

### Добавление зависимости JMH

Для Maven в `pom.xml`:

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.37</version>
    <scope>provided</scope>
</dependency>
```

### Код бенчмарка

Сравним скорость работы `String.concat` и `StringBuilder`:

```java
import org.openjdk.jmh.annotations.*;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime) // Замер среднего времени выполнения
@OutputTimeUnit(TimeUnit.NANOSECONDS) // Результат в наносекундах
@Warmup(iterations = 3, time = 1, timeUnit = TimeUnit.SECONDS) // 3 итерации прогрева
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS) // 5 итераций замера
@Fork(1) // Количество запусков JVM
public class StringBenchmark {

    @Benchmark
    public String testStringConcat() {
        String result = "";
        for (int i = 0; i < 100; i++) {
            result += i; // Пенальти: создает новый объект String каждый раз!
        }
        return result;
    }

    @Benchmark
    public String testStringBuilder() {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 100; i++) {
            sb.append(i); // Оптимизированная версия
        }
        return sb.toString();
    }
}
```

### Запуск бенчмарка

Через main-метод:

```java
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

public class Main {
    public static void main(String[] args) throws Exception {
        Options opt = new OptionsBuilder()
                .include(StringBenchmark.class.getSimpleName())
                .build();
        new Runner(opt).run();
    }
}
```

### Пример вывода

```text
Benchmark                       Mode  Cnt      Score      Error  Units
StringBenchmark.testStringConcat  avgt    5  14523.123 ± 456.789  ns/op
StringBenchmark.testStringBuilder avgt    5    234.567 ±  12.345  ns/op
```

**Вывод:** `StringBuilder` в ~60 раз быстрее для этой задачи.

---

## Применение JMH

1. **Оптимизация критического кода:** алгоритмы, библиотеки, высоконагруженные сервисы.
2. **Сравнение реализаций:** например, `ArrayList` vs `LinkedList`, разные методы хеширования.
3. **Тестирование влияния JVM-параметров:** размер heap, выбор сборщика мусора (GC).

---

## Советы по использованию

- **Избегайте "мёртвого кода":** результат бенчмарка должен использоваться (например, возвращаться из метода).
- **Настройте прогрев (warm-up):** достаточное количество итераций для стабилизации результатов.
- **Проверяйте дисперсию:** большая погрешность (`Error`) указывает на нестабильность теста.

---

## Альтернативы JMH

- **JUnit + ручные замеры:** подходит для простых случаев, но менее точный.
- **Apache Benchmark (ab):** для HTTP-запросов.

---

**Итог**

JMH — это "золотой стандарт" для микробенчмарков в Java. Используйте его для надежных замеров производительности.
