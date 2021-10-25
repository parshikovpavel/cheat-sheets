# `https://github.com/lib/pq`

https://pkg.go.dev/github.com/lib/pq

https://github.com/lib/pq

*Package* с драйвером для PostgreSQL. 

Установка:

```go
go get github.com/lib/pq
```

После подключения *driver* мы сможем пользоваться всеми возможностями `database/sql` при его подключении к Postgres.

## Получение подробной ошибки PostgreSQL

`lib/pq` может возвращать ошибки типа `*pq.Error`. Это позволяет узнать конкретную ошибку от PostgreSQL. Для ошибки можно попробовать сделать *type assertion* ([link](../lang.md#type-assertion)) к типу `*pq.Error`. После это можно использовать поля и методы типа `pg.Error` ([link](TODO)):

```go
import (
	"database/sql"
	"github.com/lib/pq"
)

err = db.Exec("update users set name = ‘Valera’")

if err != nil {
  if err, ok := err.(*pq.Error); ok {
  	fmt.Println("pq error:", err.Code.Name())
    return
  }
  
  // какая-то другая обработка
  return
}

 else {
// ...
}

```



# Структура

## `type Error`

`type Error` представляет ошибку коммуникации с сервером.

```go
type Error struct {
	Severity         string
	Code             ErrorCode
	Message          string
	Detail           string
	Hint             string
	Position         string
	InternalPosition string
	InternalQuery    string
	Where            string
	Schema           string
	Table            string
	Column           string
	DataTypeName     string
	Constraint       string
	File             string
	Line             string
	Routine          string
}
```

## `type ErrorCode`

`type ErrorCode` – пятисимвольный код ошибки.

```go
type ErrorCode string
```

### `Name()`

```go
func (ec ErrorCode) Name() string
```

`Name()` возвращает понятное для человека описание *error code*. В документации PostgreSQL это называется [*condition name*](http://www.postgresql.org/docs/9.3/static/errcodes-appendix.html).