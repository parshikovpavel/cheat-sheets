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
  [  ]               
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

Связывает текстовое содержимое элемента (`textContent`). Эквивалентно, можно использовать текстовую *interpolation*. Текстовую *interpolation* нужно использовать, если требуется изменять только часть текста.  

```html
<tag v-text="<expression>"></tag>
<tag>{{ expression }}</tag>      <!-- Аналогично -->
<tag>Text {{ expression }}</tag> <!-- Изменяется часть текста -->    
```

#### `v-bind`

Связывает атрибуты тега.  Т.к. текстовая *interpolation* не работает с  HTML-атрибутами. 

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

Связывает HTML-содержимое элемента (`innerHTML`), экранирование специальных символов не выполняется.  

```html
<tag v-html="<value>"></tag>
<div v-html="html"></div>
```

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

Используется `v-on` *directive*.

# Инструменты

Расширение для Google Chrome – [vue-devtools](). Необходимо в настройках включить "Разрешить открывать локальные файлы по ссылкам". Добавляет в консоль закладку *Vue*.

github.com/ErikCH/VuejsInActionCode