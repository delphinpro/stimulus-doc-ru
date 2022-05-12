# Действия (Actions)

-----
* [Контроллеры](controllers.md)
* [Хуки жизненного цикла](lifecycle.md)
* Действия (Actions)
* [Объекты (Targets)](targets.md)
* [Переменные контроллера (Values)](values.md)
* [CSS классы](css_classes.md)
-----

Действия – это то, как вы обрабатываете события DOM в своих контроллерах.

```html
<div data-controller="gallery">
    <button data-action="click->gallery#next">…</button>
</div>
```

```javascript
// controllers/gallery_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    next(event) {
        // …
    }
}
```

Действие – это связь между:

* методом контроллера
* элементом контроллера
* слушателями событий DOM

## Дескрипторы

В атрибуте `data-action` значение `click->gallery#next` называется дескриптором действия. В этом дескрипторе:

* `click` – это имя прослушиваемого события DOM
* `gallery` – идентификатор контроллера
* `next` – имя метода контроллера, которое будет вызвано

### Короткое обозначение событий

Stimulus позволяет сокращать дескрипторы действий для некоторых распространенных пар элемент/событие, таких как пара button/click, опустив имя события:

```html
<button data-action="gallery#next">…</button>
```

Полный набор этих сокращенных пар выглядит следующим образом:

| Элемент             | Имя события по умолчанию |
|---------------------|--------------------------|
| `a`                 | `click`                  |
| `button`            | `click`                  |
| `details`           | `toggle`                 |
| `form`              | `submit`                 |
| `input`             | `input`                  |
| `input type=submit` | `click`                  |
| `select`            | `change`                 |
| `textarea`          | `input`                  |

### Глобальные события

Иногда контроллеру необходимо прослушивать события, вызываемые на глобальных объектах `window` или `document`.

Вы можете добавить `@window` или `@document` к имени события в дескрипторе действия, чтобы установить слушателя события на `window` или `document`, соответственно, как в следующем примере:

```html
<div data-controller="gallery"
     data-action="resize@window->gallery#layout">
</div>
```

### Опции

Вы можете добавить одну или несколько опций к дескриптору действия, если вам нужно указать [опции слушателя событий DOM](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Parameters).

```html
<div
    data-controller="gallery"
    data-action="scroll->gallery#layout:!passive"
>
    <img data-action="click->gallery#open:capture">
```

Stimulus поддерживает следующие опции:

| Опция действия | Опция слушателя события DOM |
|----------------|-----------------------------|
| `:capture`     | `{ capture: true }`         |
| `:once`        | `{ once: true }`            |
| `:passive`     | `{ passive: true }`         |
| `:!passive`    | `{ passive: false }`        |

## Event Objects

Метод действия – это метод в контроллере, который служит слушателем событий действия.

Первым аргументом метода действия является объект события DOM. Вы можете захотеть получить доступ к событию по ряду причин, включая:

* для считывания кода клавиши из события клавиатуры
* для чтения координат события мыши
* считывание данных из события input
* для чтения параметров из элемента отправителя действия
* чтобы предотвратить поведение браузера по умолчанию для события
* чтобы узнать, какой элемент отправил событие до того, как оно перешло к этому действию

Следующие основные свойства являются общими для всех событий:

| Свойство события      | Значение                                                                                                                  |
|-----------------------|---------------------------------------------------------------------------------------------------------------------------|
| `event.type`          | Имя события (например, 'click')                                                                                           |
| `event.target`        | Элемент, который отправил событие (т.е. самый внутренний элемент, на который был сделан щелчок)                           |
| `event.currentTarget` | Элемент, на который установлен слушатель событий (либо элемент с атрибутом `data-action`, либо `document`, либо `window`) |
| `event.params`        | The action params passed by the action submitter element                                                                  |

Следующие методы событий дают вам больше контроля над тем, как обрабатываются события:

