---
aliases:
  - Abstract Syntax Tree
  - AST
---
## ✅ Что такое AST?

> **Abstract Syntax Tree (AST)** — это **дерево, представляющее структуру исходного кода**, которое компилятор или IDE используют для:
- Анализа
- Проверки ошибок
- Рефакторинга
- Генерации байт-кода

---

## ✅ Пример: Код на Java

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}
```

**🔧 Как выглядит AST?**
🌳 Дерево:

```text
CompilationUnit
├── TypeDeclaration [Calculator]
│   ├── Modifier: public
│   ├── ClassOrInterfaceDeclaration [Calculator]
│   │   ├── MethodDeclaration [add]
│   │   │   ├── Modifier: public
│   │   │   ├── Type: int
│   │   │   ├── Name: add
│   │   │   ├── Parameters
│   │   │   │   ├── Parameter [a: int]
│   │   │   │   └── Parameter [b: int]
│   │   │   └── BlockStatement
│   │   │       └── ReturnStatement
│   │   │           └── InfixExpression (+)
│   │   │               ├── VariableAccess: a
│   │   │               └── VariableAccess: b
```

 ✅ Визуализация в Mermaid

```mermaid
flowchart LR
      A["CompilationUnit"] --> B["TypeDeclaration<br> Calculator"]
      B --> C["ClassOrInterfaceDecl<br> Calculator"]
      C --> D["MethodDeclaration<br> add"]
      D --> E["Modifier: public"] & F["Return Type: int"] & G["Parameters"] & J["Block"]
      G --> H["int a"] & I["int b"]
      J --> K["Return Statement"]
      K --> L["Infix Expression: a + b"]
      L --> M["Variable: a"] & N["Variable: b"]


      B:::BCDJ
      C:::BCDJ
      D:::BCDJ
      J:::BCDJ
      L:::LMN
      M:::LMN
      N:::LMN

      style A fill:#e9ecef,stroke:#6c757d
      classDef BCDJ fill:#d4edda,stroke:#155724
      classDef LMN fill:#fff3cd,stroke:#856404
```

---

## 💡 Как строится AST?

1. **Лексический анализ** → разбивка на токены:
   - `public`, `class`, `Calculator`, `{`, `return`, `+`, `;`
2. **Синтаксический анализ** → построение дерева
3. **Анализ семантики** → проверка типов, ссылок
4. **Генерация байт-кода**

---

## 📦 Где используется AST?

| Инструмент | Использование |
|-----------|----------------|
| **IDEA / Eclipse** | Подсветка, автодополнение |
| **Checkstyle / PMD** | Поиск плохого кода |
| **Lombok** | Модификация AST при компиляции |
| **Spring Annotation Processing** | Генерация классов |
| **ByteBuddy / ASM** | Манипуляция байт-кодом |

---

## 🔍 Пример: AST при обработке аннотаций

```java
@Data  // Lombok
public class User {
    private String name;
    private int age;
}
```

→ [[lombok]] читает AST → добавляет геттеры/сеттеры → модифицирует дерево → генерирует новый класс

---

## ✅ Финальный вывод

> ✅ **AST — это внутреннее представление вашего кода**, как его видит компилятор.

| Исходный код | AST |
|--------------|------|
| Человек | Машина |

> 🔁 Без AST не работают:
- Компиляторы
- Линтеры
- Рефакторинг
- Генерация кода

---