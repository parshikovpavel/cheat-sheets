`slice` – наиболее часто используемая коллекция. 

`slice` (срез) – это дескриптор для непрерывного сегмента *underlying* `array` (базового массива), обеспечивающий доступ к пронумерованной последовательности элементов из этого *array*. Т.е. *slice* – это такой тип, который может хранить все возможные срезы *array* с элементами указанного *type*. 

Slice, *под капотом*, представляет собой контейнер (*slice header*), содержащий:

- указатель на данные (внутри *underlying array*)
- length (длину)
- capacity (емкость)

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

#### Реализация *stack* (стек)

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

 



### Общее

#### Обращение к элементам

Аналогично другим языкам, получение n-го элемента среза:

```go
slice[n]
```



## Адреса

- `&slice` – адрес *slice header*
- `&slice[0]` – адрес самих данных (в *underlying array*), который совпадает с адресом первого элемента



# Задачи

## Задача 1

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