| Метод события           | Результат                                                                                                 |
|-------------------------|-----------------------------------------------------------------------------------------------------------|
| event.preventDefault()  | Отменяет поведение события по умолчанию (например, переход по ссылке или отправка формы).                 |
| event.stopPropagation() | Останавливает событие до того, как оно распространится среди других слушателей на родительских элементах. |

## Множественные действия

Значение атрибута `data-action` представляет собой разделенный пробелами список дескрипторов действий.

Иногда элемент имеет несколько действий. Например, следующий элемент ввода вызывает метод `highlight()` контроллера `field`, когда он получает фокус, и метод `update()` контроллера `search` при каждом изменении значения элемента:

```html
<input type="text" data-action="focus->field#highlight input->search#update">
```

Когда элемент имеет более одного действия для одного и того же события, Stimulus вызывает действия слева направо в порядке появления их дескрипторов.

Цепочка действий может быть остановлена в любой момент вызовом `Event#stopImmediatePropagation()` внутри действия. Любые дополнительные действия справа будут проигнорированы:

```javascript
highlight: function(event) {
    event.stopImmediatePropagation();
    // ...
}
```

## Соглашение об именовании

Всегда используйте camelCase для указания имен действий, поскольку они отображаются непосредственно на методы вашего контроллера.

Избегайте имен действий, которые просто повторяют имя события, например `click`, `onClick` или `handleClick`:

```html
<button data-action="click->profile#click">Don't</button>
```

Вместо этого называйте свои методы действий в зависимости от того, что произойдет при их вызове:

```html
<button data-action="click->profile#showDialog">Do</button>
```

Это поможет вам рассуждать о поведении блока HTML без необходимости заглядывать в исходный текст контроллера.

## Параметры действий

Действия могут иметь параметры, которые считываются из атрибутов данных элемента. Они имеют формат `data-[identifier]-[param-name]-param`. Параметры должны быть указаны в том же элементе, где объявлено действие которому они должны быть переданы.

Все параметры автоматически приводятся к типу `Number`, `String`, `Object` или `Boolean`, определяемому по их значению:

| Data attribute                                  | Param                | Type    |
|-------------------------------------------------|----------------------|---------|
| `data-item-id-param="12345"`                    | `123`                | Number  |
| `data-item-url-param="/votes"`                  | `"/votes"`           | String  |
| `data-item-payload-param='{"value":"1234567"}'` | `{ value: 1234567 }` | Object  |
| `data-item-active-param="true"`                 | `true`               | Boolean |

Рассмотрим такую схему:

```html
<div data-controller="item spinner">
    <button
        data-action="item#upvote spinner#start" 
        data-item-id-param="12345" 
        data-item-url-param="/votes"
        data-item-payload-param='{"value":"1234567"}' 
        data-item-active-param="true"
    >…</button>
</div>
```

Stimulus вызовет и `SpinnerController#start`, и `ItemController#upvote`, но только последнему будут переданы какие-либо параметры:

```javascript
// ItemController
upvote(event) {
    // { id: 12345, url: "/votes", active: true, payload: { value: 1234567 } }
    console.log(event.params);
}

// SpinnerController
start(event) {
    // {}
    console.log(event.params);
}
```

Если нам больше ничего не нужно от события, мы можем деструктурировать параметры:

```javascript
upvote({ params }) {
    // { id: 12345, url: "/votes", active: true, payload: { value: 1234567 } }
    console.log(params);
}
```

Или деструктурировать только те параметры, которые нам нужны, в случае, если несколько действий на одном контроллере используют один и тот же элемент:

```javascript
upvote({ params: { id, url } }) {
    console.log(id); // 12345
    console.log(url); // "/votes"
}
```

-----
* [Контроллеры](controllers.md)
* [Хуки жизненного цикла](lifecycle.md)
* Действия (Actions)
* [Объекты (Targets)](targets.md)
* [Переменные контроллера (Values)](values.md)
* [CSS классы](css_classes.md)
-----
