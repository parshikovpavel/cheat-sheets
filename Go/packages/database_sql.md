# `database/sql`

 https://pkg.go.dev/database/sql@go1.17

*Package* `sql` предоставляет общий высокоуровневый интерфейс для *SQL database*.

*Package* `sql` необходимо использовать вместе с *database driver*. Драйвер – это отдельный *package*. Драйвер необходим для того, чтобы через `database/sql` и его расширения подключаться к БД. Полный список *database driver*'s – https://golang.org/s/sqldrivers.



Драйверы, не поддерживающие отмену контекста, не вернутся до тех пор, пока запрос не будет завершен.

Доки:

- примеры использования https://github.com/golang/go/wiki/SQLInterface
- Хороший tutorial http://go-database-sql.org/index.html

# Примеры кода

## Подключение к database

```go
package main

import (
	"database/sql"
	_ "github.com/lib/pq"
	"log"
)

func main() {
	dsn := "postgres://127.0.0.1:5432/test?sslmode=disable"
	db, err := sql.Open("postgres", dsn)
	if err != nil { log.Fatal(err) }

	defer db.Close()
	err = db.Ping()
	if err != nil {
		// do something here
	}
}
```



- `_ "github.com/lib/pq"` – подключение *database driver* ([подробно про `lib/pq`](../github/lib_pg.md)). Чтобы *driver* работал – его необходимо импортировать. Т.к. мы не собираемся использовать ни одну функцию напрямую из `github.com/lib/pq`, то мы его импортируем с алиасом `_`.

  Все *database driver* регистрируют себя в *package* `database/sql` в `init()` функции. В первый раз, когда импортируется `github.com/lib/pq`, в нем вызывается функция вида:

  ```go
  // где-то в github.com/lib/pq
  
  func init() {
  	sql.Register("postgres", &Driver{})
  }
  ```

   которая указывает, что при запросах к `driverName = "postgres"`, должен использоваться инстанс `&Driver{}`. 

  В итоге, единожды импортировав *database driver*, мы можем использовать его по всей программе.

- формат DSN:

  ```
  протокол://хост:порт/база
  postgres://127.0.0.1:5432/test?sslmode=disable
  ```

