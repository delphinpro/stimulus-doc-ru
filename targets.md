# Объекты (Targets)

_Объекты_ позволяют обращаться к важным элементам по имени.

```html
<div data-controller="search">
    <input type="text" data-search-target="query">
    <div data-search-target="errorMessage"></div>
    <div data-search-target="results"></div>
</div>
```

## Атрибуты

Атрибут `data-search-target` называется целевым атрибутом, и его значение представляет собой разделенный пробелами список имен объектов, которые вы можете использовать для ссылки на элемент в контроллере `search`.

```html
<div data-controller="search">
    <div data-search-target="results"></div>
</div>
```

## Определение

Определите имена объектов в классе контроллера, используя массив `static targets`:

```javascript
// controllers/search_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = [ 'query', 'errorMessage', 'results' ]
    // …
}
```

## Свойства

Для каждого имени объекта, определенного в массиве `static targets`, Stimulus добавляет следующие свойства в ваш контроллер, где `[name]` соответствует имени объекта:

| Тип           | Имя                    | Значение                                                                               |
|---------------|------------------------|----------------------------------------------------------------------------------------|
| Единичное     | `this.[name]Target`    | Первый совпадающий объект в области видимости                                          |
| Множественное | `this.[name]Targets`   | Массив всех совпадающих объектов в области видимости                                   |
| Наличие       | `this.has[Name]Target` | Булево значение, указывающее, существует ли в области видимости соответствующий объект |

**Примечание:** Доступ к единичному свойству объекта приведет к ошибке, если нет подходящего элемента.

## Общие объекты

Элементы могут иметь более одного атрибута объекта, и часто объекты могут быть общими для нескольких контроллеров.

```html
<form data-controller="search checkbox">
    <input type="checkbox" data-search-target="projects" data-checkbox-target="input">
    <input type="checkbox" data-search-target="messages" data-checkbox-target="input">
    …
</form>
```

В приведенном выше примере флажки доступны внутри контроллера `search` как `this.projectsTarget` и `this.messagesTarget`, соответственно.

Внутри контроллера `checkbox`, `this.inputTargets` возвращает массив с обоими флажками.

## Optional Targets

Если ваш контроллер должен работать с объектом, который может присутствовать или не присутствовать, пишите ваш код, основываясь на значении свойства наличия объекта:

```javascript
if (this.hasResultsTarget) {
    this.resultsTarget.innerHTML = "…"
}
```

## Хуки добавления и удаления объектов

Хуки объектов позволяют вам реагировать всякий раз, когда объект добавляется или удаляется в элементе контроллера.

Определите метод `[name]TargetConnected` или `[name]TargetDisconnected` в контроллере, где `[name]` - имя объекта, за добавлениями или удалениями которого вы хотите наблюдать. Метод получает элемент в качестве первого аргумента.

Stimulus вызывает каждый хук объекта каждый раз, когда его элементы добавляются или удаляются после `connect()` и до `disconnect()` хуков жизненного цикла.

```javascript
export default class extends Controller {
    static targets = [ 'item' ];

    itemTargetConnected(element) {
        this.sortElements(this.itemTargets);
    }

    itemTargetDisconnected(element) {
        this.sortElements(this.itemTargets);
    }

    // Private
    sortElements(itemTargets) { /* ... */ }
}
```

**Примечание.** Во время выполнения хуков `[name]TargetConnected` и `[name]TargetDisconnected` экземпляры `MutationObserver` за кулисами приостанавливаются. Это означает, что если хук добавляет или удаляет объект с совпадающим именем, соответствующий хук не будет вызван снова.

## Соглашение об именовании

Всегда используйте camelCase для указания имен объектов, поскольку они отображаются непосредственно на свойства вашего контроллера.
