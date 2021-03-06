**Остановился на 6.4. Определение шаблона для компонента**

*Vue* называют прогрессивным фреймворком (*progressive framework*), так как его можно использовать:

- для выполнения простых задач на отдельных страницах, по аналогии с *Jquery*
- в качестве фундамента для полноценных промышленных приложений.

Использует [MVVM паттерн](Framework.md#mvvm). Для описания *View* используются *HTML* и *CSS*, для описания *View Model* – *Javascript*.

*Vue*-приложение содержит как минимум один экземпляр объекта `Vue`. Объект `Vue` связывает разметку в *View* с данными в *ViewModel*. *View*, объект `Vue`, *ViewModel* хранятся в браузере на клиенте и поэтому могут работать при полном отсутствии серверной части, или при проблемах на сервере. Браузер-клиент вообще не перезагружает страницу.

# Vue instance

Создание  *Vue instance*'а:

```javascript
var vm = new Vue({  // vm - View Model
  [ el: <string | Element> ]
  [ data: <Object | Function> ]
  [ methods: { [key: string]: Function } ] 
  [ filters: { [key: string]: Function } ]
  [ components: { [id: string]: [definition: Object] }]
  [ computed: { [key: string]: Function } ]
  [ watch: { [key: string]: Function } ]
  
})
```

Пример:

```html
<div id="app"></div>
<script>
var vm = new Vue({
  el: '#app',
  data: {
      sitename: "Shop",
      product: {
          id: 1,
          
      }
  }  
});  
</script>
```

- `el` – указывает в какой существующий DOM-элемент монтировать экземпляр `Vue`. Может быть как строковым CSS-селектором, так и объектом типа `HTMLElement`.

- `data` –  объект с данными  *Vue instance*'а. Данные из data, будут добавлены в  *reactivity system*  (систему реактивности) *Vue* и связаны (*bind*) с *View*. Поэтому *View* будет «реагировать» на их изменения, обновляясь в соответствии с новыми значениями.  

  Для каждого `property` из объекта `data` автоматически создаются *getter* и *setter*. Поэтому *property* доступны через *Vue instance*:

  ```javascript
  console.log(vm.<property>)
  vm.<property> = <value>            
  ```

- `filters` – [фильтры](#фильтры).

- `components` – регистрация [*local component*'s](#local-регистрация)

## *Method*'ы

- `methods` – методы, которые будут подмешаны (*mixed*) к *Vue instance*. 

*Method*'ы доступны через *Vue instance*:

```javascript
vm.<method>()
```

Внутри *method*'а переменная `this` указывает на *Vue instance*. 

Используются в качестве [*handler*'а *event*'ов](#события)

Существует распространенное заблуждение, что *method*'ы «вызываются только один раз» при отрисовке, если использованы в *expression* в *template*. На самом деле это не так. На самом деле при вызове *method*'а из *templat*'а, устанавливается реактивная зависимость на те поля из массива `data` (только `data`????), которые используются по всему стеку (!) вызовов *method*'а. Если эти поля из массива `data` изменяют свое значение, это вызывает re-rendering элемента.

## Вычисляемые свойства

- `computed` – определение вычисляемых свойств – динамических, рассчитываемых свойств. Позволяют логику вычислений убрать из *View* и перенести в *ViewModel*. 

Определение:

```javascript
  computed: {
    <property> () {
	  return ...;
    }
  }
```

## Method'ы и computed свойства

*Method*'ы и *Computed* свойства по сути очень похожи, т.к. доступны через *Vue instance* и определяются через `function`.

Различия:

- *method*'ы могут принимать параметры, *computed* свойства – нет?
- *computed* свойства кэшируются и пересчитываются заново, когда изменяется какая-либо  реактивная зависимость (например, счетчик вызовов внутри работать не будет). *Method* выполняется полностью при каждом вызове.

## Наблюдатели (watcher)

- `watch` – объект, ключи которого – выражения для наблюдения, а значения – *callback*'и, вызываемые при их изменении. 

  ```javascript
  watch: {
    <property>: function (newVal, oldVal) {
      /* ... */
    }
  }
  ```

  Можно передать опцию `immediate: true`, тогда обработчик будет вызван сразу же после начала наблюдения:
  
  ```javascript
  watch: {
    <property>: {
      immediate: true,
      handler(newVal, oldVal) { ... }
    }
  }
  ```
  
  Другой способ управлять вызовом обработчика – вынести обработчик в *method* и вызывать его из любого *hook*'а.

# Ограничения реактивности

Вследствие ограничений JavaScript, есть виды изменений, которые Vue **не может обнаружить**. Однако существуют способы обойти их, чтобы сохранить реактивность.

## Изменения `Object`

Vue не может обнаружить добавление (через `object.property` и `object["property"]`) или удаление *property* (через `delete object.property` и `delete object["property"]`). 

Невозможно никак динамически добавить новое реактивное *property* верхнего уровня в объект `data` (корневое реактивное *property* *Vue instance*'а).

Однако можно добавить или удалить реактивное *property*, которое вложено в *property* из объекта `data`.

Добавление *reactive property*:

```javascript
this.$set( this.<someObject>, <propertyName/index>, <value> )
this.$set( this.someObject, 'b', 2 )
```

Удаление *reactive property*:

```javascript
this.$delete( this.<someObject>, <propertyName/index> )
this.$delete( this.someObject, 'b' )
```





На одной странице можно запустить несколько *Vue instance*'ов, которые примонтированы к разным элементам DOM. Это имеет смысл использовать, если *Vue instance* примонтирован к небольшим компонентам – карусель или веб-форма.

`Vue` *instance*'ы доступен также через переменные `$vm<N>`.

# Жизненный цикл

Жизненный цикл *Vue instance*  состоит из нескольких этапов. 

![lifecycle](https://ru.vuejs.org/images/lifecycle.png)

Смотреть [Lifecycle hook](#Lifecycles-hook)

# Lifecycle hook

Жизненный цикл имеется у *Vue instance* и *component*'ов.

Между этапами жизненного цикла вызываются функции, называемые хуками жизненного цикла (*lifecycle hook*), с помощью которых можно выполнять свой код в определённый момент жизненного цикла.

Список *Lifecycle hook*'ов:

- `beforeCreate`

- `created`. 

- `beforeMount`

- `mounted`.  В хуке `mounted` вы получите полный доступ к реактивному компоненту, шаблонам и отрисованному DOM.

  DOM-элемент, к которому примонтирован экземпляр Vue, будет доступен через `this.$el`.

   `mounted` — самый популярный хук жизненного цикла. Обычно его используют для извлечения данных для компонента (вместо этого применяйте `created`) и изменения DOM, зачастую ради интегрирования не-Vue библиотек. 

  ```html
  ExampleComponent.vue
  <template>
    <p>I'm text inside the component.</p>
  </template>
  
  <script>
  export default {
    mounted() {
      console.log(this.$el.textContent) // I'm text inside the component.
    }
  }
  </script>
  ```

  

- `beforeUpdate`

- `updated`

- `beforeDestroy`

- `destroyed`

Большую часть времени программа проводит в цикле событий (`beforeUpdate` и `updated` *hook*'и). 

Все хуки жизненного цикла автоматически привязывают контекст `this` к *Vue instance*. 

 https://habr.com/ru/company/mailru/blog/350962/ 

# Template

Vue.js использует *template* синтаксис на основе HTML, который позволяет связывать (*bind*) элементы DOM с `data` данными *Vue instance*'а. 

## Интерполяции

Интерполяции (*Interpolations*) – способ отражения данных *Vue* на элементы DOM. 

Все *interpolation*'s принимают в качестве параметра *JavaScript expression*. 

Особенности *Javascript expression* в *interpolation*:

- *Property*'s из `data` текущего *Vue instance*'а в *expression* доступны без `this`. 
- В *expression* можно использовать ограниченный набор стандартных глобальных объектов (вроде,  `Math` и `Date`). Невозможно напрямую обращаться к пользовательским глобальным объектам. Нужно использовать промежуточный доступ через *computed* свойства.

Например:

```html
{{ sitename }}
{{ product.id }}
```

Сложные *JavaScript expression* не рекомендуются, т.к. переносят часть бизнес-логики в *View*:

```html
{{ product.title.substr(1,4) }}
```



### Текст

Вставляет текстовое значение параметра напрямую в HTML-код, при этом выполняет экранирование специальных символов? *Expression* записывается в следующем виде (т.н. синтаксис *Mustache*):

```html
{{ expression }}
```



### Директивы

Директивы (*directive*) – это атрибуты, которые *bind*'ят динамическое *Javascript expression* с некоторым состоянием элемента DOM. Начинаются с префикса `v-`. 

#### `v-text`

*Bind*'ит текстовое содержимое элемента (`textContent`). Эквивалентно, можно использовать текстовую *interpolation*. Текстовую *interpolation* нужно использовать, если требуется изменять только часть текста.  

```html
<tag v-text="<expression>"></tag>
<tag>{{ expression }}</tag>      <!-- Аналогично -->
<tag>Text {{ expression }}</tag> <!-- Изменяется часть текста -->    
```

#### `v-bind`

*Bind*'ит атрибуты тега.  Т.к. текстовая *interpolation* не работает с  HTML-атрибутами. 

```html
<tag v-bind:<attr>="<expression>">
<img v-bind:src="imageSrc">
```

Поддерживается сокращенная запись

```html
<tag :<attr>="<expression>">
<img :src="imageSrc">
```

#### `v-html`

*Bind*'ит HTML-содержимое элемента (`innerHTML`), экранирование специальных символов не выполняется.  

```html
<tag v-html="<expression>"></tag>
<div v-html="html"></div>
```

#### `v-show`

Переключает CSS-свойство `display` элемента, в зависимости от того, истинно ли указанное выражение.

```html
<tag v-show="<expression>"></tag>
<div v-show="isVisible"></div>
```

#### `v-if`, `v-else-if`, `v-else`

*Directive*'s используются для вставки *tag*'а (или блока) в HTML по условию. В DOM присутствуют только те *tag*'и, которые соответствуют условию. Хорошо подходят, когда есть альтернативные блоки и один из них всегда должен быть виден.

`v-if` – *tag* отображается при значении `true` условия

```html
<tag v-if="<expression>"></tag>
<div v-if="isVisible"></div>
```

`v-else-if` – аналог `v-if`, должен следовать сразу после `v-if` или `v-else-if`

```html
<tag v-if="<expression"></tag>
<tag v-else-if="<expression>"></tag>
```

`v-else` – без аргументов, должен следовать сразу после `v-if` или `v-else-if`

```html
<tag v-if="<expression"></tag>
<tag v-else-if="<expression>"></tag>
<tag v-else></tag>
```

*Tag*'и с атрибутами `v-if`, `v-else-if`, `v-else` должны идти строго друг за другом. 



## Фильтры

Фильтры используются для форматирования текста.

Могут использоваться в текстовой *interpolation*  или с `v-bind`:

```html
{{ message | capitalize }}
<div v-bind:id="rawId | formatId"></div>
```

Сам фильтр – *Javascript function*. Может задаваться либо в момент создания [Vue instance](#vue-instance) или через метод `filter`:

```javascript
Vue.filter( {string} id, [ {Function} definition ])
Vue.filter( 'my-filter', function (value) { /* return ... */  })
```

# События

*Binding to event's* (привязка к событиям) позволяет обрабатывать наступление события.

Для *binding*'а *handler*'а используется `v-on` *directive*.

```html
<tag v-on:<eventname>="<Function | Expression>"></tag>    
```

Возможна сокращенная запись *directive* через `@`:

```html
<tag @<eventname>="<Function | Expression>"></tag>  
```

Типы аргументов:

- Привязка *expression*:

  ```html
  <button v-on:click="counter+=1">+1</button>
  <button @click="counter+=1">+1</button>
  ```

- Привязка *Function*. Для доступа к элементу, на котором произошло событие следует использовать `event.target`.

  ```html
  <button v-on:click="func">+1</button>
  <script>
  new Vue({
    /* ... */
    methods: {
      func (event) { 
          event.target....
      }
    }
  });
  </script>
  ```

Чтобы передать параметр в *method*, например в цикле, следует подставить этот параметр в момент вызова *method*'а:

```html
<tag 
    v-for="item in items" 
	@click="func(item)"
></tag>
<script>
new Vue({
  /* ... */
  methods: {
    func (item) { 
        ...
    }
  }
});
</script>
```

Если необходимо передать в *method* помимо пользовательского параметра, оригинальное событие `event` ,  то следует передать в *method* специальную переменную `$event`: 

```html
<tag 
    v-for="item in items" 
	@click="func(item, $event)"
></tag>
<script>
new Vue({
  /* ... */
  methods: {
    func (item, event) { 
        ...
    }
  }
});
</script>
```



# Формы

С помощью `v-model` организуется двунаправленный *binding* – *ViewModel* ↔ с атрибутами элементов формы (`input`, `textarea`, `select`). В зависимости от HTML *tag*'а, Vue автоматически управляет одним из атрибутов: `value`, `checked`, `selected`. Работа `v-model` *directive* основана на `v-bind` *directive*, которая *bind*'ит атрибуты тегов. И `v-model` может быть заменена на `v-bind` со сложным *expression*.

 `v-model` игнорирует начальное значение атрибутов `value`, `checked` или `selected`. В качестве значения для `v-model` необходимо использовать *property* из `data`. Значение этого *property* с самого начала определяет состояние элемента управления.

Поддерживаемые *tag*'и:

- `input type="text"`:

  ```html
  <input v-model="<property>">
  ```

- `textarea`

  ```html
  <textarea v-model="<property>"></textarea>
  ```

- `input type="checkbox"`:

  - скалярное *property*, одиночный  `input`

    ```html
    <input type="checkbox" id="<id>" v-model="<property>">
    <label for="<id>">Name</label>
    ```

  - *property*-массив, множественные `input`

    ```html
    <input type="checkbox" id="<id1>" value="<value1>" v-model="<property>">
    <label for="<id1>">Name1</label>
    <input type="checkbox" id="<id2>" value="<value2>" v-model="<property>">
    <label for="<id2>">Name2</label>
    ```

  

- `input type="radio"`

  ```html
  <input type="radio" id="<id1>" value="<value1>" v-model="<property>">
  <label for="<id1>">Name1</label>
  <input type="radio" id="<id2>" value="<value2>" v-model="<property>">
  <label for="<id2>">Name2</label>
  ```

- `select`

  ```html
  <select v-model="<property>">
    <!-- рекомендуется добавлять disabled вариант со значением "" -->
    <option disabled value="">Выберите один из вариантов:</option>
    <option value="<value1>">Text1</option>
    <option value="<value2>">Text2</option>
  </select>
  <script>
  new Vue({
    data: {
      selected: ''
    }
  })
  </script>
  ```

<u>*Binding* произвольного *expression* на атрибут `value`</u>

По умолчанию, при переключении состояния `checkbox` в *ViewModel* подставляются значения `true` и `false`. Можно сделать *binding* произвольных *expression*'s на состояния `checkbox` через атрибуты `bind:true-value` и `bind:false-value`:

```html
<input type="checkbox" bind:true-value="<expr1>" bind::false-value="<expr2>" ...>
```

Вместо конкретных значений `value` для `input` и `select`, которые при выборе подставляются в *ViewModel* можно сделать *binding* произвольных *expression*'s через атрибут `v-bind:value`:

```html
<input type="radio" v-bind:value="<expr>">
```

```html
<select>
    <option v-bind:value="<expr>">Name</option>
</select>
```

## Модификаторы

Используются для автоматической обработки введенного в `input` значения.

- `.number` – привести строку к числу

  ```html
  <input v-model.number="<property>" type="number">
  ```

- `.trim` – удалить пробелы в начале и конце строки

  ```html
  <input v-model.trim="<property>">
  ```

## Особенные атрибуты

### `disabled`

*Bind*'ится как любой другой атрибут

```html
<input type="text" :disabled="validated == 1">
```



 

# `v-for`

Многократно отрисовывает *tag* или блок шаблона. 

Формат `v-for` *directive* :

```
v-for="<value> in <array>"
v-for="(<value>, <index>) in <array>"
v-for="(<value>, <key>) in <object>"
```

где в `<value>`, `<key>`, `<index>` – подставляются значения для текущей итерации. 

Примеры:

```html
<ul>
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
```

```html
<select v-model="<property>">
  <option disabled value="">Выберите один из вариантов:</option>  
  <option v-for="(value, key) in object" v-bind:value="key">
    {{ value }}
  </option>
</select>
```

<u>Повторение tag'а фиксированное число раз</u>

Для этого нужно указать в `v-for` целое число `<count>`

```html
<tag v-for="<variable> in <count>">
<span v-for="n in 10">{{ n }} </span>
```

Переменная `variable` примет значения `1 ... <count>`. Переменная `variable` может участвовать в любых *expression*, передаваться в качестве аргумента в *method*'ы

# HTML class и style

Можно вручную работать со строками целиком в атрибутах `class` и `style` для *HTML tag*'ов, через `v-bind` *directive*. Однако гораздо удобнее передавать в `v-bind` объекты и массивы.

Способы указания *HTML class*'ов (и *styl*'ей):

- через объект, объявленный в `v-bind:class` атрибуте.

  ```html
  <tag [class="..."] v-bind:class="{'<class1>': <bool_expr>, <class2>: <bool_expr>, ... }"/>
  
  <div v-bind:class="{ a: isA, b: isB }"></div>   
  <script>
      data: {
        isA: true,
        isB: false
      }
  </script>  
  ```

- через указания имени объекта, объявленного вне `v-bind:class` атрибута:

  ```html
  <tag [class="..."] v-bind:class="<object>"/>
  
  <div v-bind:class="obj"></div>   
  <script>
    data: {
      obj: {
        isA: true,
        isB: false        
      }  
    }
  </script>    
  ```

- через указание *method*'а или *computed* свойства, возвращающего объект

  ```html
  <div v-bind:class="computedProperty"></div>   
  <script>
    computed: {
     computedProperty: function () {
      return {
        ...
      }
    }
  </script> 
  ```

# Component

Компоненты — это переиспользуемые *Vue instance*'s со своим именем. 

Преимущества использования *component*'ов:

- разделение кода на автономные части. 
- многократное переиспользование *component*'ов
- используют параметры и вызывают события

Так же как *Vue instance* имеет жизненный цикл, также и каждый *component* имеет жизненный цикл. Так же можно создать [hook'и жизненного цикла](#lifecycle-hook). 

## Создание *component*'а

Так как компоненты это переиспользуемые *Vue instance*'s, то они принимают те же опции что и `new Vue`, такие как `data`, `computed`, `watch`, `methods`, хуки жизненного цикла. За исключением нескольких специфичных для корневого *Vue instance* опций, например `el`.

Рекомендуется именовать *component*'ы в стиле *kebab-case*.

Есть два способа регистрации *component*'ов: *global* и *local*.

### *global* регистрация

При *global* регистрации, *component* регистрируется один раз и может далее использоваться в шаблоне любого *Vue instance*'а. *Global* регистрация *component*'а должна быть выполнена до создания *Vue instance*'а. 

Используется метод:

```javascript
Vue.component(id, definition)
```

```javascript
Vue.component('button-counter', {
  template: '<div>...</div>',
  data () {
    return {
      <prop>: <value>,
      ...
    }    
  },
  ...  
})
```

### *local* регистрация

При *local* регистрации, *component* регистрируется в опциях конкретного *Vue instance*, внутри которого будет использоваться. *Component* доступен только тому *Vue instance*, в опциях которого он зарегистрирован.

- определение *component*'а сохраняется в константу (или переменную):

  ```javascript
  const MyComponent = {
    template: '...',
    ...  
  };
  ```

- *component* регистрируется в опции `components`:

  ```javascript
  new Vue({
    ...
    components: {
      '<id>': <definition>,  
      'my-component': MyComponent
    }
  });
  ```

  

### Свойства

#### `data`

Свойство `data` должно быть функцией, а не объектом:

```json
data () {
  return {
    <prop>: <value>,
    ...
  }  
}
```

В этом случае каждый *component instance* получает свой экземпляр `data`. Если использовать не функцию, а объект, то все *component instance*'s будут делить один экземпляр `data`.

#### `template`

Определяет HTML-шаблон *component*'а. 

Шаблон должен иметь один корневой элемент. Поэтому весь шаблон должен быть заключен внутри какого-нибудь тега (например, `div`):

```json
template: '<div>...</div>'
```

#### Входные параметры `props`

*Component* должен явно задекларировать свои входные параметры в опции `props`. 

Рекомендуется именовать входные параметры в стиле *kebab-case*.

Входные параметры передаются только от родительского *component*'а к дочернему, дочерний *component* не может их использовать для возврата результата. ОДНАКО! Т.к. значения передаются в дочерний *component* по ссылке, при изменении объекта или массива в дочернем *component*'е – его значение изменится и в родительском *component*'е. 

<u>Без валидации</u>

```
props: ['name1', 'name2', ...]
```

<u>С валидацией</u>

Возможные способы задания:

```javascript
props: {
    // Проверка одного типа
    prop1: Number,
    // Проверка типа и дополнительные проверки
    prop2: {
      type: Number,
      required: true,
      default: 100
    }
}
```

- `type`. Возможные типы:
  - `String`
  - `Number`
  - `Boolean`
  - `Array`
  - `Object`
  - `Date`
  - `Function`
  - `Symbol`
- `default` – значение по умолчанию, если входной параметр не инициализирован. 
- `required` – параметр должен быть указан при использовании *component*'а.



## Использование *component*'а

*Component* используется как пользовательский тег внутри некоторого родительского *template* (вплоть до корневого *Vue instance*).

Например, для корневого *Vue instance*:

```html
<div id="app">
  <my-component></my-component>
</div>

<script>
new Vue({
  el: '#app',
  ... 
});  
</script>
```

### Входные параметры

Входные параметры передаются в *component* через пользовательский атрибут. 

<u>Передача статического текстового значения</u> 

```javascript
<my-component
  prop1="value1"
  prop2="value2"
></my-component>
```

<u>Динамический входной параметр</u>

Динамические входные параметры привязаны к динамическому *Javascript expression* (как [директивы](#директивы)) .  Для привязывания используется `v-bind` *directive*:

```html
<my-component v-bind:prop1="value1">
```

Или сокращенная запись:

```html
<my-component :prop1="value1">
```



## Изменение состояния *parent component*'а из *child component*'а

*Parent component* должен сделать *binding*'а *handler*'а к *custom event*'у, аналогично как для обычных [событий](#события):

 ```html
<component @<eventname>="<Function | Expression>"></tag>
 ```

Теперь в *Javascript component*'а мы можем вызвать этот *handler* через `this.$emit`:

```javascript
this.$emit('<eventname>')
```

Также в 1, 2... аргументах `this.$emit` можно передать аргументы в *handler* для *parent component*'а

```html
<-- Child component -->
<script>
this.$emit('<eventname>', <arg1>, <arg2>)
</script>
    
<-- Parent component -->
<component @<eventname>="<method>"></tag> 
<script>
...
	methods: {
        <method>(<arg1>, <arg2>) {
            ...
        }
        
    }
	
...
</script>    
```



## Доступ к экземплярам дочерних tag'ов или component'ов из Javascript

Необходимо назначить ID дочернему *tag*'у или *component*'у с помощью атрибута `ref`:

```html
<component ref="<id>">
<tag ref="<id>">
```

Теперь в этом же файле, где используется этот `<component>`, мы можем получить из *Javascript* доступ к экземпляру дочернего *tag*'а или *component*'а:

```
this.$refs.<id>
```

Можно вызывать методы у дочернего компонента:

```html
this.$refs.<component>.<method>
```

Пример для *tag*'а:

```html
<input ref="input">
<script>
    ...
    this.$refs.input.focus()
    ...
</script>
```

## Доступ к данным, свойствам и методам дочернего component'а

(Мое) Возможен из Javascript через точку, по ссылке на component:

```javascript
this.$refs.<component>.<property>
```



# Mixin

Mixin (примесь) – это объект с методами и свойствами, которые могут быть  «подмешаны» (*mixed*)  к свойствам *component*'а. Могут использоваться любые методы и свойства, которые поддерживаются для *component*'ов Vue (и *Vue instanc*'а). 

Алгоритм слияния (*merging*) свойств, если они перекрываются:

- объекты объединяются, причем приоритет имеют свойства *component*'а. Например, объекты `data`, `methods`. Объединение выполняется рекурсивно.
- *hook*'и не заменяются, а объединяются в массив и все из них вызываются. *Hook*'и *mixin*'а вызываются перед *hook*'ами *component*'а.

*Mixin* подключается к *component*'у через свойство `mixins`.

```javascript
// Примесь
var myMixin = {
  created () {
    ...
  },
  methods: {
    ...
  },
  data () {
    return {
      ...
    }
  }  
}

// Компонент
var Component = Vue.extend({
  mixins: [ myMixin ]
})
```

## Глобальный mixin

Глобальный mixin подмешивается во все *Vue instanc*'ы, созданные после его объявления. 

```javascript
// Объявление глобального mixin'а
Vue.mixin({
  methods: {
    ...
  },
})
```

## Mixin в однофайловых component'ах

При использовании однофайловых компонентов каждый *component* и *mixin* должны быть в отдельных файлах.

`mixin.js`

```javascript
export default {
  data () {
    return {
      ...  
    }
  },
  methods: {
    ...
  }  
}
```

`Component.vue`

```javascript
import mixin from '../mixin';

export default {
  mixins: [ mixin ],
  ...  
}
```

# Система сборки

Порядок подключение нового *npm*-пакета:

- добавить описание пакета в *package.json*:

  ```json
  "dependencies": {
      "fabric": "^3.5.0",
      ...
    },
  ```

- переустановить пакеты (возможно, вначале удалить целиком `node_modules`):

  ```bash
  yarn install
  ```

- подключить пакет в нужном *Vue*-компоненте  `.vue`:

  ```javascript
  import fabric from 'fabric'
  ```

  

# Axios


Axios – HTTP-клиент для браузера и *node.js* на основе *Promise*.

## `axios(config)`

Выполнение запроса:

```javascript
axios(config);
```

```javascript
axios({
  method: '<method>',
  url: '<data>',
  /* Заголовки */
  headers: {
    '<header_name>': '<value>',
    ...  
  },
  /* Параметры запроса (GET, POST...) */    
  params: {
    <param>: <value>,
    ...  
  },
  /* Полезные данные, отправляемые в теле запроса. 
     Применимо только к 'PUT', 'POST', 'PATCH' */    
  data: {
    <prop>: '<value>',
    ...
  }
})
  .then(function (response) {
    // success
  })
  .catch(function (error) {
    // error
  })
  .then(function () {
    // always executed
  });
```

Возможные *method*'ы:

- `get`
- `post`
- ...

Структура `response`:

```json
{
  // Данные от сервера
  data: {},

  // Код состояния (status code) от сервера
  status: 200,

  // Пояснение к коду состояния (reason phrase)
  statusText: 'OK',

  // Заголовки ответа
  headers: {},

  // `config` из запроса
  config: {},

  // `request` - XMLHttpRequest instance, созданный для запроса
  request: {}
}
```

## `axios.<method>`

Для удобства можно использовать отдельные методы-псевдонимы для каждого *method*'а: 

- `axios.get (url [, config])`

  ```javascript
  axios.get('/user', {
      ...
  })
      .then(...)
  ```

- `axios.post(url[, data[, config]])`

  ```javascript
  axios.post('/user', {
      <prop>: '<value>',
      ...
    }, {
      ...
  })
      .then(...)
  ```

## hook'и для использования

Для загрузки данных с сервера и записи их в данные *Vue instance*'а можно использовать:

- `created` *hook* (из книги): 

  - ``` javascript
created: function() {
       axios.get('...')
         .then((response) => { 
           this.<property> = response.data.<property>;
       })
     }         
    ```

- `mounted` *hook* (из документации)

# Vue-cli

Установка:

```bash
npm install -g @vue/cli
```



#  tiptap

 https://github.com/scrumpy/tiptap 

 https://tiptap.scrumpy.io/

 https://tiptap.scrumpy.io/docs 



# Инструменты

Расширение для Google Chrome – [vue-devtools](). Необходимо в настройках включить "Разрешить открывать локальные файлы по ссылкам". Добавляет в консоль закладку *Vue*.

github.com/ErikCH/VuejsInActionCode

# ECMAScript 2015

https://babeljs.io/docs/en/learn

 https://learn.javascript.ru/import-export 