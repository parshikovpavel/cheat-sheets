# `string`

Строки – *immutable* (неизменяемы): после создания невозможно изменить содержимое строки.

Строки хранятся в кодировке UTF-8 и символы строки могут быть представлены двумя и более байтами. 

При обращении к определенной позиции в строке по индексу `str[1]` возвращается байт (тип `byte`), хранящийся в этой позиции. 

https://go.dev/blog/strings

## Code point, character, rune

Стандарт Unicode использует термин «*code point*» для обозначения элемента, представленного одним значением. *Code point* U+2318, имеющая шестнадцатеричное значение 2318, представляет символ ⌘. 

«*Code point*» звучит немного громоздко, поэтому Go вводит более короткий термин для этого понятия: *rune* . Этот термин встречается в библиотеках и исходном коде и означает то же самое, что и «*code point*», с одним интересным дополнением.

В языке Go слово  `rune` - это псевдоним для типа `int32`, поэтому программам понятно, когда *integer value* представляет собой *code point*. В Go *character constant* называется *rune constant* . *Type* и *value* выражения

```
'⌘'
```

имеет `rune` c *integer value* `0x2318`.

Основные моменты:

- Исходный код Go всегда использует кодировку UTF-8.
- *String* содержит произвольное количество байтов.
- *String literal* всегда содержит допустимые последовательности UTF-8.
- Эти последовательности представляют собой *Unicode code point*'ы, называемые *rune*.

## Итерирование `string`

Существует лишь один способ, которым Go обрабатывает UTF-8 особым образом – при использовании цикла `for-range`для работы со *string*. При итерировании через `for - range`, происходит корректное итерирование по *Unicode runes*.

На каждой итерации цикла, *index* цикла — это начальная позиция текущей *rune*, измеряемая в байтах, а *value* — это её *code point*. 

```go
func main() {
	name := "Вася" // строка

	for index, rune := range name {
		fmt.Printf("%c starts at byte position %d\n", rune, index)
	}
}

/*
В starts at byte position 0
а starts at byte position 2
с starts at byte position 4
я starts at byte position 6
*/
```

При итерировании циклом `for` по индексам массива, происходит итерирование по байтам строки, что в случае Unicode строк будет некорректным.

```go
name := "Вася Пупкин" // строка

for i := 0; i < len(name); i++ {
	fmt.Printf("%c", name[i]) // name[i] — байт
} // ÐÐ°ÑÑ ÐÑÐ¿ÐºÐ¸Ð½
```

## Библиотека [`unicode/utf8`](https://go.dev/pkg/unicode/utf8/)

