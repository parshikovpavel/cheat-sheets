https://go.dev/blog/go1.13-errors

https://pkg.go.dev/errors









## `errors`

*Package* `errors` реализуют функции для манипулирования *error*'s.

Функция `New()` создает *error*, единственным содержимым которых является текстовое *message*.

Функции `Unwrap()`, `Is()` и `As()` работают с *error*, которые могут *wrap* другие *error*. *Error wrap* другую *error*, если ее *type* имеет *method*

```
Unwrap() error
```

Если `e.Unwrap()` возвращает *non-nil error* `w`, мы говорим, что `e` *wrap* `w`.

`errors.Unwrap()` распаковывает *wrapped error*. Если *type* его аргумента имеет метод `Unwrap()`, `errors.Unwrap()` вызывает этот метод один раз. Иначе, возвращает `nil`.

Простой способ создать *wrapped error* — вызвать `fmt.Errorf()` и применить `%w` *verb* к аргументу *error*.

Пример, как сделать *wrap* и *unwrap error*:

```go
wrappedErr := fmt.Errorf("... %w...",..., err,...)
errors.Unwrap(wrappedErr) == err
```

`Is()` *unwrap* свой первый аргумент, последовательно пытаясь найти *error*, соответствующий (*equal*???) второму аргументу. Он сообщает, найдено ли совпадение. Его следует использовать вместо простых проверок на *equality* (равенство).

Так:

```go
if errors.Is(err, fs.ErrExist)
```

предпочтительнее, чем:

```go
if err == fs.ErrExist
```

Т.к. `errors.Is()` вернет `true`, если `err`  *wrap* `fs.ErrExist`.

`As()` *unwrap* свой первый аргумент, последовательно пытаясь найти *error*, которую можно *assign* (присвоить) второму аргументу, который должен быть *pointer* (даже *pointer* на *pointer*):

- в случае успеха он выполняет *assignment* (присвоение) и возвращает `true`. 
- в противном случае – возвращает `false`. 

Поэтому, проверка соответствия типа error должна выполняться так:

```go
var validationErr *tinkoff.ValidationError
	if errors.As(err, &validationErr) {
		// ...
	}
```

и это предпочтительнее, чем

```go
if validationErr, ok := err.(*tinkoff.ValidationError); ok {
	// ...
}
```

Потому что `errors.As()` вернет `true`, если `err` *wrap* `*tinkoff.ValidationError`.



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

