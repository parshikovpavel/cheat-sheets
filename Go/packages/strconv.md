# `strconv`

*Package* `strconv` реализует преобразования базовых типов в и из строковых представлений

## Структура

### `FormatInt()`

```go
func FormatInt(i int64, base int) string
```

`FormatInt()` возвращает строковое представление `i` в данной системе счисления `base`, где 2 <= `base` <= 36. В результате используются строчные буквы от `a` до `z` для цифр > = 10.

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	v := int64(-42)

	s10 := strconv.FormatInt(v, 10)
	fmt.Printf("%T, %v\n", s10, s10) // string, -42

	s16 := strconv.FormatInt(v, 16)
	fmt.Printf("%T, %v\n", s16, s16) // string, -2a
}
```



### `Itoa()`

```go
func Itoa(i int) string
```

Преобразует `int` в `string`

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	i := 10
	s := strconv.Itoa(i)
	fmt.Printf("%T, %v\n", s, s) // string, 10
}
```

