---
aliases:
  - Canvas
  - Canvas transformations
  - Content Categories
  - Content Models
  - ctx.fillRect
  - ctx.rotate
  - ctx.translate
  - Declaration
  - DOCTYPE
  - Document Object Model
  - DOM
  - Embedded
  - Flow
  - Heading
  - hgroup
  - HTML
  - HTML 5
  - HTML5
  - Hypertext Markup Language
  - HyperText Markup Language
  - Interactive
  - Metadata
  - Phrasing
  - rotate
  - Scalable Vector Graphics
  - scale
  - Sectioning
  - SVG
  - translate
  - Встраиваемый контент
  - Заголовочный контент
  - Интерактивный контент
  - Канвас
  - Категории контента
  - Масштабируемая векторная графика
  - Метаданные
  - Модель содержимого
  - Объектная модель документа
  - Объявление типа документа
  - Потоковый контент
  - Секционный контент
  - Содержание документа
  - Фразовый контент
  - Язык гипертекстовой разметки
---

## Declaration

Объявление типа документа и кодировки:

```html
<!DOCTYPE HTML>
<meta charset="UTF-8">
```

![](1225bb30b883124d0d770f7a46af329d.jpeg)

## Content Models

Категории контента HTML5:

1. Metadata
   1. `<base>`, `<link>`, `<meta>`, `<noscript>`, `<script>`, `<style>`, `<title>`
2. Embedded
   1. `<audio>`, `<video>`, `<canvas>`, `<iframe>`, `<img>`, `<math>`, `<object>`, `<svg>`
3. Interactive
   1. `<a>`, `<audio>`, `<video>`, `<button>`, `<details>`, `<embed>`, `<iframe>`, `<img>`, `<input>`, `<label>`, `<object>`, `<select>`, `<textarea>`
4. Heading
   1. `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`, `<hgroup>`
5. Phrasing
   1. `<img>`, `<span>`, `<strong>`, `<label>`, `<br />`, `<small>`, `<sub>` и другие
6. Flow
   1. Содержит большинство элементов HTML5, которые входят в обычный поток документа
7. Sectioning
   1. `<article>`, `<aside>`, `<nav>`, `<section>`

## SVG vs Canvas

### Canvas

- Элементы рисуются программно.
- Рисование выполняется пикселями.
- Анимации не встроены.
- Высокая производительность для операций попиксельного рисования.
- Зависит от разрешения.
- Нет поддержки обработчиков событий.
- Полученное изображение можно сохранить как `.png` или `.jpg`.
- Хорошо подходит для графически насыщенных игр.

### SVG

- Элементы являются частью DOM страницы (Document Object Model).
- Рисование выполняется векторами.
- Такие эффекты, как анимации, встроены.
- Основан на стандартном синтаксисе XML, что обеспечивает лучшую доступность.
- Не зависит от разрешения.
- Поддерживаются обработчики событий.
- Не подходит для игровых приложений.
- Лучше всего подходит для приложений с большими областями отрисовки (например, Google Maps).

### Canvas transformations

Методы трансформации канваса:

```javascript
.translate(), .rotate(), scale()
```

Пример с `ctx.translate()`:

```html
<html>
  <head></head>
  <body>
    <canvas id="canvas1" width="400" height="300">
    </canvas>
    <script>
      var c=document.getElementById("canvas1");
      var ctx=c.getContext("2d");
      ctx.font="bold 22px Tahoma";
      ctx.textAlign="start";
      ctx.fillText("start", 10, 30);
      ctx.translate(100, 150);
      ctx.fillText("after translate", 10, 30);
    </script>
  </body>
</html>
```

Пример с `ctx.rotate()` и `ctx.fillRect()`:

```html
<html>
  <head></head>
  <body>
    <canvas id="canvas1" width="400" height="300">
    </canvas>
    <script>
      var c=document.getElementById("canvas1");
      var ctx=c.getContext("2d");
      ctx.fillStyle = "#FF0000";
      ctx.fillRect(10,10, 100, 100);
      ctx.rotate( (Math.PI / 180) * 25);  //rotate 25 degrees.
      ctx.fillStyle = "#0000FF";
      ctx.fillRect(10,10, 100, 100);
    </script>
  </body>
</html>
```
