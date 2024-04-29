https://yourbasic.org/golang/how-to-sort-in-go/

# `sort`

`sort` *package* предоставляет конструкции для сортировки *slice*'s  и *user-defined collection*'s.

## Примеры

### Пример 1

```go
package main

import (
	"fmt"
	"sort"
)

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%s: %d", p.Name, p.Age)
}

// ByAge реализует sort.Interface для []Person на основе Age field
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
	people := []Person{
		{"Bob", 31},
		{"John", 42},
		{"Michael", 17},
		{"Jenny", 26},
	}

  // Существует два способа отсортировать slice. Во-первых, можно определить
  // набор method'ов для slice type, как для ByAge, и
  // вызвать `sort.Sort() ``
	sort.Sort(ByAge(people))

  // Другой способ - использовать `sort.Slice()` с указанием функции Less, 
  // которая может быть задана как closure. В этом
  // случае никакие method'ы не нужны (и если они существуют, то они
  // игнорируются) 
  sort.Slice(people, func(i, j int) bool {
		return people[i].Age > people[j].Age
	})
}
```



TODO!!!



## `Int()`

```go
func Ints(x []int)
```

`Ints()` сортирует *slice* из *int*'ов в восходящем порядке.

## `Slice()`

```go
func Slice(x interface{}, less func(i, j int) bool)
```

`Slice()` сортирует *slice* `x` с учетом предоставленной функции `less()`. 

Не гарантируется, что сортировка будет *stable*: эквивалентные элементы могут быть переставлена в другом порядке. Для *stable* сортировки используйте `SliceStable()`.

Функция `less()` должна удовлетворять тем же требованиям, что и метод `Interface.Less()`.

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	people := []struct {
		Name string
		Age  int
	}{
		{"Gopher", 7},
		{"Alice", 55},
		{"Vera", 24},
		{"Bob", 75},
	}
	sort.Slice(people, func(i, j int) bool { return people[i].Name < people[j].Name })
	fmt.Println("By name:", people)

	sort.Slice(people, func(i, j int) bool { return people[i].Age < people[j].Age })
	fmt.Println("By age:", people)
}

```

## `Sort()`

`Sort()` сортирует данные в порядке возрастания, в соответствии с тем как это определено методом `data.Less()`. Функция `Sort()` делает один вызов метода `data.Len()` для определения размера данных `n` и `O(n*log(n))` вызовов методов `data.Less()` и `data.Swap()`. Стабильность сортировки не гарантируется.

## `type Interface`

```go
type Interface interface {
  // Len — количество элементов в collection.
	Len() int

  // Less сообщает, должен ли элемент с индексом i
	// находиться перед элементом с индексом j. 
	// 
	// Если и Less(i, j), и Less(j, i) – false, 
	// тогда элементы с индексами i и j считаются equal. 
  // Sort() может размещать equal element'ы в любом порядке в конечном результате, 
  // в то время как Stable() сохраняет исходный input order для equal element'ов. 
	// 
  // Less() должен описывать транзитивный порядок: 
	// - если Less(i, j) и Less(j, k) – true , то Less(i, k) также должен быть true. 
	// - если Less(i, j) и Less(j, k) – false, то Less(i, k) также должен быть false. 
	Less(i, j int) bool

  // Swap() swap'ает element'ы c index'ами i и j
	Swap(i, j int)
}
```



ЗАКОНЧИЛ ТУТ!!!