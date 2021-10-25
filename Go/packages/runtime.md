*Package* `runtime` содержит операции, которые взаимодействуют с Go's *runtime system*, такие как функции управления *goroutine*'s. Он также включает (???) *low-level type information*, используемую `reflect` *package*.



# Структура

## `NumGoroutine()`

```go
func NumGoroutine() int
```

`NumGoroutine()` возвращает число *goroutine*'s, которые существуют в настоящий момент