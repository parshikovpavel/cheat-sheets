https://yourbasic.org/golang/how-to-sort-in-go/

# `sort`

`sort` *package* предоставляет конструкции для сортировки *slice*'s  и определенных пользователей *collection*'s (????).

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

