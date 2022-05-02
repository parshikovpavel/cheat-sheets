https://habr.com/ru/post/457728/

`map` – это неупорядоченная (!!!) группа *element*'ов (*value*'s) одного *type* (*element type*), проиндексированные уникальными *key*'s другого *type* (*key type*). Значение неинициализированного `map` – `nil`.

<pre>
MapType     = "map" "[" KeyType "]" ElementType .
KeyType     = <a href="#type">Type</a> .
ElementType = <a href="#type">Type</a> .
</pre>



```go
map[string]int
map[*T]struct{ x, y float64 }
map[string]interface{}
```

Для *key type* должна быть полностью определены операторы `==` и `!=`. 

Поэтому в качестве *key type* можно использовать:

- *Numeric types*
- *String types*
- *Array types*
- *Struct types* 
- *Pointer types*
- `interface`. При этом операторы `==`, `!=`  должны быть определены для динамических значений в ключе (*dynamic key values*)

В качестве *key type* нельзя использовать:

- *Slice types*
- `function`
- `map`
- `slice`

### Internal

Если передать `map` *by value* в *function* и там изменять ее содержимое через формальный параметр, то изменения будут видны и через фактический параметр (как `object` в PHP).

This variable `m` is a map of string keys to int values:

```
var m map[string]int
```

Map types are reference types, like pointers or slices, and so the value of `m` above is `nil`; it doesn't point to an initialized map. A nil map behaves like an empty map when reading, but attempts to write to a nil map will cause a runtime panic; don't do that. To initialize a map, use the built in `make` function:

```
m = make(map[string]int)
```

The `make` function allocates and initializes a hash map data structure and returns a map value that points to it. The specifics of that data structure are an implementation detail of the runtime and are not specified by the language itself. In this article we will focus on the *use* of maps, not their implementation.

TODO!!!! https://blog.golang.org/maps

### Частые операции

- создание (инициализация):

  - создание через *composite literal*. При этом можно создать пустой `map` или сразу инициализировать его:

    ```go
    var timeZone = map[string]int{
        "a": 1,
        "b": 2,
        "c": 3,
    }
    ```

  - создание пустого `map` через `make`:

    ```go
    make(map[string]int)       // небольшого capacity
    make(map[string]int, 100)  // с указанным начальным capacity
    ```

    Первоначальная *capacity* не ограничивает его *size*: `map` растет автоматически с добавлением *element*'ов, кроме случая когда `map = nil`. `map = nil` – то же самое что пустая `map` , но нельзя добавлять *element*'ы.

- Длина:

  [`len(a)`](#len()) 

- Добавить *element* ([assignment](#assignment))

  `a["b"] = 1` 

- Получить *element* (*[index expression](#index-expression)*)

  `c := a["b"]` 

  Если `map` – `nil` или не содержит ключ `"b"`, то `a["b"]` – [*zero value*](#zero-value) для *element type*.

- Получить *element* и получить флаг, что *element* существует в `map`. Используется *multiple assignment*. Конструкция называется *"comma ok" idiom*.

  ```go
  var c int
  var ok bool
  c, ok = a["b"]
  ```

  Получить *element* и сразу проверить, что *element* существует в `map`.

  ```go
  if c, ok = a["b"]; ok {
    return c;
  }
  ```

  Если нужно получить только флаг, что *element* существует в `map` и сам *element* не нужен, то используем [blank identifier (`_`)](#blank-identifier):

  ```go
  _, ok = a["b"]
  ```

  

- Удалить *element*

  [`delete(a, "b")`](#delete())

  Если `a = nil` или `a[key]` не существует, `delete()` ничего не делает.

### Реализация *set* (множества)

*Set* (множество) можно реализовать как `map` с *element type* – `bool`. 

Операции:

- инициализировать *set*:

  ```go
  set := map[string]bool{
      "Ann": true,
      "Joe": true,
      ...
  }
  ```

- добавить элемент в *set*:

  ```go
  set["a"] = true
  ```

- проверить наличие элемента в *set*:

  ```go
  if set[person] { // будет false если person не в set
      ...
  }
  ```



(!!!) Но более экономным по памяти вариантом является использование `map[keyType]struct{}`:

Операции:

- инициализировать *set*:

  ```go
  type T struct{}
  
  set := map[string]T{
    "Ann": T{},
    "Joe": T{},
      ...
  }
  ```

  ...

### Ошибка Cannot assign to (struct field in map)

Например, такое выдает ошибку:

```go
package main

import "fmt"

type Animal struct {
	count int
}

func main() {
	m := map[string]Animal{"cat": Animal{2}, "dog": Animal{3}, "mouse": Animal{5}}
        fmt.Println(m)
	m["dog"].count = 4
	
	fmt.Println(m)

}
```

<u>Причина:</u>

Левая часть присвоения должна быть «*addressable*» ([1](https://golang.org/ref/spec#Assignments))

> Каждый левый операнд должен быть *addressable*, *map index expression* или (только для = присвоения) *blank identifier*.

и [2](https://golang.org/ref/spec#Address_operators) 

> *Operand* должен быть *addressable*, то есть быть либо *variable*, либо косвенным указателем (*pointer indirection*), либо *slice indexing operation*; или *field selector of an addressable struct operand*; или array *indexing operation of an addressable array*.

Решения:

1. Взять структуру целиком, изменить и присвоить обратно:

   ```go
   package main
   
   import "fmt"
   
   type Animal struct {
   	count int
   }
   
   func main() {
   	m := map[string]Animal{"cat": Animal{2}, "dog": Animal{3}, "mouse": Animal{5}}
   
   	fmt.Println(m)
   
   	var x = m["dog"]
   	x.count = 4
   	m["dog"] = x
   
   	fmt.Println(m)
   
   
   }
   ```

2. Использовать в *map* *pointer*'ы на структуру:

   ```go
   package main
   
   import "fmt"
   
   type Animal struct {
   	count int
   }
   
   func main() {
   	m := map[string]*Animal{"cat": &Animal{2}, "dog": &Animal{3}, "mouse": &Animal{5}}
   	fmt.Printf("%#v\n",m["dog"])
   	
   	m["dog"].count = 4
   
   	fmt.Printf("%#v", m["dog"])
   
   }
   ```

### Присваивание map