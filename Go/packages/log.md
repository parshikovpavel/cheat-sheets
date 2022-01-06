# `log`

*Package* `log`  реализует функции логгирования. 

*Package* определяет *type* `Logger` с методами для форматированного вывода в лог. Он также имеет предопределенный *'standard' Logger*, доступный через *helper function*'s `Print[f|ln]`, `Fatal[f|ln]` и `Panic[f|ln]`, которые проще в использовании, чем создание `Logger` вручную. Этот `Logger` пишет в *standart error* и печатает дату и время каждого сообщения. Каждое сообщение выводится в отдельной строке: если сообщение не заканчивается *newline*, *logger* добавит ее. 

Функции `FatalX` вызывают `os.Exit(1)` после вывода сообщения. Функции `PanicX` вызывают *panic* после вывода сообщения.

# Структура

## `Fatal()`

```go
func Fatal (v ... interface {})
```

`log.Fatal()` эквивалентен `log.Print()` + вызов `os.Exit (1)`.

## `Fatalf()`

```go
func Fatalf(format string, v ...interface{})
```

`log.Fatalf()` эквивалентен `log.Printf()` + вызов `os.Exit (1)`.

## `Print()`

```go
func Print(v ...interface{})
```

`log.Print()` вызывает `log.Output` для вывода в *standard logger*'е. Аргументы обрабатываются аналогично `fmt.Print()`.

## `Println()`

```go
func Println(v ...interface{})
```

`Println()` вызывает `log.Output` для вывода в *standard logger*'е. Аргументы обрабатываются аналогично `fmt.Println()`.