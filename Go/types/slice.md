## Определение

`slice` – наиболее часто используемая коллекция. 

`slice` (срез) – это дескриптор для непрерывного сегмента *underlying* `array` (базового массива), обеспечивающий доступ к пронумерованной последовательности элементов из этого *array*. Т.е. *slice* – это такой тип, который может хранить все возможные срезы *array* с элементами указанного *type*. 

*Zero value* для *slice* – `nil`.

Slice, *под капотом*, представляет собой контейнер (*slice header*), содержащий:

- указатель на данные (внутри *underlying array*)
- *length* (длину)
- *capacity* (емкость)

*Slice header* можно смоделировать с помощью типа [`reflect.SliceHeader`](https://golang.org/pkg/reflect/#SliceHeader):

```golang
type SliceHeader struct {
        Data uintptr
        Len  int
        Cap  int
}
```

После инициализации срез всегда связан с *underlying* `array` (базовым массивом), который содержит его элементы. Таким образом, *slice* разделяет хранилище со своим `array` и с другими `slice`'s того же `array`.

*Underlying* `array` может выходить за пределы конца `slice`. *Capacity* – это длина `slice` + длина `array` за пределами `slice`; `slice` длиной до этой *capacity* можно создать из исходного `slice`, применив [*slice expression*](#slice-expression). Значение *capacity* можно определить с помощью функции `cap()`.

Определение одномерного *slice*:

```
SliceType = "[" "]" ElementType 
```

```go
[]Тип        // без инициализации
[]Тип{1,2,3} // с инициализацией
```

```go
a := []string{"1", "2", "3"} // сокращенная форма
var b = []int{2, 3, 5}       // полная форма
```

Размер *slice* указывать не нужно, компилятор сам может подсчитать количество элементов.

Чтобы определить многомерный *slice*, необходимо в качестве `Тип` указать другой `slice`:

```go
[][]Тип
```

```go
var bigDigits = [][]string{
    {"1","2","3"},
    {"4","5","6"},
}
```

Основные операции со *slice*: (https://github.com/golang/go/wiki/SliceTricks)

- *clone (copy)*:

  ...

## `nil` и пустой *slice*

Если рассмотреть *slice header*:

```golang
[pointer] [length] [capacity]
```

тогда:

```golang
nil slice:   [nil][0][0]
empty slice: [addr][0][0] // pointer указывает на address
```

```go
func main() {

    var nil_slice []int
    var empty_slice = []int{}

    fmt.Println(nil_slice == nil, len(nil_slice), cap(nil_slice))				// true 0 0
    fmt.Println(empty_slice == nil, len(empty_slice), cap(empty_slice))	// false 0 0
}
```

Таким образом, и для *nil slice* и для *empty slice*:

- `len(slice) == 0`
- `cap(slice) == 0`
- *built-in function* `append` – обрабатывает их одинаково



## Реализация *stack* (стек)

*stack* на Go реализуется через *slice*:

- для push – используем функцию `append`
- для pop – используем *slice expression* для исключения *top element*.

```go
type Stack []int

// Push a new value onto the stack
func (s *Stack) Push(v int) {
	*s = append(*s, v) // Simply append the new value to the end of the stack
}

// Remove and return top element of stack. Return false if stack is empty.
func (s *Stack) Pop() (int, bool) {
	if s.IsEmpty() {
		return 0, false
	} else {
		index := len(*s) - 1 // Get the index of the top most element.
		element := (*s)[index] // Index into the slice and obtain the element.
		*s = (*s)[:index] // Remove it from the stack by slicing it off.
		return element, true
	}
}
```

Но эта реализация НЕ *thread safe*, т.к. мы можем попасть в race condition и попытаться сделать *pop* из пустого *stack* (если два потока попадут в функцию `Pop()`).

Чтобы избежать race condition – необходимо обернуть *slice* структуру и добавить в нее *mutex*:

```go
type stack struct {
     lock sync.Mutex // you don't have to do this if you don't want thread safety
     s []int
}

func NewStack() *stack {
    return &stack {make([]int,0),}
}

func (s *stack) Push(v int) {
    s.lock.Lock()
    defer s.lock.Unlock()

    s.s = append(s.s, v)
}

func (s *stack) Pop() (int, error) {
    s.lock.Lock()
    defer s.lock.Unlock()


    l := len(s.s)
    if l == 0 {
        return 0, errors.New("Empty Stack")
    }

    res := s.s[l-1]
    s.s = s.s[:l-1]
    return res, nil
}


func main(){
    s := NewStack()
    s.Push(1)
    s.Push(2)
    s.Push(3)
    fmt.Println(s.Pop())
    fmt.Println(s.Pop())
    fmt.Println(s.Pop())
}
```

 



## Общее

#### Обращение к элементам

Аналогично другим языкам, получение n-го элемента среза:

```go
slice[n]
```



## Адреса

- `&slice` – адрес *slice header*
- `&slice[0]` – адрес самих данных (в *underlying array*), который совпадает с адресом первого элемента



*Slice expression* (выражения для *slice*) позволяют получить *substring* из `string` или *slice* из `array`, `*array` или `slice`. 

Существует два варианты *slice expression*:

- простой
- полный

# *Slice expression*

## Простой *slice expression*

**✔**

В простой форме *slice expression* указывается нижняя и верхняя граница:

```
a[<low> : <high>]
```

В результате получается *substring* или *slice*, который:

- содержит элементы с индексами, начиная с `<low>` и заканчивая `<high>-1`, т.е. из диапазона `[<low>;<high>)` (открытый справа). 
- `len(res) = <high>-<low>`.
- `cap(res) = len(a) - <low>`

Например:

```go
a := [5]int{1, 2, 3, 4, 5}
s := a[1:4]                // тип `s` – []int            
fmt.Println(s)             // {2, 3, 4}
fmt.Println(len(s))        // 3
fmt.Println(cap(s))        // 4
```

Если `a` имеет тип  `*array`, то `a[<low> : <high>]` это сокращение для `(*a)[<low> : <high>]`.

`<low>` или `<high>` можно не указывать, в этом случае *by default*:

- `<low> = 0`
- `<high> = len(a)`

```go
a[2:]  // same as a[2 : len(a)]
a[:3]  // same as a[0 : 3]
a[:]   // same as a[0 : len(a)]
```

```go
func main() {
  a := []int{1, 2}

  fmt.Println(a[2:])  // == fmt.Println(a[2:2]), выводит []
  fmt.Println(a[:0])  // == fmt.Println(a[0:0]), выводит []
}
```



Для *array* и *string*, индексы должны находиться в диапазоне `0 <= low <= high <= len(a)`, в противном случае они находятся *out of range*. Для *slice*, верхней границей индекса является *slice capacity* `cap(a)`, а не его длина. [Константный](#constant) индекс должен быть неотрицательным и [representable](https://go.dev/ref/spec#Representability) значением типа `int`; для *array* и константных *string* – константные индексы также должны находиться в диапазоне (каком???). Если оба индекса являются [константами](#constant), они должны удовлетворять условию `low <= high`. Если индексы оказываются *out of range* во время выполнения, бросается  [run-time panic](#run-time-panic).

Результаты *slice expression*:

- для *string* и *slice* (за исключением [untyped string](#constant)) – результатом является *non-constant value* того же типа, что и операнд. 
- для *untyped string* – результатом является *non-constant value* типа `string`. 
- для *array*, он должен быть [addressable](https://go.dev/ref/spec#Address_operators), – результатом операции является *slice* с тем же типом элементов, что и *array*.

Если операнд для корректного *slice expression* – `nil` *slice* (корректным, наверное, будет только *slice expression* с верхним и нижним индексом – 0) , то результатом будет также `nil` *slice*. 

```go
func main() {
  var a []int

  fmt.Println(a[:] == nil) // true
  _ = a[1:]                // выбрасывает panic: runtime error: slice bounds out of range [1:0]
}
```

Если результат *expression* – *slice*, то он разделяет свой *underlying array* с операндом. Например:

```go
var a [10]int
s1 := a[3:7]   // underlying array of s1 is array a; &s1[2] == &a[5]
s2 := s1[1:4]  // underlying array of s2 is underlying array of s1 which is array a; &s2[1] == &a[5]
s2[1] = 42     // s2[1] == s1[2] == a[5] == 42; все элементы массива - это один и тот же underlying array element
```

## Полный *slice expression*

Полный *slice expression* позволяет указать *capacity*.

Может использоваться для `array`, `*array` или `slice` (но не для `string`):

```
a[<low> : <high> : <max>]
```

При этом будет создан *slice* как и для простого *slice expression* `a[<low> : <high>]`. Но для него устанавливается `capacity = <max> - <low>`.

TODO!!!



## Задачи

### Задача 1

Эта программа выводит `[]`, т.к. в функции `main()` не происходит изменение заголовка, который состоит из *pointer*, *len*, *capacity*. Поэтому в функции `main()` как был, так и остается *slice* с `len = 0` и `cap = 0`.

```go
package main

import "fmt"

func main() {
  s := []int{}
  add(s)
  fmt.Println(s)  // []
}

func add(s []int) {
  s = append(s, 4)
}
```

Эта программа выведет `[4]`, т.к. в функции `add()` произойдет изменение в *underlying array* и это изменение будет видно в `main()`.

```go
package main

import "fmt"

func main() {
	s := make([]int, 1)
	add(s)
	fmt.Println(s)  // [4]
}

func add(s []int) {
	s[0] = 4
}
```

## Удаление элемента из *slice*

1. Если порядок имеет значение, то нужно сделать *append* всех элементов слева и справа от удаляемого элемента:  

   ```go
   func remove(slice []int, s int) []int {
       return append(slice[:s], slice[s+1:]...)
   }
   ```

   Такой прием называется *re-slicing* и слишком дорог и неэффективен, т.к. в итоге приходится сдвинуть (скопировать) все весь хвост *slice*. Хотя этот прием является идеоматическим для Go.

2. Если порядок не важен, то можно сделать *swap* с последним элементов в *slice* и затем обрезать *slice* на 1 элемент:

   ```go
   func remove(s []int, i int) []int {
       s[i] = s[len(s)-1]
       return s[:len(s)-1]
   }
   ```

Удаление всех элементов поочереди из *slice* в 1 000 000 элементов в 1 случае – занимает 224 с, во втором — всего 0,06 нс.

Однако, нужно быть осторожными с этими простыми примерами, потому что они изменяют *underying array*, а значит влияют и на исходный *slice*:

```go
func remove(slice []int, s int) []int {
	return append(slice[:s], slice[s+1:]...)
}

func main() {
	original := []int{1, 2, 3}

	result := remove(original, 1)

	fmt.Println(result)   // [1 3]
	fmt.Println(original) // [1 3 3]
}
```

  

3. Еще один вариант – использовать функцию `slices.Delete()`.[]()