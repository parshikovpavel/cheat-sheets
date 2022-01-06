# `math/rand`

В *package* `rand` реализованы *pseudo-random number generator*'s, неподходящие для *security-sensitive* работы.

*Random number*'s генерируются *Source*. Функции верхнего уровня, такие как `Float64()` и `Int()`, используют общий *default Source*, который создает детерминированную последовательность значений при каждом запуске программы. Используйте функцию `Seed()` для инициализации *default Source*, если для каждого запуска требуется различное поведение. *Default Source* безопасен для конкурентного использования несколькими *goroutine*'s, а *Source*, созданный через `NewSource()` – нет.

Сгенерированные этим *package* значения можно легко предсказать, независимо от того, как был сделан `Seed()`. *Random number*'s, подходящие для *security-sensitive* работы, см. `crypto/rand` *package*.

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

