# `unsafe`

*Package* `unsafe` содержит операции, которые обходят безопасность типов программ Go.

*Package*'s, которые делают `import "unsafe"`, могут быть непереносимыми и не защищены рекомендациями по совместимости Go 1.

[смотреть также про Alignment / padding](../Internal.md#alignment-padding)

# Структура

## `Alignof()`

```go
func Alignof(x ArbitraryType) uintptr
```

`Alignof()` принимает *expression* `x` любого типа и возвращает *required alignment* гипотетической *variable* `v`, как если бы `v` была объявлена через `var v = x`. Это наибольшее значение `m` такое, что всегда `адрес v % m = 0` (т.е. это и есть результат *alignment*, данные *aligned*, когда они имеют длину *n* *byte*'s и адрес элемента данных *n-byte aligned*, [link](../Internal.md#alignment-padding)). 

Это то же самое, что и значение, возвращаемое функцией `reflect.TypeOf(x).Align()`. 

Можно использовать специальный вызов `Alignof()`, если переменная `s` имеет *struct type*, а `f` является *field* в этой *struct*, то `Alignof(s.f)` вернет *required alignment* для этого *field* в *struct*. Это аналогично вызову функции `reflect.TypeOf(s.f).FieldAlign()`. Возвращаемое значение `Alignof()` - это `Go constant`.

Пример:

```go
type myStruct struct {
	myInt   int64   // 8 bytes
	myBool  bool    // 1 byte
}

func main() {
	a := myStruct{}
	fmt.Println(unsafe.Alignof(a))        // 8
	fmt.Println(unsafe.Alignof(a.myInt))  // 8
	fmt.Println(unsafe.Alignof(a.myBool)) // 1
}
```

##  `Offsetof()`

```go
func Offsetof(x ArbitraryType) uintptr
```

`Offsetof()` возвращает *offset* для *field* `x` внутри *struct*. Аргумент `x` должен иметь форму `structValue.field`. Другими словами, он возвращает количество байтов между началом *struct* и началом *field*. Возвращаемое значение `Offsetof()` - это *Go constant*.

Пример:

```go
type myStruct struct {
	myBool1  bool    // 1 byte
	myBool2  bool    // 1 byte
	myInt   int64    // 8 bytes
	myBool3  bool    // 1 byte
	myBool4  bool    // 1 byte
}

func main() {
	a := myStruct{}
	fmt.Println(unsafe.Offsetof(a.myBool1))  // 0
	fmt.Println(unsafe.Offsetof(a.myBool2))  // 1
	fmt.Println(unsafe.Offsetof(a.myInt))    // 8
	fmt.Println(unsafe.Offsetof(a.myBool3))  // 16
	fmt.Println(unsafe.Offsetof(a.myBool4))  // 17
}
```





## `Sizeof()`

```go
func Sizeof(x ArbitraryType) uintptr
```

`Sizeof()` принимает выражение `x` любого типа и возвращает размер в байтах гипотетической переменной `v`, как будто бы было сделано объявление `var v = x`. Размер не включает размер блоков памяти, на которые `x` содержит *pointer*'ы внутри себя. Например, если `x` – это *slice*, `Sizeof()` возвращает размер *slice descriptor*, а не размер памяти, *pointer* на который содержит *slice*. Возвращаемое значение `Sizeof()` – *Go constant*.







# Internal

https://aticleworld.com/data-alignment-and-structure-padding-bytes/

https://en.wikipedia.org/wiki/Data_structure_alignment

https://stackoverflow.com/questions/61053746/memory-alignment-and-padding-in-c

https://www.google.com/search?q=alignment+padding+memory+golang&ei=Xb2BYfyMJcKFrwT0xadQ&oq=alignment+padding+memory+golang&gs_lcp=Cgdnd3Mtd2l6EAM6BwgAEEcQsAM6BggAEBYQHjoICCEQFhAdEB5KBAhBGABQr6kDWMCxA2CrswNoAXACeACAAasBiAGPBpIBAzQuM5gBAKABAcgBCMABAQ&sclient=gws-wiz&ved=0ahUKEwi8r_bd3vrzAhXCwosKHfTiCQoQ4dUDCA4&uact=5

https://www.geeksforgeeks.org/structure-member-alignment-padding-and-data-packing/

https://itnext.io/structure-size-optimization-in-golang-alignment-padding-more-effective-memory-layout-linters-fffdcba27c61

https://go101.org/article/memory-layout.html