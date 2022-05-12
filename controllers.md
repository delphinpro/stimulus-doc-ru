# Контроллеры

Контроллер – это основная организационная единица приложения Stimulus.

```javascript
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    // …
}
```

Контроллеры – это экземпляры JavaScript классов, которые вы определяете в своем приложении. Каждый класс контроллера наследуется от базового класса `Controller`, экспортируемого модулем `@hotwired/stimulus`.

## Свойства

Каждый контроллер принадлежит экземпляру Stimulus `Application` и связан с элементом HTML. Внутри класса контроллера вы можете получить доступ к его элементам:

* приложению, через свойство `this.application`
* HTML элемент, через свойство `this.element`

## Модули

Определите классы контроллеров в модулях JavaScript, по одному в каждом файле. Экспортируйте каждый класс контроллера как объект по умолчанию, как в примере выше.

Поместите эти модули в каталог `controllers/`. Назовите файлы `[identifier]_controller.js`, где `[identifier]` соответствует идентификатору каждого контроллера.

## Идентификаторы

Идентификатор – это имя, которое используется для ссылки на класс контроллера в HTML.

Когда вы добавляете атрибут `data-controller` к элементу, Stimulus считывает идентификатор из значения атрибута и создает новый экземпляр соответствующего класса контроллера.

Например, у этого элемента есть контроллер, который является экземпляром класса, определенного в `controllers/reference_controller.js`:

```html
<div data-controller="reference"></div>
```

Ниже приведен пример того, как Stimulus будет генерировать идентификаторы для контроллеров:

| Если файл контроллера имеет имя… | его идентификатор будет… |
|----------------------------------|--------------------------|
| `clipboard_controller.js`        | `clipboard`              |
| `date_picker_controller.js`      | `date-picker`            |
| `users/list_item_controller.js`  | `users--list-item`       |
| `local-time-controller.js`       | `local-time`             |

## Области видимости

Когда Stimulus подключает контроллер к элементу, этот элемент и все его дочерние элементы составляют область видимости контроллера.

Например, `<div>` и `<h1>` ниже являются частью области видимости контроллера, а окружающий элемент `<main>` - нет.

```html
<main>
  <div data-controller="reference">
    <h1>Reference</h1>
  </div>
</main>
```

## Вложенные области видимости

При вложении каждый контроллер знает только о своей области видимости, исключая область видимости всех контроллеров, вложенных в него.

Например, контроллер `#parent` ниже знает только об объектах `item`, находящихся непосредственно в его области видимости, но не об объектах контроллера `#child`.

```html
<ul id="parent" data-controller="list">
  <li data-list-target="item">One</li>
  <li data-list-target="item">Two</li>
  <li>
    <ul id="child" data-controller="list">
      <li data-list-target="item">I am</li>
      <li data-list-target="item">a nested list</li>
    </ul>
  </li>
</ul>
```

## Несколько контроллеров на элементе

Значение атрибута `data-controller` представляет собой список идентификаторов, разделенных пробелами:

```html
<div data-controller="clipboard list-item"></div>
```

Иногда элемент на странице может иметь несколько контроллеров. В приведенном выше примере `<div>` имеет два подключенных контроллера, `clipboard` и `list-item`.

Аналогичным образом, часто бывает, что несколько элементов на странице ссылаются на один и тот же класс контроллера:

```html
<ul>
  <li data-controller="list-item">One</li>
  <li data-controller="list-item">Two</li>
  <li data-controller="list-item">Three</li>
</ul>
```

Здесь каждый `<li>` имеет свой собственный экземпляр контроллера `list-item`.

## <a name="naming_convention"></a>Соглашение об именовании

Всегда используйте camelCase для имен методов и свойств в классе контроллера.

Если идентификатор состоит из нескольких слов, пишите слова в kebab-case (т.е. с использованием дефиса: `date-picker`, `list-item`).

В именах файлов разделяйте несколько слов подчеркиванием или тире (snake_case или kebab-case: `controllers/date_picker_controller.js`, `controllers/list-item-controller.js`).

## Регистрация

Если вы используете Stimulus for Rails с картой импорта или Webpack вместе с пакетом `@hotwired/stimulus-webpack-helpers`, ваше приложение будет автоматически загружать и регистрировать классы контроллеров, следуя приведенным выше соглашениям.

Если нет, ваше приложение должно вручную загрузить и зарегистрировать каждый класс контроллера.

## Ручная регистрация контроллеров

Чтобы вручную зарегистрировать класс контроллера с идентификатором, сначала импортируйте класс, а затем вызовите метод `Application#register` на объекте вашего приложения:

```javascript
import ReferenceController from './controllers/reference_controller';

application.register('reference', ReferenceController);
```

Вы также можете зарегистрировать анонимный класс контроллера вместо того, чтобы импортировать его из модуля:

```javascript
import { Controller } from '@hotwired/stimulus';

application.register('reference', class extends Controller {
    // …
});
```

## Предотвращение регистрации на основе факторов окружения

Если вы хотите, чтобы контроллер регистрировался и загружался только при соблюдении определенных факторов среды – например, заданного агента пользователя – вы можете переписать статический метод `shouldLoad`:

```javascript
class UnloadableController extends ApplicationController {
    static get shouldLoad() {
        return false;
    }
}

// This controller will not be loaded
application.register('unloadable', UnloadableController);
```

## Связь между контроллерами с помощью событий

Если вам нужно, чтобы контроллеры взаимодействовали друг с другом, следует использовать события. В классе `Controller` есть удобный метод `dispatch`, который облегчает эту задачу. В качестве первого аргумента он принимает _имя события_, к которому затем автоматически добавляется имя контроллера, разделенное двоеточием. Полезная нагрузка хранится в `detail`. Это работает следующим образом:

```javascript
class ClipboardController extends Controller {
    static targets = [ 'source' ];

    copy() {
        this.dispatch('copy', { detail: { content: this.sourceTarget.value } });
        this.sourceTarget.select();
        document.execCommand('copy');
    }
}
```

И это событие может быть направлено в действие на другом контроллере:

```html
<div data-controller="clipboard effects" data-action="clipboard:copy->effects#flash">
  PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
  <button data-action="clipboard#copy">Copy to Clipboard</button>
</div>
```

Поэтому, когда вызывается действие `Clipboard#copy`, вызывается и действие `Effects#flash`:

```javascript
class EffectsController extends Controller {
    flash({ detail: { content } }) {
        console.log(content); // 1234
    }
}
```

## Прямое обращение к другим контроллерам

Если по какой-то причине невозможно использовать события для связи между контроллерами, вы можете получить экземпляр контроллера через метод `getControllerForElementAndIdentifier` из приложения. Этот метод следует использовать только в том случае, если у вас есть уникальная проблема, которую невозможно решить с помощью более общего способа использования событий:

```javascript
class MyController extends Controller {
    static targets = [ 'other' ];

    copy() {
        const otherController = this.application.getControllerForElementAndIdentifier(this.otherTarget, 'other');
        otherController.otherMethod();
    }
}
```
