- http://jmoiron.github.io/sqlx/
- https://github.com/jmoiron/sqlx
- https://pkg.go.dev/github.com/jmoiron/sqlx

# `jmoiron/sqlx`

`database/sql` представляет слишком низкоуровневый интерфейс. И не очень хорошо подходит для повседневной работы, т.к. требует вручную сохранять поля из результата запроса в поля структур. Это неудобно в поддержке кода.

`jmoiron/sqlx` предоставляет более высокоуровневый интерфейс поверх `database/sql`.

Он позволяет:

- (туда) прокидывать структуры в запрос в качестве параметров
- (обратно) делать `Scan()` в структуру целиком (а не по отдельным полям)

## Установка

Вначале необходимо установить `sqlx` и *database driver* ([подробно про `lib/pq`](../github/lib_pg.md)). 

```go
$ go get github.com/jmoiron/sqlx
$ go get github.com/lib/pq
```

## Подключение к database

Выполняется аналогично [подключению к базе данных для `database/sql`](../packages/database_sql.md#подключение-к-database). 

Можно создать экземпляр `sqlx.DB`:

- через `sqlx.Open` (это *promoted method* `sql.Open()`): 

  ```go
  db, err := sql.Open("postgres", dsn)
  ```

- из существующего `sql.DB` с помощью `NewDb` (необходимо указать *driverName*):

  ```go
  db, err := sqlx.NewDb(sql.Open("postgres", dsn), "postgres")
  ```



Полный пример:

```go
package main

import (
	"github.com/jmoiron/sqlx"
	_ "github.com/lib/pq"
	"log"
)

func main() {
	dsn := "postgres://127.0.0.1:5432/test?sslmode=disable"
	db, err := sqlx.Open("postgres", dsn)
	if err != nil { log.Fatal(err) }

	defer db.Close()
	err = db.Ping()
	if err != nil {
		// do something here
	}
}
```



## Scannable type

*Type* является *scannable* (и значит к нему можно применить `sql.Row.Scan()` или `sql.Rows.Scan()`, если:

- он не является `struct`, например является `string` или `int`
- или он реализует `sql.Scanner`
- или это `struct` без *exported field*'s (например `time.Time`)

## Handle Types

Объекты `sqlx` оборачивают объекты из `database/sql`. Но ни один из методов `database/sql` не изменяется, все расширенное поведение реализуется в новых методах.

`sqlx` спроектирован, чтобы быть похожим на  `database/sql`. 

Существует 4 *handle types* (обрабатывающих типа):

- `sqlx.DB` – аналогично `sql.DB` (*database*)
- `sqlx.Tx` – аналогично `sql.Tx` (*transaction*)
- `sqlx.Stmt` – аналогично `sql.Stmt` (*prepared statement*)
- `sqlx.NamedStmt` – включает `sql.Stmt`, но расширяет возможности *prepared statement* для поддержки *named parameters*.

И 2 *cursor types*:

- `sqlx.Rows` – аналогично `sql.Rows` (возвращается из `Queryx()`)
- `sqlx.Row` – аналогично `sql.Row` (возвращается из `QueryRowx()`)

*Handle types* и *cursor types* – оборачивают аналоги из `database/sql`. Например, при вызове `sqlx.DB.Query` происходит вызов того же кода, что и при `sql.DB.Query`. 

## Запросы в БД

`sqlx` реализует следующие методы для выполнения запросов в БД:

- неизменные из `database/sql`:
  - `Exec(...) (sql.Result, error)`
  - `Query(...) (*sql.Rows, error)`
  - `QueryRow(...) *sql.Row` 
- расширения для методов из `database/sql`:
  - `MustExec(...) sql.Result` – `Exec()`, которые вызывает *panic* при *error*
  - `Queryx(...) (*sqlx.Rows, error)` – `Query()`, который возвращает `sqlx.Rows`
  - `QueryRowx(...) *sqlx.Row` – `QueryRow()`, который возвращает `sqlx.Row`

- новые методы (похожих нет в `database/sql`:
  - `Get(dest interface{}, ...) error`
  - `Select(dest interface{}, ...) error`



## *Scanning* в *struct*

`sqlx.Row` и `sqlx.Rows` реализуют метод [`StructScan()`](#structscan), с помощью которого можно сканировать *result* в *struct fields*. Также можно использовать удобные методы [`Get()`](#get) и [`Select()`](#select), реализующие `StructScan()` внутри себя.

*Struct fields* должны быть *exported*, чтобы `sqlx` мог писать в них. Можно использовать `db` *struct tag*, чтобы указать, какое *column name* мапится на конкретное *struct field*. Т.е. *struct tag* работает аналогично библиотеке `encoding/json`. Можно установить *mapping by default* с помощью метода `db.MapperFunc()`. По умолчанию, *column name* мапится на *field name*, к которому применен `strings.Lower()`.

Пример:

```go
type Place struct {
    Country       string
    City          sql.NullString
    TelephoneCode int `db:"telcode"`
}
 
rows, err := db.Queryx("SELECT * FROM place")
if err != nil { // ... }
  
for rows.Next() {
    var p Place
    err = rows.StructScan(&p)
    if err != nil { // ... }
}

err = rows.Err()
if err != nil {
  return
}
```

При сканировании можно в структуре использовать типы [`type Null*`](../packages/database_sql.md#типы-null):

```go
type User struct {
	FirstName	string			`db:"first_name"`
	LastName	sql.NullString	`db:"last_name"`
}

var user User
err := db.Get(&user, "select * from users where id = 1 limit 1")
if err != nil { // ... }

if user.LastName.Valid {
	fmt.Println(user.LastName.String)
} else {
	fmt.Println("last_name is null")
}
```





# Обработка значений `NULL`

Также как значения `NULL` обрабатываются в библиотеке `database/sql` ([link](../packages/database_sql.md#обработка-значений-null)), можно в структуру поместить:

-  поля с  `type Null*` . Недостаток этого подхода – бизнес-логика (структура) теперь привязана к `database/sql`, хотя структура сама по себе не связана с базой данный – в нее просто загружаются данные из БД.

  ```go
  type Product struct {
  	ProductNo     int `db:"product_no"`
	Name          sql.NullString
  }
  
  p := Product{}
  
  err = db.Get(&p, "SELECT product_no, name FROM products where product_no = $1", 1)
  if err != nil {  }
  
  fmt.Printf("%#v\n", p) // Product{ProductNo:1, Name:sql.NullString{String:"", Valid:false}}
  
  if p.Name.Valid {
  	fmt.Println(p.Name.String)
  } else {
  	fmt.Println("name is null")
  }
  
  ```
  
-  вместо *value* использовать *pointer*. Тогда для значений `NULL` этот *pointer* будет принимать значение `nil`. При этом описание структур не будет содержать никакого кода, привязанного к `database/sql`:

   ```go
   type Product struct {
   	ProductNo     int `db:"product_no"`
   	Name          *string
   }
   
   p := Product{}
   
   err = db.Get(&p, "SELECT product_no, name FROM products where product_no = $1", 1)
   if err != nil {  }
   
   fmt.Printf("%#v\n", p) // Product{ProductNo:1, Name:(*string)(nil)}
   
   if p.Name != nil {
   	fmt.Println(*p.Name)
   } else {
   	fmt.Println("name is null")
   }
   ```

   



# Структура



## `type DB`

```go
type DB struct {
	*sql.DB

	Mapper *reflectx.Mapper
	// contains filtered or unexported fields
}
```

`DB` - это обертка вокруг `sql.DB`.

### `Get()`

```go
func (db *DB) Get(dest interface{}, query string, args ...interface{}) error
```

`Get` выполняет следующие действия:

- `QueryRowx()` – для выбора одной *row*
- сканирует полученную *row* в `dest`:
  - если `dest` – имеет *scannable type*. Использует `Row.Scan()`. При этом *result set* должен иметь только один *column*.
  - если `dest` – имеет *non-scannable type*. Использует `Row.StructScan()`. 

*Placeholder*'s заменяются предоставленными аргументами `args`. 

Если *result set* пуст, то возвращается *error* `sql.ErrNoRows`, как для `row.Scan`

Пример:

```go
type Place struct {
    Country       string
    City          sql.NullString
    TelephoneCode int `db:"telcode"`
}

p := Place{}

// Пример non-scannable type
// Сканирует строку в структуру p
err = db.Get(&p, "SELECT * FROM place LIMIT 1")
if err != nil { // ... }
 
// Пример scannable type
// Сканирует количество строк (тип int) в переменнуюwell
var id int
err = db.Get(&id, "SELECT count(*) FROM place")
if err != nil { // ... }
```

### `QueryRowx()`

```go
func (db *DB) QueryRowx(query string, args ...interface{}) *Row
```

`QueryRowx()` – аналог [`sql.QueryRow`](../packages/database_sql.md#queryrow), который возвращает `*sqlx.Row`. `sqlx.Row` является расширением `sql.Row` и позволяет [scanning в struct](#scanning-в-struct). *Placeholder*'ы заменяются аргументами `args`.

### `Queryx()`

```go
func (db *DB) Queryx(query string, args ...interface{}) (*Rows, error)
```

`Queryx()` – аналог [`sql.Query`](../packages/database_sql.md#query), который возвращает `*sqlx.Rows`. `sqlx.Row` является расширением `sql.Row` и позволяет [scanning в struct](#scanning-в-struct). *Placeholder*'ы заменяются аргументами `args`.

### `Select()`

```go
func (db *DB) Select(dest interface{}, query string, args ...interface{}) error
```

`Select` выполняет следующие действия:

- `Queryx()` – для выбора множества *row*'s
- сканирует каждую *row* в `dest`, который должен иметь тип *slice*. 
  - если элементы этого `dest` *slice* – имеют *scannable type*. Использует `Row.Scan()`. При этом *result set* должен иметь только один *column*. 
  - если `dest` – имеет *non-scannable type*. Использует `Row.StructScan()`. 

`*sql.Rows` закрывается автоматически.

*Placeholder*'s заменяются предоставленными аргументами `args`. 

Пример:

```go
type Place struct {
    Country       string
    City          sql.NullString
    TelephoneCode int `db:"telcode"`
}

pp := []Place{}

// Пример non-scannable type
// Сканирует множество строк в slice pp
err = db.Select(&pp, "SELECT * FROM place WHERE telcode > ?", 50)
if err != nil { // ... }
 
// Пример scannable type
// извлекает 10 names с типом string
var names []string
err = db.Select(&names, "SELECT name FROM place LIMIT 10")
if err != nil { // ... }
```

`Select`, в отличии от `Queryx`, загружает в память сразу весь *result set*. Если этот *result set* очень большой, то лучше просто итерироваться по *result set* с применением `Queryx() + StructScan()`, в этом случае потребуется выделение памяти только для одной `Place`,  а не для целого `[]Place`.



## `type Row`

```go
type Row struct {
	Mapper *reflectx.Mapper
	// contains filtered or unexported fields
}
```

`type Row` – это дублирующая реализация `sql.Row` для получения доступа к данным `sql.Rows.Columns()`, необходимым для `StructScan()`.

### `StructScan()`

```go
func (r *Row) StructScan(dest interface{}) error
```

`StructScan()` похож на `sql.Row.Scan`, но сканирует одну `Row` в одну *struct* `dest`. Необходимо использовать для [*non-scannable type*'s](#scannable-types).

## `type Rows`

```go
type Rows struct {
	*sql.Rows

	Mapper *reflectx.Mapper
	// contains filtered or unexported fields
}
```

`type Rows` – это обертка над `sql.Rows`.

### `StructScan()`

```go
func (r *Rows) StructScan(dest interface{}) error
```

`StructScan()` похож на `sql.Rows.Scan`, но сканирует одну `Row` в одну *struct* `dest`. Да!!! одну *row*, т.к. необходимо итерироваться по *row*'s с помощью [`sql.Rows.Next()`](../packages/database_sql.md#next). Необходимо использовать для [*non-scannable type*'s](#scannable-types).

`StructScan()` необходимо использовать вместо `Select()`, когда загрузка памяти методом `Select()` может быть черезмерной (подробней в [`Select()`](#select))

 `Rows.StructScan()` кэширует сопоставление позиций *column*'s на *field*'s, поэтому небезопасно выполнять `StructScan()` для одного экземпляра `Rows` с разными *struct type*'s.

