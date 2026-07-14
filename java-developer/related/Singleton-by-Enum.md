---
aliases:
  - Singleton
  - Singleton-by-Enum
  - Singleton при помощи Enum
  - Double-Checked Locking
---
## Паттерны: Как работает Singleton в многопоточной среде? Enum-способ.


**Проблема Singleton в многопоточной среде** — несколько потоков могут одновременно создать несколько экземпляров.

**Решения (от простого к сложному):**

| Способ | Механизм | Плюсы | Минусы |
|--------|----------|-------|--------|
| **`synchronized` метод** | Блокировка всего метода `getInstance()` | Просто | Низкая производительность |
| **Double-Checked Locking** | Две проверки + `volatile` | Быстрый, ленивый | Сложный, `volatile` обязателен |
| **Static inner class** | Инициализация при загрузке вложенного класса | Ленивый, потокобезопасный, без синхронизации | Нельзя передать параметры |
| **Enum (рекомендуемый)** | Одиночное значение enum | **Самый простой, сериализация, отражение** | Нет ленивой инициализации |

**Enum-способ:**

```java
public enum Singleton {
    INSTANCE;
    
    private final DatabaseConnection connection = new DatabaseConnection();
    
    public void doSomething() {
        connection.query();
    }
}

// Использование
Singleton.INSTANCE.doSomething();
```

**Почему Enum лучше:**
- Потокобезопасен из коробки (JVM гарантирует)
- Защищает от создания через рефлексию (enum нельзя создать рефлексивно)
- Защищает от десериализации (дополнительные экземпляры не создаются)
- Минимальный код

**Сравнение с Double-Checked Locking:**
```java
// Сложный вариант — не рекомендуется
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Итог:** **Enum** — лучший способ для Singleton в Java (Эффективная Java #📘 , Джошуа Блох #👨 ). Единственный минус — инициализация при загрузке класса (не ленивая).