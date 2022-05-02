# `squirrel`

https://pkg.go.dev/github.com/Masterminds/squirrel

github.com/Masterminds/squirrel

## Подключение

```go
import "github.com/Masterminds/squirrel"
```

`package squirrel` помогает билдить *SQL query* из составных частей.

## Примеры

```go
import sq "github.com/Masterminds/squirrel"

users := sq.Select("*").From("users").InnerJoin("emails USING (email_id)")

active := users.Where(sq.Eq{"deleted_at": nil})

sql, args, err := active.ToSql()

sql == "SELECT * FROM users JOIN emails USING (email_id) WHERE deleted_at IS NULL"
```



# Структура

## Variable

```go
var (
	// Question – это экземпляр типа PlaceholderFormat, который оставляет placeholder как есть – `?`
	Question = questionFormat{}

	// Dollar – это экземпляр типа PlaceholderFormat, который заменяет placeholder на placeholder c долларом (e.g. $1, $2, $3).
	Dollar = dollarFormat{}

	// Colon is a PlaceholderFormat instance that replaces placeholders with
	// colon-prefixed positional placeholders (e.g. :1, :2, :3).
	Colon = colonFormat{}

	// AtP is a PlaceholderFormat instance that replaces placeholders with
	// "@p"-prefixed positional placeholders (e.g. @p1, @p2, @p3).
	AtP = atpFormat{}
)
```





## Type

### `type Eq`

```go
type Eq map[string]interface{}
```

`Eq` используется для описания равенства `=`  в методах `Where()`/`Having()`/`Set()`.

Реализует `interface Sqlizer`

```go
Select("id", "created", "first_name").From("users").Where(Eq{
	"company": 20, // "company = 20"
})
```



### `type InsertBuilder`

```go
type InsertBuilder builder.Builder
```

`InsertBuilder()` билдит `INSERT` *statement*.

#### `Insert()`

```go
func Insert(into string) InsertBuilder
```

`Insert()` возвращает новый `InsertBuilder()` с указанным в `into` именем таблицы.

##### Примеры

Вставка нескольких *row*'s:

```go
sql, args, err := sq.
    Insert("users").Columns("name", "age").
    Values("moe", 13).Values("larry", sq.Expr("? + 5", 12)).
    ToSql()

sql == "INSERT INTO users (name,age) VALUES (?,?),(?,? + 5)"
```

Возврат `id`:

```go
query := sq.Insert("nodes").
    Columns("uuid", "type", "data").
    Values(node.Uuid, node.Type, node.Data).
    Suffix("RETURNING \"id\"").
    RunWith(m.db).
    PlaceholderFormat(sq.Dollar)

query.QueryRow().Scan(&node.id)
```



#### `Columns()`

```go
func (b InsertBuilder) Columns(columns ...string) InsertBuilder
```

`Columns()` добавляет в *query* – *column*'s.





#### `SetMap()`

```go
func (b InsertBuilder) SetMap(clauses map[string]interface{}) InsertBuilder
```

`SetMap()` устанавливает *column*'s и *value*'s для *insert builder* из `clauses` map c *column name* и *value*. Обратите внимание, что это сбросит все предыдущие *column*'s и *value*'s, если они были установлены.



#### `Values()`

```go
func (b InsertBuilder) Values(values ...interface{}) InsertBuilder
```

