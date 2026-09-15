---
aliases:
  - async
  - await
  - Javascript
  - JavaScript
  - JS
  - promise
  - Promise
  - spread
  - spread operator
  - Спред оператор
  - Спред-оператор
  - Яваскрипт
---
## Спред-оператор

Спред-оператор `…` раскладывает итерируемые объекты на отдельные элементы.

Пример раскрытия массива через спред:

```javascript
'use strict';
let [firstName, lastName, ...rest] = "Юлий Цезарь Император Рима".split(" ");
alert(firstName); // Юлий
alert(lastName);  // Цезарь
alert(rest);      // Император,Рима (массив из 2х элементов)
```

## Промисы и async/await

Промисы (от англ. `promise` — «обещание»).

Ключевое слово `async` заворачивает результат функции в промис. Значения других типов оборачиваются в успешно завершившийся промис автоматически.

Пример функции, возвращающей промис:

```javascript
async function f() {
  return 1;
}
f().then(alert); // 1
// OR
async function f() {
  return Promise.resolve(1);
}
f().then(alert); // 1
```

Ключевое слово `await` заставит интерпретатор JavaScript ждать до тех пор, пока промис справа от `await` не выполнится. После чего оно вернёт его результат, и выполнение кода продолжится.

Пример ожидания результата промиса:

```javascript
async function f() {
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("готово!"), 1000)
  });
  let result = await promise; // будет ждать, пока промис не выполнится (*)
  alert(result); // "готово!"
}
f();
```
