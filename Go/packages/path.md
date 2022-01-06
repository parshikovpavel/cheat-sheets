# `path`

*Package* `path` реализует утилиты для управления *slash-separated path* (т.е. с разделителем `/`).

*Package* `path` следует использовать только для *path*, разделенных *slash*'ами, таких как *path* который является частью URL (вида `/css/my.css`). Т.е. его нельзя использовать для целого URL, а только для *path*. 

Этот *package* не имеет работать с *Windows path*, где используются буква диска (`C:\`) и *backslash* (`\`); для управления *path* в операционной системе используйте пакет `path/filepath`.



## Структура

```go
func Join(elem ...string) string
```

`Join()` объединяет любое количество *path element*'ов в единый *path*, разделяя их *slash*'ами. Пустые *element*'ы игнорируются. Результат пропускается через функцию `Clean()`. Если *argument list* пуст или все его *element*'ы пусты, `Join()` возвращает пустую *string*.

```go
func main() {
	fmt.Println(path.Join("a", "b", "c"))
	fmt.Println(path.Join("a", "b/c"))
	fmt.Println(path.Join("a/b", "c"))

	fmt.Println(path.Join("a/b", "../../../xyz"))

	fmt.Println(path.Join("", ""))
	fmt.Println(path.Join("a", ""))
	fmt.Println(path.Join("", "a"))
}
```

Вывод:

```
a/b/c
a/b/c
a/b/c
../xyz

a
a
```

