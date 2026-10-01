*Generic*'и добавлены в Go 1.18. 

*Generic*'и — это способ написания кода, который не зависит от конкретных используемых *typ*'ов. Функции и типы теперь могут быть написаны для использования любого из набора типов.

Дженерики добавляют в язык три новых важных вещи:

1. Параметры типа для функции и типов.
2. Определение типов интерфейсов как наборов типов, включая типы, не имеющие методов.
3. Вывод типа, позволяющий во многих случаях опускать аргументы типа при вызове функции.







### Generic Method

Возьмём *type parameter* - обёртка над *slice*

```go
type List[T any] []T
```

Добавим к нему метод *Reverse*:

```go
func (s List[T]) Reverse() List[T] {
	res := make(List[T], len(s))

	for i := range s {
		res[i] = s[len(s)-i-1]
	}

	return res
}

func main() {
	numbers := List[int]{1, 2, 3, 4}
	reversed := numbers.Reverse()

	fmt.Println(numbers)  // [1 2 3 4]
	fmt.Println(reversed) // [4 3 2 1]
}
```

*Method* не выходит за пределы `T`: `List[T]` на входе, `List[T]` на выходе — содержимое изменяется, но тип элемента остаётся прежним. То есть, нас полностью устраивает фиксированный `T`, доставшийся от *receiver*.

Но что если нам хочется реализовать метод `Map()`, который превратит `List[T]` в `List[U]` — например, `List[int]` в `List[string]`? Здесь нас ждёт проблемка: `List[T]` знает только о `T`, а тип для `U` определяется при каждом вызове `Map()`. Значит, *method*'у нужен собственный *type parameter* — тот, которого нет у *receiver*'а.

```go
// type parameter U в методе -> ошибка
func (s List[T]) Map[U any](f func(T) U) List[U] {...}

// syntax error: method must have no type parameters
```

Именно это до Go 1.27 было запрещено: метод не мог объявить новый *type parameter*. Поэтому `Map()` жил в виде отдельной функции и засорял неймспейс пакета:

```go
func Map[T, U any](s List[T], f func(T) U) List[U] {...}
```

В Go 1.27 такой метод наконец компилируется. Параметр типа `U` объявляется прямо у метода — между *receiver*'ом и списком аргументов:

```go
type List[T any] []T

func (s List[T]) Map[U any](f func(T) U) List[U] {
    out := make(List[U], len(s))
    
    for i, v := range s {
        out[i] = f(v)
    }
    
    return out
}

func main() {
    nums := List[int]{1, 2, 3}
    strs := nums.Map(strconv.Itoa)
    fmt.Printf("%q\n", strs) // ["1" "2" "3"]
}
```

`T = int` задан типом самого `nums`, а `U = string` компилятор выводит из сигнатуры функции `strconv.Itoa` — `func(int) string`. Отсюда выводится и тип результата: `List[string]`.