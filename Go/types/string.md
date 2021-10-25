# `string`

Строки – *immutable* (неизменяемы): после создания невозможно изменить содержимое строки.

Строки хранятся в кодировке UTF-8 и символы строки могут быть представлены двумя и более байтами. 

При обращении к определенной позиции в строке по индексу `str[1]` возвращается байт (тип `byte`), хранящийся в этой позиции. 



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





Символы *carriage return* (возврата каретки, `'\ r'`) внутри *raw string literal* отбрасываются из итогового значения строки (*raw string value*).

TODO!!!

https://golang.org/ref/spec#String_literals

## Итерирование `string`

При итерировании циклом `for` по индексам массива, происходит итерирование по байтам строки, что в случае Unicode строк будет некорректным.

```go
name := "Вася Пупкин" // строка

for i := 0; i < len(name); i++ {
	fmt.Printf("%c", name[i]) // name[i] — байт
} // ÐÐ°ÑÑ ÐÑÐ¿ÐºÐ¸Ð½
```

При итерировании через `for - range`, происходит корректное итерирование по *Unicode runes*.

```go
for _, rune := range name {
	fmt.Printf("%c", rune) // rune — utf8-символ
} // Вася Пупкин
```

## Конкатенация `string`

TODO:

https://www.calhoun.io/6-tips-for-using-strings-in-go/

https://golangdocs.com/concatenate-strings-in-golang

https://www.geeksforgeeks.org/different-ways-to-concatenate-two-strings-in-golang/

https://stackoverflow.com/questions/1760757/how-to-efficiently-concatenate-strings-in-go



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