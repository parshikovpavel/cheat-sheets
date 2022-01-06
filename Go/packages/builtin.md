## `builtin`

https://pkg.go.dev/builtin

Встроенные функции, которые являются частью языка.

### `len()`

Возвращает длину `v`:

```go
func len(v Type) int
```

Результат:

```
String    string type      длина string в байтах
Array     [n]T, *[n]T      длина массива (== n)
Slice     []T              число элементов
Map       map[K]T          число элементов
Channel   chan T           количество элементов, стоящих в очереди в chain
```

### `cap()`

Возвращает *capacity* (емкость) `v` 

```go
func cap(v Type) int
```

Результат:

```
Array     [n]T, *[n]T      длина array (== n, == len())
Slice     []T              максимальная длина, которую slice может достичь, т.е. размер underlying array 
Channel   chan T           емкость буфера channel, в штуках элементов;
```

Для *slice* всегда сохраняется зависимость между `len()` и `cap()`:

```
0 <= len(s) <= cap(s)
```

### `close()`

[подробнее `close()`](../types/channel.md#close)







### `delete()`

```go
func delete(m map[Type]Type1, key Type)
```

Удаляет *element* с указанным *key* (`m[key]`) из `map`. Если `m = nil` или `m[key]` не существует, `delete()` ничего не делает.

### `make()`

```go
func make(t Type, size ...IntegerType) Type
```

Может использоваться для выделения памяти и инициализации объектов с *type*'s:

- `slice`
- `map`
- `chan`

Особенности:

- возвращает `Type` (а не `*Type`, как `new()`). 
- возвращает не *zero value* (`nil`), а уже инициализированное значение (например, *sliсe* некоторого размера).

Смысл введения этой функции в том, что эти три типа (`slice`, `map`, `chain`) представляют собой, под капотом, ссылки на структуры данных, которые должны быть инициализированы перед использованием. Это позволяет за одну операцию сразу получить инициализированную и готовую к использованию переменную, отличную от *zero value* (`nil`).  Например, для *slice* можно с помощью `make()` сразу инициализировать *length*, *capacity* и соответствующий *underlying array* (??? внутри этого *underlying array* данные будут очищены в *zero value*). Поэтому с этими типами `new()` используется очень редко.

Также важно, что `make` возвращает *value* (не *pointer*). Но т.к. они уже внутри содержать *reference*, их всегда и используются *by value* и получать специально *pointer* не нужно. 

TODO!!! https://golang.org/doc/effective_go#allocation_make

Возвращаемое значение зависит от `Type`:

- *slice*:

  ```
  Call             Type T     Result
  
  make(T, n)       slice      slice of type T with length n and capacity n
  make(T, n, m)    slice      slice of type T with length n and capacity m
  ```

  - Если указан только `size1`, то получаем *slice* с параметрам `len = cap = size1`.
  - Если указан также `size2`, то `cap = size2`.

  Пример: выделить *slice* с `len = 0`, `cap = 10`. 

  ```go
  make([]int, 0, 10) 
  ```

- *map*:

  ```
  Call             Type T     Result
  
  make(T)          map        map of type T
  make(T, n)       map        map of type T with initial space for approximately n elements
  ```

  * Если *size* не указан - выделяется пустой *map* с небольшим *capacity*

  * Если *size* указан - выделяется пустой *map* с указанным начальным *capacity*

    Первоначальная capacity не ограничивает его size

- *channel*:

  ```
  Call             Type T     Result
  
  make(T)          channel    unbuffered channel of type T
  make(T, n)       channel    buffered channel of type T, buffer size n
  ```

  * Если *size* не указан - *unbuffered channel*
  * Если *size* указан, то  *buffered channel*, `buffer capacity = size`



Разница между `new()` и `make()`:

```go
p := new([]int)       // *p == nil; редко используется
v := make([]int, 100) // v - slice на 100 элементов (каждый элемент инициализирован zero value)
```



### `new()`

```go
func new(Type) *Type
```

Функция, выполняет *allocate* памяти указанного *type* с *zero value* и возвращает *pointer* на нее. Подробнее про *zero value* ([link](#zero-value)).

-  Аргумент:
   - `Type`  – *type* (не *value*). 
-  Возвращаемое значение:
   - `*Type` – *Pointer* на выделенный участок памяти указанного *type* с *zero value*.

По сути, `new()` аналогично *variable declaration* ([link](#variable-declaration)), но возвращает *pointer*, а не *value*.

```go
a := new(int) // type *int
var b int     // type int
```

`new()` аналогично следующей форме *Composite literal*:

```go
s := new(S)
s := &S{}
```

### `panic()`

[смотреть тут](../Lang.md#panic)

### `print()`

```go
func print(args ...Type)
```

Форматирует аргументы некоторым образом, в зависимости от реализации языка, и записывает результат в *standard error*. 

Не гарантируется сохранение этой функции в языке (не рекомендуется использовать для продакшена).

Полезна для отладки:

- выводит адрес для *pointer type*:

  ```go
  	var s string = "syz"
  	var i interface{} = &s
  
  	println(s)  // syz
  	println(&s) // 0xc000044768
  ```



### `println()`

```go
func println(args ...Type)
```

Форматирует аргументы некоторым образом, в зависимости от реализации языка, и записывает результат в *standard error*. 

Между аргументами добавляются пробелы, в конце добавляется *newline*.

Полезна для отладки (смотреть `print()` [1](#print))



### `recover()`

[смотреть тут](../Lang.md#recover)



### `type error`

Смотреть [1](errors.md#type-errors)

