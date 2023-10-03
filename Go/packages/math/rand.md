# `math/rand`

В *package* `rand` реализованы *pseudo-random number generator*'s, неподходящие для *security-sensitive* работы.

*Random number*'s генерируются  [Source](https://pkg.go.dev/math/rand#Source), обычно обернутым в [Rand](https://pkg.go.dev/math/rand#Rand). Оба типа должны использоваться одной *goroutine* одновременно: совместное использование нескольких *goroutine*'s требует некоторой синхронизации.

Функции верхнего уровня, такие как [`Float64()`](https://pkg.go.dev/math/rand#Float64) и [`Int()`](https://pkg.go.dev/math/rand#Int) , безопасны для одновременного использования несколькими *goroutine*'s.

Сгенерированные этим *package* значения можно легко предсказать, независимо от того, как был сделан `Seed()`. *Random number*'s, подходящие для *security-sensitive* работы, см. `crypto/rand` *package*.

Пример:

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	answers := []string{
		"It is certain",
    ....
	}
	fmt.Println("Magic 8-Ball says:", answers[rand.Intn(len(answers))])
}

```



# Структура

## `Int()`

```go
func Int() int
```

`Int()` возвращает неотрицательное псевдослучайное *int* из *default Source*.





## `Int63()`

```go
func Int63() int64
```

`Int63()` возвращает неотрицательное *63-bit integer* в виде типа `int64` из *default Source*.





## `Intn()`

```go
func Intn(n int) int
```

`Intn()` возвращает в виде типа `int` – неотрицательное псевдослучайное число в полуоткрытом интервале [0, n) из *default Source*. 

