# Constant

✓

## Виды untyped constant

В Go есть *untyped constant*'ы соответствующих видов:

- *boolean*
- *numeric*, к ним относятся:
  - *rune*
  - *integer*
  - *floating*
  - *complex*
- *string*. 

| Constant                | Examples              |
| :---------------------- | :-------------------- |
| integer constant        | `1000`, `67413`       |
| floating-point constant | `4.56`, `128.372`     |
| boolean constant        | `true`, `false`       |
| rune constant           | `'C'`, `'ä'`          |
| complex constant        | `2.7i`, `3 + 5i`      |
| string constant         | `"Hello"`, `"Rajeev"` |

Вид *untyped constant*'ы определяется ее синтаксисом при записи (`1000` – *integer*, `1000.0` – *float-pointing*, `'1'` – *rune*).

И эти *untyped constant* могут быть приведены только к *typed constant*'ам, с которыми они [assignable](Lang.md#assignability) (как указано в [разделе](#typed-and-untyped-constantы))

Между *numeric constant*'ами разных типов возможны преобразования при выполнении [constant expression](#constant-expression) (как указано в разделе про них).

*Constant*'ы могут иметь *identifier* (имя) или не иметь. *Constant*'а — это просто простое неизменное значение





*Constant value* представляется:

- [rune literal](https://go.dev/ref/spec#Rune_literals)
- [integer literal](https://go.dev/ref/spec#Integer_literals)
- [floating-point literal](https://go.dev/ref/spec#Floating-point_literals)
- [imaginary literal](https://go.dev/ref/spec#Imaginary_literals)
- [string literal](https://go.dev/ref/spec#String_literals)
- *identifier*, обозначающий *constant*
- [constant expression](https://go.dev/ref/spec#Constant_expressions)
- [conversion](https://go.dev/ref/spec#Conversions) с результатом, которое является *constant*
- значением результата некоторых *built-in function*'s, таких как `unsafe.Sizeof`, примененных к любому *value*
- `cap` или `len`, примененных к [некоторым expressions](https://go.dev/ref/spec#Length_and_capacity)
- `real` и `imag`, применененных к *complex constant*
- `complex`, примененных к *numeric constant*
- *boolean truth value* представляется с помощью *predeclared constant* `true` и `false`.
- *predeclared identifier* [iota](https://go.dev/ref/spec#Iota) обозначает *integer constant*.

(не важно)

*Numeric constant* представляют точные значения произвольной точности и не переполняются. (не важно)

*Constant*'ы могут быть [typed](https://go.dev/ref/spec#Types) или *untyped*. 

К *untyped constant*'ам относятся:

- *Literal constant*'ы
- `true`
- `false`
- `iota`
- некоторые [constant expressions](https://go.dev/ref/spec#Constant_expressions) содержащие только *untyped constant* операнды

[*Typed constant*'ы](#typed-constantы)

[Default type для untypes constant]()

(не важно)

## *Typed and Untyped Constant*'ы

Go не позволяет выполнять операции, которые смешивают *numeric type*'s. Например, вы не можете сложить `float64` и `int`, и даже `int64` и `int`

```go
var myFloat float64 = 21.54
var myInt int = 562
var myInt64 int64 = 120

var res1 = myFloat + myInt  // Not Allowed (Compiler Error)
var res2 = myInt + myInt64  // Not Allowed (Compiler Error)
```

Необходимо явно привести *variable*'s к одному *type*:

```go
var res1 = myFloat + float64(myInt)  // Works
var res2 = myInt + int(myInt64)      // Works
```

Эта строгость устраняет распространенную причину ошибок в языке C. Но от программистов это требует украшать свой код неуклюжими *numeric conversion*, чтобы ясно выразить их значение.

Каким же образом тогда мы можем писать `const i = 0`? Какой *type* у `0`? Было бы неразумно требовать, чтобы *constant*'ы имели *type conversion* в простых контекстах, таких как `i = int(0)`. Поэтому *numeric constant*'ы работают иначе, чем в других C-подобных языках (и чем variable в Go). Этот дизайн не требует постоянного *constant conversion*, при этом имея возможность писать такие вещи, как `math.Sqrt(2)`.

### *Untyped constant*

Любая *constant* в Go является *untyped*, если явно не указан *type*. При этом может быть указано имя или нет:

- если у нее не указано имя (по сути, это *literal*):

  ```go
  1       // untyped integer constant
  4.5     // untyped floating-point constant
  true    // untyped boolean constant
  "Hello" // untyped string constant
  ```

- если указано имя (через [constant declaration]()):

  ```go
  const a = 1      // untyped integer constant'а, которая обозначает integer literal
  const f = 4.5    // untyped floating-point constant'а, которая обозначает floating-point literal
  const b = true   // ...
  const s = "Hello"
  ```

Хотя мы говорим *integer constant*, *string constant*,.... , они все же являются *untyped*.  Хотя понятно, что значение которое они хранят имеет некоторый тип. Но при этом эти *constant*'ы еще не получили фиксированный *type*, например, `int32` или `float64` или `string`, который заставил бы их подчиняться строгим правилам типизации Go.

Один из способов думать о *untyped constant*'ах состоит в том, что они живут в своего рода идеальном пространстве значений, пространстве менее ограниченном, чем полная система типов Go.

Т.к. `const a = 1` не имеет фиксированного *type*, мы можем присвоить его любой *variable*, *type* которой [assignable](Lang.md#assignability) с *integer*.

```go
func main() {
  const a = 1
  var myInt int = a
  var myFloat float64 = a
  var myComplex complex64 = a
}
```

Хотя `const a = 1` – *untyped*, она все же является *untyped integer*. Поэтому ее можно использовать только там, где разрешено использование *integer* (т.е. [assignable](Lang.md#assignability)). Например, вы не можете присвоить его `string` или `boolean` *variable*.

```go
  const a = 1
  var s string = a  // cannot use a (type untyped int) as type string in assignment
```

Точно так же *untyped floating-point constant*, может использоваться везде, где разрешено *floating-point value*:

```go
const f = 4.5
var myFloat32 float32 = f
var myComplex64 complex64 = f
```



#### *Default type* для *untyped constant*

У *untyped constant*'ы есть *default type*, в который *constant*'а неявно преобразуется, когда требуется значение с явно определенным *type*.

*Default type* для *untyped constant*:

| Constants                   | Default Type |
| :-------------------------- | :----------- |
| *integer* (`10`, `76`)      | `int`        |
| *float* (`3.14`, `7.92`)    | `float64`    |
| *complex* (`3+5i`)          | `complex128` |
| *rune* (`'a'`, `'♠'`)       | `rune`       |
| *boolean* (`true`, `false`) | `bool`       |
| *string* (`"Hello"`)        | `string`     |



Например при использовании *constant*'ы в *short variable declaration*, когда явно не указан *type*, используется *default type*. Т.к. *variable* (не самой *constant*'е) нужен *type*:

```go
  i := 0
  fmt.Printf("%T", i)  // int
```



Также при использовании функции с параметром `interface{}`, например:

```go
func Printf(format string, a ...interface{}) (n int, err error)
```

для создания аргумента используется *default type*:

```go
fmt.Printf("%T", 1) // int
```





### *Typed constant*'ы

*Constant*'e можно явно присвоить *type*:

- явно в [constant declaration](#constant-declaration) (при объявлении *constant*),

  ```go
  const typedInt int = 1  // Typed constant
  ```

- явно при [conversion](#conversion)

  ```go
  const m = string(k)        // m == "x"   (typed string constant)
  ```

- неявно при [variable declaration](https://go.dev/ref/spec#Variable_declarations),

- неявно при [assignment](https://go.dev/ref/spec#Assignments) 

- неявно при использовании как *operand* в *expression*. 

Если *constant*'а не может быть представлена с помощью указанного *type*, то выбрасывается ошибка.

При определении *typed constant*'ы через *constant declaration* выполняется связывание *identifier*'ов c *constant expression*. *Constant expression* должно быть [assignable](Lang.md#assignability) присваиваемоему *type*. Поэтому можно определить *constant*'ы следующих типов:

- `bool`

- unsigned int: `byte/uint/uint8/uint16/uint32/uint64/uintptr`

- signed int: `int/int8/int16/int32/int64`

- `rune`

- floating-point: `float32`, `float64`

- ...

- не только базовых *type*'s (`string`, `bool`, ...), но и *defined type* для которых используется какой-либо базовый *underlying type*.

  ```go
  func main() {
    type Myint int
    const i1 Myint = 1            // typed constant с underlying type `int`
    const i2 = Myint(2)
    fmt.Printf("%T %v\n", i1, i1) // main.Myint 1
    fmt.Printf("%T %v\n", i2, i2) // main.Myint 2
  }
  ```

Определение *constant* других типов сообщит об ошибке во время компиляции:

```go
const u User = User{} // invalid const type User
const p *int = &i     // invalid const type *int
```



*Array type* ([1](#array-type)) не может использоваться для *constant*, также как и *array* не может быть частью *constant expression*. Поэтому в качестве константных *array* нужно использовать *variable*.





К *typed constant*'ами, как и к *variable*, применяются все правила системы типов Go. Например, вы не можете присвоить *float variable* = *typed integer constant*:

```go
var myFloat64 float64 = typedInt  // Compiler Error
```

*Typed constant*'ы не имеют такой гибкости, как *untyped constant*'ы:

- их нельзя присваивать *variable*, у которой тип [assignable](Lang.md#assignability)
- нельзя использовать в математической формуле *constant*'ы с разными *type*'s.

Таким образом, вы должны объявлять *type* для *constant*'ы только, если это необходимо. При этом вы теряете гибкость *untyped constant*.







# Defined type

Пусть определен *defined type* (`type myString`), для которого *underlying type* – один из *predefined type*'s (`string`). 

Мы не можем присвоить: `myString` *variable*  = `string` *variable*  (из-за строгой типизации).

Но при этом можем присвоить: `myString` *variable*  =  *untyped string constant*, т.к. *untypes string constant* [assignable](Lang.md#assignability) c типом `myString`:

```go
func main() {
  type MyString string

  var myVar string = "Hello"
  const myConst = "Hello"

  var t MyString
  t = myConst // Работает, т.к. справа untyped string constant
  t = "Hello" // Работает, т.к. справа untyped string constant
  t = myVar   // Не работает, из-за строгой типизации
  
 }
```









# Constant declaration

*Constant declaration* связывает *list* из *identifier*'ов (имен *constant*) со значениями list'а из [constant expressions](https://go.dev/ref/spec#Constant_expressions). Количество *identifier*'ов должно быть равно количеству *expression*'ов, а *n*-й *identifier* в списке слева привязывается к *n*-ому *expression*'у в списке справа.

<pre>
ConstDecl      = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
ConstSpec      = IdentifierList [ [ Type ] "=" ExpressionList ] .
IdentifierList = identifier { "," identifier } .
ExpressionList = Expression { "," Expression } . 
</pre>

Если *type* указан, все *constant*'ы принимают указанный *type*, и *expression*'ы должны быть [assignable](https://go.dev/ref/spec#Assignability) к этому *type*. 

```go
const Pi float64 = 3.14159265358979323846
const u, v float32 = 0, 3    // u = 0.0, v = 3.0
```

Если *type* не указан, *constant*'ы принимают отдельные *type* соответствующих *expression*. 

```go
const Pi = float64(3.14)
```

Если *expression value* являются [untyped constant'ами](#constant), объявленные *constant*'ы остаются *untyped*, а *constant identifier*'s обозначают *constant value*'s. Например, если *expression* является *floating-point literal*, *constant identifier* обозначает *floating-point constant*'у, даже если дробная часть литерала равна нулю.

```go

const zero = 0.0         // untyped floating-point constant
const a, b, c = 3, 4, "foo"  // a = 3, b = 4, c = "foo", untyped integer и string constant'ы

```

В *constant declaration* с круглыми скобками (`()`), *expression list* может быть опущен у любого, кроме первого `ConstSpec`. Такой пустой *expression list*  эквивалентен текстовой подстановке первого предыдущего непустого *expression list*'а и его типа, если он есть. Таким образом, пропуск *expression list*'а эквивалентен повторению предыдущего *expression list*'а. Количество *identifier*'ов должно быть равно количеству элементов в предыдущем *expression list*'е. 

```go
  const (
    a = 1
    b      // b == 1
    c      // c == 1
  )
```

Вместе с [`iota` constant generator](#iota) этот механизм позволяет легко объявить последовательные значения:

```go
const (
	Sunday = iota  // 0
	Monday         // 1
	Tuesday        // ...
	Wednesday
	Thursday
	Friday
	Partyday
	numberOfDays  // 7, эта constant – non-exported
)
```

*Constant*'ы создаются во время компиляции, даже если они определены как локальные в функциях. В *constant decalaration* ([1](#constant-declaration)) можно использовать только *constant expression* ([1](#constant-expression)), которое вычисляется компилятором.

*Constant*'ы, как и следовало ожидать, не могут быть изменены. То есть вы не можете повторно присвоить *constant*'е другое значение после ее инициализации:

```go
const a = 123
a = 321 // Compiler Error (Cannot assign to constant)
```



# Iota

В [constant declaration](#constant-declarations) предварительно объявленный идентификатор `iota` представляет последовательные *untyped integer constant*'ы. Его значением является индекс (номер по порядку) соответствующего [`ConstSpec`](#constant-declaration) в этом *constant declaration*, начиная с 0. Другими словами, значением `iota` является номер строка, в которой оно находится в *constant declaration*.

Его можно использовать для построения набора связанных *constant*:

```go
const (
	c0 = iota  // c0 == 0
	c1 = iota  // c1 == 1
	c2 = iota  // c2 == 2
)

// Либо так даже проще, засчет текстовой подстановке предыдущего непустого expression list'а (#constant-declaration)
const (
	c0 = iota  // c0 == 0
	c1         // c1 == 1
	c2         // c2 == 2
)

const (
	a = 1 << iota  // a == 1  (iota == 0)
	b = 1 << iota  // b == 2  (iota == 1)
	c = 3          // c == 3  (iota == 2, unused)
	d = 1 << iota  // d == 8  (iota == 3)
)

const (
	u         = iota * 42  // u == 0     (untyped integer constant)
	v float64 = iota * 42  // v == 42.0  (float64 constant)
	w         = iota * 42  // w == 84    (untyped integer constant)
)

const x = iota  // x == 0
const y = iota  // y == 0
```

По определению, многократное использование `iota` в одном и том же `ConstSpec` имеет одно и то же значение (т.е. повторное использование `iota` не инкрементирует его значение):

```go
const (
	a0, b0 = 1 << iota, 1<<iota - 1  // a0 == 1, b0 == 0  (iota == 0)
	a1, b1                           // a1 == 2, b1 == 1  (iota == 1)
	_, _                             //                   (iota == 2, unused)
	a3, b3                           // a3 == 8, b3 == 7  (iota == 3)
)
```

`iota` не обязательно должна появляться в 1 строке, при этом ее значение также равно номеру строки в *constant declaration*:

```go
const (
  A int = 1
  B int = 2
  C int = iota + 1  // C == 3
  D                 // D == 4
  E                 // E == 5
)
```

Значения `iota` можно игнорировать, присвоив *blank identifier*:

```go
const (
  _ int = iota
  A // 1
  B // 2
  C // 3
  D // 4
  E // 5
)
```









# Constant expression

*Constant expression* может содержать только [constant](#constant) операнды и вычисляются во время компиляции.

Т.е. это не работает:

```go
func addMinutes(minutes int) {
	const more = minutes + 60
	return more
}
```

т.к. *constant expression* вычисляется по *runtime variable* `minutes`.



*Untyped constant* (*boolean, numeric, string*) могут использоваться в качестве операндов везде, где допустимо использование операнда соответствующего типа. 

При этом в любом *expression* (и в *constant expression*) можно смешивать различные *untyped constant*'ы, если они совместимы (??? [assignable](Lang.md#assignability)) друг с другом. Можно думать, что все *numeric constant*'ы, будь то *integer, floating-point, complex, rune*, живут в своего рода едином пространстве.



Правила по выводу типов в результате *constant expression*:

1. [Сравнение](https://go.dev/ref/spec#Comparison_operators) *constant* всегда возвращает *untyped boolean constant*.

   ```go
   const a = 7.5 > 5          // a == true (untyped boolean constant)
   const h = "foo" > "bar"    // h == true  (untyped boolean constant)
   ```

   

2. [Shift expression](https://go.dev/ref/spec#Operators)

   - Правый операнд *shift expression* должен либо иметь *unsigned integer type* либо быть *untyped constant*, которая может быть представлена значением типа `uint`.
   - Левый операнд должен либо иметь *integer type*, либо быть *untyped constant*, которая может быть представлена значением типа `int`.
   - (a) Если левый операнд [shift expression](https://go.dev/ref/spec#Operators) является *untyped constant*, результатом является *integer constant*; (b) в противном случае это *constant*'а того же типа, что и левый операнд, который должен иметь [integer type](https://go.dev/ref/spec#Numeric_types).

   ```go
   const d = 1 << 3.0         // d == 8     (untyped integer constant) (правило a)
   const e = 1.0 << 3         // e == 8     (untyped integer constant) (правило a)
   const d = 32 >> 3.0        // d == 4     (untyped integer constant) - 3.0 может быть представлена `uint`
   
   const e = 10.50 << 2       // illegal    (10.50 не может быть представлено значением типа `int`)
   const f = 64 >> -2         // illegal    (-2 не может быть представлено значением типа `uint`)
   const f = int32(1) << 33   // illegal    (constant 8589934592 переполняет int32) (правило b)
   const g = float64(2) >> 1  // illegal    (float64(2) – typed floating-point constant) (правило b)
   ```

   

3. Любая другая *operation* над *untyped constant*'ами возвращает *untyped constant*'у того же самого вида. [Виды *untyped constant*](#виды-constant): *boolean, numeric (integer, rune, floating-point, complex), string*. Например, *expression* с двумя *untyped integer constant* возвращает также *untyped integer constant*'у, результат *truncate* к *integer*.

4. Если *untyped numeric constant*'ы в *binary operation* (кроме [shift expression](https://go.dev/ref/spec#Operators)) – разных видов, то результат будет иметь вид того операнда, который появляется позже в этом списке: `integer < rune < floating-point < complex`.

   ```go
   const a = 2 + 3.0          // a == 5.0   (untyped floating-point constant)
   const b = 15 / 4           // b == 3     (untyped integer constant) (3 правило)
   const c = 15 / 4.0         // c == 3.75  (untyped floating-point constant) (4 правило)
   const Θ float64 = 3/2      // Θ == 1.0   (type float64, 3/2 – integer operation)
   const Π float64 = 3/2.     // Π == 1.5   (type float64, 3/2. – float operation)
   
   const k = 'w' + 1          // k == 'x'   (untyped rune constant)
   
   const Σ = 1 - 0.707i       //            (untyped complex constant)
   ```



Пример вычисления сложного *constant expression*:

```
result = 4.5 + (10 - 5) * (3 + 2)/2

4.5 + (10 - 5) * (3 + 2)/2
          ↓
  4.5 + (5) * (3 + 2)/2
          ↓
   4.5 + (5) * (5)/2
          ↓
     4.5 + (25)/2
          ↓
      4.5 + 12
          ↓
        16.5
```



Можно даже написать:

```go
var f = 'a' * 1.5
```



(не важно)



*Constant expression* всегда вычисляются точно; промежуточные значения и сами *untyped constant* могут хранить значения за пределами размеров, которые поддерживатся каким-либо *type* в языке. *Numeric constant*'ы живут в числовом пространстве произвольной точности. И над такими значениями большой точности можно выполнять арифметические операции. 

Ниже приведены корректные *constant declaration*:

```go
const Huge = 1 << 100 // Huge == 1267650600228229401496703205376 (untyped integer constant, больше чем int)
```

Но мы не можем его использовать в *assignment* или даже напечатать. Т.е. их необходимо преобразовать обратно к нормальному диапазону значений для конкретного *type* при использовании (при *assignment*, передача в качестве параметра). Если значение превышает нормальный диапазон для *type*, то при компиляции произойдет ошибка. Этот оператор даже не скомпилируется:

```go
                         // преобразование в default type - `int`
  fmt.Println(Huge)      // constant 1267650600228229401496703205376 overflows int
```

Но `Huge` может быть полезно: мы можем использовать его в выражениях с другими *constant*'ами и использовать значение этих выражений, если результат может быть представлен в диапазоне `int`:

```go
const Four int8 = Huge >> 98 // Four == 4 (тип int8)
```

Аналогичным образом, *floating-point constant*'ы могут иметь очень высокую точность, так что арифметические операции с ними будут более точными. *Constant*'ы, определенные в *package* `math`, содержат намного больше цифр, чем доступно в  `float64`. Вот определение `math.Pi`:

```go
Pi  = 3.14159265358979323846264338327950288419716939937510582097494459
```

Когда это значение присваивается *variable*, часть точности будет потеряна:

```
    pi := math.Pi
    fmt.Println(pi)  // 3.141592653589793
```

Наличие такого количества доступных цифр означает, что вычисления, подобные `Pi/2`, будут иметь большую точность до тех пор, пока результат не будет присвоен. 







Делитель для операции деления или нахождения остатка не должен быть равен нулю:

```go
3.14 / 0.0   // недопустимо: деление на ноль
```

Значения *typed constant* всегда должны быть точно [representable](Lang.md#Representability)  значением соответствующего *constant type*. Это также указано в правилах *conversion* для *constant* (раздел [conversion](Lang.md#conversion)) (для *non-constant value* там указаны другие правила). Следующие *constant expression* недопустимы:

```go
uint(-1) // -1 – не representable как uint
int(3.14) // 3.14 – не representable как uint

const Huge = 1 << 100 // Huge == 1267650600228229401496703205376 (untyped integer constant, больше чем uint64)
int64(Huge) // 1267650600228229401496703205376 не может быть представлен как int64, т.к. не помещается в него без переполнения

const Four int8 = 4
Four * 300 // операнд 300 не может быть представлен как int8 (тип Four)
Four * 100 // продукт 400 не может быть представлен как int8 (тип Four)
```



(не важно)



# Pattern

*Constant* безопаснее, чем *variable*.

Используйте *constant* везде, где это возможно. При этом вы защищены от случайного изменения значения.

*Global variable* — это плохо. *Variable* должны принадлежать к наименьшему возможному *scope*. 

Но нет *ничего плохого* в объявлении *global constant*, т.к. ее значение не меняется. Конечно, если *constant* используется только в одном месте, иметь смысл объявить ее там. 