`Values()` добавляет в *query* – value's для одной *row*. Т.е. можно добавить несколько *row*'s, как в [примере](#примеры)



### `type PlaceholderFormat`

```go
type PlaceholderFormat interface {
	ReplacePlaceholders(sql string) (string, error)
}
```

`type PlaceholderFormat` — это *interface*, включающий метод `ReplacePlaceholders()`.

`ReplacePlaceholders()` берет *SQL statement* и заменяет каждый *placeholder* `?` на (возможно) другой *placeholder*.



### `type SelectBuilder`

```go
type SelectBuilder builder.Builder
```

`type SelectBuilder` билдит *SQL SELECT statement*.



#### `From()`

```go
func (b SelectBuilder) From(from string) SelectBuilder
```

`From()` устанавливает `FROM` *clause* для *query*.

Пример:

```go
Select("id", "created", "first_name").From("users")
```



#### `InnerJoin()`

```go
func (b SelectBuilder) InnerJoin(join string, rest ...interface{}) SelectBuilder
```

`InnerJoin()` добавляет `INNER JOIN` *clause* в *query*.

- `rest` – это то же самое что и `args` (значения *placeholder*'ов) (посмотрел в исходном коде)

Пример:

```go
users := sq.Select("*").From("users").Join("emails USING (email_id)")
```



#### `Limit()`

```go
func (b SelectBuilder) Limit(limit uint64) SelectBuilder
```

`Limit()` устанавливает `LIMIT` *clause* для *query*.



#### `OrderBy()`

```go
func (b SelectBuilder) OrderBy(orderBys ...string) SelectBuilder
```

`OrderBy()` добавляет `ORDER BY` *expression* в *query*. В эту функция передается:

- целая строка со описаниями всех сортировок
- slice из string, ??? конкатенируются через `,`

Пример:

```go
q := squirrel.Select("event.*, p.name as project_name").
		From("event").
		LeftJoin("project as p on event.project_id=p.id").
		OrderBy("created desc")
```

```go
OrderBy("o ASC", "p DESC")
```



#### `OrderByClause()`

```go
func (b SelectBuilder) OrderByClause(pred interface{}, args ...interface{}) SelectBuilder
```

`OrderByClause()` добавляет `ORDER BY` *clause* в *query*.

В эту функцию передается (аналогично [`WHERE`](#where)):

- *SQL expression* `pred`
- Если *expression* содержит *SQL placeholder*'ы, необходимо также передать набор аргументов `args`, по одному для каждого *placeholder*'а.

Пример:

```go
	b := Select("a", "b").
		From("e").
		OrderByClause("? DESC", 1).
		OrderBy("o ASC", "p DESC").
```





#### `PlaceholderFormat()`

```go
func (b SelectBuilder) PlaceholderFormat(f PlaceholderFormat) SelectBuilder
```

`PlaceholderFormat()` устанавливает формат *placeholder*'а (например, `sq.Question` или `sq.Dollar`) для *query*.



#### `Select()`

```go
func Select(columns ...string) SelectBuilder
```

`Select()` возвращает новый `type SelectBuilder`, опционально устанавливая *column*'s для результата.

Пример:

```go
Select("id", "created", "first_name").From("users") // ... continue building up your query

// можно использовать sql method в select column
Select("first_name", "count(*)").From("users")

// можно использовать column alias
Select("first_name", "count(*) as n_users").From("users")
```



#### `ToSql()`

```go
func (b SelectBuilder) ToSql() (string, []interface{}, error)
```

`ToSql()` билдит *query* в *SQL string* и связанные *args*.

Пример:

```go
var db *sql.DB

query := Select("id", "created", "first_name").From("users")

sql, args, err := query.ToSql()
if err != nil {
	log.Println(err)
	return
}

rows, err := db.Query(sql, args...)
if err != nil {
	log.Println(err)
	return
}

defer rows.Close()

for rows.Next() {
	// scan...
}
```



#### `Where()`

```go
func (b SelectBuilder) Where(pred interface{}, args ...interface{}) SelectBuilder
```

`Where()` добавляет *expression* в `WHERE` *clause* для *query*.

Отдельные *expression* объединяются с помощью `AND`  в результирующем SQL.

В качестве аргумента `pred` можно использовать:

- `nil` и `""` – игнорируются

- `string` – *SQL expression*. Если *expression* содержит *SQL placeholder*'ы, необходимо также передать набор аргументов `args`, по одному для каждого *placeholder*'а.

  ```go
  Select("id", "created", "first_name").From("users").Where("company = ?", companyId)
  ```

- `map[string]interface{}` или `type Eq` (`type Eq` является *defined type* для `map[string]interface{}`) — *map* для сопоставления *SQL expression* → *value*. Каждый *key* преобразуется в *expression* вида `<key> = ?`, с соответствующим *value*, привязанным к *placeholder*'у. Если `value == nil`, *expression* будет `<key> IS NULL`. Если *value* имеет тип *array* или *slice*, *expression* будет иметь вид `<key> IN (?,?,...)` с одним *placeholder*'ом для каждого элемента в *value*. Эти *expression*'ы объединяются с помощью `AND`.

`Where()` бросает *panic*, если `pred` не является ни одним из вышеперечисленных типов.

Примеры:

- использование разных типов условий и логических операторов:

  ```go
  Select("id", "created", "first_name").From("users").Where(Eq{
  	"company": companyId,
  })
  
  Select("id", "created", "first_name").From("users").Where(GtOrEq{
  	"created": time.Now().AddDate(0, 0, -7),
  })
  
  Select("id", "created", "first_name").From("users").Where(And{
  	GtOrEq{
  		"created": time.Now().AddDate(0, 0, -7),
  	},
  	Eq{
  		"company": companyId,
  	},
  })
  ```

- несколько вызовов `Where()` в цепочке:

  ```go
  Select("id", "created", "first_name").
  	From("users").
  	Where("company = ?", companyId).
  	Where(GtOrEq{
  		"created": time.Now().AddDate(0, 0, -7),
  	})
  ```




### `type UpdateBuilder`

```go
type UpdateBuilder builder.Builder
```

`type UpdateBuilder` билдит SQL `UPDATE` *statement*.



#### `Update()`

```go
func Update(table string) UpdateBuilder
```

`Update()` возвращает новый `UpdateBuilder` с заданным *table name*.



#### `Set()`

```go
func (b UpdateBuilder) Set(column string, value interface{}) UpdateBuilder
```

`Set()` добавляет `SET` *clause* в *query*.



#### `SetMap()`

```go
func (b UpdateBuilder) SetMap(clauses map[string]interface{}) UpdateBuilder
```

`SetMap()` — это удобный метод, который вызывает `Set()` для каждой пары *key/value* в `clauses`.



#### `Where()`

```go
func (b UpdateBuilder) Where(pred interface{}, args ...interface{}) UpdateBuilder
```

`Where()` добавляет `WHERE` *expression* в *query*.



# `github.com/lann/builder`

https://pkg.go.dev/github.com/lann/builder



## `type Builder`

```go
type Builder struct {
	// contains filtered or unexported fields
}
```

`type Builder` хранит *set* из *named value*'s (именованных значений).



