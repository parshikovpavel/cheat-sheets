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
	// Len is the number of elements in the collection.
	Len() int

	// Less reports whether the element with index i
	// must sort before the element with index j.
	//
	// If both Less(i, j) and Less(j, i) are false,
	// then the elements at index i and j are considered equal.
	// Sort may place equal elements in any order in the final result,
	// while Stable preserves the original input order of equal elements.
	//
	// Less must describe a transitive ordering:
	//  - if both Less(i, j) and Less(j, k) are true, then Less(i, k) must be true as well.
	//  - if both Less(i, j) and Less(j, k) are false, then Less(i, k) must be false as well.
	//
	// Note that floating-point comparison (the < operator on float32 or float64 values)
	// is not a transitive ordering when not-a-number (NaN) values are involved.
	// See Float64Slice.Less for a correct implementation for floating-point values.
	Less(i, j int) bool

	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```

An implementation of Interface can be sorted by the routines in this package. The methods refer to elements of the underlying collection by integer index.

ЗАКОНЧИЛ ТУТ!!!