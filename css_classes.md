# CSS классы

-----
* [Контроллеры](controllers.md)
* [Хуки жизненного цикла](lifecycle.md)
* [Действия (Actions)](actions.md)
* [Объекты (Targets)](targets.md)
* [Переменные контроллера (Values)](values.md)
* CSS классы
-----

В HTML класс CSS определяет набор стилей, которые могут быть применены к элементам с помощью атрибута `class`.

CSS classes are a convenient tool for changing styles and playing animations programmatically. For example, a Stimulus controller might add a “loading” class to an element when it is performing an operation in the background, and then style that class in CSS to display a progress indicator:

Классы CSS – это удобный инструмент для изменения стилей и воспроизведения анимации программным путем. Например, контроллер Stimulus может добавить класс "loading" к элементу, когда он выполняет операцию в фоновом режиме, а затем стилизовать этот класс в CSS для отображения индикатора выполнения:

```html
<form data-controller="search" class="search--busy">
```

```css
.search--busy {
    background-image: url(throbber.svg) no-repeat;
}
```

В качестве альтернативы жесткому кодированию классов с помощью строк JavaScript, Stimulus позволяет ссылаться на классы CSS по логическому имени, используя комбинацию атрибутов данных и свойств контроллера.

## Определение классов

Определите классы CSS по логическому имени в вашем контроллере с помощью массива `static classes`:

```javascript
// controllers/search_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static classes = [ 'loading' ];
}
```

## Атрибуты 

Логические имена, определенные в массиве `static classes` контроллера, отображаются на атрибуты классов CSS элемента контроллера.

```html
<form
    data-controller="search"
    data-search-loading-class="search--busy"
>
    <input data-action="search#loadResults">
</form>
```

Создайте атрибут класса CSS, соединив идентификатор контроллера и логическое имя в формате `data-[идентификатор]-[логическое имя]-class`. Значением атрибута может быть одно имя класса CSS или список из нескольких имен классов.

**Примечание:** Атрибуты классов CSS должны быть указаны на том же элементе, что и атрибут `data-controller`.

Если вы определяете несколько классов для одного логического имени, то разделите их пробелами:

```html
<form
    data-controller="search"
    data-search-loading-class="bg-gray-500 animate-spinner cursor-busy"
>
    <input data-action="search#loadResults">
</form>
```

## Свойства

Для каждого логического имени, определенного в массиве `static classes`, Stimulus добавляет следующие свойства CSS класса в ваш контроллер:

| Тип           | Имя                          | Значение                                                                        |
|---------------|------------------------------|---------------------------------------------------------------------------------|
| Единичное     | `this.[logicalName]Class`    | Значение атрибута CSS class, соответствующего logicalName                       |
| Множественное | `this.[logicalName]Classes`  | Массив всех классов в соответствующем атрибуте CSS class, разделенных пробелами |
| Наличие       | `this.has[LogicalName]Class` | Булево значение, указывающее на наличие или отсутствие атрибута CSS класса      |

Используйте эти свойства для применения CSS-классов к элементу с помощью методов `add()` и `remove()` из [API DOM classList](https://developer.mozilla.org/en-US/docs/Web/API/Element/classList).

Например, чтобы отобразить индикатор загрузки на элементе контроллера `search` перед получением результатов, вы можете реализовать действие `loadResults` следующим образом:

```javascript
export default class extends Controller {
    static classes = [ 'loading' ];
    
    loadResults() {
        this.element.classList.add(this.loadingClass);
    
        fetch(/* … */);
    }
}
```

Если атрибут CSS класса содержит список имен классов, его единичное свойство CSS класса возвращает первый класс в списке.

Используйте множественное свойство CSS класса для доступа ко всем именам классов в виде массива. Комбинируйте это с [синтаксисом spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) для одновременного применения нескольких классов:

```javascript
export default class extends Controller {
    static classes = [ 'loading' ];
    
    loadResults() {
        this.element.classList.add(...this.loadingClasses);
    
        fetch(/* … */);
    }
}
```

**Примечание:** Stimulus выдаст ошибку, если вы попытаетесь получить доступ к свойству класса CSS, когда соответствующий атрибут класса CSS отсутствует.

## Соглашения об именовании

Используйте camelCase для указания логических имен в определениях классов CSS. Логические имена отображаются на свойства CSS-класса в camelCase:

```javascript
export default class extends Controller {
    static classes = [ 'loading', 'noResults' ];
    
    loadResults() {
        // …
        if (results.length == 0) {
            this.element.classList.add(this.noResultsClass);
        }
    }
}
```

В HTML пишите атрибуты классов CSS в kebab-case стиле:

```html
<form data-controller="search"
      data-search-loading-class="search--busy"
      data-search-no-results-class="search--empty">
```

При создании атрибутов классов CSS следуйте соглашениям для идентификаторов, описанным в разделе [Контроллеры: Соглашения об именовании](controllers.md#naming_convention).

-----
* [Контроллеры](controllers.md)
* [Хуки жизненного цикла](lifecycle.md)
* [Действия (Actions)](actions.md)
* [Объекты (Targets)](targets.md)
* [Переменные контроллера (Values)](values.md)
* CSS классы
-----
