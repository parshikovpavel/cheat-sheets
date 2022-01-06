# `ioutil`

*Package* `ioutil` реализует некоторые I/O утилиты.

Начиная с Go 1.16, та же функциональность реализуется *package* `io` и *package* `os`, и этим *package*'s следует отдавать предпочтение.



# Структура

## `ReadAll()`

```go
func ReadAll(r io.Reader) ([]byte, error)
```

Начиная с Go 1.16, эта функция просто вызывает `io.ReadAll()`.