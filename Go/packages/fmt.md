

# `fmt`

`fmt` – *formatted* (форматированный). 

*Package* реализует форматированный I/O по аналогии с функциями `printf()` и `scanf()` в Си. 

Некоторые функции принимают `format`, в котором можно использовать спецификаторы формата `%`, напоминающие спецификаторы функций `printf()` и `scanf()` в Си.

## Плейсхолдеры

### Общие 

```
%v    значение в default формате
	    для структур флаг плюс (%+v) добавляет имена полей
%#v   представление значения в синтаксисе Go
%T    представление типа значения в синтаксисе Go
%%    символ процента (%)
```





### `%v`

- `%v` – значение в *default* формате.

  *Default* форматы:

  ```
  bool:                    %t
  int, int8 etc.:          %d
  uint, uint8 etc.:        %d, %#x if printed with %#v
  float32, complex64, etc: %g
  string:                  %s
  chan:                    %p
  pointer:                 %p
  ```

*String* и *slice of bytes* (обрабатываются одинаково с помощью этих плейсхолдеров):

- `%s` – неинтерпретированные байты из string или slice
- `%q` – строку заключает в двойные кавычки `"` и экранирует в кавычках нужные символы (например, `%q`).

#### Плейсхолдеры для `error`

Начиная с Go 1.13 функция `fmt.Errorf` (только эта функция!) поддерживает `%w` плейсхолдер для вставки `error` в вывод. 

Плейсхолдер `%w` :

- вставляет ошибку в строку. Это поведение аналогично  `%v`.
- оборачивает ошибку в другую ошибку так, что:
  - ее можно позже развернуть через `errors.Unwrap`, которая возвращает аргумент `%w` (это вложенная ошибка)
  - ошибку можно изучать через функции `errors.Is`и `errors.As`

Пример:

```
if err != nil {
    // Возвращает ошибку, которая разворачивается до err.
    return fmt.Errorf("decompress %v: %w", name, err)
}
```

Пример использования `errors.Is()`:

```
err := fmt.Errorf("access denied: %w", ErrPermission)
...
if errors.Is(err, ErrPermission) ...
```

Если не хочется использовать `%w`, тогда можно использовать `%v` (лучше работает чем `%s` для значений `nil`).

Пример:

```
if err != nil {
    return fmt.Errorf("decompress %v: %v", name, err)
}
```



# Структура



## `fmt.Errorf()`

```go
func Errorf(format string, a ...interface{}) error
```

Форматирует и возвращает строку как значение типа `error`.



## `Fprintf()`

```
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
```

`Fprintf()` форматирует в соответствии с спецификатором `format` и записывает в `w`. 

Возвращаемые значения:

- `n` – количество записанных байт
- `err` – ошибки записи.







## `fmt.Println()`

```go
func Println(a ...interface{}) (n int, err error)
```

Делает вывод операндов в *standart output* в *default* формате. Между операндами добавляются *space*, а в конце добавляется *newline*. 

- `n` – количество записанных байтов
- `err` – все обнаруженные ошибки записи.

```go
package main

import (
	"fmt"
)

func main() {
	const name, age = "Kim", 22
	fmt.Println(name, "is", age, "years old.")

	// It is conventional not to worry about any
	// error returned by Println.

}
```



## `fmt.Printf()`

```go
func Printf(format string, a ...interface{}) (n int, err error)
```

Делает вывод операндов в *standart output* в соответствии с форматом `format`. 

- `n` – количество записанных байтов
- `err` – все обнаруженные ошибки записи.