Если цикла `for-range` недостаточно для ваших целей, скорее всего, необходимая вам функция предоставляется пакетом в библиотеке [`unicode/utf8`](https://go.dev/pkg/unicode/utf8/), который содержит вспомогательные подпрограммы для проверки, декомпиляции и повторной сборки строк UTF-8. Вот программа, эквивалентная `for` `range`приведенному выше примеру, но использующая `DecodeRuneInString `функцию из этого пакета для выполнения работы. *Return value*'s функции — это *rune* и ее размер в байтах, закодированных в UTF-8.

```
    const nihongo = "日本語"
    for i, w := 0, 0; i < len(nihongo); i += w {
        runeValue, width := utf8.DecodeRuneInString(nihongo[i:])
        fmt.Printf("%#U starts at byte position %d\n", runeValue, i)
        w = width
    }
```

Цикл `for-range` и `DecodeRuneInString` выдают отдинаковый результат.





## String literal

*String literal* (строковый литерал) является описанием *string constant*'ы ([link](../lang.md#constant)). 

Есть две формы:

- *raw string literal* (необработанный строковый литерал) 
- *interpreted string literal* (интерпретированный строковый литерал).

### *Raw string literal*

*Raw string literal* – это последовательности символов между *back quote*'s (обратными кавычками), например:

```go
`foo`
```

Внутри *quote*'s может использоваться любой символ, кроме символа *back quote* (\`). Значение *raw string literal* – это строка, состоящая из *uninterpreted* (закодированных в UTF-8) символов между *quote*'s. Например, *backslash* (обратный слэш, `\`)  не имеет специального значения, а строка может содержать символы *newline*. 

```go
var a = `fsdjklj
lfsd
j;hglfj;'lf`
```





Символы *carriage return* (возврата каретки, `'\r'`) внутри *raw string literal* отбрасываются из итогового значения строки (*raw string value*).

TODO!!!

https://golang.org/ref/spec#String_literals



## Конкатенация `string`

смотреть [String concatenation](../Lang.md#string-concatenation)

TODO:

https://www.calhoun.io/6-tips-for-using-strings-in-go/

https://golangdocs.com/concatenate-strings-in-golang

https://www.geeksforgeeks.org/different-ways-to-concatenate-two-strings-in-golang/

https://stackoverflow.com/questions/1760757/how-to-efficiently-concatenate-strings-in-go



## `strings` package

[Здесь](../packages/strings.md)



## Изменение элемента строки

Строки в Go *immutable*, поэтому нельзя изменять их содержимое. Чтобы изменить значение строки, нужно создать новую строку.

Способы:

- Решение для однобайтовых символов: преобразовать `string` в *byte slice* внести изменения и преобразовать обратно:

  ```go
  s := []byte(str)
  s[2] = 'y'
  str = string(s)
  ```

- Т.к. строки UTF8, для многобайтовых строк это будет работать неправильно. Лучше использовать *rune slice*:

  ```go
  s := []rune(str)
  s[2] = 'y'
  str = string(s)
  ```

- Решение для однобайтовых символов: *slice* до заменяемого символа + нужный символ + *slice* после заменяемого:

  ```go
  str[0] = str[0][:2] + "y" + str[0][3:]
  ```

## Вывести символ строки, как символ

- для однобайтовых символов:

  ```go
  fmt.Println(string("Hello"[1]))
  fmt.Printf("%c", test[1])
  ```

- для многобайтовых символов:

  ```go
  fmt.Println(string([]rune("Hello, 世界")[1]))
  fmt.Printf("%c", []rune("HELLO")[1])
  ```

  

## Substring

Встроенной в стандартные библиотеки функции для получения *substring* – нет.

Есть такие варианты сделать это вручную:

- конвертировать *string* в `[]rune` , применить *slice expression*, конвертировать назад в *string*: 

  ```go
  substring := string([]rune(str)[:20])
  ```

  Этот вариант вызывает *panic*, если границы *slice expression* выходят за пределы, в случае когда длина *string* меньше чем длина *substring*. 

  Более сложный вариант с этой идеей в виде функции, которая проверяет граничные случаи:

  ```go
  func substr(input string, start int, length int) string {
      asRunes := []rune(input)
  
      if start >= len(asRunes) {
          return ""
      }
  
      if start+length > len(asRunes) {
          length = len(asRunes) - start
      }
  
      return string(asRunes[start : start+length])
  }
  ```

  

- без выделения дополнительной памяти (при преобразовании в `[]rune`).

  Нужно пройти в цикле `for` по строке и найти байтовые индексы начала и конца *substring*:

  ```go
  func substring(s string, start int, end int) string {
      start_str_idx := 0
      i := 0
      for j := range s {
          if i == start {
              start_str_idx = j
          }
          if i == end {
              return s[start_str_idx:j]
          }
          i++
      }
      return s[start_str_idx:]
  }
  ```

- использовать экспериментальный *package* https://pkg.go.dev/golang.org/x/exp/utf8string:

  ```go
  a := utf8string.NewString("ÄÅàâäåçèéêëìîïü")
  s := a.Slice(1, 3)
  ```

  Этот вариант тоже вызывает *panic*,  когда границы выходят за пределы *string*. При необходимости нужно проверить границы с помощью `utf8string.RuneCount()` или `utf8.RuneCountInString()`.

https://stackoverflow.com/questions/28718682/how-to-get-a-substring-from-a-string-of-runes-in-golang