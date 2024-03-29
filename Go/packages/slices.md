`golang.org/x/exp/slices`

## `func Delete()`

```go
func Delete[S ~[]E, E any](s S, i, j int) S
```

`Delete()` удаляет элементы `s[i:j]` из `s`, возвращая измененный *slice*. 

Важные моменты:

- функция является оберткой для `append(slice[:s], slice[s+1:]...)`
- поэтому `Delete()` изменяет содержимое исходного фрагмента
- Элемент(ы) от индекса i до j удаляются
- Вам нужно переназначить `slice`, иначе он будет иметь неправильную длину

