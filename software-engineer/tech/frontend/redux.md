---
aliases:
  - Action Creators
  - combineReducers
  - createStore
  - mapDispatchToProps
  - mapStateToProps
  - react-redux
  - reducer
  - reducers
  - Redux
  - Redux Library
  - Redux.js
  - Редакс
  - Редьюкс
---

## Redux

Redux — предсказуемый контейнер состояния для приложений на JavaScript. Это небольшая JavaScript-библиотека, которая может использоваться с любым фронтендом. Она использует паттерн **single source of truth**.

## Основные концепции

### Single source of truth

The global state of the app is stored in a single `store`. _Единственный источник информации о состояниях_ (паттерн).

### State is read-only

You can change the state only by dispatching `actions`. Action are objects that contain information about what should be changed.

### Pure reducers

`reducers` are functions that handle the actions and return the next state of the application. Reducers need to be pure, meaning they cannot modify the state, they need to return a new state object.

Пример store (состояние):

```javascript
{
  contacts: [{
    name: 'David'
  }, {
    name: 'Amy'
  }],
  toggle: true
}
```

Пример action (акция):

```javascript
{
  type: 'ADD_CONTACT', // обязательное поле
  name: 'James'
}
```

Пример reducer (редуктор):

```javascript
function contactsApp(state, action) {
  switch (action.type) {
    case 'ADD_CONTACT':
      return [ 'CONTACT_IS_ADDED_STATE', action.person ]
    default:
      return state
  }
}
```

## Actions (акции)

`type: 'ADD_CONTACT'` — обязательное поле. Согласно конвенции значение поля записывается в **UPPERCASE_SNAKE_CASE**.

`payload: {}` — если в акции хранится более одного поля данных, то все данные спускаются на уровень ниже в поле `payload`.

### Action Creators

Генераторы акций — простые функции, которые возвращают акции. Их стоит использовать для применения `DRY-паттерна (Don't Repeat Yourself)`.

```javascript
function addContact(person) {
  return {
    type: 'ADD_CONTACT',
    payload: person
  }
}
```

## Сочетание редукторов

Рекомендуется объединять редукторы в единый массив при помощи метода `combineReducers()`.

```javascript
const contactsApp = combineReducers({
  addContacts,
  doSomething
})
```

## Развёртывание в проекте

Установка пакетов:

```powershell
npm install redux
# если проект на React — также:
npm install react-redux
```

## React + Redux

Подключение хранилища к приложению:

```javascript
import { Provider } from 'react-redux';
import { createStore } from 'redux';
import { connect } from 'react-redux';
const el = <Provider store={store}>
  <Counter/>
</Provider>;
```

`connect()` — метод, который подключает react-компонент к redux-store.

```javascript
function connect(mapStateToProps?, mapDispatchToProps?)
/*
mapStateToProps simply returns the state variables as props to our component,
mapDispatchToProps allows to define how we dispatch actions and make the
dispatching functions available as props.
*/
function mapStateToProps(state) {
  return {
    count: state.count
  };
}
const mapDispatchToProps = {
  incrementCounter
}
function Counter(props) {
  function handleClick() {
    props.incrementCounter(1);
  }
  return <div>
    <p>{props.count}</p>
    <button onClick={handleClick}>Increment</button>
  </div>;
}
const Counter = connect(mapStateToProps, mapDispatchToProps)(Counter);
const el = <Provider store={store}>
  <Counter/>
</Provider>;
ReactDOM.render(...)
```
