# Переменные контроллера (Values)

-----
* [Контроллеры](controllers.md)
* [Хуки жизненного цикла](lifecycle.md)
* [Действия (Actions)](actions.md)
* [Объекты (Targets)](targets.md)
* Переменные контроллера (Values)
* [CSS классы](css_classes.md)
-----

Вы можете читать и записывать [атрибуты данных HTML](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/data-*) на элементах контроллера как типизированные переменные, используя специальные свойства контроллера.

```html
<div data-controller="loader"
     data-loader-url-value="/messages">
</div>
```

```javascript
// controllers/loader_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static values = {
        url: String;
    }
    
    connect() {
        fetch(this.urlValue).then(/* … */);
    }
}
```

## Определение

Определите переменные в контроллере с помощью объекта `static values`. Поместите имя каждой переменной слева, а её тип – справа.

```javascript
export default class extends Controller {
    static values = {
        url: String,
        interval: Number,
        params: Object,
    }
    
    // …
}
```

## Типы

Тип переменной – это один из стандартных типов javascript `Array`, `Boolean`, `Number`, `Object` или `String`. Тип определяет, как значение переменной перекодируется между JavaScript и HTML.

| Тип     | Кодируется как…          | Декодируется как…                  |
|---------|--------------------------|------------------------------------|
| Array   | `JSON.stringify(array)`  | `JSON.parse(value)`                |
| Boolean | `boolean.toString()`     | `value != "0" && value != 'false'` |
| Number  | `number.toString()`      | `Number(value)`                    |
| Object  | `JSON.stringify(object)` | `JSON.parse(value)`                |
| String  | –                        | –                                  |

## Свойства и атрибуты

Stimulus автоматически генерирует свойства getter, setter и existential для каждой переменной, определенной в контроллере. Эти свойства связаны с атрибутами данных в элементе контроллера:

| Тип         | Имя свойства          | Эффект                                                      |
|-------------|-----------------------|-------------------------------------------------------------|
| Getter      | `this.[name]Value`    | Читает атрибут `data-[identifier]-[name]-value`             |
| Setter      | `this.[name]Value=`   | Пишет в атрибут `data-[identifier]-[name]-value`            |
| Existential | `this.has[Name]Value` | Проверяет наличие атрибута `data-[identifier]-[name]-value` |

### Getters

Геттер для переменной декодирует связанный атрибут данных в экземпляр типа этой переменной.

Если атрибут данных отсутствует в элементе контроллера, геттер возвращает _значение по умолчанию_, в зависимости от типа переменной:

| Тип переменой | Значение по умолчанию |
|---------------|-----------------------|
| Array         | `[]`                  |
| Boolean       | `false`               |
| Number        | `0`                   |
| Object        | `{}`                  |
| String        | `''`                  |

### Setters

Сеттер переменной устанавливает связанный с ней атрибут данных на элементе контроллера.

Чтобы удалить атрибут данных из элемента контроллера, присвойте ему значение `undefined`.

### Existential

Existential-свойство для переменной оценивается как `true`, если связанный атрибут данных присутствует на элементе контроллера, и `false`, если он отсутствует.

## Хуки изменения значений переменных

Хуки изменения значения переменной позволяют вам реагировать на изменение атрибута данных.

Определите метод `[name]ValueChanged` в контроллере, где `[name]` - это имя переменной, за изменением которой вы хотите наблюдать. Метод получает декодированное значение в качестве первого аргумента и декодированное предыдущее значение в качестве второго аргумента.

Stimulus вызывает каждый хук изменения после инициализации контроллера и каждый раз, когда связанный с ним атрибут данных изменяется. Это включает в себя изменения в результате присвоения сеттеру переменной какого-либо значения.

```javascript
export default class extends Controller {
    static values = { url: String }
    
    urlValueChanged() {
        fetch(this.urlValue).then(/* … */);
    }
}
```

### Предыдущие значения

Вы можете получить доступ к предыдущему значению в хуке `[name]ValueChanged`, определив метод хука с двумя аргументами.

```javascript
export default class extends Controller {
    static values = { url: String }
    
    urlValueChanged(value, previousValue) {
        /* … */
    }
}
```

Оба аргумента могут быть названы как угодно. Вы также можете использовать `urlValueChanged(current, old)`.

## Значения по умолчанию

Переменные, которые не были указаны на элементе контроллера, могут быть установлены значениями по умолчанию, указанными в определении контроллера:

```javascript
export default class extends Controller {
    static values = {
        url: { type: String, default: '/bill' },
        interval: { type: Number, default: 5 },
        clicked: Boolean,
    }
}
```

Когда требуется значение по умолчанию, используется расширенная форма описания `{ type, default }`. Эту форму можно смешивать с обычной формой, которой не требуется значение по умолчанию.

## Соглашение об именовании

Записывайте имена значений в camelCase в JavaScript и kebab-case в HTML. Например, значение с именем `contentType` в контроллере `loader` будет иметь связанный с ним атрибут данных `data-loader-content-type-value`.

-----
* [Контроллеры](controllers.md)
* [Хуки жизненного цикла](lifecycle.md)
* [Действия (Actions)](actions.md)
* [Объекты (Targets)](targets.md)
* Переменные контроллера (Values)
* [CSS классы](css_classes.md)
-----
