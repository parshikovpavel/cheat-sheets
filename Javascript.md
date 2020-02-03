- *enumerable* – перечисляемые *properties*
- *own* – собственные *properties*





### `Object.assign()`

(ES6) Метод копирует все *enumerable own properties* из `sources` объектов в `target` объект.  Последующие *properties* перезаписывают предыдущие. 

```javascript
Object.assign(target, ...sources)
```

Возвращаемое значение – `target` объект.

```javascript
var o1 = { a: 1 };
var o2 = { b: 2 };

var obj = Object.assign(o1, o2, o3); // obj = o1 = { a: 1, b: 2}
```

Клонирование объекта:

```javascript
var obj = { a: 1 };
var copy = Object.assign({}, obj); // copy = { a: 1 }
```





# Function

Объявление метода:

```javascript
{
    name: function () {}
}
```

Стандарт ES6 (другое название ES2015) поддерживает сокращенный синтаксис объявления метода

```javascript
{
    name() {}
}
```

 

