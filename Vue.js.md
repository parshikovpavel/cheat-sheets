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

- `methods` – методы, которые будут подмешаны (*mixed*) к *Vue instance*. *Method*'ы доступны через *Vue instance*:

  ```javascript
  vm.<method>()
  ```

  Внутри *method*'а' переменная `this` указывает на *Vue instance*. Поэтому для доступа к *property* в `data` необходимо писать `this.<property>`.

  Используются в качестве [*handler*'а *event*'ов](#события)
  
- `filters` – [фильтры](#фильтры).

## Вычисляемые свойства

- `computed` – определение вычисляемых свойств – динамических, рассчитываемых свойств. Позволяют логику вычислений убрать из *View* и перенести в *ViewModel*. 

Определение:

```javascript
  computed: {
    <property>() {
	  return ...;
    }
  }
```

## Наблюдатели (watcher)

- `watch` – объект, ключи которого – выражения для наблюдения, а значения – *callback*'и, вызываемые при их изменении. 

  ```javascript
  watch: {
    <property>: function (newVal, oldVal) {
      /* ... */
    }
  }
  ```

  

 

  

На одной странице можно запустить несколько *Vue instance*'ов, которые примонтированы к разным элементам DOM. Это имеет смысл использовать, если *Vue instance* примонтирован к небольшим компонентам – карусель или веб-форма.

`Vue` *instance*'ы доступен также через переменные `$vm<N>`.

# Жизненный цикл

Жизненный цикл *Vue instance*  состоит из нескольких этапов. 

![lifecycle](https://ru.vuejs.org/images/lifecycle.png)

Между этими этапами вызываются функции, называемые хуками жизненного цикла (*lifecycle hook*), с помощью которых можно выполнять свой код в определённый момент жизненного цикла.

Список *Lifecycle hook*'ов:

- `beforeCreate`
- `created`
- `beforeMount`
- `mounted`
- `beforeUpdate`
- `updated`
- `beforeDestroy`
- `destroyed`

Большую часть времени программа проводит в цикле событий (`beforeUpdate` и `updated` *hook*'и). 

# Template

Vue.js использует *template* синтаксис на основе HTML, который позволяет связывать (*bind*) элементы DOM с `data` данными *Vue instance*'а. 

## Интерполяции

Интерполяции (*Interpolations*) – способ отражения данных *Vue* на элементы DOM. 

Все *interpolation*'s принимают в качестве параметра *JavaScript expression*, например:

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

Директивы (*directive*) – это атрибуты, которые *bind*'ят данные *Vue* (или Javascript выражение с этими данными) с некоторым состоянием элемента DOM. Начинаются с префикса `v-`. 

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

Поддерживает сокращенная запись

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

Tag'и с атрибутами `v-if`, `v-else-if`, `v-else` должны идти строго друг за другом. 



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

- Привязка *Function*:

  ```html
  <button v-on:click="func">+1</button>
  <script>
  new Vue({
    /* ... */
    methods: {
      func: function (event) { /* ... */ }
    }
  });
  </script>
  ```

# Формы

С помощью `v-model` организуется двунаправленный *binding* – *ViewModel* **↔** с атрибутами элементов формы (`input`, `textarea`, `select`). В зависимости от HTML *tag*'а, Vue автоматически управляет одним из атрибутов: `value`, `checked`, `selected`. Работа `v-model` *directive* основана на `v-bind` *directive*, которая *bind*'ит атрибуты тегов. И `v-model` может быть заменена на `v-bind` со сложным *expression*.

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

По умолчанию, на состояния одиночного `checkbox` сделан *binding* значений `true/false`. Можно сделать *binding* произвольных *expression*'s через атрибуты `bind:true-value` и `bind:false-value`:

```html
<input type="checkbox" bind:true-value="<expr1>" bind::false-value="<expr2>" ...>
```

Вместо конкретных значений `value` для `input` и `select` можно сделать *binding* произвольных *expression*'s через атрибут `v-bind:value`:

```html
<input type="radio" v-bind:value="<expr>">
```

```html
<select>
    <option v-bind:value="<expr>">Name</option>
</select>
```

<u>Модификаторы</u>

Используются для автоматической обработки введенного в `input` значения.

- `.number` – привести строку к числу

  ```html
  <input v-model.number="<property>" type="number">
  ```

- `.trim` – удалить пробелы в начале и конце строки

  ```html
  <input v-model.trim="<property>">
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



#  tiptap

 https://github.com/scrumpy/tiptap 

 https://tiptap.scrumpy.io/

 https://tiptap.scrumpy.io/docs 

# Инструменты

Расширение для Google Chrome – [vue-devtools](). Необходимо в настройках включить "Разрешить открывать локальные файлы по ссылкам". Добавляет в консоль закладку *Vue*.

github.com/ErikCH/VuejsInActionCode
