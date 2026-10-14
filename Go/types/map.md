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

Эта переменная `m` представляет собой `map` из *string key*'s и *int value*'s:

```go
var m map[string]int
```

*Map type*'s - это *reference type*'s, как *pointer* или *slice*, поэтому выше `m == nil`; оно не указывает на *initialized map*. 

Для *nil map*:

- при чтении ведет себя как *empty map*
- при записи произойдет *runtime panic* (!!!) 

Для инициализировать map необходимо использовать *built-in function* `make` ([ссылка](../packages/builtin.md#make())):

```go
m = make(map[string]int)
```

Функция `make` выделяет память и инициализирует структуру данных в виде *hash map*, а затем возвращает *map value*, указывающее на нее. Особенности этой структуры данных зависят от реализации среды выполнения и не определены самим языком. 

TODO!!!! https://blog.golang.org/maps

### Частые операции

- создание (инициализация):

  - создание через [*composite literal*](../Lang.md#composite-literal). При этом можно создать пустой `map` или сразу инициализировать его:

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

    Первоначальная *capacity* не ограничивает его *size*: `map` растет автоматически с добавлением *element*'ов, кроме случая когда `map = nil`. `map = nil` – то же самое что пустая `map` , но нельзя добавлять *element*'ы (?)

- Длина:

  [`len(a)`](#len()) 

- Добавить *element* ([assignment](#assignment))

  `a["b"] = 1` 

  Если `map` – `nil` 

- Получить *element* (*[index expression](#index-expression)*)

  `c := a["b"]` , то произойдет *runtime panic* (!!!) 

  Если `map` – `nil` или не содержит ключ `"b"`, то `a["b"]` – [*zero value*](#zero-value) для *element type*.

- Можно выполнять операции с *zero value*:

  ```go
  	m := make(map[string]int)
  
  	m["a"]++
  	m["b"] += 10
  
  	fmt.Print(m) // map[a:1 b:10]
  ```

  Это возможно, т.к. *index expression* возвращает [zero value](https://golang.org/ref/spec#The_zero_value) для отсутствующих *key*.

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

### Механизм разрешения коллизий

**До Go 1.24:**

- использовался вариант метода цепочек — цепочки бакетов, а не связанный список отдельных элементов.

- Каждый бакет вмещает **8 пар ключ–значение**.
- Младшие биты хеша выбирают бакет; часть старших битов хранится в `tophash`, чтобы быстрее искать внутри него.
- Если бакет заполнен, создаётся **overflow-бакет**, связанный с ним указателем. Получается цепочка бакетов.
- При росте элементы постепенно переносятся из старых бакетов в новые — *эвакуация*.

```
бакет → overflow-бакет → overflow-бакет
 8 пар       8 пар           8 пар
```

**Начиная с Go 1.24 — Swiss Tables, открытая адресация:**

- Группа содержит **8 слотов** и управляющие байты с состоянием слотов и частью хеша.
- По хешу выбирается начальная группа.
- Если ключ не найден и поиск нужно продолжить, проверяется следующая группа по последовательности пробирования.
- **Цепочек overflow-бакетов нет**: коллизии разрешаются поиском других слотов внутри таблицы.
- При заполнении отдельная таблица расширяется либо разделяется, элементы перераспределяются.

```
группа → проверяем другую группу → ещё одну
         по последовательности пробирования
```

### Эвакуация map

Когда бакеты сильно заполнены, необходимо увеличить размер map'ы

В Go это происходит, когда коэфициент заполнения превышает 6.5

Он рассчитывается, как количество элементов в map'е/количество бакетов.

В этот момент необходимо увеличит размер map'ы. Создается map'а, которая имеет в 2 раза больший размер

Дальше необходимо эвакуировать данные из старой map'ы в новую. Они эвакуируются при операциях записи (добавления, обновления элементов в map'у).

### Конкурентный доступ

`map` не является safe-concurrent. Если несколько *goroutine* одновременно пытаются записать в неё данные (или даже просто читать при одновременной записи) без какой-либо синхронизации, возникает **состояние гонки (race condition)**. Go специально отслеживает такие ситуации и при их обнаружении вызывает панику с сообщением `fatal error: concurrent map writes`.  Это сделано для защиты от повреждения внутренней структуры хеш-таблицы.

Внутри `map` — хеш-таблица, организованная как массив бакетов (ячеек). При коллизиях хешей создаются цепочки. Проблема возникает, когда карта автоматически меняет размер (resizing): в этот момент одна *goroutine* может находиться в середине перемещения элементов между бакетами, а другая в это же время пытается записать новый ключ. В результате цепочки указателей внутри `map` могут повредиться: программа начнёт ссылаться на освобождённую память или на перекрывающиеся данные.

Поэтому конкурентный доступ к *map* должен происходить с использование механизмов синхронизации (мьютекс).