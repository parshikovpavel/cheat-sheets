# `strings`

Package реализует функции для изменения UTF-8 строк



## Function

### `ReplaceAll()`

```go
func ReplaceAll(s, old, new string) string
```

`ReplaceAll()` возвращает копию *string* `s` со всеми непересекающимися экземплярами строки `old`, замененными на `new` (т.е. не перепроверяет на сопоставление те куски, которые уже заменены). Если `old` пусто, `new` (???) совпадает с началом *string* и местом после каждого символа UTF-8, что дает до `k+1` замен для *string* из k-*rune*.

Пример:

```go
fmt.Println(strings.ReplaceAll("oink oink oink", "oink", "moo")) // moo moo moo
```





### `Split()`

```go
func Split(s, sep string) []string
```

Разбивает `s` на все *substring*'s, разделенные `sep`, и возвращает map из substring's между разделителями `sep`.

Если `s` не содержит `sep`, а `sep` не пуст, `Split()` возвращает  *map* длиной 1, единственным элементом которого является `s`.

```go
str := "hi, this is, educative" 
split := strings.Split(str, ",")
fmt.Println(split)
```



### `Trim()` 

```go
func Trim(s, cutset string) string
```

`Trim()` возвращает *string* `s` со всеми удаленными начальными и конечными *Unicode code point*'ами, содержащимися в `cutset`.

Пример:

```go
fmt.Print(strings.Trim("¡¡¡Hello, Gophers!!!", "!¡")) // Hello, Gophers
```



### `TrimSpace()`

```go
func TrimSpace(s string) string
```

`Trim()` возвращает *string* `s` со всеми удаленными начальными и конечными *space* (` `. `\n`, `\t`, ...), как определено в Unicode.

Пример:

```go
fmt.Println(strings.TrimSpace(" \t\n Hello, Gophers \n\t\r\n")) // Hello, Gophers
```



## Type

### `Builder`

```go
type Builder struct {
	// contains filtered or unexported fields
}
```

`Builder` используется для эффективной сборки `string` с использованием методов `Write`. `Builder` сводит к минимуму копирование памяти. *Zero value* готово к использованию. Не копируйте *non-zero* `Builder`.

```go
import (
	"fmt"
	"strings"
)

func main() {
	var b strings.Builder
	for i := 3; i >= 1; i-- {
		fmt.Fprintf(&b, "%d...", i)
	}
	b.WriteString("ignition")
	fmt.Println(b.String())
}
```



Как и `bytes.Buffer`, `strings.Builder` поддерживает четыре метода для *write* в *builder*:

```go
func (b *Builder) Write(p []byte) (int, error) 
func (b *Builder) WriteByte(c byte) error 
func (b *Builder) WriteRune(r rune) (int, error) 
func (b * Builder) WriteString(s string) (int, error)
```

Это позволяет писать в *builder*: *slice of byte*, *byte*, *rune* или *string*.



#### Использование

Самый простой вариант конкатенации строк без `strings.Builder` – следующий.

```go
func join(strs ...string) string {
	var ret string
	for _, str := range strs {
		ret += str
	}
	return ret
}
```

Хотя это работает, это неэффективно. Т.к. каждый раз, когда мы вызываем `ret += str` новую строку, она должна выделяться в памяти. Это происходит потому, что `string` в Go *immutable*, поэтому, если мы хотим изменить *string* или добавить к ней содержимое, нам нужно создать совершенно новую *string*. Чтобы избежать создания новых *string* при построении нашей окончательной *string*, мы можем использовать `Builder` *type*:

```go
func join(strs ...string) string {
	var sb strings.Builder
	for _, str := range strs {
		sb.WriteString(str)
	}
	return sb.String()
}
```

`Builder` реализует множество *interface*'s:

- `io.Writer`: это позволяет использовать `Builder` с различными функциями, например, `fmt.Fprintf()`:

  ```go
  var b strings.Builder
  for i := 3; i >= 1; i-- {
  	fmt.Fprintf(&b, "%d...", i)
  }
  b.WriteString("ignition")
  fmt.Println(b.String())
  ```



#### `String()`

```go
func (b *Builder) String() string
```

`String()` возвращает аккумулированную строку.



#### `WriteString()`

```go
func (b *Builder) WriteString(s string) (int, error)
```

`WriteString()` добавляет содержимое `s` в буфер `b`. Возвращает длину `s` и *nil error*.