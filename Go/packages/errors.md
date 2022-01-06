## `errors`

Смотреть аналог – [github.com/pkg/errors](). Но кажется стандартный `errors` покрывает уже все возможности этой библиотеки.

`errors` – это *package*

### `type error`

`error` – это *type*, встроенный *interface* тип для представления состояния ошибки, значение `nil` – отсутствие ошибки.

```go
type error interface {
    Error() string
}
```

В соответствии с соглашениями, принятыми в языке Go, значение ошибки (или успеха когда `error = nil`) типа `error` возвращается в последнем (или единственном) значении, возвращаемом функцией или методом.

```go
item, err := haystack.Pop()
```

### `New()`

```go
func New(text string) error
```

Возвращает ошибку с текстом `text`.

Часто вместе `errors.New()` полезно использовать `fmt.Errorf()` ([1](#fmterrorf))  или есть такая же функция в пакете [github.com/pkg/errors]()

```go
func ... (int, error) {
    if ... {
        return nil, errors.New("...")
    }
    ...
    return x, nil
}
```



## Wrap для error

https://itnext.io/golang-error-handling-best-practice-a36f47b0b94c

### Встроенный с Go 1.13

Используется функция `fmt.Errorf()`

Если *format specifier* включает `%w` и *error* в качестве соответствующего операнда, то возвращаемая *error* будет реализовывать метод `Unwrap()`, возвращающий исходный операнд. Нельзя включать более одного `%w` или в качестве операнда использовать НЕ *error*. 

```go
import (
   "database/sql"
   "errors"
   "fmt"
)

func bar() error {
   if err := foo(); err != nil {
      return fmt.Errorf("bar failed: %w", foo())
   }
   return nil
}

func foo() error {
   return fmt.Errorf("foo failed: %w", sql.ErrNoRows)
}

func main() {
   err := bar()
   if errors.Is(err, sql.ErrNoRows) {
      fmt.Printf("data not found,  %+v\n", err)
      return
   }
   if err != nil {
      // unknown error
   }
}
/* Outputs:
data not found,  bar failed: foo failed: sql: no rows in result set
*/
```

Недостатки:

- не поддерживает вывод *call stack*

