# `math`

*Package* `math` предоставляет базовые *constant*'s и математические *function*'s.

## Структура

### `Pow()`

```go
func Pow(x, y float64) float64
```

`Pow()` возвращает `x` в степени `y`.

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	c := math.Pow(2, 3)
	fmt.Printf("%.1f", c)  // 8.0
}
```

В Go нет оператора возведения в степень (вроде `**` или `^`). `Pow()` реализует вычисления для `float64`. 

Для *integer* необходимо:

1. либо делать *conversion* из `int` во `float64` и использовать `math.Pow()`

   ```go
   // MathPow calculates n to the mth power with the math.Pow() function
   func MathPow(n, m int) int {
     return int(math.Pow(float64(n), float64(m)))
   }
   ```

2. либо реализовать метод самостоятельно:

   ```go
   // IntPow calculates n to the mth power. Since the result is an int, it is assumed that m is a positive power
   func IntPow(n, m int) int {
     if m == 0 {
       return 1
     }
     result := n
     for i := 2; i <= m; i++ {
       result *= n
     }
     return result
   }
   ```

По бенчмаркам функция, подобная `IntPow()` работает в 4 раза быстрее на одном ядре ЦП по сравнению с использованием `math.Pow`.



  

  

### `Round()`

```go
func Round(x float64) float64
```

`Round()` возвращает ближайшее целое число, округленное на половину от нуля (мое: т.е. как будто к числу прибавили 0,5 и отбросили дробную часть).

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	p := math.Round(10.5)
	fmt.Printf("%.1f\n", p)  // 11.0

	n := math.Round(-10.5)
	fmt.Printf("%.1f\n", n)  // -11.0
}

```

Функция возвращает значение с типом `float64`.

Если необходим `int`, то можно:

```go
int(f+0.5)          // Вариант 1
int(math.Round(f))  // Вариант 2
```