Часто вместе `errors.New()` полезно использовать `fmt.Errorf()` ([1](#fmterrorf))

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

### Создание кастомной *error*, которая может заворачивать в себя другую *error*

https://stackoverflow.com/questions/61407054/using-standard-library-only-how-to-wrap-error-in-custom-error

https://dev.to/tigorlazuardi/go-creating-custom-error-wrapper-and-do-proper-error-equality-check-11k7

Необходимо использовать для *error* тип `struct`, который:

- включает в себя внутреннюю ошибку `inner`
- реализует метод `Unwrap()`

```go
type ValidationError struct {
	inner error
	message string
}

func NewValidationError(err error, message string) *ValidationError {
	return &ValidationError{
		inner: err,
		message: message,
	}
}

func (e *ValidationError) Error() string {
	return fmt.Sprintf("%s due to: %v", e.message, e.inner)
}

func (e *ValidationError) Unwrap() error {
	return e.inner
}

```



# Особенности сравнения *error*'s

Особенности сравнения *error*'s заключаются в особенностях работы [operator'а `==`](../Lang.go#comparison-operator) в Go. 

Случай 1. Две отдельные *error*'s с одним и тем же *message*, созданные разными вызовами функции `errors.New()` – не равны:

```go
func main() {
  const message = "A custom error message"

  var e1, e2 error

  e1 = errors.New(message)
  e2 = errors.New(message)

  // Не равны
  fmt.Println(e1 == e2)     // false

  // Потому что хранят pointer'ы
  fmt.Printf("%#v\n", e1)   // &errors.errorString{s:"A custom error message"}
  fmt.Printf("%T\n", e1)    // *errors.errorString

  // Причем pointer'ы на разные адреса
  fmt.Printf("%p\n", e1)    // 0xc000010230
  fmt.Printf("%p\n", e2)    // 0xc000010240

  // ------------------------------------------------------------------------------------------
  // fmt.Errorf делегирует создание error – функции errors.New() и поэтому ведет аналогично
  e1 = fmt.Errorf(message)
  e2 = fmt.Errorf(message)

  // Не равны
  fmt.Println(e1 == e2)     // false

  // Потому что хранят pointer'ы
  fmt.Printf("%#v\n", e1)   // &errors.errorString{s:"A custom error message"}
  fmt.Printf("%T\n", e1)    // // *errors.errorString

  // Причем pointer'ы на разные адреса
  fmt.Printf("%p\n", e1)    // 0xc000010260
  fmt.Printf("%p\n", e2)    // 0xc000010270
}
```

В этом случае мы сравниваем два *interface value*'s с типом `error`. Они будут равны, когда равны *dynamic type*'s и *dynamic value*'s. *Dynamic type*'s у обоих переменных – *pointer type* (т.е. *dynamic type* совпадает). А *dynamic value* не совпадает, т.к. *pointer value*'s указываются на разные *variable*'s (области в памяти).

Поэтому нельзя сравнивать *error*, которая пришла из вызова функции с новой только-что созданной *error*. Не нужно вообще в коде постоянно делать `errors.New()`, например:

```go
if err == errors.New("Token is expired") {  //  Будет всегда false
      // ...
}
```

Нужно где-то заранее в коде определить *variable* с *error* и потом использовать ее везде:

```go
package fruits

var NoMorePumpkins = errors.New("No more pumpkins")
```

```go
package shop

if err == fruits.NoMorePumpkins {
     ...
}
```





Случай 2. При этом две *error*'s с одним и тем же *message*, которые созданы вручную и не являются *pointer*'ами, – равны:

```go
type CustomError struct {
  Message string
}

func (ce CustomError) Error() string {
  return ce.Message
}

func main() {
  const message = "A custom error message"

  var e1, e2 error

  e1 = CustomError{Message: message}
  e2 = CustomError{Message: message}

  // Равны, потому что это struct, а не pointer
  fmt.Println(e1 == e2)   // true
}
```

В этом случае мы также сравниваем два *interface value*'s с типом `error`. Они будут равны, когда равны *dynamic type*'s и *dynamic value*'s. *Dynamic type*'s у обоих переменных – *struct type* (т.е. *dynamic type* совпадает). И *dynamic value* совпадает, т.к. у обеих *struct*'s совпадают значения всех их *field*'s. 

# Не просто проверяйте *error*, обрабатывайте их *gracefully*

https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

## *Error* – это просто *value*

Обработку *error* в Go можно разделить на три основные стратегии.

### Sentinel error

*Sentinel error* (ошибка часового, похоже на [guard clause](Design.md#guard-clause)). Имеют вид:

```go
if err == ErrSomething { … }
```

*Sentinel* означает, что дальнейшая обработка невозможна. Т.е. есть некоторое специальное *value*, которое обозначает *error*. Примеры таких *value*: `io.EOF`, `syscall.ENOENT`.

Использование *sentinel error* является наименее гибкой стратегией обработки *error*, поскольку *caller* (вызывающий) должен сравнить результат с *predeclared value* с помощью оператора `=`. Это создает проблему, когда вы хотите предоставить больше контекста, так как возврат другой *error* нарушит проверку на равенство. Вместо этого *caller* будет вынужден просмотреть вывод метода `error.Error()` чтобы убедиться, что он соответствует определенной строке.

Никогда не нужно проверять вывод метода `error.Error()`, т.к. он предназначен для использования людьми, а не проверки в коде. Эта строка выводится в log file или отображается на экране. 

TODO!!!

# *error* в виде *constant*

https://dave.cheney.net/2016/04/07/constant-errors

Допустим мы используем стратегию *sentinel error*. В идеале, *sentinel error* должно быть описана как *constant*, и в этом случае оно становится *immutable* и взаимозаменяемой.

При использовании *error* в виде *external variable*, например `io.EOF`, любой код, который делает *import* для `io` *package* может изменить значение `io.EOF`. Это может привести к следующей проблеме:

```go
func f() error {
  return io.EOF
}

func main() {
	err := f()

	io.EOF = fmt.Errorf("whoops")

	fmt.Println(x == io.EOF)      // false
}
```

Но т.к. можно создавать *constant*'ы не только базовых *type*'s (`string`, `bool`, ...), но и *defined type* для которых используется какой-либо базовый *underlying type* ([link](../constant.md#typed-constantы)) То можно написать следующую реализацию `error` *interface*, которая использует `string` в качестве *underlying type*, и может использоваться в *constant expression*.

```go
type Error string

func (e Error) Error() string { 
  return string(e) 
}

func main() {
  const a Error = "123"
}
```

У таких константных *error* есть два преимущества:

- их значение изменить нельзя:

  ```go
  const err = Error("EOF") 
  err = Error("not EOF") // error, нельзя присвоить err
  ```

- две *string* равны, если их содержимое одинаково ([link](../Lang.md#comparison-operator)). Поэтому два значения *type* `Error` с одинаковым содержимым будут равны.

  ```go
  const err = Error("EOF") 
  fmt.Println(err == Error("EOF")) // true
  ```

