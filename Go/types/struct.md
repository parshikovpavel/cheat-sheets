`struct` (структура) – *composite type* (составной тип), последовательность именованных элементов, называемых *field* (поле), у каждого из которых есть *name* и *type*.

<pre>
StructType    = "struct" "{" { FieldDecl ";" } "}" .
FieldDecl     = (<a href="#объявление-constant">IdentifierList</a> <a href="#types">Type</a> | EmbeddedField) [ Tag ] .
EmbeddedField = [ "*" ] <a href="#types">TypeName</a> .
Tag           = string_lit .
</pre>



- Пример 1

  ```go
  // An empty struct.
  struct {}
  ```

- Пример 2

  ```
  StructType    = "struct" "{" { FieldDecl ";" } "}" .
  FieldDecl     = IdentifierList Type
  ```

  ```go
  // A struct with 6 fields.
  struct {
  	x, y int
  	u float32
  	_ float32  // padding
  	A *[]int
  	F func()
  }
  ```



# *Embedded field*

*Field*, объявленное с указанием `TypeName`, но без указания `IdentifierList`, называется *embedded field* (встроенным полем). 

*Embedded field* может быть:

- [`TypeName`](../Lang.md#type) (e.g. `T`) (в том числе *interface*, *struct*, введенные через [type declaration](#type-declaration))
- *pointer* тип (e.g. `*T`). При этом `T` – *non-interface* тип и *non-pointer* тип

Для *еmbedded field* в качестве *field name* выступает *unqualified type name*:

```go
struct {
	T1        // field name is T1
	*T2       // field name is T2
	P.T3      // field name is T3
	*P.T4     // field name is T4
}
```

# *Promoted field* и *method*

Пусть имеем:

```go
type A struct {
  B  // embedded field, field name is B
}
a := A{}
```

где `B` –  *еmbedded field*, имеет структуру:

```go
type B struct {
  field int
}

func (b B) method() int {
    return ...
}
```

Тогда можно использовать сокращенную форму *selector*'а: 

- вместо  `a.B.field` – `a.field`
- вместо `a.b.method` – `a.method`

(когда `a.field` и `a.method` – допустимые [*selector*]()'s) 

Такие `field` и `method` (внутри *embedded field*) называются *promoted*.

*Promoted fields* могут использоваться как обычные *fields* c одним ограничением:

- *promoted fields* не могут использовать в *composite literal*. Нужно явно описать весь *embedded field*:

  ```go
  a := A{field: 1} // Error!!!
  a: = A{B{field: 1}} // OK!
  ```

# *Tag*

После *field declaration* ([`FieldDecl`](#struct-тип)) можно указать `Tag` в виде [*string literal*](#string-literal)'а.

*Field tag* можно извлечь с помощью [reflection interface](). И они используются некоторыми пакетами, для описание полей данных, полученных по некоторому протоколу (*protobuf*, *json*).

С точки зрения языка, tag игнорируется. 

```go
struct {
	name string  "any string"
  microsec  uint64 `protobuf:"1"`
}
```



# Примеры

*Declaration* для типа `struct` 

```go
type A struct {
    b int
}
```

*Variable declaration* типа `struct` с инициализацией:

```go
a := A{b: 1}
```



# Empty struct

*Empty struct* – это *struct type*, у которого нет *field*'s. 

Примеры:

```go
type Q struct{}
var q struct{}
```

*Empty struct* имеет нулевой размер:

```go
func main() {
	var s struct {}
	fmt.Println (unsafe.Sizeof(s)) // 0
}
```

*Empty struct* является *struct type*, как и любая другая *struct*. Все свойства, которые можно использовать с обычными *struct*, также применимы к *empty struct*.

Примеры *composite type*'s с `struct{}`:

- *array* из `struct{}` с `size = 0`:

  ```go
  var x [1000000000]struct{}
  fmt.Println(unsafe.Sizeof(x)) // prints 0
  ```

- *struct* из `struct{}` с `size = 0`:

  ```go
  var s struct {
  	A struct{}
  	B struct{}
  }
  
  fmt.Println(unsafe.Sizeof(s)) // prints 0
  ```

- *slice* из `struct{}`. *Slice header* занимает 12 bytes. Элементы *slice* занимают 0 byte.

  ```go
  func main() {
  	var d = make([]struct{}, 10000000)
  	fmt.Println(unsafe.Sizeof(d))    // prints 24
  	fmt.Println(unsafe.Sizeof(d[0])) // prints 0
  }
  ```

Причем у всех `struct{}` одинаковый *address*:

```go
func main() {
	var a struct {}
	var b struct {
		A struct{}
		B struct{}
	}
	var c [1000000000]struct{}
	var d = make([]struct{}, 10000000)

	fmt.Printf("%p\n", &a)     // 0x119f9e0
	fmt.Printf("%p\n", &b)     // 0x119f9e0
	fmt.Printf("%p\n", &b.A)   // 0x119f9e0
	fmt.Printf("%p\n", &b.B)   // 0x119f9e0
	fmt.Printf("%p\n", &c)     // 0x119f9e0
	fmt.Printf("%p\n", &d[0])  // 0x119f9e0
}
```

Это отмечено в *Go specification*: [две разные variables нулевого размера могут иметь один и тот же адрес в памяти.](http://golang.org/ref/spec#Size_and_alignment_guarantees)

## Выделение дополнительной памяти для `struct{}`

Есть отдельный случай, когда добавление `struct{}` приводит к выделению дополнительной памяти. Это происходит если:

- `struct{}` добавляется в конце непустой *struct*
- в конце *struct* отсутствует *padding*

```go
	// Struct без padding
	var structWitoutPadding1 struct {
		_ int64
	}
	var structWitoutPadding2 struct {
		_ int64
		_ struct{}
	}

	fmt.Println(unsafe.Sizeof(structWitoutPadding1))     // 8
	fmt.Println(unsafe.Sizeof(structWitoutPadding2))     // 16

	// Struct с padding
	var structWithPadding1 struct {
		_ int64
		_ bool
	}
	var structWithPadding2 struct {
		_ int64
		_ bool
		_ struct{}
	}
	
	fmt.Println(unsafe.Sizeof(structWithPadding1))     // 16
	fmt.Println(unsafe.Sizeof(structWithPadding2))     // 16
```

Причина в том, что хотя *empty struct* `struct{}` не потребляют памяти, но вы можете получить их адрес. И если бы не проиходило выделения дополнительной памяти, то `t.Y` указывало бы за пределы *struct* (это приводит к разным проблемам с GC, разыменованию недопустимого *pointer*'а). Поэтому *field* нулевого размера, такие как `struct{}` и `[0]byte`, встречающиеся в конце *struct*, имеют размер *1 byte*. 

```go
var t struct {
      X uint32
      Y struct{}
}

// приводит к чему-то вроде
var t struct {
      X uint32
      Y [1]byte
}
```

Адрес `struct{}`, которые находятся внутри непустой *struct* указывает внутрь этой *struct* и не совпадает с адресом простой пустой `struct{}`:

```go
func main() {
	var s struct{}

	var t struct {
		A struct{}
		B int64
		C struct{}
	}

	fmt.Printf("%p\n", &s)       // 0x119f9e0

	fmt.Printf("%p\n", &t)       // 0xc000018070
	fmt.Printf("%p\n", &t.A)     // 0xc000018070
	fmt.Printf("%p\n", &t.B)     // 0xc000018070
	fmt.Printf("%p\n", &t.C)     // 0xc000018078
}
```



Применение *empty struct* на практике:

- в каналах `chan struct{}` для передачи сигналов между *goroutine*'s ([link](channel.md)).
- в `map[string]struct{}` для реализации *set* (множества) ([link](map.md#реалилизация-set-множества))