- `sql.Open()` ([link](#open))

- `db.Close()` ([link](#close))

- [`db.Ping()`](#ping) – необходимо для того, чтобы установить соединение и проверить, что оно работает. Т.к. `sql.Open` не устанавливается соединение к *database*.

## Выборки из базы

### Выбор множества строк

```go
// здесь код из раздела #подключение-к-databasen

var id int
var name string


rows, err := db.Query("select product_no, name from products where product_no = $1", 1)
if err != nil {
  return
}

defer rows.Close()

for rows.Next() {
  err := rows.Scan(&id, &name)
  if err != nil {
    return
  }

  log.Println(id, name)
}

err = rows.Err()
if err != nil {
  return
}
```

- `DB.Query` ([link](#query))
- `Rows.Close` ([link](#close)) – *rows* надо не забывать закрывать
- Чтобы проверить, что при выборе следующей строки не произошла ошибка (а не просто закончился цикл при возврате `false` методом `Rows.Next()`), необходимо использовать метод `Err()` ([тут подробней](#next))

### Выбор одной строки

```go
var name string

err = db.QueryRow("select name from users where id = $1", 1).Scan(&name)
if err != nil {
	if err == sql.ErrNoRows {
		fmt.Println("user not found")
	} else {
		// ...
	}
}

fmt.Println(name)
```

### Получение детального описания ошибки

Способ получения человеко-понятного описания ошибки PostgreSQL ([link](../github/lib_pg.md#получение-подробной-ошибки-postgresql))

### Транзакция

```go
tx, err = db.Begin()
if err != nil { // ... }
defer tx.Rollback()

result, err := tx.Exec("update users set name = ‘Vasya’")
if err != nil { // ... }

id, err := result.LastInsertId() // 0, nil
affected, err := result.RowsAffected() // 1, nil

err = tx.Commit()
if err != nil { // ... }
```

- `defer tx.Rollback()` – удобнее всего закрывать *transaction* с помощью отложенного вызова. Проблемы с `Tx.Rollback()` в `defer` – нет, т.к. даже если до этого был `Tx.Commit()`, то `Tx.Rollback()` не будет иметь никакого эффекта. 





# Prepared statement

На *database level* –  *prepared statement* привязан к одному соединению с *database*. Типичный *flow* состоит в том, что клиент отправляет *SQL statement* с *placeholder*'s на сервер для подготовки, сервер возвращает *statement ID*, а затем *client* выполняет *statement*, отправляя его ID и *parameters*.

TODO!!! http://go-database-sql.org/prepared.html

*Parameter placeholder* в *prepared statement* различаются в зависимости от используемой *database* и *database driver*:

```
MySQL               PostgreSQL            Oracle
=====               ==========            ======
WHERE col = ?       WHERE col = $1        WHERE col = :col
VALUES(?, ?, ?)     VALUES($1, $2, $3)    VALUES(:val1, :val2, :val3)
```

`https://github.com/lib/pq` использует Postgres-native порядковые *placeholder*. Один и тот же *placeholder* можно использовать несколько раз, подставляя в запрос один и тот же *parameter*:





# Обработка значений `NULL`

В БД поля могут принимать значение `null`. 

Если такие поля сканировать напрямую в переменную, которая *non-pointer*, то будет выдана ошибка:

```go
	var name string


	err = db.QueryRow("select name from products where product_no = $1", 1).Scan(&name)

	fmt.Printf("%#v\n", name) // ""
	
	fmt.Printf("%#v\n", err) // &errors.errorString{s:"sql: Scan error on column index 0, name \"name\": 
                           // converting NULL to string is unsupported"}


```

Поэтому для сканирования таких полей (???? а также для записи значения `null` в базу данных) необходимо:

- В *package* `database/sql`  для таких значений есть набор типов `type Null*` (например, [`NullString`](#type-nullstring)). 

  `NullString` описывает `string`, которая может быть `null`. `NullString` реализует *interface* `Scanner`, поэтому его можно использовать в методе `Scan()`.

  - Если необходимо проверить значение на `NULL`, то следует делать так:

    ```go
    var s sql.NullString
    err := db.QueryRow("SELECT name FROM foo WHERE id=?", id).Scan(&s)
    ...
    if s.Valid {
       // use s.String
    } else {
       // NULL value
    }
    ```

  - Если различать `null` и пустое значение `""` не нужно, то можно просто обращаться к значению  `product.FirstName.String`:

    ```go
    var name sql.NullString
    
    err = db.QueryRow("select name from products where product_no = $1", 1).Scan(&name)
    
    fmt.Printf("%#v\n", name) // sql.NullString{String:"", Valid:false}
    fmt.Printf("%#v\n", name.String) // ""
    fmt.Printf("%#v\n", err) // <nil>
    ```

- Другой вариант, вместо *value* использовать *pointer*. Тогда для значений `NULL` этот *pointer* будет принимать значение `nil`:

  ```go
  	var name *string
  
  	err = db.QueryRow("select name from products where product_no = $1", 1).Scan(&name) // значение NULL
  	fmt.Printf("%#v\n", name) // (*string)(nil)
  
  	err = db.QueryRow("select name from products where product_no = $1", 2).Scan(&name) // значение "abc"
  	fmt.Printf("%#v\n", name) // (*string)(0xc0000133a0)
  	fmt.Printf("%#v\n", *name) // "abc"
  
  ```

- Если различать `null` и пустое значение `""` не нужно, то можно создать собственный *type* с *underlying type* `string`, который будет реализовывать *interface* `Scannable()` и нужную логику обработки значения:

  ```go
  type myString string
  
  func (s *myString) Scan(src interface{}) error {
      if src == nil {
          return nil
      }
      if value, ok := src.(string); ok {
          *s = value
          return nil
      }
      return fmt.Errorf("Unsupported type for myString: %T", src)
  }
  ```

  

  



# Структура

## Variables

### `ErrNoRows`

```go
var ErrNoRows = errors.New("sql: no rows in result set")
```

`ErrNoRows` возвращает метод `Scan()`, когда `QueryRow()` не выбрало никакой строки. Т.е. `QueryRow()` возвращает такое значение `*Row`, которое возвращает `ErrNoRows` при вызове метода `Scan()`.



## `Open()`

```go
func Open(driverName, dataSourceName string) (*DB, error)
```

`Open()` открывает базу данных, указанную ее *driver Name* и *dataSource Name*, обычно состоящего *database name* и *connection information*.

`Open()` может просто валидировать корректность аргументов, не создавая подключения к *database*. Чтобы проверить валидность *data source name*, необходимо вызвать `DB.Ping()`.

Возвращенный `*DB` безопасен для конкурентного использования несколькими *goroutine*'s и поддерживает собственный *pool* незанятых соединений (*idle connections*). Таким образом, функцию `Open()` нужно вызывать только один раз. `Close()` делать обычно не требуется (????).

## `type DB`

```go
type DB struct {
	// contains filtered or unexported fields
}
```

`DB` – это *database handle*, представляющий *pool* из нуля или более базовых *connection*'s. Он безопасен для конкурентного использования несколькими *goroutine*'s.

Т.е. экземпляр `DB` – не является соединением к *database*, это абстракция для доступа к *database* с *connection pool*'ом внутри. Поэтому создание экземпляра `DB` не выполняет подключение к *database*, не возвращает *error* (??? `sql.Open()` возвращает *error*) и не вызывает *panic*.  

`DB` автоматически создает и освобождает *connection*'s; он также поддерживает свободный *pool* из *idle connection*'s. Поэтому `DB` выполнит подключение к *database* только тогда, когда *connection* потребуется в первый раз.

 Если в базе данных есть понятие состояния каждого соединения, такое состояние можно надежно наблюдать в *transaction* (`type Tx`) или соединении (`type Conn`). После вызова `DB.Begin()` возвращаемый `Tx` привязан к одному соединению. Как только для *transaction* вызывается `Commit()`  или `Rollback()`, соединение этой *transaction* возвращается в *pool* незанятых соединений БД. Размер пула можно контролировать с помощью `SetMaxIdleConns`.

### `Begin()`

```go
func (db *DB) Begin() (*Tx, error)
```

`Begin()` стартует *transaction*. *Isolation level* по умолчанию зависит от драйвера.

`Begin()` использует `context.Background()` под капотом. Чтобы указать контекст, используйте `BeginTx()`.

Возвращает *transaction* `*Tx`.

### `Close()`

```go
func (db *DB) Close() error
```

`Close()` закрывает (??? что закрывает?) *database* и предотвращает запуск новых запросов. Затем `Close()` ожидает завершения всех запросов, которые начали обрабатываться на сервере. `Close()` используется редко, поскольку *database handle* должен быть долговечным и совместно использоваться многими *goroutine*'s.

### `DB.Exec()`

```go
func (db *DB) Exec(query string, args ...interface{}) (Result, error)
```

`Exec()` выполняет *query* и не возвращает *row*'s. Полезно для команд вроде `INSERT` и `UPDATE`. Аргументы предназначены для любых параметров-заполнителей в запросе.

- `args` используются для *prepared statement*. Это *parameter*'s для *placeholder*'s внутри текста *query*. (аналогично [`Query()`](#query))

### `Ping()`

```go
func (db *DB) Ping() error
```

`Ping()` проверяет, что соединение с базой данных все еще *alive*, и при необходимости устанавливает соединение.



### `Query()`

```go
func (db *DB) Query(query string, args ...interface{}) (*Rows, error)
```

`Query()` выполняет запрос (*query*) (обычно `SELECT`), который возвращает *row*'s. 

- `args` используются для *prepared statement*. Это *parameter*'s для *placeholder*'s внутри текста *query*. Могут быть нюансы при работе с *pgbouncer*.



### `QueryRow()`

```go
func (db *DB) QueryRow(query string, args ...interface{}) *Row
```

`QueryRow()` выполняет *query* и возвращает:

- единственную *row*
- первую *row*, если таких *row*'s несколько. Остальные *row*'s отбрасываются.

`QueryRow()` никогда не возвращает `nil`. Если произошла *error*, то она потом возвращается методом `Scan()`, который будет вызван для `*Row`. Например, если в результате *query* не были выбраны *row*'s, то вызов `Row.Scan()` вернет `ErrNoRows`.





## `type NullString`

```go
type NullString struct {
	String string
	Valid  bool // Valid is true if String is not NULL
}
```

`NullString` описывает `string`, которая может быть `null`. `NullString` реализует *interface* `Scanner`, поэтому его можно использовать в методе `Scan()`.

Пример использования [здесь](#обработка-значений-null).



## `type Result` 

```go
type Result interface {
	LastInsertId() (int64, error)
	RowsAffected() (int64, error)
}
```

### `LastInsertId()`

```go
LastInsertId() (int64, error)
```

`LastInsertId()` возвращает `integer`, сгенерированное базой данных в ответ на команду. Обычно это будет значение из автоинкрементного *column* при вставке новой *row*. Не все базы данных поддерживают эту функцию, и синтаксис таких *statement*'s различен.

В PostgreSQL эту информацию можно получить с помощью `RETURNING` *clause*.

### `RowsAffected()`

```go
RowsAffected() (int64, error)
```

`RowsAffected()` возвращает количество *row*'s, затронутых командами `UPDATE`, `INSERT` или `DELETE`. Не каждая БД или драйвер БД могут поддерживать это.







## `type Row`

```go
type Row struct {
	// contains filtered or unexported fields
}
```

`Row` – это результат вызова метода `QueryRow()` для выбора одной *row*.

### `Row.Scan()`

```go
func (r *Row) Scan(dest ...interface{}) error
```

`Scan()` копирует *column*'s из сматченной *query* в переменные, на которые указывают *pointer*'ы *dest*. Подробно смотрите документацию по `Rows.Scan()` ([link](#rows-scan)). Если на *query* матчится несколько *row*'s, то `Scan()` использует первую *row* и отбрасывает остальные. Если ни одна *row* не матчится на *query*, то `Scan()` возвращает `ErrNoRows`.



## `type Rows`

```go
type Rows struct {
	// contains filtered or unexported fields
}
```

`Rows` – это результат *query*. Его *cursor* начинается перед первой *row* полученного *result set*. 

### `Close()`

```go
func (rs *Rows) Close() error
```

`Close()` закрывает `Rows`, предотвращая дальнейший перебор. Если `Next()` вызывается и возвращает `false` и больше нет других *result set*'s, `Rows` закрывается автоматически и достаточно проверить результат `Err()`. `Close()` идемпотентен и не влияет на результат `Err()`.



### `Err()`

```go
func (rs *Rows) Err() error
```

`Err()` возвращает *error*, если она случилась во время итерирования *row*'s (??? методом `Next()`). `Err()` можно вызывать после явного или неявного `CLose()`.



### `Next()`

```go
func (rs *Rows) Next() bool
```

`Next()` подготавливает следующую *row* из *result set* для чтения с использованием метода `Scan()`. 

Возвращаемое значение:

- `true` – успех
- `false` – следующая *row* в *result set* отсутствует или при ее подготовке произошла ошибка. Чтобы различит два этих *case* необходимо использовать метод `Err()`.

Каждому вызову `Scan()`, даже первому, должен предшествовать вызов `Next()`.

После вызова `Next()` необходимо вызвать `Err()`, чтобы убедиться, что цикл прошел до конца, все строки были выбраны и ошибок не было.

### `Rows.Scan()`

```go
func (rs *Rows) Scan(dest ...interface{}) error
```

`Scan()` копирует *column*'s (??? наверное *field*'s) в текущей строке в переменные, *pointer*'ы на которые передаются в `dest`. Количество значений в `dest` должно быть таким же, как и количество *column*'s в `Rows`.

Возвращаемое значение:

- `error` – ошибка, если что-то пошло не так. Например, отвалилось соединение (??? точно, вроде ж значения уже прочитаны).

`Scan()` конвертирует *column*'s, считанные из базы данных, в следующие общие *Go type*'s и специальные *type*'s, предоставляемые пакетом `database/sql`:

```
*string
*[]byte
*int, *int8, *int16, *int32, *int64
*uint, *uint8, *uint16, *uint32, *uint64
*bool
*float32, *float64
*interface{}
*RawBytes
*Rows (cursor value)
и type реализующий Scanner (see Scanner docs)
```

В самом простом случае, если *value type* `T` из исходного *column* – `integer`, `bool` или `string`, а `dest` имеет *type* `*T`, `Scan()` просто присваивает значение по указанному *pointer*'у.

`Scan()` также конвертирует между `string` и *numeric type*'s, если информация не будет потеряна:

- преобразует значения из *numeric column*'s в `*string`
- преобразует значения в *numeric type*'s, если не произойдет *overflow* (переполнение). Например, `float64(300)` или `string("300")`могут быть преобразованы в `uint16`, но не в `uint8`. При этом `float64 (255)` или `string("255")` могут сканировать в `uint8` . Единственным исключением является то, что сканирование некоторых чисел `float64` в `string` может потерять часть информации при преобразовании. Рекомендуется, сканировать *floating point column* сканировать в `*float64`.

Значения типа `time.Time` можно сканировать в значения типа `*time.Time`, `*interface{}`, `*string` или `*[]byte`. 

Значения типа `bool` можно сканировать в `*bool`, `*interface{}`, `*string`, `*[]byte` или `*RawBytes`.

При сканировании в `*bool` источником может быть `true`, `false`, `1`, `0` или `string`, которые парсятся с помощью `strconv.ParseBool`.

## `type Scanner`

```go
type Scanner interface {
	Scan(src interface{}) error
}
```

`Scanner` – это *interface*, используемый `Row.Scan()`, `Rows.Scan()`,....

### `Scan()`

```go
Scan(src interface{}) error
```

`Scan()` должен сохранить значение, полученное из базы данных.

`src` будет иметь один из следующих типов: 

- `int64`
- `float64`
- `bool`
- `[]byte`
- `string`
- `time.Time`
- `nil` - для значений `NULL`

Должна быть возвращена *error*, если значение не может быть сохранено без потери информации. 

*Reference type*'s (`[]byte`) действительны в момент вызова и не должны сохраняться без копирования. Underlying значение может быть изменено драйвером. Если значение необходимо сохранить – скопируйте его. 









## `type Tx`

```go
type Tx struct {
	// contains filtered or unexported fields
}
```

`Tx` – это *database transaction* в прогрессе.

*Transaction* должна заканчиваться вызом `Commit()` или `Rollback()`.

`type Tx` предоставляет методы – аналогичные методам `type DB` (например, `Tx.Exec()`)

### `Commit()`

```go
func (tx *Tx) Commit() error
```

`Commit()` коммитит *transaction*.

### `Tx.Exec()`

```go
func (tx *Tx) Exec(query string, args ...interface{}) (Result, error)
```

Аналогично [`DB.Exec()`](#db-exec) только внутри *transaction*.





### `Rollback()`

```go
func (tx *Tx) Rollback() error
```

`Rollback()` прерывает *transaction*.

