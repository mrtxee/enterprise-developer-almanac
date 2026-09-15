---
aliases:
  - char[]
  - immutable
  - java-passwords
  - Java Cryptography Architecture
  - JCA
  - Password storage
  - String
  - terminal null
  - Терминальный ноль
  - Хранение пароля
  - Хранение паролей в Java
  - Чувствительные данные
---

## Хранение паролей в Java

### Чувствительные данные в `char[]`

В Java пароль (и другие чувствительные данные) принято хранить в `char[]`, а не в `String`.

`String` в Java — неизменяемый (immutable) тип, он остаётся в памяти до сборки мусора, а `char[]` можно очистить вручную сразу после использования. Это снижает риск утечки пароля через дамп памяти или отладку.

**Пример**

```java
String password = "secret123"; // в куче создаётся объект, который нельзя изменить
password = ""; // ← создаётся НОВЫЙ объект, старый остаётся в памяти!
```

Старый объект с паролем остаётся в памяти, пока его не соберёт GC, а это может занять неопределённое время.

**Почему `char[]` безопаснее**

```java
char[] password = "secret123".toCharArray();
try {
  // валидация, хеширование, проверка...
  if (isValidPassword(username, password)) {
    grantAccess();
  }
} finally {
  // ВАЖНО: очищаем даже при исключении!
  Arrays.fill(password, '\0');
}
```

> [!tip] Терминальный ноль — `\0`
> Терминальный ноль (terminal null) `\0` — символ конца строки в стиле C.

**Не абсолютное решение**

`char[]` — не абсолютное решение, но лучшее из возможных, так как минимизирует продолжительность жизни пароля в куче.

Если создать `char[] password = "secret123".toCharArray();`, то строка `"secret123"` останется в heap (куче) как объект `String` до тех пор, пока её не соберёт сборщик мусора (GC).

Единственный способ не столкнуться с этой проблемой — сразу вычитывать пароль в `char[]`.

### Официальная рекомендация

Oracle прямо рекомендует использовать `char[]` для паролей.

**Java Cryptography Architecture (JCA) Reference Guide**

> «It would seem to be a no-brainer to use `String` objects to store passwords, but `String` objects are immutable and there is no way to zero out the characters. `char` arrays can be zeroed out after use, so they are a better choice.»

**Java Secure Coding Guidelines**

> «Sensitive information, such as passwords, should be stored in `char` arrays rather than in `String` objects.»
