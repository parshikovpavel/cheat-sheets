# Общие особенности

* компилируемый язык

* имеет сборщик мусора

* CSP-style, асинхронная модель, возможность в процессе выделять потоки и писать параллельные программы (аналог OpenMP)

* на выходе получаем готовый бинарник без зависимостей, которые надо было бы скачивать менеджером зависимостей

* строгая статическая типизация, типы переменных не динамические, и поэтому нет оверхеда по памяти на хранение типа.

# Конфигурирование

Запуск сервера с документацией:

```bash
godoc -http=:8000
```

Сборка программ на языке Go выполняется в два этапа: 

- компиляция
- компоновка.

Стандартный компилятор языка Go называется `gc`, в состав его инструментов входят программы: 

- `5g`, `6g` и `8g` – для компиляции
- `5l`, `6l` и `8l` – для компоновки 
- `godoc` – для просмотра документации.

Инструмент сборки программ `go` автоматически выбирает нужный компилятор и компоновщик и выполняет:

- собирает локальные программы и пакеты
- загружает, собирает и устанавливает сторонние программы и пакеты.

Программы записываются в виде текста на UTF-8. Все ключевые слова и операторы языка Go записываются символами ASCII, однако идентификаторы в языке Go могут начинаться с любых алфавитных символов *Unicode* и содержать любые алфавитные символы *Unicode* или
цифры. Поэтому идентификаторы могут определяться на любом языке (в том числе русском).

## Проблемы с загрузкой библиотек

- проект должен быть за пределами `GOPATH` 

  ```bash
  $ go mod init go.avito.ru/bx/service-user-search-history
  ```

- команда

  ```bash
  $ go mod vendor
  $ avito service run
  ```

- в GoLand нужно проверить, что в `Settings/Preferences | Go | Go Modules` включена поддержка *Go modules*, и сделать `File -> Invalidate Caches / Restart` 

- почистить кеш go modules:

  - 

## Создание сценариев на Go

Существует возможность создания сценариев на Go, начинающихся со строки `#!` ([shebang]()). Для этого необходимо использовать инструмент для
компиляции и запуска программ. Возможные варианты:

- `gonow` (`github.com/kless/gonow`)
- `gorun` (`wiki.ubuntu.com/gorun`)

Чтобы создать сценарий на Go необходимо:

- добавить строку `#!/usr/bin/env gonow` или `#!/usr/bin/env gorun` в начало файла с расширением `.go`, содержащем функцию `main()` (в пакете `main`). 
- дать файлу права на выполнение (`chmod +x`).

При первом запуске команда `gonow` или `gorun` скомпилирует файл с расширением `.go` и запустит его. 

При последующих запусках перекомпиляция будет выполняться, только если исходный файл `.go` изменился с момента предыдущей компиляции.

Такие сценарии удобны для решения задач системного администрирования.

## Environment variable's

### `GOROOT`

Если инструмент для сборки программ `go` выдает ошибку:

```bash
go version
```

то необходимо в файле `~/.bash_profile`:

```bash
export GOROOT=/usr/local/opt/go/libexec # 1. Создать environment variable, содержащую путь 
                                        #    к каталогу установки Go
export PATH=$PATH:$GOROOT/bin           # 2. В переменную PATH включить путь $GOROOT/bin (???)
```

### `GOPATH`

Местоположение [*workspace*](#организация-кода) определяется через `GOPATH` *environment variable*. По умолчанию, это каталог `$HOME/go`.

Если *workspace* расположен в другом месте, то необходимо его указать в `GOPATH`

```bash
export GOPATH=$HOME/repos/goeg
```

Если нужно хранить программы в нескольких каталогах, то в `GOPATH` *environment variable* можно вписать несколько путей к каталогам, разделенных двоеточием:

```bash
export GOPATH=$HOME/app/go:$HOME/goeg
```

Получить текущий `GOPATH`:

```
go env GOPATH
```

### `PATH`

Для удобства необходимо добавить каталог `$GOPATH/bin` в `PATH`:

```bash
export PATH=$PATH:$GOPATH/bin
```



# Типизация

Язык Go использует строгую типизацию. Это не позволяет, например, сложить значения типов `int32` и `int16` без явного их преобразования.







# Lexical elements

## Literal

Literal (литерал) является описанием *constant*'ы (константы). 



## Rune literal

Rune literal (строковый литерал) – *integer value*, которое задает *Unicode code point*. Записывается в *single quote*'s, например, `'x'`или `'\n'`. 

По умолчанию, *rune literal* приводится к типу `rune` (псевдоним для `int32`).

```go
tst := 'a'
fmt.Println(reflect.TypeOf(tst)) // int32
```

Однако может быть приведено к любому *integer type*, необходимо чтобы значение поместилось в этот integer type:

```go
var a byte

a = 'w'  // OK
a = 'ц'  // constant 1094 overflows byte
```

## String literal

*String literal* является описанием *string constant*'ы. 

# Constant

В Go есть *constant*'ы:

- *boolean*
- *numeric*, к ним относятся:
  - *rune*
  - *integer*
  - *floating*
  - *complex*
- *string*. 

*Constant*'ы создаются во время компиляции, даже если они определены как локальные переменные в функциях. В *constant decalaration* ([1](#constant-declaration)) можно использовать только *constant expression* ([1](#constant-expression)), которое вычисляется компилятором.

*Constant*'e можно явно присвоить тип при объявлении *constant*, объявлении *variable*, как *operand* в *expression* и т.д. Если *constant*'а не может быть представлена с помощью указанного типа, то выбрасывается ошибка.

У *constant*'ы есть тип *by default*, в который *constant*'а неявно преобразуется, когда требуется значение с явно определенным типом, например *short variable declaration*  `i := 0`. 

Типы *constant* *by default*:

- *boolean* – `bool`
- *rune* – `rune`
- *integer* – `int`
- *floating-point* – `float64`
- *complex* – `complex128`
- *string* – `string`





# Block

*Block* – последовательность *declaration*'s и *statement*'s, заключенных в фигурные скобки.

<pre>
Block = "{" StatementList "}" .
StatementList = { <a href="#statement">Statement</a> ";" } .  
</pre>

Выделяют следующие *block*'и:

- явно объявленные *block*'и в коде с помощью `{}`
- неявно объявленные *block*'и:
  - *Universe block* – весь код.
  - *Package block* – код в *package*. 
  - *File block* – код в *file*.
  - `if`, `for` и `switch`  *statement*'s – отдельный *block*
  - *clause* (`case`) внутри `switch` и `select` – отдельный *block*

# Declarations и scope

<pre>
Declaration   = <a href="#constant-declaration">ConstDecl</a> | <a href="#type-declaration">TypeDecl</a> | <a href="#variable-declaration">VarDecl</a> .
TopLevelDecl  = Declaration | <a href="#function-declaration">FunctionDecl</a> | <a href="#method-declaration">MethodDecl</a> .  
</pre>

Один и тот же *identifier* не может быть объявлен одновременно на уровне *file* и *package block*.

*Scope* любого *identifier* определяется [block](#block)'ом, в котором он объявлен:

- *Scope* любой *constant*'ы, *type*, *variable* и *function* (но не *method*), объявленных на *top level* (вне *function*), – *package block*.
- *Scope* любого [`PackageName`](#import)  из директивы [`import`](#import-declaration) – *file block*.
- *Scope* любого *method receiver*'а, *function parameter*'а и *result variable* – *function body*.

- *Scope* любой *constant* или *variable* внутри *function* начинается с точки *declaration* и до конца самого вложенного *block*'а ([1](#block))
- *Scope* любого `type` внутри *function* начинается с точки *declaration* и до конца самого вложенного *block*'а ([1](#block)).

*Identifier*, объявленный внутри *block*'а, может быть переобъявлен в любом внутреннем *block*'е. До тех пора пока внутренний *identifier* находится в *scope*, он перекрывает внешние *identifier*'s.

## Identifier

### Predeclared identifier

*Predeclared identifier* всегда объявлены в [universe block](#block).

````
Types:
	bool byte complex64 complex128 error float32 float64
	int int8 int16 int32 int64 rune string
	uint uint8 uint16 uint32 uint64 uintptr

Constants:
	true false iota

Zero value:
	nil
````

#### `nil`

`nil` – *Predeclared identifier* (!!!)

*Identifier* `nil` используется как значение для пустых *pointer*'ов и пустых значений.

`nil` может использоваться в операциях сравнения и присваивания. У `nil` нет методов.

`nil` играет почти ту же роль, что и `null` в PHP и Java.

### Exported identifier

Чтобы получить доступ к *identifier* из другого *package*, он должен быть объявлен как *exported*. 

*Identifier* является *exported*, если выполняется два условия:

1. первый символ *identifier*'а – это буква в *Unicode upper case*; И!!!!
2. *identifier* объявлен в *package block* ([1](#block)) или является *field* ([1](#struct-type)) или *method* ([1](#method-declaration)) (т.е. *field* и *method* могут быть *exported*, и должны в этом случае начинаться с буквы в *Unicode upper case*)

Все остальные *identifier*'s – *non-exported*.

## *Constant declaration*

<pre>
ConstDecl      = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
ConstSpec      = IdentifierList [ [ Type ] "=" ExpressionList ] .
IdentifierList = identifier { "," identifier } .
ExpressionList = Expression { "," Expression } . 
</pre>


[Constant](#constant)

*Array type* ([1](#array-type)) не может использоваться для *constant*, также как и *array* не может быть частью *constant expression*. Поэтому в качестве константных *array* нужно использовать *variable*:





## `type` *declaration*

В Go отсутствуют такие понятия, как классы и наследование (отношение IS-A). В языке Go поддерживается возможность определения пользовательских *type*'s и чрезвычайно простой способ агрегации типов (отношение Has-A).

Обеспечивается возможность полного отделения типов данных от их поведения и поддерживается динамическая (утиная) типизация. Динамическая (утиная) типизация – мощный механизм абстракций, позволяющий обрабатывать значения (например, передаваемые функциям) с помощью предоставляемых ими методов, независимо от их фактических типов.

Имеется возможность определять пользовательские типы данных, опираясь на фундаментальные типы. А также определять функции для работы с ними.

Объявление `type` привязывает идентификатор (*type name*) к какому-то фундаментальному [*type*](#types). 

Существует два вида объявлений `type`: 

- объявление *alias* (псевдоним, *alias declaration*, `AliasDecl`)
- определение *type* (*type definition*, `TypeDef`) 

```
TypeDecl = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
TypeSpec = AliasDecl | TypeDef .
```

### Объявление *alias*

Объявление *alias* привязывает идентификатор `identifier` к какому-то типу `Type` (фундаментальному или полученного из фундаментального)

```
AliasDecl = <identifier> = <Type> .
```

```go
type nodeList = []*Node  // nodeList и []*Node - один и тот же type
```

Указанный идентификатор выступает в качестве *alias*'а для *type*.

Такие *alias*'ы, привязанные к одному *type*, могут использовать взаимозаменяемо, однако они не могут иметь *method*'ы. 

### Определение *type*

Определение *type* создает новый *type* с тем же *underlying type* (базовым типом) и операциями, что и указанный тип `Type`, и связывает с ним идентификатор `identifier`.

```
TypeDef = <identifier> <Type>
```

```go
type TreeNode struct {
	left, right *TreeNode
	value *Comparable
}

type Block interface {
	BlockSize() int
	Encrypt(src, dst []byte)
	Decrypt(src, dst []byte)
}
```

Новый *type* называется *defined* (определенный) *type* . Он отличается от любого другого *type*, в том числе от *type*, из которого он создан. То есть такие *type*'s, даже с одинаковой структурой, не могут использоваться взаимозаменяемо.

*Defined type* может иметь связанные с ним *method*'ы. Он не наследует никаких методов, привязанных к указанному `Type`, но *method set* (набор методов) для типа `interface` и элементов (!!!) *composite type* (составного типа) остается неизменным:

```go
// Mutex - это тип данных с двумя методами: Lock и Unlock.
type Mutex struct { /* Поля Mutex */}
func (m *Mutex) Lock () { /* Реализация Lock */}
func (m *Mutex) Unlock () { /* Реализация Unlock  */}

// NewMutex имеет те же поля, что и Mutex, но его method set пуст.
type NewMutex Mutex

// Method set базового типа PtrMutex - *Mutex остается неизменным,
// но method set PtrMutex – пуст.
type PtrMutex *Mutex

// Method set *PrintableMutex содержит методы
// Lock и Unlock, которые привязаны к его встроенному полю Mutex.
type PrintableMutex struct {
	Mutex
}

// MyBlock - это interface type, который имеет тот же самый method set, что и Block (определен чуть выше).
type MyBlock Block 
```

Пример определения нового *type* для *slice* со значениями типа `interface{}`:

```go
type Stack []interface{}
```

### Конструктор

Go не является традиционным ООП языком, `struct` не обладают всеми свойствами объектов и и не могут иметь конструктор.

Однако *variable* любого типа (в том числе `struct`) инициализируются *zero value*. А *composite type*'s инициализируются рекурсивно. 

Если нужно получать *variable*, инициализированную некоторым значением, можно вручную написать функцию-конструктор и вызывать ее явно. Например, в пакете `errors` есть функция `New()`, которая возвращает *variable* встроенного типа `error`:

```go
err := errors.New("Description")
```



## *Variable declaration*

*Variable declaration* создает одну или более *variable*'s, привязывает к ним *identifier*'s и дает им *type* и *initial value*.

<pre>
VarDecl     = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
VarSpec     = IdentifierList ( Type [ "=" ExpressionList ] | "=" ExpressionList ) .  
</pre>

```go
var i int
var U, V, W float64
var k = 0
var x, y float32 = -1, -2
var (
	i       int
	u, v, s = 2.0, 3.0, "bar"
)
var re, im = complexSqrt(-1)
var _, found = entries[name]  // map lookup; only interested in "found"
```

Если `ExpressionList` не указан, *variable* инициализируется *zero value*.



Виды объявлений:

- *Short variable declaration* (сокращенное объявление переменной). Одновременно объявляет и инициализирует переменные. 

  ```go
  a := "b"; // Без указания типа
  a := string("b") // С указанием типа
  ```

- Полная форма объявления.

  ```go
  var a = "b"; // Без указания типа
  var a string = "b" // С указанием типа
  ```

Указание типа переменной:

- можно не указывать, в этом случае компилятор Go автоматически определит его по присваиваемому значению. 

  В этом случае переменная `a` будет иметь тип `string`:

  ```go
  a := "b"; // Сокращенная форма
  ```

- можно указывать:

  ```go
  a := string("World!") // Сокращенная форма
  ```



После объявления переменной (с указанием типа или автоматическим определением типа), в следствие строгой типизации, ей могут присваиваться значения только того же типа.

## *Function declaration*

Смотреть [function type и `Signature`](#function-type)

Объявление *function* связывает *function name* с ее *signature* и *body*.

<pre>
FunctionDecl = "func" FunctionName <a href="#function-type">Signature</a> [ FunctionBody ] .
FunctionName = identifier .
FunctionBody = <a href="#block">Block</a> .  
</pre>

```go
func min(x int, y int) int {
	if x < y {
		return x
	}
	return y
}
```

Открывающая фигурная скобка ставится на одной строке с `Signature`.

Существует два способа передачи *parameter*'ов (и *receiver*'а):

- *by value* (`T`). Внутри *function* мы используем копию оригинального значения. Любые изменения переменной, произведенные внутри функции, никак не отразятся на оригинальном значении.
- *by pointer* (`*T`). Внутри *function* мы можем изменить оригинальное значение, на которое ссылается *pointer*.



Существуют следующие причины передачи *parameter*'ов (и *receiver*'а) *by pointer*:

- если необходимо в *function* измененить оригинальное значение фактического параметра. 
- в Go не используется техника *copy-on-write* (????). Для маленьких по размеру *type* (например, для которых *underlying type* – `struct` из нескольких значений `int` или `string`) – можно использовать передачу *by value*. Для значений большого типа, намного дешевле передать в параметре *pointer* на это значение, чем копировать само значение. Потому что операция передачи указателя (обычно 32- или 64-битного значения) намного дешевле. И это имеет смысл даже для методов, не изменяющих значения *parameter*'ов (или *receiver*'а). 



В соответствии с соглашениями, принятыми в языке Go, значение ошибки (или успеха когда `error = nil`) типа `error` возвращается в последнем (или единственном) значении, возвращаемом функцией или методом.

```go
item, err := haystack.Pop()
```

## Объявление *method*'а

Смотреть [function type и `Signature`](#function-type).

*Method* – это *function* с *receiver*'ом (приёмником). Объявление *method*'а связывает *method name* с *signature*, *body* и *base type receiver*'а. 

При объявлении *method*'а, после ключевого слова `func` и перед `MethodName`, указывается *receiver*, заключенный в круглые скобки `()`.

```
MethodDecl = "func" Receiver MethodName Signature [ FunctionBody ] .
Receiver   = ( [ identifier ] Type ) .
MethodName = identifier .
FunctionBody = Block .
```

*Receiver* должен иметь:

-  [defined type](#определение-type) (`T`). В этом случае *receiver* передается *by value* и любые изменения, произведенные в *receiver*'е, никак не отразятся на оригинальном значении.
- или быть *pointer*'ом на [defined type](#определение-type) (`*T`). В этом случае *method* может изменять значение *receiver*'а.

Этот тип `T` (*base type receiver*'а) не может быть *pointer* или `interface`, и он должен быть определен в том же *package*, что и *method*. Говорят, что *method* привязан к своему *base type receiver*'а и доступен только на *selector*'ах для типов `T` и `*T`.

В других языках *receiver* обычно имеет имя `this` или `self`. В языке Go тоже можно использовать такие имена, но считается, что это не соответствует стилю Go.

Если значение *receiver*'а не используется в методе, для него можно не указывать в объявлении `identifier`.

Пример: привязать к *receiver* типу `*Point` (*base type* – `Point`) методы `Length` и `Scale`.  

```go
func (p *Point) Length() float64 {
	return math.Sqrt(p.x * p.x + p.y * p.y)
}

func (p *Point) Scale(factor float64) {
	p.x *= factor
	p.y *= factor
}

func (stack Stack) Cap() int {
    return cap(stack)
}
```





# Type

*Type* может задаваться:

- именем типа (`TypeName`). Возможны варианты:
  - Есть набор предопределенных *type name*'s (`int`, `bool`, ...). 
  - Другие вводятся с помощью [объявления type](#объявление-type). В этом случае у этого типа `T` есть *underlying type* (базовый тип)
-  Литералом типа (`TypeLit`). Так задаются *composite types* (составные типы) – `array`, `struct`, *pointer*, `func`, `interface`, `slice`, `map` и `channel` .

```
Type      = TypeName | TypeLit | "(" Type ")" .
TypeName  = identifier | QualifiedIdent .
TypeLit   = ArrayType | StructType | PointerType | FunctionType | InterfaceType |
	    SliceType | MapType | ChannelType .
```

Каждый type имеет некоторое [zero value](#zero-value).

## `string`

Строки – *immutable* (неизменяемы): после создания невозможно изменить содержимое строки.

Строки хранятся в кодировке UTF-8 и символы строки могут быть представлены двумя и более байтами. 

При обращении к определенной позиции в строке по индексу `str[1]` возвращается байт (тип `byte`), хранящийся в этой позиции. 

### Итерирование `string`

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



## Числовые типы

```
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts
complex128  the set of all complex numbers with float64 real and imaginary parts

byte        alias for uint8
rune        alias for int32

uint     either 32 or 64 bits
int      same size as uint
uintptr  an unsigned integer large enough to store the uninterpreted bits of a pointer value
```

## `struct` тип

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

### *Embedded field*

*Field*, объявленное с указанием `TypeName`, но без указания `IdentifierList`, называется *embedded field* (встроенным полем). 

*Embedded field* может быть:

- [`TypeName`](#types) (e.g. `T`)
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

### *Promoted field* и *method*

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

-  *promoted fields* не могут использовать в *composite literal*. Нужно явно описать весь *embedded field*:

  ```go
  a := A{field: 1} // Error!!!
  a: = A{B{field: 1}} // OK!
  ```

### *Tag*

После *field declaration* ([`FieldDecl`](#struct-тип)) можно указать `Tag` в виде [*string literal*](#string-literal)'а.

*Field tag* можно извлечь с помощью [reflection interface](). И они используются некоторыми пакетами, для описание полей данных, полученных по некоторому протоколу (*protobuf*, *json*).

С точки зрения языка, tag игнорируется. 

```go
struct {
	name string  "any string"
  microsec  uint64 `protobuf:"1"`
}
```



### Примеры

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



















 

## Коллекции

### `array` type

`array` – это нумерованная последовательность элементов одного *type*. 

```
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

Длина является частью *type* массива. Поэтому в качества параметра функции может подставляться только массив определенного размера. Поэтому массивы практически никогда не используются напрямую. А используются через `slice`.

Длину массива можно узнать с помощью встроенной функции `len` ([1](#len)). Элементы имеют индексы от `0` до `len(a)-1`.

 `array` всегда одномерны, но в одномерный `array` могут быть вложены другие одномерные `array` – получается многомерный `array`.

```
[32]byte
[2*N] struct { x, y int32 }
[1000]*float64
[3][5]int
[2][2][2]float64  // same as [2]([2]([2]float64))
```

### `slice`

`slice` – наиболее часто используемая коллекция. 

`slice` (срез) – это дескриптор для непрерывного сегмента *underlying* `array` (базового массива), обеспечивающий доступ к пронумерованной последовательности элементов из этого *array*. Т.е. *slice* – это такой тип, который может хранить все возможные срезы *array* с элементами указанного *type*. 

После инициализации срез всегда связан с *underlying* `array` (базовым массивом), который содержит его элементы. Таким образом, *slice* разделяет хранилище со своим `array` и с другими `slice`'s того же `array`.

*Underlying* `array` может выходить за пределы конца `slice`. *Capacity* – это длина `slice` + длина `array` за пределами `slice`; `slice` длиной до этой *capacity* можно создать из исходного `slice`, применив [*slice expression*](#slice-expression). Значение *capacity* можно определить с помощью функции `cap()`.

Определение одномерного *slice*:

```
SliceType = "[" "]" ElementType 
```

```go
[]Тип        // без инициализации
[]Тип{1,2,3} // с инициализацией
```

```go
a := []string{"1", "2", "3"} // сокращенная форма
var b = []int{2, 3, 5}       // полная форма
```

Размер *slice* указывать не нужно, компилятор сам может подсчитать количество элементов.

Чтобы определить многомерный *slice*, необходимо в качестве `Тип` указать другой `slice`:

```go
[][]Тип
```

```go
var bigDigits = [][]string{
    {"1","2","3"},
    {"4","5","6"},
}
```

### Общее

#### Обращение к элементам

Аналогично другим языкам, получение n-го элемента среза:

```go
slice[n]
```

## *Pointer* тип

*Pointer* – это переменная, хранящая адрес в памяти, где находится другое значение. *Pointer*'ы в Go практически ничем не отличаются от *pointer*'ов в языках C и C++, за исключением того, что Go не поддерживает арифметику с указателями. *Pointer* объявляется добавлением символа звездочки `*` перед *type* `*T`. 

Переменной типа *pointer* (`*T`) может быть присвоен указатель на любую переменную типа `T`. Тип `T` называется *base type* (бызовым типом) *pointer*'а. 

Значение *by default* – `nil`.

```
PointerType = "*" BaseType .
BaseType    = Type .
```

Примеры объявлений:

```go
*Point
*[4]int
```



## *Function type*

Смотреть также [объявление function](#function-declaration)

Функции и методы в языке Go определяются с помощью ключевого слова `func`. 

Тип `func`:

```
FunctionType   = "func" Signature .
Signature      = Parameters [ Result ] .
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .
ParameterList  = ParameterDecl { "," ParameterDecl } .
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
IdentifierList = identifier { "," identifier } .
```


Функции и методы могут возвращать ноль или более значений. 

В списке *parameter*'ов и *result*'ов (`ParameterList`) имена переменных (`IdentifierList`) должны быть:

- либо все указаны (отдельно в списке *parameter*'ов и отдельно в списке *result*'ов)
- либо ни одно из имен не указано (отдельно в списке *parameter*'ов и отдельно в списке *result*'ов):

Список *parameter*'ов и список *result*'ов (`Parameters`) всегда заключаются в круглые скобки `()`. *Parameter*'ы и *result*'ы указываются через запятую `,`.

```go
func(x int) int // имена parameter'а указаны, result'а - нет 
func(a, b int, z float64) (success bool) // имена parameter'ов и result'ов указаны
func(int, int, float64) (float64, *[]int) // имена parameter'ов и result'ов не указаны
func(n int) func(p *T) // имена parameter'а указаны, result'а - нет
```

Только в одном случае скобки `()` не обязательны – в списке *result*'ов указан один `Type` без указания имени (`identifier`), он может быть записан как тип без скобок.

```go
func(a, _ int, z float32) bool // Type result'а записан без скобок
func(a, b int, z float32) (bool) // или можно Type result'а записать в скобках
```

К последнему *parameter*'у можно добавить префикс  `...` и тогда функция может быть вызвана с нулем или более аргументов для этого параметра. Такая функция называется *variadic* (вариативной). 

```go
func(prefix string, values ...int)
```

## *Interface* *type*

`interface`  – описывает некоторый *method set*.

`interface` является абстрактными типом и не позволяет создавать его экземпляры.

`interface` лежит в основе *duck typing*, поддерживаемой в Go (аналогично реализовано в Python). Для того чтобы конкретный `type` удовлетворял *interface*'у, достаточно (!!!) чтобы он имел реализацию *method*'ов, определяемых *interfacе*'ом. Т.е. не требуется формально определять связь между `interface` и конкретным `type`. `type` будет автоматически удовлетворять всем `interface`'s, методы которых он реализует.

Реализующий `interface` конкретный `type` может подставляться в те места кода, где требуется этот `interface`. 

```
InterfaceType      = "interface" "{" { ( MethodSpec | InterfaceTypeName ) ";" } "}" .
MethodSpec         = MethodName Signature .
MethodName         = identifier .
InterfaceTypeName  = TypeName .
```

 В определении *interface*'а могут:

- явно указываться спецификации *method*'ов (`MethodSpec`)

  ```go
  interface {
  	Read([]byte) (int, error)
  }
  ```

- или указываться другие *interface*'ы, методы которых будут встроены в этот *interface* (`InterfaceTypeName`). `A` называется *embedding* (встроенным) *interface*'ом в `B`.

  ```go
  type A interface {}
  type B interface {
  	A
  }
  ```

*Method set interface*'а – это объединение явно указанных *method*'ов и *method*'ов *embedding interface*'s.

### Empty interface `interface{}`

Все `type`'s реализуют *empty interface* (пустой интерфейс), т.к. он не требует от `type` реализации ни каких методов:

```go
interface{}
```

*Empty interface* может использоваться для ссылки на значение любого `type` (подобно `void*` в C++)

Таким образом, объявление *slice*, который может хранит элементы любых `type`:

```go
type Stack []interface{}
```

### Internal

Рассмотрим тип:

```go
type Binary uint64

func (i Binary) String() string {
    return strconv.Uitob64(i.Get(), 2)
}

func (i Binary) Get() uint64 {
    return uint64(i)
}
```

в результате присвоения выделяется ячейка памяти:

```go
B := Binary(200)
```

![interface1](img/go/interface1.png)

Если объявить интерфейс:

```go
type Stringer interface {
    String() string
}
```

и привести переменную к типу этого интерфейса:

```go
s := Stringer(b)
```



![interface2](img/go/interface2.png)



то значение будет представлено в виде пары из двух word:

- *pointer* на информацию о типе, хранящемся в интерфейсе. Указывает на *interface table* или *itable*. *itable* относится к *interface type* (`Stringer`), а не к к *dynamic type* (`Binary`). В начале *Itable* расположена информация о типе (*type(Binary)*), затем – список *pointer*'ов на *method*'s. *itable* для интерфейса `Stringer`, в который вложен тип `Binary`, содержит только *pointer*'ы на *method*'ы, которые определены в интерфейсе `Stringer` (метод `String`). *Pointer*'s на те *method*'s, которые отсутствуют в типе `Stringer` (`Binary.Get()`), отсутствуют в этой itable. 
- *pointer* на данные. 



https://research.swtch.com/interfaces

https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go

https://stackoverflow.com/questions/23148812/whats-the-meaning-of-interface/23148998#23148998

https://stackoverflow.com/questions/27178635/cast-a-struct-pointer-to-interface-pointer-in-golang

https://stackoverflow.com/questions/27178635/cast-a-struct-pointer-to-interface-pointer-in-golang/27178682

https://stackoverflow.com/questions/51530429/type-inode-is-pointer-to-interface-not-interface

https://stackoverflow.com/questions/62146693/whats-the-difference-between-interface-pointer-and-interface-value-in-golang

https://stackoverflow.com/questions/59630086/is-empty-interface-in-golang-as-function-argument-is-pass-by-value-or-pointer

https://www.airs.com/blog/archives/277

https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go

https://stackoverflow.com/questions/20874798/does-assigning-value-to-interface-copy-anything

https://stackoverflow.com/questions/13511203/why-cant-i-assign-a-struct-to-an-interface

https://stackoverflow.com/questions/40823315/x-does-not-implement-y-method-has-a-pointer-receiver

https://stackoverflow.com/questions/40823315/x-does-not-implement-y-method-has-a-pointer-receiver

ТУТ!!!



## `map` *type*

`map` – это неупорядоченная (!!!) группа *element*'ов (*value*'s) одного *type* (*element type*), проиндексированные уникальными *key*'s другого *type* (*key type*). Значение неинициализированного `map` – `nil`.

<pre>
MapType     = "map" "[" KeyType "]" ElementType .
KeyType     = <a href="#type">Type</a> .
ElementType = <a href="#type">Type</a> .
</pre>

```go
map[string]int
map[*T]struct{ x, y float64 }
map[string]interface{}
```

Для *key type* должна быть полностью определены операторы `==` и `!=`. 

Поэтому в качестве *key type* можно использовать:

- *Numeric types*
- *String types*
- *Array types*
- *Struct types* 
- *Pointer types*
- `interface`. При этом операторы `==`, `!=`  должны быть определены для динамических значений в ключе (*dynamic key values*)

В качестве *key type* нельзя использовать:

- *Slice types*
- `function`
- `map`
- `slice`

Если передать `map` *by value* в *function* и там изменять ее содержимое через формальный параметр, то изменения будут видны и через фактический параметр (как `object` в PHP).

### Частые операции:

- создание (инициализация):

  - создание через *composite literal*. При этом можно создать пустой `map` или сразу инициализировать его:

    ```go
    var timeZone = map[string]int{
        "a": 1,
        "b": 2,
        "c": 3,
    }
    ```

  - создание пустого `map` через `make`:

    ```go
    make(map[string]int)       // небольшого capacity
    make(map[string]int, 100)  // с указанным начальным capacity
    ```

    Первоначальная *capacity* не ограничивает его *size*: `map` растет автоматически с добавлением *element*'ов, кроме случая когда `map = nil`. `map = nil` – то же самое что пустая `map` , но нельзя добавлять *element*'ы.

- Длина:

  [`len(a)`](#len()) 

- Добавить *element* ([assignment](#assignment))

  `a["b"] = 1` 

- Получить *element* (*[index expression](#index-expression)*)

  `c := a["b"]` 

  Если `map` – `nil` или не содержит ключ `"b"`, то `a["b"]` – [*zero value*](#zero-value) для *element type*.

- Получить *element* и получить флаг, что *element* существует в `map`. Используется *multiple assignment*. Конструкция называется *"comma ok" idiom*.

  ```go
  var c int
  var ok bool
  c, ok = a["b"]
  ```

  Получить *element* и сразу проверить, что *element* существует в `map`.

  ```go
  if c, ok = a["b"]; ok {
    return c;
  }
  ```

  Если нужно получить только флаг, что *element* существует в `map` и сам *element* не нужен, то используем [blank identifier (`_`)](#blank-identifier):

  ```go
  _, ok = a["b"]
  ```

  

- Удалить *element*

  [`delete(a, "b")`](#delete())

  Если `a = nil` или `a[key]` не существует, `delete()` ничего не делает.

### Реализация *set* (множества)

*Set* (множество) можно реализовать как `map` с *element type* – `bool`. 

Операции:

- инициализировать *set*:

  ```go
  set := map[string]bool{
      "Ann": true,
      "Joe": true,
      ...
  }
  ```

- добавить элемент в *set*:

  ```go
  set["a"] = true
  ```

- проверить наличие элемента в *set*:

  ```go
  if set[person] { // будет false если person не в set
      ...
  }
  ```



# Zero value

Когда для *variable* выделяется память, через:

- объявление *variable*
- вызов `new`
- когда создается новое значение (???)
- создание *composite literal*
- вызов `make`

и не обеспечивается явная инициализация, переменной присваивается *zero value* (нулевое значение, значение *by default*). Поэтому в языке
Go не может быть неинициализированных переменных,

*Zero value* зависит от типа:

- для `bool` – `false`
- для `numeric` – `0`
- для `string` – `""`
- для *pointer*, *function*, `interface`, `slice`, *channel* и *map* – `nil`

Для *composite type*'s эта инициализация выполняется рекурсивно, поэтому, например, поля каждого элемента `struct` внутри `array` будут установлены в *zero value*.

После

```
type T struct { i int; f float64; next *T }
var t T
```

Получаем

```
t.i == 0
t.f == 0.0
t.next == nil
```

Если нужно получать *variable*, инициализированную некоторым значением, можно вручную написать [функцию-конструктор](#конструктор) и вызывать ее явно.



# Operator

*Operator*'ы собирают *operand*'ы в *expression*'s.

<pre>
Expression = UnaryExpr | Expression binary_op Expression .
UnaryExpr  = PrimaryExpr | unary_op UnaryExpr .
binary_op  = "||" | "&&" | rel_op | add_op | mul_op .
rel_op     = "==" | "!=" | "<" | "<=" | ">" | ">=" .
add_op     = "+" | "-" | "|" | "^" .
mul_op     = "*" | "/" | "%" | "<<" | ">>" | "&" | "&^" .
unary_op   = "+" | "-" | "!" | "^" | "*" | "&" | "<-" 
</pre>  








## `++`, `--`

В отличие от C++, операторы `++` и `—` могут использоваться только как оператор, но не как выражение. 

```go
a++
```

Они могут применяться лишь в постфиксной форме. Это предотвращает появление проблем, связанных с неправильным порядком вычислений.

## Конкатенация `string`

`string` можно конкатенировать с помощью оператора `+` или оператора присваивания `+=`:

```
s := "hi" + " "
s += "and good bye"
```

В результате будет создана новая строка (т.к. строки *immutable*).



## Comparison operators

*Comparison operator*'ы (операторы сравнения) сравнивают два *operand*'а и выбрасывают *untyped* значение типа `bool`.

```
==    equal
!=    not equal
<     less
<=    less or equal
>     greater
>=    greater or equal
```

Первый *operand* должен быть [assignable]() с типом второго operand'а, или наоборот.







## Address operator

*Address operator* (адресные операторы)

Для операнда `x` типа `T` операция взятия адреса `&x` генерирует *pointer* типа `*T` на  `x`:

```go
type T int

func main() {
	var x T
	var p *T = &x

	fmt.Println(reflect.TypeOf(p))  // *main.T
}
```

```go
&x
&a[f(2)]
```

Для *pointer*'а  `x` типа `*T `, разыменование (dereferencing, косвенное обращение, inderection) *pointer*'а `*x` обозначает переменную типа `T`, на которую указывает `x`. Разыменование выполняется добавлением символа звездочки `*` перед именем переменной.

```go
*p
*pf(x)
```





# Область видимости

Область видимости при использовании оператора сокращенного объявления переменной `:=`  – ограничена блоком. 

Например, здесь область видимости переменной ограничена телом инструкции `if`:

```go
if <condition> {
    variable := "value"
}
```





# Комментарии

Комментарии оформляются в стиле языка C++: 

```go
// Однострочные комментарии, заканчивающиеся в конце строки

/* Блочные комментарии, занимающие несколько
строк 
*/
```









# Expression

*Expression* (выражение) позволяет вычислить значение путем применения *operator*'s и *function*'s к *operand*'s.

## *Qualified identifier*

Обращение к `type`, `func`, `var` (переменным) и другим элементам *package* записывается в виде `<package>.<элемент>`, где `<package>` – это последний (или единственный) компонент в имени *package*. Например, обращение к функции `Reverse()` в *package* `stringutil` записывается так `stringutil.Reverse()` 

## *Composite literal*

*Composite literal* (составной литерал) конструирует значение следующих типов:

- `struct`
- `array`
-  `slice`
- `map`. 

Новое значение конструируется при каждом очередном вычислении.

------

<pre>
CompositeLit  = LiteralType LiteralValue .
LiteralType   = StructType | ArrayType | "[" "..." "]" <a href="#array-type">ElementType</a> |
                SliceType | MapType | <a href="#type">TypeName</a> .
LiteralValue  = "{" [ ElementList [ "," ] ] "}" .
ElementList   = KeyedElement { "," KeyedElement } .
KeyedElement  = [ Key ":" ] Element .
Key           = FieldName | Expression | LiteralValue .
FieldName     = identifier .
Element       = Expression | LiteralValue .  
</pre>



------

Для *composite literal* – *underlying type* должен быть: 

- *struct type*
- *array type*
- *slice type*
- *map type*



The LiteralType's underlying type must be a struct, array, slice, or map type (the grammar enforces this constraint except when the type is given as a TypeName). The types of the elements and keys must be [assignable](https://golang.org/ref/spec#Assignability) to the respective field, element, and key types of the literal type; there is no additional conversion. The key is interpreted as a field name for struct literals, an index for array and slice literals, and a key for map literals. For map literals, all elements must have a key. It is an error to specify multiple elements with the same field name or constant key value. For non-constant map keys, see the section on [evaluation order](https://golang.org/ref/spec#Order_of_evaluation).

For struct literals the following rules apply:

- A key must be a field name declared in the struct type.
- An element list that does not contain any keys must list an element for each struct field in the order in which the fields are declared.
- If any element has a key, every element must have a key.
- An element list that contains keys does not need to have an element for each struct field. Omitted fields get the zero value for that field.
- A literal may omit the element list; such a literal evaluates to the zero value for its type.
- It is an error to specify an element for a non-exported field of a struct belonging to a different package.



TODO!!! Zero value

Если есть такие *type declation*'s:

```go
type Point3D struct { x, y, z float64 }
type Line struct { p, q Point3D }
```

то возможны такие *composite literal*'s:

```go
origin := Point3D{}                            // zero value for Point3D
line := Line{origin, Point3D{y: -4, z: 12.3}}  // zero value for line.q.x
```

Можно использовать *address operator* ([1](#address-operator)) для *composite literal*, чтобы получить pointer на новое значение:

```go
var pointer *Point3D = &Point3D{y: 1000}
```





Способы указания длины для *array literal*:

- явно указать длину в `ArrayType`. Варианты:

  - если в *array literal* записано меньше элементов, чем длина в `ArrayType`, для отсутствующих элементов устанавливается *zero value* в соответствии с *element type*:

    ```go
    buffer := [10]string{}             // len(buffer) == 10
    intSet := [6]int{1, 2, 3, 5}       // len(intSet) == 6
    ```

  - если в *array literal* записано больше элементов, чем длина в `ArrayType`, это приводит к ошибке.

-  Конструкция `[...]ElementType` позволяет указать длину массива, равную максимальному индексу элемента +1:

  ```go
  days := [...]string{"Sat", "Sun"}  // len(days) == 2
  ```

  

Каждая строка *composite literal* должна заканчиваться запятой `,`, даже если это последняя или единственная строка в expression. Это результат  автоматической простановки точек с запятой ([1](#точка-с-запятой)).

```go
mapa := map[string]string{
    "jedan": "one",
    "dva":   "two"  // ошибка, здесь будет поставлена ;
}
```



## *Function literal*



## *Index expression*

Получение значения по индексу для `array`, `*array`, `slice`, `string` и `map`:

```go
a[x]
```

Если `a` – НЕ `map`:

- индекс  `x` должен быть *integer type* или *untyped constant*

Для `string`:

-  `a[x]` – значение байта по индексу `x`, тип `a[x]` – `byte`

Для `map` :

- тип `x` должен быть *assignable* c *key type*
- если `map` – `nil` или не содержит ключ `x`, то `a[x]` – [*zero value*](#zero-value) для *element type*.
-  

TODO!!!

## *Slice expression*

*Slice expression* (выражения для *slice*) позволяют получить *substring* из `string` или *slice* из `array`, `*array` или `slice`. 

### Простой *slice expression*

В простой форме *slice expression* указывается нижняя и верхняя граница:

```
a[<low> : <high>]
```

В результате получается *substring* или *slice*, который содержит элементы с индексами, начиная с `<low>` и заканчивая `<high>-1`, т.е. из диапазона `[<low>;<high>)`.

Например:

```go
a := [5]int{1, 2, 3, 4, 5}
s := a[1:4]                // {2, 3, 4}
```

Если `a` –  `*array`, то `a[<low> : <high>]` это сокращение для `(*a)[<low> : <high>]`.

`<low>` или `<high>` можно не указывать, в этом случае *by default*:

- `<low> = 0`
- `<high> = len(a)`

```go
a[2:]  // same as a[2 : len(a)]
a[:3]  // same as a[0 : 3]
a[:]   // same as a[0 : len(a)]
```

Если результат *expression* – *slice*, то он разделяет свой *underlying array* с операндом.

Например:

```go
var a [10]int
s1 := a[3:7]   // underlying array of s1 is array a; &s1[2] == &a[5]
s2 := s1[1:4]  // underlying array of s2 is underlying array of s1 which is array a; &s2[1] == &a[5]
s2[1] = 42     // s2[1] == s1[2] == a[5] == 42; все элементы массива - это один и тот же underlying array element
```

### Полный *slice expression*

Полный *slice expression* позволяет указать *capacity*.

Может использоваться для `array`, `*array` или `slice` (но не для `string`):

```
a[<low> : <high> : <max>]
```

При этом будет создан *slice* как и для простого *slice expression* `a[<low> : <high>]`. Но для него устанавливается `capacity = <max> - <low>`.

## Type assertions

Позволяет проверить, что `x != nil` и что значение в `x`  имеет тип `T`.

```go
x.(T)
```

где:

- `x` – *expression of interface type*
- `T` – *type*

Если *type assertions* выполняется, возвращает значение, хранящееся в `x` и его тип - `T`. 

Возможны такие формы *type assertions*:

- 1 форма:

  Если *type assertions* не выполняется, происходить *run-time panic*:

  ```go
  v = x.(T)
  ```

- 2 форма:

  ```go
  v, ok = x.(T)
  ```

  Если *type assertions* выполняется – `ok == true`, иначе – `ok == false` и значение `v` – *zero value* для типа `T`. *Run-time panic* не происходит.

  Аналог *type switch*:

  ```go
  if v, ok := any.(Stringer); ok {
    return v.String()
  }
  ```

  





## Вызов *function* и *method*

Вызов *function*:

```
f(a1, a2, … an)
```

Вызов *method*:

```
math.Atan2(x, y) 
```

Если возвращаемые значения *function*'и (*method*'а) `g` совпадают (по количеству и типам) с параметрами другой *function*'и (*method*'а) `f`, то можно сделать вызов вида:

```
f(g(parameters_of_g))
```

Вызов `f` не должен иметь других параметров, кроме вызова `g`. Последний параметр `f` может быть вида `...a`. 

```go
func g(...) (string, string) {
  ...
	return p1, p2
}

func f(s, t string) string {
  ...
	return p
}

f(g(p1, p2))
```

Каждая строка списка параметров должна заканчиваться запятой `,`, даже если это последняя или единственная строка. Это результат  автоматической простановки точек с запятой ([1](#точка-с-запятой)).

```go
response.BadRequest(
			web1brokerautodictionariesget.BadRequestRespData{
				...
			}  // ошибка, здесь будет поставлена ;
)
```





## Operator

*Operator* соединяет *operand*'ы в *expression*:

<pre>
Expression = UnaryExpr | Expression binary_op Expression .
UnaryExpr  = PrimaryExpr | unary_op UnaryExpr .
binary_op  = "||" | "&&" | rel_op | add_op | mul_op .
rel_op     = "==" | "!=" | "<" | "<=" | ">" | ">=" .
add_op     = "+" | "-" | "|" | "^" .
mul_op     = "*" | "/" | "%" | "<<" | ">>" | "&" | "&^" .
unary_op   = "+" | "-" | "!" | "^" | "*" | "&" | "<-" .  
</pre>



# Statement

Statement контролируют исполнение.

<pre>
Statement =
	Declaration | LabeledStmt | SimpleStmt |
	GoStmt | ReturnStmt | BreakStmt | ContinueStmt | GotoStmt |
	FallthroughStmt | Block | IfStmt | SwitchStmt | SelectStmt | ForStmt |
	DeferStmt .
SimpleStmt = EmptyStmt | ExpressionStmt | SendStmt | IncDecStmt | Assignment | ShortVarDecl .  
</pre>







Блоки программного кода заключаются в фигурные скобки:

- тело функций 

  ```go
  func main() {
  	// ...
  }
  ```

- тело управляющих конструкций:

  ```go
  if ... { 
      //...
  }
  ```

Отступы используются исключительно для удобства человека.

## Точка с запятой `;`

При разделении операторов:

- можно использовать точку с запятой `;`. Обязательно, если в одной строке располагается несколько инструкций

  ```go
  a := 1;
  ```

- можно не использовать точку с запятой `;`. Тогда они автоматически будут добавлена лексическим анализатором.

  ```go
  a := 1
  ```

Лексический анализатор использует следующее правило для автоматической вставки точек с запятой `;`  перед *newline*:  если *newline* стоит после токена, который может завершить *statement*, лексический анализатор вставляет точку с запятой `;`.

<u>Последствия из правила:</u>

Нельзя поставить открывающую фигурную скобку `{` структуры управления ( `if`, `for`, `switch`, или `select`) на следующей строке. Иначе перед скобкой будет вставлена точка с запятой:

```
if i <f () // ошибка, здесь будет вставлена ;
{ 
    г()
}
```

## `if`

<pre>
IfStmt = "if" [ <a href="#statement">SimpleStmt</a> ";" ] <a href="#operator">Expression</a> <a href="#block">Block</a> [ "else" ( IfStmt | <a href="#block">Block</a> ) ] .  
</pre>


Условное выражение в инструкции `if` не заключается в круглые скобки.

- Пример 1:

  ```
  "if" Expression Block
  ```

  ```go
  if a > 1 { 
  	// ...    
  }
  ```

- Пример 2:

  ```
  IfStmt = "if" Expression Block "else" IfStmt.
  ```

  ```go
  if x < y {
  	return x
  } else if x > z {
  	return z
  }
  ```

*Expression*'у может предшествовать *simple statement* (`SimpleStmt`), который выполняется до вычисления *expression*'а.

- Пример:

  ```
  IfStmt = "if" SimpleStmt ";" Expression Block .
  ```

  ```go
  if x := f(); x < y {
  	return x
  }
  ```

  

## `switch` *statement*

У `switch` *statement* есть две формы: 

- *expression switch*
- *type switch*

<pre>
SwitchStmt = <a href="#expression-switch">ExprSwitchStmt</a> | <a href="#type-switch">TypeSwitchStmt</a> .  
</pre>
*Expresson* внутри `switch` вычисляется только один раз

### *Expression switch*



### *Type switch*

Специальная форма `switch`, в которой сравниваются не *value*, а *type*. 

*Expression* внутри `switch` имеет специальную форму, похожую на [type assertion](#type-assertion), но используется специальное зарезервированное слово `type`.

```go
TypeSwitchStmt  = "switch" [ SimpleStmt ";" ] TypeSwitchGuard "{" { TypeCaseClause } "}" .
TypeSwitchGuard = [ identifier ":=" ] PrimaryExpr "." "(" "type" ")" .
TypeCaseClause  = TypeSwitchCase ":" StatementList .
TypeSwitchCase  = "case" TypeList | "default" .
TypeList        = Type { "," Type } .
```

```go
switch x.(type) {
case <type1>:
  ...
case <type2>:
  ...
default:
  ...
}
```

Как и в *type assertion* ([1](#type-assertion)), `x` должен быть *interface type*, `Type` внутри `case` должен быть *non-interface type*. 

`TypeSwitchGuard` может включать *short variable declaration*. *Scope* такой *variable* – неявно объявленный *block* ([1](#block)) для `swith` *statement*.



```go
switch i := x.(type) {
case nil:
	printString("x is nil")                // type of i is type of x (interface{})
case int:
	printInt(i)                            // type of i is int
case float64:
	printFloat64(i)                        // type of i is float64
case func(int) float64:
	printFunction(i)                       // type of i is func(int) float64
case bool, string:
	printString("type is bool or string")  // type of i is type of x (interface{})
default:
	printString("don't know the type")     // type of i is type of x (interface{})

```











ТУТ!!!



## `for`

Существует три формы цикла `for`:

- с *single* `Condition` ([1](#с-одиночным-condition))
- с `ForClause`
- с `RangeClause`

<pre>
ForStmt = "for" [ <a href="#с-single-condition">Condition</a> | ForClause | RangeClause ] Block . 
</pre>

### С *single* `Condition`

<pre>
Condition = Expression .  
</pre>



В других языках используется `while`. Цикл выполняется до тех пор, пока `Condition` – `true`. 

```go
for a < b {
	// ...
}
```

Условие может быть вообще не указано, тогда считается что оно всегда `true` (аналог `while(1)`, бесконечный цикл):

```go
for {
		// ...
}
```



### c `range` *clause*

<pre>
RangeClause = [ ExpressionList "=" | IdentifierList ":=" ] "range" Expression .  
</pre>



(??? Не разобрался почему слева `ExpressionList`)

Итерирует все элементы *array*, *slice*, *string* или *map* или значений, полученных по *channel*. Для каждого элемента присваивает переменным (может быть несколько переменных в `IdentifierList`) некоторые значения, зависящие от типа `Expression` (см. таблицу ниже), а затем выполняет блок.


```go
for index, value := range <slice> {
  // ...
}

for key, val = range <map> {
	h(key, val)
}
```



*Value*'s зависят от типа `Expression` следующим образом:

```
Range expression                          1st value          2nd value

array or slice  a  [n]E, *[n]E, or []E    index    i  int    a[i]       E
string          s  string type            index    i  int    see below  rune
map             m  map[K]V                key      k  K      m[k]       V
channel         c  chan E, <-chan E       element  e  E
```

### С *for clause*

<pre>
ForClause = [ InitStmt ] ";" [ Condition ] ";" [ PostStmt ] .
InitStmt = SimpleStmt .
PostStmt = SimpleStmt .  
</pre>





Стандартная форма для C++ и PHP:

```go
for i := 0; i < 10; i++ {
	// ...
}
```

## `return`

<pre>
ReturnStmt = "return" [ <a href="#constant-declaration">ExpressionList</a> ] .  
</pre>


В функции без [*result parameters*](#result-parameters) используется пустой `return`:

```go
func noResult() {
	return
}
```

Если у функции есть [*result parameters*](#result-parameters):

1. явно указать *value* в `return`:

   ```go
   func simpleF() int {
   	return 2
   }
   ```

2. указать вызов функции в `return`. Работает, как будто каждое значение, возвращаемое функцией, было присвоено временной переменной, за которыми следует `return`, перечисляющий эти переменные (как в 1 случае):

   ```go
   func complexF2() (re float64, im float64) {
   	return complexF1()
   }
   ```

3. Если для [*result parameters*](#result-parameters) указаны имена, то можно использовать пустой `return`. `return` просто возвращает значения этих переменных.

   ```go
   func complexF3() (re float64, im float64) {
   	re = 7.0
   	im = 4.0
   	return
   }
   ```

   







## `break`

Завершает выполнение самого внутреннего оператора `for`, `switch` или `select`

```
BreakStmt = "break" [ Label ]
```

Можно поставить `label` перед внешним `for`, `switch` или `select` и указать этот `label` в операторе `break`. Тогда будет завершено выполнение внешнего *statement*. 

```
OuterLoop:
	for i = 0; i < n; i++ {
		for j = 0; j < m; j++ {
			break OuterLoop
		}
	}
```



## *Concurrency*

### *Channel*

Одна из основных проблем *concurrent programming* – корректный доступ к *shared variable*. При разработке на Go рекомендуется передавать *shared variable* по *channel*. В итоге, доступ к конкретной ячейке памяти имеет только одна *goroutine*. *Race condition* не может произойти *by design*.

### *Goroutine*

В Go было выбрано название *goroutine*, т.к. существующие термины (*thread*, *coroutine*, *process* и др.) – наполнены неточным смыслом. 

*Goroutine* – это *function*, которая выполняется в независимом *concurent thread of control*, в том же самом *address space* (вместе с другими *goroutine*'s). 

*Goroutine* –  *lightweight* (легковесная), и стоит немного больше, чем *allocation* в *stack space* (??? почему *stack*). *Stack* изначально маленький по размеру, поэтому он дешевый, и растет путем *allocating* (и *freeing*) в *heap storage* (почему *heap*???) по мере необходимости.

*Goroutine*'s мультиплексируются (что это значит???) в множество *OS thread*'s, поэтому если одна из них блокируется (например, по I/O), другие продолжают выполняться. *Goroutine design* скрывает многие сложности создание и управления *thread*'ами.

Выполнение программы не дожидается завершения вызванной *function* (*goroutine*). Вызванная *function* начинает выполняться независимо в новой горутине. Когда функция завершается, ее горутина также тихо (???) завершается. Если функция имеет какие-либо возвращаемые значения, они отбрасываются по завершении функции.

### `go` *statement*

`go` *statement* начинает выполнение *goroutine* (1) в в том же самом *address space*. 

<pre>
GoStmt = "go" <a href="#operator">Expression</a> .   
</pre>

*Expression* должнл быть *function call* или *method call*, оно не может быть заключено в круглые скобки `(...)`. Использовать *built-in function* нельзя. 

```go
go foo()                    // использование function declaration
go func() {                 // использование function literal
        time.Sleep(delay)
        fmt.Println(message)
}()                         // Важно, что ставяться скобки () - необходимо вызвать function
```

*Function literal* – *closure*, поэтому *variable*'s, которые используются внутри *function*, не будут удалены в вызывающей *goroutine* и будут существовать до завершения выполнения *function*.

# Автоматическое приведение *pointer→value* и *value→pointer*

Выполняются автоматические приведения:

- Если `pa` – *pointer* `a` типа, то:
  - для *index expression* можно писать сокращенно `pa[x]` вместо `(*pa)[x]`
  - для простого *slice expression* можно писать сокращенно `pa[<low> : <high>]` вместо `(*pa)[<low> : <high>]`
  - для полного *slice expression* можно писать сокращенно `pa[<low> : <high> : <max>]` вместо `(*pa)[<low> : <high> : <max>]`
- если `x` – *defined type*, и метод `m()` в качестве *receiver* принимает `*x` , то можно писать сокращенно `x.m()`, вместо `(&x).m()`.
- если `px` – *pointer* на *defined type* `x`, и метод `m()` в качестве *receiver* принимает `x` (или `x` имеет *property* `p`), то можно писать сокращенно `px.m()` (`x.p`), вместо `(*px).m()`  (`(*px).p`)

Поэтому в Go отсутствует оператор `->` из C и C++, т.к. Go самостоятельно автоматически приводит *pointer→value* и *value→pointer*





# Инициализация программы и исполнения

### Инициализация пакета

Порядок объявлений и определений в общем случае не имеет большого значения. 

Переменная может быть объявлена:

- как до ее использования:

  ```go
  var a = 1
  
  func main() {
      fmt.Print(a)
  }
  ```

- так и после ее использования (в соответствии с [declaration и scope](#declaration-и-scope)):

  ```go
  func main() {
      fmt.Print(a)
  }
  
  var a = 1
  ```

  



# Garbage collector

 Garbage collector (сборщик мусора) съедает в момент работы 1мс.



# Go modules

*Go modules* полностью заменяют старый подход с управлением зависимостями через `GOPATH`. 

Выделяют два режима работы:

- *module-aware mode* – режим с поддержкой *Go Modules*
- *GOPATH mode* – *legacy* режим с разрешением зависимостей через `GOPATH`

## Поддержка Go modules

Поддержка *module*'s включена прямо в `go` *command*'у. 

*Module-aware mode* включен автоматически, если:

- `go.mod` есть в текущем каталоге
- `go.mod` есть в любом родительском каталоге.

Также управлять *mode* можно с помощью `GO111MODULE` *environment variable*. Возможные значения:

- не задан или `GO111MODULE = auto` – включен или нет *module-aware mode* зависит от наличия файла `go.mod`  в текущем каталоге и родительских каталогах, как описано выше. 
- `GO111MODULE=on` – включен *module-aware mode*, `GOPATH` игнорируется
- `GO111MODULE=off` – включен *GOPATH mode*, возможности *Go modules* не используются. Зависимости ищет в каталоге `vendor` и `GOPATH`.

При *module-aware mode*:

- значение `GOPATH` *environment variable* не используется при поиске *dependency*'s
- загруженные *dependency*'s продолжают сохраняться в `GOPATH/pkg/mod`
- *command*'ы (*executable*???) устанавливаются в `GOPATH/bin` (если `GOBIN` не установлена).

## Module

*Module* – набор связанных *package*'s, которые *released* (релизятся) вместе. *Module* – это единица обмена исходным кодом и единица версионирования. 

*Module* содержит *package*'s в каталоге, содержащем файл `go.mod`, а также подкаталогах этого каталога, вплоть до следующего подкаталога, содержащего другой файл `go.mod` (если есть).

*Module* бывают:

- *executable* (исполняемый) – который содержит `package main`.
- *non-executable* ???

Даже если *module* изначально не опубликован в *repository* (*github*), желательно его сразу структурировать так, как будто он будет опубликован в *repository* (*github*).

*Repository*, как правило, содержит только один *module*, расположенный в корне репозитория (но может и несколько *module*'s). 

### Module path

*Module path* (путь к модулю) – префикс для *import path* для всех *package*'s внутри *module*. Объявляется в файле `go.mod`. 

```
module github.com/parshikovpavel/hello
```

*Module path* позволяет указать, где `go` *command* будет искать его его для загрузки. Т.е. `go` *command* при поиске некоторого *module* по его *module path* (например, `github.com/parshikovpavel/hello`) сделает запрос по соответствующему HTTPS URL-адресу (например, `https://github.com/parshikovpavel/hello`). Также по этому адресу будут считаны некоторые дополнительные метаданные из ответа HTML (популярные VCS хостинги автоматически присылают эти данные). 

Поэтому самый простой способ использовать и делиться *module* – указать *repository URL* в качестве *module path*. Поэтому принято, что если код хранится в некотором удаленном *repository*, то корень этого удаленного *repository* используется в качестве *module path*.  

## Package

Программы размещаются внутри *package*. 

*Package* – набор из *file*'s, размещенных в одном каталоге (и его подкаталогах?), которые компилируются вместе. `func`, `type`, `var` и `constant`, определенные в любом *file* некоторого *package* видны внутри всех других *file*'s этого же *package* (т.е. может быть несколько *file*'s в одном *package* !!!!).

Подробно про `package` *clause* и `PackageName` ([1](#package-clause)).

### File

Язык Go оперирует в терминах *package*'s, а не файлов. То есть *package* можно разбить на любое количество файлов, и поместить в один каталог. Им всем необходимо задать одинаковое объявление `package`, и все они будут являться частями одного и же *package*, как если бы все их содержимое находилось в единственном файле.

Особенности:

- Несколько *file*'s могут принадлежать одному и тому же *package* (!!!! отличие от PHP). 
- Несколько file's могут содержать `package` *clause* с одним и тем же `PackageName`.
- *File name* никак не связано с `PackageName` (вообще никак, `PackageName` – *by conventional* последняя часть *import path* ([1](#выбор-packagename)), а file name не имеет вообще никакого отношения к *import path* и `PackageName`) (!!!!).

### Import path

*Import path* (путь импорта) – строка, используемая для импорта *package* (не *file*, а *package*). *Import path* = *module path* + подкаталог внутри модуля.

Если в *module* с *module path* `github.com/google/go-cmp` находится *package* в каталоге `cmp/`, то *import path* этого *package* - `github.com/google/go-cmp/cmp`. 

*Package*'s из *standard library* имеют короткие *import path* (нет *module path*), такие как `"fmt"` и `"net/http"`.







### `package` *clause*

Любой фрагмент программного кода должен быть включен в *package* (пакет).

Объявление `package` указывает, к какому *package* принадлежит файл. Все *file*'s с одним и тем же `PackageName` составляют *package* (т.е. несколько *file*'s могут входить в один *package* !!!!). 

`PackageName` – состоит из одной части, он содержит только последнюю часть *import path*, он не содержит `/` (!!!!).

Первый оператор в исходном файле Go должен быть:

```
PackageClause  = "package" PackageName .
PackageName    = identifier .
```

```go
package stack
```



### Выбор `PackageName`

*By convention*, последняя часть *import path* указывается в качестве `PackageName` (*"Last Segment" Convention*).

Например, для файла `github.com/parshikovpavel/hello/xxx/yyy.go` (у которого *import path* `github.com/parshikovpavel/hello/xxx`) указываем:

```go
// github.com/parshikovpavel/hello/xxx/yyy.go
package xxx
```

Но ничего не мешает не следовать этой *convention*. Например, указать `zzz` в качестве `PackageName`:

```go
// github.com/parshikovpavel/hello/xxx/yyy.go
package zzz

var Variable = "b"
```

Никаких проблем при *import*'е нет (здесь при *import*'е `PackageName` опущен, поэтому используется `PackageName` `zzz` из импортированного *package*) ([1](#import-decalration))

```go
// github.com/parshikovpavel/hello/main.go
package main
import (
    "fmt"
    "github.com/parshikovpavel/hello/xxx"
)
func main() {
    fmt.Println(zzz.Variable)
}
```

Т.е. `PackageName` в *qualified identifier* (здесь `zzz`) не обязан совпадать с последней частью в *import path* (когда `PackageName` в *import declaration* опущено) (здесь `xxx`). И `PackageName` в *qualified identifier* нужно искать соответствие среди `PackageName` в директивах `package` импортированных пакетов.



### `package main`

*Executable module* должен иметь `main` *package* с функцией `main()` , которая является точкой входа в программу. `main` *package* используется для создания *executable binary*. Выполнение программы начинается в  `main` *package* с вызова функции `main()`.

```go
package main

func main() {
	// ...    
}
```

Функция `main()` всегда не имеет аргументов и ничего не возвращает. Когда функция `main.main()` завершается, одновременно с ней завершается выполнение программы, и она возвращает операционной системе значение 0.

Также можно использовать функцию `init()`, которая выполняется перед функцией `main()`.



### `import` *declaration*

`import` позволяет импортировать *package*. Из этого импортированного *package* можно использовать только [*exported identifier*'s](#exported-identifier).

<pre>
ImportDecl       = "import" ( ImportSpec | "(" { ImportSpec ";" } ")" ) .
ImportSpec       = [ "." | <a href="#package-clause">PackageName</a> ] ImportPath .
ImportPath       = string_lit .  
</pre>

`PackageName` может быть:

1. указан, тогда `PackageName` в импортирующем файле может использоваться в *qualified identifier*'s для обращения к *exported identifier*'s из импортированного *package*. 
2. опущен, используется `PackageName` из импортированного *package*.
3. вместо `PackageName` указана точка `.`, exported identifier's будут доступны без се экспортируемые идентификаторы пакета, объявленные в [блоке](https://golang.org/ref/spec#Blocks) пакета этого [пакета,](https://golang.org/ref/spec#Blocks) будут объявлены в блоке файла импортируемого исходного файла и должны быть доступны без квалификатора.



Примеры:

- одна `ImportSpec`:

  ```go
  import "io"    // `ImportSpec`
  ```

- несколько `ImportSpec`:

  ```go
  import (
      "io"       // `ImportSpec`
      "bufio"    // `ImportSpec`
  )
  ```

  

Здесь указывается *import path* (не файла, а *package*).

Импортируемые пакеты можно не отделять друг от друга запятыми.

Смотреть *qualified identifier* ([1]())





## Порядок создания *executable module*

- Создать *repository* для *module*. Например, `https://github.com/parshikovpavel/hello`.

- Клонируем *repository*. При этом будет создан каталог `hello` в текущем каталоге.

  ```bash
  git clone https://github.com/parshikovpavel/hello
  ```

- заходим в каталог `hello` . 

  Инициализируем новый *module* с корнем в текущем каталоге. Для это создается файл `go.mod` в текущем каталоге.

  ```bash
  go mod init github.com/parshikovpavel/hello
  ```

  Содержимое файла:

  ```bash
  $ cat go.mod 
  module github.com/parshikovpavel/hello
  
  go 1.15
  ```

- Для *executable module* необходимо создать *file* (имя любое [1](#file)), входящий в [`package main`](#package-main) с функцией `main()`:

  ```go
  package main
  
  import "fmt"
  
  func main() {
  	fmt.Println("Hello!")
  }
  ```

- выполнить *build* и *install* программы:

  ```bash
  go install example.com/user/hello
  ```

  При этом *executable* помещается в соответствующий каталог ([1](#go-install))

## Порядок создания *package* внутри своего *module*

Пример создания `morestrings` (*package name*) *package* внутри `github.com/parshikovpavel/hello` (*module path*) *module* и использования его.

- Создаем каталог `morestring` (*package name*) для *package* внутри каталога c `hello` (*module name*) *module*:

  ```bash
  mkdir .../hello/morestring
  ```

- внутри каталога `../hello/morestring` размещаем файл (файлы) этого *package*. Имя (имена) файла могут быть любыми ([1](#file)). Например, имя файла – `reverse.go`. 

- Файл `reverse.go` внутри *package* должен иметь структуру:

  ```
  package morestrings // Указан <PackageName>
  
  func Name(...) ... { // Начинается с заглавной буквы
  	...
  }
  ```

  Все *exported identifier*'s ([1](#exported-identifier)), которые должны быть доступны из других *package*'s), как `Name`, должны быть начинаться с буквы в *Unicode upper case*.



<u>ТУТ!!! Закончил Importing packages from your module</u>



## `go mod`

Операции с *module*'s.

```
go mod <command> [arguments]
```

Перечень команд:

```
download    загрузить module's в локальный кеш
edit        редактировать go.mod из инструментов или скриптов
graph       напечатать граф требований для module
init        инициализировать новый module в текущей директории
tidy        добавить отсутствующие и удалить неиспользуемые module's
vendor      сделать копию dependency's в vendor
verify      верифицировать, что dependency's имеют ожидаемый контент (???)
why         объяснить, почему package's или module's требуется
```

## `go mod init`

Инициализирует новый *module* с корнем в текущем каталоге. Для это создается файл `go.mod` в текущем каталоге.

```bash
go mod init [modulePath]
```

- `modulePath` – *module path*.

## `go get`

Разрешить и добавить *dependency*'s к текущему *module*, а затем *build* и *install* их.

```bash
go get [-d] [-t] [-u] [-v] [-insecure] [build flags] [packages]
```










Павел 8:26

create a directory for the package named $HOME/hello/morestrings, and then a file named reverse.go i

ReverseRunes function begins with an upper-case letter, it is exported, and can be used in other packages that import our morestrings package.

Let's test that the package compiles with go build
18 сентября
Павел
Павел 2:21

This won't produce an output file. Instead it saves the compiled package in the local build cache.

After confirming that the morestrings package builds, let's use it from the hello program

The go tool uses this property to automatically fetch packages from remote repositories.
Павел
Павел 8:41

When you run commands like go install, go build, or go run, the go command will automatically download the remote module and record its version in your go.mod file:
19 сентября
Павел
Павел 1:26

Module dependencies are automatically downloaded to the pkg/mod subdirectory of the directory indicated by the GOPATH environment variable. The downloaded contents for a given version of a module are shared among all other modules that require that version, so the go command marks those files and directories as read-only. To remove all downloaded modules, you can pass the -modcache flag to go clean
Павел
Павел 20:36

You write a test by creating a file with a name ending in _test.go that contains functions named TestXXX with signature func (t *testing.T). The test framework runs each such function; if the function calls a failure function such as t.Error or t.Fail, the test is considered to have failed.
Павел
Павел 21:15

Then run the test with go test:
20 сентября
Павел
Павел 13:36











https://golang.org/doc/tutorial/getting-started
Tutorial: Get started with Go - The Go Programming..
golang.org

Создайте каталог hello для ваше

$ go run hello.go

Visit pkg.go.dev and search for a "quote" package.
8 сентября
Павел
Павел 2:31

айдите и щелкните rsc.io/quoteпакет в
Павел
Павел 8:44

Вы можете использовать сайт pkg.go.dev, чтобы найти опубликованные модули, в

When your code imports packages from another module, a go.mod file lists the specific modules and versions providing those packages. That file stays with your code, including in

To create a go.mod file, run the go mod init command, giving it the name of the module your code will be in (here, just use "hello"):

$ go mod init hello
Павел
Павел 21:59

Run your code to see the message generated by the function you're calling.$ go run hello.go go: finding module for package rsc.io/quote go: found rsc.io/quote in rsc.io/quote v1.5.2 Don't communicate by sharing memory, share memory by communicating.
Павел
Павел 22:10

https://golang.org/doc/tutorial/create-module
Tutorial: Create a Go module - The Go Programming..
golang.org

Go code is grouped into packages, and packages are grouped into modules. Your package's module specifies the context Go needs to run the code, including the Go version the code is written for and the set of other modules it requires.
Павел
Павел 23:28

Create a greetings directory for your Go module source code

Start your module using the go mod init command to create a go.mod file.

Run the go mod init command, giving it the path of the module your code will be in. Here, use example.com/greetings for the module path — in production code, this would be the URL from which your module can be downloaded.

$ go mod init example.com/greetings go: creating new go.mod: module example.com/greetings
Example Domain
example.com
9 сентября
Павел
Павел 2:23

The go mod init command creates a go.mod file that identifies your code as a module that might be used from other code. The file you just created includes only the name of your module and the Go version your code supports. But as you add dependencies — meaning packages from other modules — the go.mod file will list the specific module versions to use

In Go, a function whose name starts with a capital letter can be called by a function not in the same package. This is known in Go as an exported name.
Павел
Павел 8:16

import ( "fmt" "example.com/greetings" ) func main() { // Get a greeting message and print it. message := greetings.Hello("Gladys") fmt.Println(message) }
Павел
Павел 8:50

Create a new module for this hello package.

From the command line at the hello directory, run the go mod init command
Павел
Павел 12:56

giving it the name of the module your code will be in (here, just use "hello").

$ go mod init hello

For production use, you’d publish your modules on a server, either inside your company or on the internet, and the Go command will download them from there. For now, you need to adapt the caller's module so it can find the greetings code on your local file system
Павел
Павел 13:10

To do that, make a small change to hello module’s go.mod file.

In the hello directory, open the go.mod file, change it so that it looks like the following, and save the file.module hello go 1.14 replace example.com/greetings => ../greetings
Example Domain
example.com

Here, the replace directive tells Go to replace the module path (the URL example.com/greetings) with a path you specify
Павел
Павел 14:07

n the hello directory, run go build to make Go locate the module and add it as a dependency to the go.mod file.$ go build go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
Павел
Павел 15:12

ook at go.mod again to see the changes made by go build, including the require directive Go added.module hello go 1.14 replace example.com/greetings => ../greetings require example.com/greetings v0.0.0-00010101000000
Example Domain
example.com
Павел
Павел 22:45

To build the module, Go found the local code in the ../greetings directory, then added a require directive to specify that hello is dependent on (requires) example.com/greetings
Example Domain
example.com

To reference a published module, a go.mod file would omit the replace directive and use a require directive with a tagged version number at the end.
Павел
Павел 8:16

log.Fatal(err)
Павел
Павел 12:33

"math/rand"
Павел
Павел 21:08

rand.Seed(time.Now().UnixNano())

rand.Intn(len(formats))]
Павел
Павел 22:53

randomFormat starts with a lowercase letter, making it accessible only to code in its own package (in other words, it's not exported).

Use the math/rand package to generate a random number for selecting an item from the slice.

Add an init function to seed the rand package with the current time. Go executes init functions automatically at program startup, after global variables have been initialized.

11 сентября
Павел
Павел 22:55

initialize a map with the following syntax: make(map[key-type]value-type).

copy of the item's value. You don't need the index, so you use the Go blank identifier (an underscore) to ignore it.
12 сентября
Павел
Павел 11:10

Go's built-in support for unit testing makes it easier to test as you go. Specifically, using naming conventions, Go's testing package, and the go test command, you can quickly write and execute tests.
Павел
Павел 17:34

In the greetings directory, create a file called greetings_test.go.

Ending a file's name with _test.go tells the go test command that this file contains test functions.
Павел
Павел 22:31

import ( "testing" "regexp" )

Implement test functions in the same package as the code you're testing.
13 сентября
Павел
Павел 16:59

Test function names have the form TestName, where Name is specific to the test

, test functions take a pointer to the testing package's testing.T as a parameter. You use this parameter's methods for reporting and logging
вчера
Павел
Павел 8:21

The go test command executes test functions (whose names begin with Test) in test files (whose names end with _test.go). You can add the -v flag to get verbose output that lists all of the tests and their results.
Павел
Павел 8:44

go run command is a useful shortcut for compiling and running a single-file program, it doesn't generate a binary executable you can easily run again.

Ещё



15 сентября
Павел
Павел 8:12

Программы Go организованы в пакеты. Пакет представляет собой набор исходных файлов в той же директории, которые скомпилированы вместе. Функции, типы, переменные и константы, определенные в одном исходном файле, видны всем другим исходным файлам в том же пакете.

Репозиторий содержит один или несколько модулей. Модуль представляет собой набор связанных пакетов Go, которые выбрасываются вместе. Репозиторий Go обычно содержит только один модуль, расположенный в корне репозитория. В названном go.modтам файле объявляется путь к модулю : префикс пути импорта для всех пакетов в модуле. Модуль содержит пакеты в каталоге, содержащем его go.modфайл, а также подкаталоги этого каталога, вплоть до следующего подкаталога, содержащего другой go.modфайл (если есть).



# `go` *command*

## Параметр `packages`

Для команд:

- `go install`
- ...

параметр `packages` применяется следующим образом:

- если `packages` указан, например:

  ```bash
  go install example.com/user/hello
  ```

  то команда ищет *package* в контексте *module*, внутри которого находится текущий каталог. Если текущий каталог не находится внутри указанного *module* (или *package*???) (здесь внутри `example.com/user/hello`), то команда выдает ошибку.

- если `packages` не указан, то используется *package* в текущем каталоге. Например:

   ```bash
  go install
  ```

- в качестве `packages` можно использовать относительный *import path*, относительно текущего каталога. Например:

  ```bash
  go install .
  ```

Если в текущем каталоге находится *package* с *import path* – `example.com/user/hello`, то следующие команды эквивалентны:

```bash
go install
go install .
go install example.com/user/hello
```



## `go build`

Сборка *package* по его *import path*, вместе с его dependency's, но без процесса *install* (размещения исполняемого файла в `bin`).

```bash
go build [-o output] [-i] [build flags] [packages]
```



Поведение:

- при компиляции `package main` – записывает полученный исполняемый файл в выходной файл с именем первого исходного файла (`go build ed.go rx.go` записывает в файл `ed`) или в файл с именем каталога исходного кода (`go build unix/sam` пишет в файл `sam`)

- При компиляции нескольких *package*'s или одного не `package main` – компилирует пакеты, но отбрасывает получившийся файл, и может использоваться только для проверки возможности сборки *package*.

Флаги:

-  `-o` заставляет `build` писать результирующий исполняемый файл или объект в указанный *output file* или *directory* вместо поведения по умолчанию, описанного ранее. 

## `go install`

```bash
go install [-i] [build flags] [packages]
```

*Compile* и *install* package (или *package*'s). 

- `packages` – опционально, *import path* пакета (пакетов). Подробно про параметр `packages` ([1](#параметр-packages))

  ```bash
  $ go install github.com/user/hello
  ```

  Если текущей является *import path*, то можно запускать без указания *import path*.

  ```bash
  $ go install
  ```

  



- *Executable* выполняется *installing* в каталог:

  - по пути `GOBIN` *environment variable*
  - если `GOBIN` не установлено, то по умолчанию значение `GOBIN`:
    - `$GOPATH/bin`.
    - если `$GOPATH` не установлена, то `$HOME/go/bin`

  Таким образом, различные *executable*'s помещаются в один и тот же каталог. Можно включить путь к этому каталогу в `PATH`:

  ```bash
  export PATH=$PATH:$GOPATH/bin
  ```

- *Non-executable* (подключаемый package):

  -  если *module-aware mode* включен, выполняется *buiding* и *caching*, без.*installing*.
  - если *module-aware mode* отключен, выполняется *installing* в каталог `$GOPATH/pkg/$GOOS_$GOARCH`.

## `go list`

```bash
go list [-f format] [-json] [-m] [list flags] [build flags] [packages]
```

Команда выводит информацию об указанных `packages`, информация об одном *package* на каждой строке. 

По умолчанию, команда выводит список *package import path*'s:

```bash
$ go list
github.com/parshikovpavel/hello
```

Флаг `-f` позволяет задать формат для вывода информации о *package*. Команда по умолчанию эквивалентна:

```bash
go list -f '{{.ImportPath}}'
```

В шаблоне могут использоваться поля из следующей структуры:

```go
type Package struct {
    Dir           string   // directory containing package sources
    ImportPath    string   // import path of package in dir
    ImportComment string   // path in import comment on package statement
    Name          string   // package name
    Doc           string   // package documentation string
    Target        string   // install path
    Shlib         string   // the shared library that contains this package (only set when -linkshared)
    Goroot        bool     // is this package in the Go root?
    Standard      bool     // is this package part of the standard Go library?
    Stale         bool     // would 'go install' do anything for this package?
    StaleReason   string   // explanation for Stale==true
    Root          string   // Go root or Go path dir containing this package
    ConflictDir   string   // this directory shadows Dir in $GOPATH
    BinaryOnly    bool     // binary-only package (no longer supported)
    ForTest       string   // package is only for use in named test
    Export        string   // file containing export data (when using -export)
    Module        *Module  // info about package's containing module, if any (can be nil)
    Match         []string // command-line patterns matching this package
    DepOnly       bool     // package is only a dependency, not explicitly listed

    // Source files
    GoFiles         []string // .go source files (excluding CgoFiles, TestGoFiles, XTestGoFiles)
    CgoFiles        []string // .go source files that import "C"
    CompiledGoFiles []string // .go files presented to compiler (when using -compiled)
    IgnoredGoFiles  []string // .go source files ignored due to build constraints
    CFiles          []string // .c source files
    CXXFiles        []string // .cc, .cxx and .cpp source files
    MFiles          []string // .m source files
    HFiles          []string // .h, .hh, .hpp and .hxx source files
    FFiles          []string // .f, .F, .for and .f90 Fortran source files
    SFiles          []string // .s source files
    SwigFiles       []string // .swig files
    SwigCXXFiles    []string // .swigcxx files
    SysoFiles       []string // .syso object files to add to archive
    TestGoFiles     []string // _test.go files in package
    XTestGoFiles    []string // _test.go files outside package

    // Cgo directives
    CgoCFLAGS    []string // cgo: flags for C compiler
    CgoCPPFLAGS  []string // cgo: flags for C preprocessor
    CgoCXXFLAGS  []string // cgo: flags for C++ compiler
    CgoFFLAGS    []string // cgo: flags for Fortran compiler
    CgoLDFLAGS   []string // cgo: flags for linker
    CgoPkgConfig []string // cgo: pkg-config names

    // Dependency information
    Imports      []string          // import paths used by this package
    ImportMap    map[string]string // map from source import to ImportPath (identity entries omitted)
    Deps         []string          // all (recursively) imported dependencies
    TestImports  []string          // imports from TestGoFiles
    XTestImports []string          // imports from XTestGoFiles

    // Error information
    Incomplete bool            // this package or a dependency has an error
    Error      *PackageError   // error loading package
    DepsErrors []*PackageError // errors loading dependencies
}
```







## `go run`

запуск программы с компиляцией во временную папку

```
go run <package>
```

Компилирует и исполняет указанный *main* (?) пакет `<package>`. 

- `package` – может задаваться, как:

  - список исходных файлов `.go` из одного каталога

  - *import path*. 

    Например для каталога `$GOPATH\src\hello` из любого места можно запустить:

    ```bash
    go run hello
    ```

    

  - путь в файловой системе

  - шаблон, соответствующий одному известному пакету (?). Например, `go run .` или `go run my/cmd`.



## Запуск программы

Запуск программы:

```bash
./hello
```

# Legacy GOPATH

Код должен быть структурирован определенным образом.

Основные принципы:

- Весь код размещается в одном *workspace* (рабочей области). Большинство программистов на Go хранят *весь* *source code* и *dependency*'s в одном *workspace*. Местоположение *workspace* определяется через `GOPATH`
- *Worspace* содержит множество *repository*'s.
- Каждый *repository* содержит один или несколько *package*'s или *command*'s
- Каждый *package* состоит из одного или нескольких *source file*'s в одной *directory*.
- Путь к *directory* определяет *package*'s *import path* (путь импорта пакета).

*Workspace* должен содержать две *directory*'s в корне:

- `src` – с *source file*'s
- `bin` – с *binary file*'s.

Инструмент `go`  *build* и *install binary file*'s в каталог `bin`.

Каталог `src` содержит множество *repository*'s , каждый из которых хранит один или несколько *package*'s или *command*'s

Пример структуры *workspace*:

```
bin/
    hello                          # binary file
    outyet                         # binary file
src/
    github.com/golang/example/     # repository
        .git/                      
        hello/                     # command
            hello.go             
        outyet/                    # command
            main.go               
            main_test.go          
        stringutil/                # package
            reverse.go             
            reverse_test.go        
    golang.org/x/image/            # repository
        .git/                      
        bmp/                       # package
            reader.go              
            writer.go            
```

### Общий алгоритм импорта и использования package

Например, в *package* по адресу `github.com/user/stringutil` будет размещен файл `reverse.go` .

- создаем каталог для *package*:

  ```
  $GOPATH/src/github.com/user/stringutil
  ```

- размещаем в этом каталоге файл `reverse.go` (полный путь `github.com/user/stringutil/reverse.go`):

  ```go
  package stringutil // Название package
  
  // Функция в package
  func Reverse(s string) string {
  	// ...
  }
  ```

- используем этот *package* в другом файле `$GOPATH/src/github.com/user/hello/hello.go`.

  *Import path* – `github.com/user/stringutil`

  При использовании *function* из *package*, нужно обращаться в виде: `<package_name>.<func_name>`:

  ```go
  package main
  
  import (
    fmt
    "github.com/user/stringutil" // Тут указываем import path (не файл)
  )
  
  func main() {
  	fmt.Println(stringutil.Reverse("!oG ,olleH")) // <package_name>.<func_name>
  }
  ```

- Запуск *main package*:

  ```bash
  go run github.com/user/hello
  ```

### Import path

*Import path* (путь импорта) – строка, используемая для импорта *package* (не файла, а *package*). *Import path* соответствует местоположению *package* внутри *workspace* (в папке `src`) или в *remote repository* (удаленном репозитории).

*Package*'s из *standard library* имеют короткие *import path*, такие как `"fmt"` и `"net/http"`.

Для *package* можно выбрать любой произвольный *path*. Принято, что если код хранится в некотором удаленном *repository*, то необходимо использовать корень этого удаленного *repository* в качестве *base path*. 

Например, если код размещается в *repository*  `github.com/user`:

- путь к каталогу с *repository* – `$GOPATH/src/github.com/user`
- путь к каталогу с `hello` *package* – `$GOPATH/src/github.com/user/hello`.  *Package import path* – `github.com/user/hello`

# Тестирование

Имя файла с тестами должно иметь вид:

```
<name>_test.go
```

В файле с тестами размещаются функции вида. 

```go
func Test<FuncName>(t *testing.T) {
 	// ... 
}
```

Пример теста:

```go
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want) // неудачный тест
		}
	}
}
```

Запуск процесса тестирования:

```bash
go test <import_path>
go test github.com/user/stringutil
```

При этом будут запущены:

- из package `<import_path>` 
- из файлов вида `<name>_test.go`
- каждая функция с сигнатурой вида `Test<FuncName>` 

Если функция вызывает функцию вида  `t.Error` (`t.Errorf`, ...) или `t.Fail`, тест считается неудачным.

## Запуск в GoLand

GoLand позволяет запустить под отладкой тест

кроме того в makefile есть цель tests или test



# Standard library

Фундаментальные типы данных в языке Go поддерживают привычные операторы (`+`, `-`,...), а стандартная библиотека Go добавляет дополнительные пакеты функций для работы с фундаментальными типами (например, `string`). 

При использовании в коде *package*, который логически включен в другой *package* (например, `path/filepath`) необходимо указывать только последний компонент их имени (например, `filepath`).

## `builtin`

Встроенные функции, которые являются частью языка.

### `len()`

Возвращает длину `v`:

```go
func len(v Type) int
```

Результат:

```
String    string type      длина string в байтах
Array     [n]T, *[n]T      длина массива (== n)
Slice     []T              число элементов
Map       map[K]T          число элементов
Channel   chan T           количество элементов, стоящих в очереди в chain
```

### `cap()`

Возвращает *capacity* (емкость) `v` 

```go
func cap(v Type) int
```

Результат:

```
Array     [n]T, *[n]T      длина array (== n, == len())
Slice     []T              максимальная длина, которую slice может достичь, т.е. размер underlying array 
Channel   chan T           емкость буфера channel, в штуках элементов;
```

Для *slice* всегда сохраняется зависимость между `len()` и `cap()`:

```
0 <= len(s) <= cap(s)
```

### `delete()`

```go
func delete(m map[Type]Type1, key Type)
```

Удаляет *element* с указанным *key* (`m[key]`) из `map`. Если `m = nil` или `m[key]` не существует, `delete()` ничего не делает.

### `make()`

Может использоваться для выделения памяти и инициализации объектов *type*'s:

- `slice`
- `map`
- `chan`

Возвращает `Type` (а не `*Type`, как `new()`). 

```go
func make(t Type, size ...IntegerType) Type
```

Возвращаемое значение зависит от `Type`:

```
Slice:      Если указан только size1, то получаем slice с параметрам len = cap = size1.
            Если указан также size2, то cap = size2. 
            Например, make([]int, 0, 10) выделяет slice с len = 0, cap = 10. 

Map:        Если size не указан - выделяется пустой map с небольшим capacity
            Если size указан - выделяется пустой map с указанным начальным capacity
            Первоначальная capacity не ограничивает его size
            
Channel:    Если size не указан - channel небуферизованный
            Если size указан, то buffer capacity = size
```

### `error` (builtin)

Смотреть [1](#error)



## `fmt`

`fmt` – *formatted* (форматированный). 

*Package* реализует форматированный I/O по аналогии с функциями `printf()` и `scanf()` в Си. 

Некоторые функции принимают `format`, в котором можно использовать спецификаторы формата `%`, напоминающие спецификаторы функций `printf()` и `scanf()` в Си.

### Плейсхолдеры

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



### `fmt.Errorf()`

```go
func Errorf(format string, a ...interface{}) error
```

Форматирует и возвращает строку как значение типа `error`.





### `fmt.Println()`

```go
func Println(a ...interface{}) (n int, err error)
```

Делает вывод операндов в *standart output* в *default* формате. Между операндами всегда добавляются *space* и *newline*. 

- `n` – количество записанных байтов
- `err` – все обнаруженные ошибки записи.

### `fmt.Printf()`

```go
func Printf(format string, a ...interface{}) (n int, err error)
```

Делает вывод операндов в *standart output* в соответствии с форматом `format`. 

- `n` – количество записанных байтов
- `err` – все обнаруженные ошибки записи.

## `json`

### Encoding

Для выполнения *encoding* данных в JSON нужно использовать функцию `Marshal()`:

```go
func Marshal(v interface{}) ([]byte, error)
```

<u>Пример:</u>

```go
type Message struct {
    Name string
    Body string
    Time int64
}

func main() {
	m := Message{"Alice", "Hello", 1294706395881547000}
	b, err := json.Marshal(m)
}
```

В случае успеха:

```go
err == nil
b == []byte(`{"Name":"Alice","Body":"Hello","Time":1294706395881547000}`)
```

Особенности использования `Marshal` с разными *type*'s:

- `struct` type – выполняется *encode* только (!!!) для *exported field*'s ([1](#exported-identifier), которые начинаются с *Unicode upper case* )
- `map` type – поддерживаются только `string` в качестве ключей; поэтому `map` должен быть типа `map[string]T` (где `T` – любой тип).

### Decoding

Для выполнения *decoding* данных из JSON необходимо использовать функцию `Unmarshal()`:

```go
func Unmarshal(data []byte, v interface{}) error
```

Сначала нужно создать место, где будут храниться декодированные данные

```go
var m Message
```

и вызвать `json.Unmarshal()`, передав ему `[]byte` с JSON данными и *pointer* на `m`

```go
err := json.Unmarshal(b, &m)
```

Если `b` содержит допустимый JSON, который соответствует `m`, после вызова `err == nil` и данные из `b` будут сохранены в структуре `m`, так что:

```go
m = Message{
    Name: "Alice",
    Body: "Hello",
    Time: 1294706395881547000,
}
```

Чтобы найти, куда декодировать данные из поля `Foo` в структуру, `Unmarshal`  просматривает поля структуры назначения, чтобы найти *field* (в порядке предпочтения):

- *Exported field* (!!!) с тегом `"Foo"` 
- *Exported field* (!!!) с именем `"Foo"`
- *Exported field* (!!!) с именем `"FOO"` или `"FoO"` или другое нечувствительное к регистру совпадение `"Foo"`

Если JSON  данные не совсем соответствует типу Go, например:

```go
b := []byte(`{"Name":"Bob","Food":"Pickle"}`)
var m Message
err := json.Unmarshal(b, &m)
```

`Unmarshal()` будет декодировать только те *field*'s, которые он может найти в типе назначения. В этом случае будет заполнено только поле `Name` в `m`, а поле `Food` будет игнорироваться. Это можно использовать, чтобы декодировать только несколько определенных полей из большого JSON-объекта. Это также означает, что `Unmarshal()` не затронет любые *non-exported field*'s в структуре назначения.

Чтобы декодировать поля JSON, которые не начинаются с заглавных букв, необходимо использовать *tags*:

```golang
type Sample struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}
```

<u>Пример 1:</u>

Декодирование JSON вида:

```json
{"fruits":["apple","banana","cherry","date"]}
```

```go
json := []byte(`{"fruits":["apple","banana","cherry","date"]}`)
var m map[string][]string
err := json.Unmarshal(json, &m)
```

<u>Пример 2:</u>

Декодирование JSON, который имеет нерегулярную структуру (разные структуры в ключах):

```json
{
    "sendMsg":{"user":"ANisus","msg":"Trying to send a message"},
    "say":"Hello"
}
```

Для этого декодирование выполняется по частям. На каждом шаге вложенная структура декодируется в `json.RawMessage`. 

```go
var objmap map[string]json.RawMessage
err := json.Unmarshal(data, &objmap)
```

Затем вложенную структуру можно декодировать дальше:

```go
var s sendMsg
err = json.Unmarshal(objmap["sendMsg"], &s)
```







ТУТ!!!









## `log`

*Package* `log`  реализует функции логгирования. Функции пишут в *standart error* и печатают дату и время каждого сообщения. Каждое сообщение выводится в отдельной строке: если сообщение не заканчивается *newline*, функции ее добавят. 

Функции *Fatal* вызывают `os.Exit (1)` после вывода сообщения.

### `log.Fatal()`

```go
func Fatal (v ... interface {})
```

`log.Fatal()` эквивалентен `log.Print()` + вызов `os.Exit (1)`.

### `log.Fatalf()`

```go
func Fatalf(format string, v ...interface{})
```

`log.Fatalf()` эквивалентен `log.Printf()` + вызов `os.Exit (1)`.

### `log.Print()`

```
func Print(v ...interface{})
```

`log.Print()` вызывает `log.Output` для печати в стандартном *logger*'е (?). 

## `os`

*Package* содержит функции для выполнения различные действия в операционной системе. Эти функции устраняют различия платформ (Unix, MacOS, ...).

### `os.Args`

Хранит аргументы командной строки, начиная с имени программы

```go
var Args []string
```







## `path`

*Package* включает функции для управления путями, разделенными *slash* `/`. *Package* следует использовать для путей в URL-адресах.

Этот *package* не следует использовать для работы с путями файловой системы (в первую очередь, разделенными *backslash* `\` и содержащими букву диска `C:\`, как в Windows). Для управления ими необходимо использовать пакет `path/filepath`.

### `path/filepath`

*Package* включает функции для управления путями файловой системы. При этом учитываются особенности файловой системы (какие слэши используются для разделения).

*Package* не следует использовать для путей в URL-адресах.

#### `filepath.Base()`

Возвращает последний (базовый) элемент пути (имя файла или имя директории) в пути файловой системы.

```go
func Base(path string) string
```

## `reflect`

### `TypeOf()`

Возвращает тип `i`. 

```go
func TypeOf(i interface{}) Type
```

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
  tst := 'a'
  fmt.Println(reflect.TypeOf(tst))
}
```

## `strings`

Package реализует функции для изменения UTF-8 строк

### `Split()`

```go
func Split(s, sep string) []string
```

Разбивает `s` на все *substring*'s, разделенные `sep`, и возвращает map из substring's между разделителями `sep`.

Если `s` не содержит `sep`, а `sep` не пуст, `Split()` возвращает  *map* длиной 1, единственным элементом которого является `s`.

```go
str := "hi, this is, educative" 
split := strings.Split(str, ",")
fmt.Println(split)
```















## `time`

### `Sleep()`

```go
func Sleep(d Duration)
time.Sleep(100 * time.Millisecond)
```

Приостанавливает текущую *goroutine* как минимум на время `d`.

Пример:

```go
fmt.Printf("Current Unix Time: %v\n", time.Now().Unix())
time.Sleep(2 * time.Second)
fmt.Printf("Current Unix Time: %v\n", time.Now().Unix())
```









## `errors`

`errors` – это *package*

### `error`

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

### `errors.New()`

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



# Стандартные приемы

## `in_array()`

В Go нет такой встроенной функции для *array type* и *slice type*.

Можно использовать три приема:

- использовать map:

  ```go
  func belongsToMap(lookup string) bool {
  list := map[string]bool{
      "900898296857": true,
      "900898302052": true,
      "900898296492": true,
      "900898296850": true,
      "900898296703": true,
      "900898296633": true,
      "900898296613": true,
      "900898296615": true,
      "900898296620": true,
      "900898296636": true,
  }
  if _, ok := list[lookup]; ok {
      return true
  } else {
      return false
  }
  }
  ```

- использовать *slice* и перебор в цикле:

  ```go
  func belongsToList(lookup string) bool {
  list := []string{
      "900898296857",
      "900898302052",
      "900898296492",
      "900898296850",
      "900898296703",
      "900898296633",
      "900898296613",
      "900898296615",
      "900898296620",
      "900898296636",
  }
  for _, val := range list {
      if val == lookup {
          return true
      }
  }
  return false
  }
  ```

- использовать *switch statement*:

  ```go
  func belongsToSwitch(lookup string) bool {
  switch lookup {
  case
      "900898296857",
      "900898302052",
      "900898296492",
      "900898296850",
      "900898296703",
      "900898296633",
      "900898296613",
      "900898296615",
      "900898296620",
      "900898296636":
      return true
  }
  return false
  }
  ```

По тестам `switch` всегда быстрее, `slice` близок по скорости, `map` гораздо медленнее (наверно, это только для случая когда нужно еще строить сам map).









Павел
Павел 12:09

Пакет bufio предоставляет функции буферизованного ввода/
вывода, включая функции чтения и записи строк из и в текстовые
файлы в кодировке UTF-8. Пакет io предоставляет низкоуровневые
функции ввода/вывода, а также интерфейсы io.Reader и io.Writer,

Пакет io/ioutil – высокоуров-
невые функции для работы с файлами. Пакет regexp дает поддержку

регулярных выражений.





26 августа
Павел
Павел 0:18

Файлы в языке Go представлены ука-
зателями на значения типа os.File, потому в программе создаются
две переменные этого типа и инициализируются ссылками на пото-
ки стандартного ввода и вывода (оба имеют тип *os.File). Так как
функции и методы в языке Go могут возвращать несколько значе-
ний, из этого следует, что Go поддерживает множественное присва-
ивание, как в данном примере (?, ? в листинге выше).
Павел
Павел 13:32



Любая функция, передаваемая инструкции defer
(§5.5), должна вызываться, поэтому после имени функции указываются
круглые скобки (? и ? в листинге выше), но фактический вызов про-
исходит в момент завершения функции, вызвавшей инструкцию defer.
То есть инструкция defer «замораживает» вызов функции, откладывая
его на более позднее время. Это означает, что на выполнение инструк-
ции defer почти не расходуется времени и управление немедленно пе-
редается следующей за ней инструкции. Таким образом, задержанный
вызов метода os.File.Close() не будет выполнен, пока не завершится
вызывающая функция (или не возникнет аварийная ситуация, о чем
рассказывается чуть ниже) – в данном случае функция main(), поэтому

файл остается открытым на протяжении всего времени работы с ним и
гарантированно закрывается по завершении или в случае аварии.
Павел
Павел 20:56

система времени выполнения языка Go автоматически
закроет все открытые файлы, механизм сборки мусора освободит па-
мять, занятую программой, и корректно будут закрыты все сетевые
соединения и подключения к базам данных,

Аварией в языке Go называется ошибка времени выполнения
(напоминает исключения в других языках). Аварийную ситуацию
можно создать вручную, вызвав встроенную функцию panic()
, а
остановить развитие аварии можно с помощью функции recover()

В языке Go принято сообщать об
ошибках
, возвращая значение ошибки из функций и методов в ви-
де единственного или последнего возвращаемого значения или nil,
при отсутствии ошибки, и проверять наличие ошибки в вызываю-
щем программном коде. Функции panic/recover предназначены для
обработки по-настоящему исключительных (то есть неожиданных)
ситуаций, а не обычных ошибок1.
Павел
Павел 21:08

C++, Java и Python, где часто механизм исключений используется
и для обработки исключительных ситуаций, и для обработки ошибок.
Павел
Павел 21:27

функция
может принимать любые значения, независимо от их типов, при
условии что они реализуют ожидаемые интерфейсы, то есть любые
значения, имеющие методы, определяемые интерфейсами.
27 августа
Павел
Павел 8:12

новое значение error вызовом функции fmt.Errorf()
28 августа
Павел
Павел 0:56



для возвращаемых значений определяются не только их типы,
но и имена. Благодаря этому при входе в функцию возвращаемым
переменным присваиваются пустые значения
Павел
Павел 12:40

fmt.Errorf()

(Для создания значения ошибки из строкового
литерала используется функция errors.New()
.)
Павел
Павел 12:46

Функции и методы, возвращаю-
щие одно значение или более, должны иметь хотя бы одну инструк-
цию return.

Если в функции или методе, помимо типов, указаны также
имена возвращаемых переменных, допускается использовать пустую
инструкцию return (то есть без указания значений или переменных).
В таких случаях возвращаются переменные, перечисленные в заго-
ловке функции.
Павел
Павел 13:04

Чтобы значение было доступно для стандартных операций
чтения, оно должно удовлетворять требованиям интерфейса
io.Reader
. Этот интерфейс определяет единственный метод с сиг-
натурой Read([]byte) (int, error). Метод Read() читает данные
из значения, относительно которого он вызывается, и помещает
прочитанные данные в указанный срез с байтами. Он должен воз-
вращать: количество прочитанных байтов и значение ошибки или
nil при ее отсутствии; или io.EOF (признак «конца файла»), в слу-
чае исчерпания данных и отсутствия ошибки или какое-то другое
значение, отличное от nil, в случае ошибки.

чтобы
значение было доступно для стандартных операций записи, оно
должно удовлетворять требованиям интерфейса io.Writer
. Этот
интерфейс определяет единственный метод с сигнатурой Write([]
byte) (int, error). Метод Write() должен записывать данные из
указанного среза с байтами в значение, относительно которого он
вызывается, и должен возвращать количество записанных байтов
и значение ошибки (которое может быть значением nil при от-
сутствии ошибки).
Павел
Павел 13:14

Пакет io предоставляет инструменты для чтения и записи, но они
не используют буферизацию и оперируют в терминах простых бай-
тов. Пакет bufio предоставляет инструменты ввода/вывода с буфе-
ризацией, где инструменты ввода могут работать только со значени-
ями, реализующими интерфейс io.Reader (то есть предоставляющими
соответствующий метод Read()), а инструменты вывода – только со
значениями, реализующими интерфейс io.Writer (то есть предостав-
ляющими соответствующий метод Write()). Инструменты чтения и
записи из пакета bufio обеспечивают возможность буферизации, мо-
гут оперировать в терминах байтов и строк и потому лучше подходят
для чтения и записи текстовых файлов в кодировке UTF-8.
30 августа
Павел
Павел 0:49

https://index.minfin.com.ua/reference/coronavirus/geo..

Коронавирус - Статистика по странам [30.08.2020] - Карта заражений, графики
index.minfin.com.ua
Павел
Павел 18:56

буферизованного
ввода/вывода, посредством которых можно обращаться к содер-
жимому как к двоичным байтам или, что более удобно в данном
случае, как к строкам. Функция-конструктор bufio.NewReader() при-
нимает в виде аргумента любое значение, поддерживающее интер-
фейс io.Reader (то есть любое значение, имеющее метод Read()), и
возвращает новое значение буферизованного ввода типа io.Reader,
которое будет читать данные из указанного значения.
Павел
Павел 19:49

func() {
ifif err == nilnil {
err = writer.Flush()
}
}()

анонимная отложенная функция,
31 августа
Павел
Павел 1:59

именованное

возвращаемое значение value, ему можно присваивать значения в лю-
бой точке внутри функции с помощью оператора присваивания (=).
Однако, если внутри инструкции if имеется такая инструкция, как
value := ..., будет создана новая переменная, потому что инструк-
ция if образует новый блок. В результате переменная value внутри
блока инструкции if как бы загородит собой возвращаемое значение
value.

необходимо гарантировать, что этому имени нигде
в данной функции не будет присваиваться какое-либо значение с по-
мощью оператора сокращенного объявления переменной (:=), чтобы
избежать риска создания теневой переменной.

пустого идентифика-
тора
, _ (? в листинге выше). Пустой идентификатор играет роль запол-
нителя в операции присваивания, где ожидается переменная, и помога-
ет просто отбросить присваиваемое значение. Пустой идентификатор
не считается новой переменной,

мощный пакет regexp (§3.6.5)
поддержки регулярных выражений. Этот пакет можно использовать
для создания указателей на значения типа regexp.Regexp (то есть для
создания значений типа *regexp.Regexp). Эти значения обладают мно-
жеством методов поиска и замены. В данной функции используется
метод regexp.Regexp.ReplaceAllStringFunc(), который принимает стро-
ку и функцию с сигнатурой func(string) string, вызывает эту функ-
цию для каждого найденного совпадения, передает ей совпавший текст
и замещает совпадение текстом, возвращаемым функцией.

анонимной функции,

line = wordRx.ReplaceAllStringFunc(line,
funcfunc(word string) string { returnreturn strings.ToUpper(word) })
Павел
Павел 8:16

regexp.Compile()

regexp.MustCompile()

Метод bufio.Reader.ReadString() читает (или, строго
говоря, декодирует) последовательность двоичных байтов, как текст в
кодировке UTF-8 (этот метод также может декодировать текст в 7-бит-
ной кодировке ASCII), до байта с указанным значением (включая его)
или до конца файла. Вызывающей программе возвращается прочитан-
ный текст в виде значения типа string и значение ошибки (или nil).
Павел
Павел 8:37

io/ioutil
, предоставляющий
ряд высокоуровневых функций, включая используемую здесь функ-
цию ioutil.ReadFile()
. Она читает файл целиком и возвращает все
его содержимое в виде последовательности двоичных байтов ([]byte)
вместе со значением ошибки.

преобразуются в
строку с помощью инструкции преобразования вида тип(переменная).
Павел
Павел 12:30

map (§4.3).
Тип map представляет собой коллекцию пар ключ/значение и обеспе-
чивает очень быстрый поиск по ключу
.
Павел
Павел 13:01

Отображения, срезы и каналы в языке Go создаются с помощью
встроенной функции make(). Она создает значение указанного типа
и возвращает ссылку на него.
Павел
Павел 13:10

Когда выполняется цикл по срезу (или массиву),
в каждой итерации возвращается индекс (начиная с 0) и элемент
с этим индексом (если срез непустой).
1 сентября
Павел
Павел 2:28

В
языке Go для этого можно использовать прием, удобный для слу-
чаев, когда необходимо определить наличие ключа в отображении,
который заключается в том, чтобы поместить две переменные слева
от оператора присваивания. Первой из них будет присваиваться зна-
чение, а второй – логическое значение, признак наличия искомого
ключа.
Павел
Павел 8:42

поддержке замыканий

Замыкание
– это функция, которая как бы «захватывает» окружение вызова,
например состояние функции, внутри которой было создано за-
мыкание,

недоступно за пределами функ-
ции, где оно объявлено. Локальные переменные
Павел
Павел 14:39

Одним из ключевых аспектов языка Go является возможность ис-
пользовать преимущества современных компьютеров на аппаратной
архитектуре с несколькими процессорами или ядрами,
2 сентября
Павел
Павел 8:14

go-подпрограммы
(goroutines)
, фактически очень легковесные потоки выполнения/

каналы, обеспечивающие надежное средство
одно- и двустороннего обмена данными между go-подпрограммами
и которые можно использовать для их синхронизации.

Многопоточная модель выполнения в языке Go основана на об-
мене данными, а не на их совместном использовании.

традиционными подходами,

механизма блокировок,

Пакет math предоставляет
математические функции для работы с вещественными числами
(§2.3.2), а пакет runtime – функции для доступа к свойствам про-
граммы времени выполнения, таким как тип платформы,

struct
Павел
Павел 8:34

Если в пакете имеется одна или более функций init()
, они авто-
матически будут вызваны до вызова функции main() в пакете main.

Каналы в языке Go имитируют каналы операционной системы
Unix и обеспечивают двусторонний (или, при желании, односторон-
ний) обмен данными. Каналы действуют подобно очередям FIFO
(First In, First Out – первый пришел, первый ушел), то есть они
сохраняют порядок следования элементов данных, передаваемых
через них. Элементы нельзя просто так выбросить из канала, но их
можно игнорировать при приеме.

messages := makemake(chanchan string, 10)
Павел
Павел 11:44

Каналы создаются с помощью функции make() (глава 7) и объ-
являются с помощью синтаксической конструкции chan Тип. Ин-
струкция выше создаст канал, позволяющий передавать и прини-
мать строки. Второй аргумент функции make() определяет размер
буфера (по умолчанию равен 0) – в данном случае выделяется бу-
фер, способный хранить до десяти строк. Если буфер канала оказы-
вается заполненным до конца, доступ к нему блокируется, пока из
канала не будет извлечен хотя бы один элемент. Это означает, что
через канал может передаваться любое количество элементов, при
условии что они своевременно будут извлекаться из канала, чтобы
освободить место для последующих элементов.
Павел
Павел 12:15

Канал, в котором
буфер имеет нулевой размер, может использоваться для отправки
значений, только когда на другом конце канала ожидается прием
данных. (Эффекта неблокирующих каналов можно добиться с по-
мощью инструкции select
,
Павел
Павел 12:41

отправим пару строк в канал:
messages <- "Leader"
messages <- "Follower"
3 сентября
Павел
Павел 2:26

отправим пару строк в канал:
messages <- "Leader"
messages <- "Follower"
Когда оператор передачи данных <- используется в роли двух-
местного оператора, левым операндом должен быть канал, а пра-
вым – значение, отправляемое в канал, типа указанного при объ-
явлении канала.

Обычно каналы создаются с целью обеспечить взаимодействия
между go-подпрограммами.

блокирующее поведение каналов можно использовать для синхро-
низации
.
Павел
Павел 2:35

После создания канала сразу же выполняется
инструкция defer
, вызывающая встроенную функцию close() (см.
табл. 5.1 ниже), чтобы обеспечить его закрытие, когда он станет не-
нужным.
Павел
Павел 20:17

инструкция go. Этой инструкции
передается вызов функции

будет выполняться в отдельной,
асинхронной go-подпрограмме. Это означает, что выполнение
текущей функции (то есть главной go-подпрограммы) будет не-
медленно продолжено с инструкции, следующей за объявлением
функции.
Павел
Павел 20:45

в инструкции go создается (и вызывается)
анонимная функция.
Павел
Павел 22:19

Функция выполняет бесконечный цикл,
ожидающий (то есть блокирующий выполнение собственной go-
подпрограммы, но не других go-подпрограмм и не функции, в кото-
рой go-подпрограмма была запущена) появления запросов в канале
questions, в данном случае значений типа polar struct. Когда в ка-
нале появятся полярные координаты, анонимная функция вычислит
соответствующие им декартовы координаты (с помощью пакета math
из стандартной библиотеки) и отправит ответ типа cartesian struct
(созданный с помощью синтаксиса составных литералов

Как они сконструировали cartesian?
Павел
Павел 22:25

ifif _, err := fmt.Sscanf(line, "%f %f", &radius, &?); err != nil {
fmt.Fprintln(os.Stderr, "invalid input")
continuecontinue
}
Павел
Павел 22:34

Однако требование ввести признак конца файла делает программу
polar2cartesian более гибкой, так как это дает возможность орга-
низовать чтение данных из произвольного внешнего файла с при-
менением операции перенаправления

функция прини-
мает исходную строку, формат – в данном случае два вещественных
числа, разделенных пробелом, – и один или более указателей на
переменные, куда должны быть записаны результаты разбора стро-
ки. (Оператор & получения адреса применяется для создания указа-
теля на значение, подробнее о нем рассказывается в §4.1.) Функция
возвращает количество элементов, которые ей успешно удалось выч-
ленить из исходной строки, и значение ошибки (или nil). В случае
ошибки функция выводит соответствующее сообщение в устройство
os.Stderr, что обеспечивает вывод сообщения в консоли, даже ес-
ли устройство os.Stdout программы будет перенаправлено в файл.
4 сентября
Павел
Павел 8:10

в аналогичных программах,
производящих массу тяжеловесных расчетов после каждого ввода,
можно было бы извлечь немалую выгоду от подобной организации
вычислений, например задействовав по одной go-подпрограмме на
каждую группу введенных значений.
Павел
Павел 8:23

В языке Go поддерживаются два типа комментариев
, оба заимство-
ванные из языка C++. Однострочные комментарии начинаются
с символов // и заканчиваются символом перевода строки. Они ин-
терпретируются просто как символ перевода строки. Универсальные
комментарии начинаются символами /* и заканчиваются символа-
ми */
, могут располагаться на нескольких строках. Когда универ-
сальный комментарий целиком располагается внутри строки (на-
пример, /* комментарий на одной строке */),

Когда универ-
сальный комментарий целиком располагается внутри строки (на-
пример, /* комментарий на одной строке */), он интерпретируется
как пробел, а когда целиком занимает одну или более строк – как
символ перевода строки
. (Символы перевода строки имеют большое
значение в языке Go,

Идентификаторами в языке Go могут быть непустые последова-
тельности букв и цифр, начинающиеся с буквы
Павел
Павел 22:03

Буквами считаются символ подчеркивания _
и все символы

Заглавные и строчные любые

Цифрами
считаются любые символы

Арабские цифры

Идентификаторы чувствительны к регистру символов, то есть
идентификаторы LINECOUNT, Linecount, LineCount, lineCount и
linecount, считаются разными идентификаторами. Идентифика-
торы, начинающиеся с буквы в верхнем регистре, то есть с буквы
из категории Юникода «Lu» (включая символы A–Z), считаются
общедоступными, или экспортируемыми
, в терминологии Go, а все

остальные – частными, или неэкспортируемыми
. (Это правило не
применяется к именам пакетов, которые в соответствии с соглаше-
ниями состоят только из букв в нижнем регистре.)

Пустой идентификатор _ играет роль временного идентифика-
тора в операциях присваивания, где ожидается переменная, и сбра-
сывает любые присваиваемые ему значения.
5 сентября
Павел
Павел 2:18

Константы объявляются с помощью ключевого слова const
.

Перемен-
ные могут объявляться с помощью ключевого слова var или операто-
ра сокращенного объявления переменных. Компилятор Go способен
определять тип объявляемой переменной по выражению справа от
оператора присваивания
. Однако при желании или необходимости до-
пускается явно указывать тип, например если требуется указать иной
тип, отличающийся от типа, определяемого компилятором автомати-
чески.

constconst limit = 512 // константа; совместима с любыми числовыми типами
constconst top uint16 = 1421 // константа; тип: uint16
start := -19 // переменная; определяемый компилятором тип: int

end := int64(9876543210) // переменная; тип: int64
varvar i int // переменная; значение 0; тип: int
varvar debug = falsefalse // переменная; определяемый компилятором тип: bool
checkResults := truetrue // переменная; определяемый компилятором тип: bool
stepSize := 1.5 // переменная; определяемый компилятором тип: float64
acronym := "FOSS" // переменная; определяемый компилятором тип: string

Для целочисленных литералов компилятор Go определяет тип
int
, для вещественных литералов – тип float64
, а для литералов
комплексных чисел – тип complex128 (число в названиях типов отра-
жает количество бит, занимаемых значением этого типа). Обычной
практикой считается не указывать тип явно, если только не требует-
ся использовать какой-то конкретный тип,

Типизи-
рованные числовые константы (такие как top в примере выше) мо-
гут использоваться только в выражениях с числовыми значениями
того же типа (если явно не преобразовывать их тип). Нетипизи-
рованные числовые константы могут использоваться в выражени-
ях с числовыми значениями любых встроенных типов (например,
константу limit можно использовать в выражениях с целыми или
вещественными числами).
Павел
Павел 2:34

Переменной i в примере выше не было явно присвоено какое-
либо значение. Это не влечет за собой никаких неожиданностей,
потому что в языке Go переменным всегда присваиваются нулевые
значения
, соответствующие их типам, если явно не было присвоено
другое значение. То есть числовая переменная гарантированно будет
инициализирована нулем, а строковая – пустой строкой, если явно
не указано другое значение. Благодаря этому устраняется проблема
появления бессмысленных значений в неинициализированных пере-
менных, от которой страдают некоторые другие языки.
Павел
Павел 10:36

Вместо того чтобы многократно повторять ключевое слово const,
когда требуется определить несколько констант, их можно сгруппи-
ровать в одно объявление с единственным ключевым словом const

можно использовать для груп-
пировки объявлений переменных с ключевым словом var.)

когда требуется лишь объявить константы с отличающимися

значениями, и при этом сами значения не играют никакой роли,
можно воспользоваться поддержкой перечислений
.
constconst Cyan = 0
constconst Magenta = 1
constconst Yellow = 2
const (
Cyan = 0
Magenta = 1
Yellow = 2
)
const (
Cyan = iotaiota // 0
Magenta // 1
Yellow // 2
)
Павел
Павел 10:52

В сгруппиро-
ванном объявлении констант первой из них присваивается нулевое
значение
, если явно не было указано другое (определенное значение
или iota), а второй и всем остальным – значение предшествующей
константы или iota
, если значением предшествующей константы яв-
ляется iota. При этом для каждой последующей константы значение
iota оказывается на единицу больше, чем для предшествующей.
Павел
Павел 11:14

все константы получат
значение iota (причем константы Magenta и Yellow – неявно).
Павел
Павел 11:32

если в правом фрагменте убрать явное при-
сваивание значения iota, константа Cyan получит значение 0, кон-
станта Magenta получит значение константы Cyan, и константа Yellow
получит значение константы Magenta – то есть все они получат зна-
чение 0.

если в правом фрагменте убрать явное при-
сваивание значения iota, константа Cyan получит значение 0, кон-
станта Magenta получит значение константы Cyan, и константа Yellow
получит значение константы Magenta – то есть все они получат зна-
чение 0.
Павел
Павел 18:37

Значение iota можно еще использовать в комбинации с веще-
ственными числами, простыми выражениями и пользовательскими
типами.
typetype BitFlag int
const (
Active BitFlag = 1 « iota // 1 « 0 == 1
Send // Неявно BitFlag = 1 « iota // 1 « 1 == 2
Receive // Неявно BitFlag = 1 « iota // 1 « 2 == 4

Переменным типа BitFlag можно присваивать любые значения
типа int, тем не менее BitFlag – это совершенно другой тип, и значе-
ния этого типа могут участвовать в выражениях с числами типа int,
только если они явно будут приводиться к типу int (или значения
типа int будут явно приводиться к типу BitFlags).
Павел
Павел 19:31

функции вывода из пакета fmt
используют метод String() типа, если он определен. То есть, чтобы
сделать вывод значений типа BitFlag более информативным, до-
статочно просто добавить в него соответствующий метод String().

funcfunc (flag BitFlag) String() string {
Павел
Павел 20:11

языке Go имеются два встроенных логических значения, true и
false, оба относятся к типу bool. Кроме того, в Go поддерживаются
стандартные логические операторы и операторы сравнения, возвра-
щающие результат типа bool,

К выражениям с двухместными логическими операторами (||и
&&) применяется сокращенный порядок вычислений
Павел
Павел 20:23

Операторы сравнения в языке Go (<, <=, ==, !=, >=, >) наклады-
вают определенные ограничения на сравниваемые значения. Два
значения должны иметь один и тот же тип или, если они являют-
ся интерфейсами, должны содержать реализацию одного и того же
интерфейса. Если одно из значений является константой, его тип
должен быть совместим с типом другого значения. То есть нети-
пизированные числовые константы можно сравнивать с числовыми
значениями любого типа, но числовые значения разных типов, ни
одно из которых не является константой, сравнивать нельзя, если
явно не преобразовать тип одного операнда в тип другого.
Павел
Павел 20:29

Операторы == и != могут применяться к операндам любых со-
вместимых типов, включая массивы и структуры, элементы которых
могут сравниваться между собой с помощью == и !=. Эти операторы
не могут использоваться для сравнения срезов, однако при необ-
ходимости такое сравнение можно выполнить с помощью функции
reflect.DeepEqual() из стандартной библиотеки. Операторы == и !=
можно использовать для сравнения двух указателей или двух интер-
фейсов, или для сравнения указателя, интерфейса или ссылки (на-
пример, на канал, отображение или срез) со значением nil. Осталь-
ные операторы сравнения (<, <=, >=, >) могут применяться только к
Павел
Павел 20:34

числам и строкам.

не поддерживается перегрузка операторов,
Павел
Павел 20:40

Таблица 2.3. Логические операторы и операторы сравнения
6 сентября
Павел
Павел 21:16

Числовые типы

поддержку целых значе-
ний типа big.Int и рациональных значений типа big.Rat, которые
имеют неограниченный размер (то есть их размеры ограничиваются
только доступным объемом машинной памяти). Все числовые типы
считаются отличными друг от друга, то есть к числовым значениям
разных типов (например, к значениям типов int32 и int) нельзя
применять двухместные арифметические операторы или операторы
сравнения (такие как + или <).
Павел
Павел 22:10

поэтому они могут складываться или сравниваться с другими чис-
ловыми значениями независимо от их типов (встроенных).
Чтобы выполнить арифметическую операцию или сравнить два
числовых значения разных типов, необходимо выполнить преобразо-
вание типов, обычно к большему типу, дабы избежать потери точности.
Синтаксис преобразования типа имеет вид: тип(значение),
7 сентября
Павел
Павел 7:41

constconst factor = 3 // Константа factor совместима со всеми числовыми типами
i := 20000 // i автоматически получит тип int
i *= factor
j := int16(20) // j получит тип int16; то же, что и: var j int16 = 20
i += int(j) // Типы должны совпадать, поэтому преобразование обязательно
k := uint8(0) // То же, что и: var k uint8
k = uint8(i) // Успех, но k получит значение i, усеченное до 8 бит
fmt.Println(i, j, k) // Выведет: 60020 20 116
Павел
Павел 12:10

Посмотреть все изображения
Павел
Павел 22:23

Константа
math.MaxUint8 определена в пакете math, где также имеются похожие
константы для остальных встроенных числовых типов.
Павел
Павел 22:30

Числовые значения одного типа могут сравниваться операторами
сравнения (табл. 2.3 выше). Аналогично к значениям одного типа могут

Елена николаевна Галина ивановна
Павел
Павел 22:39

применяться арифметические операторы – в табл. 2.4 перечислены
операторы, которые могут применяться к значениям любых числовых
(встроенных) типов, и в табл. 2.6 – только к целочисленным значениям.

Выражения, определяющие значения констант
, вычисляются на эта-
пе компиляции – в них могут использоваться любые арифметические
и логические операторы, а также операторы сравнения.



makefile — это файл, содержащий набор инструкций, используемых утилитой make в инструментарии автоматизации сборки.

https://ru.m.wikipedia.org/wiki/Автоматизация_сборки
Павел
Павел 13:45

утилита make,
Павел
Павел 14:35

инструкции вместе со своими зависимостями указаны в makefile
Павел
Павел 21:30

Make-файл содержит «правила» в следующей форме:

target: dependencies system command(s)

Целью обычно является имя файла, который генерируется программой

Зависимость (также называется предварительным условием) — это файл, используемый в качестве входных данных для создания цели.

Цель часто зависит от нескольких файлов.

Системная команд(ы) (также называемая способом) — это действие, которое выполняет утилита make. Способ может содержать более одной команды, либо в той же строке или каждая в своей строке.
21 сентября
Павел
Павел 0:59

make-файл выполняется при помощи команды make, например, make [options] [target1 target2 ...]. По умолчанию, когда утилита make ищет make-файл и если имя make-файла не было указано в качестве параметра, то make пробует следующие имена по порядку: makefile и Makefile

A makefile works upon the principle that files only need recreating if their dependencies are newer than the file being created/recreated.

The makefile is recursively carried out (with dependency prepared before each target depending upon them) until everything has been updated (that requires updating) and the primary/ultimate target is complete. These instructions with their dependencies are specified in a makefile. If none of the files that are prerequisites have been changed since the last time the program was compiled, no actions take place.
Павел
Павел 10:14

‘#’ in a line of a makefile starts a comment. It and the rest of the line is ignored
Павел
Павел 20:50

Note the use of meaningful indentation in specifying commands; also note that the indentation must consist of a single <tab> character.
22 сентября
Павел
Павел 23:57

The make file will recompile all objects if any of the headers change, but if an individual .c file changes, the only work that will need to be done is to recompile that file and then relink all the objects. Well written make rules can help reduce compile time by detecting what did and did not change
23 сентября
Павел
Павел 20:26

We define the same, reusable rule to make each .o from each .c, and to make each target from the objects
Павел
Павел 20:33

The targets all and clean are named .PHONY because they don't refer to real files, but are things we want make to do.
Павел
Павел 21:17

Make is invoked with a list of target file names to build as command-line arguments:

make [TARGET ...]

Without arguments, Make builds the first target that appears in its makefile, which is traditionally a symbolic "phony" target named all.
сегодня
Павел
Павел 9:26

Язык make-файлов похож на декларативное программирование . [35] [36] [37] Этот класс языка, в котором описаны необходимые конечные условия, но порядок, в котором должны выполняться действия, не важен,



makefile consists of rules. Each rule begins with a textual dependency line which defines a target followed by a colon (:) and optionally an enumeration of components (files or other targets) on which the target depends. The dependency line is arranged so that the target (left hand of the colon) depends on components (right hand of the colon). It is common to refer to components as prerequisites of the target
27 сентября
Павел
Павел 18:13

25 джеков

сырки творожные

молоко
Павел
Павел 19:01

Пшенка

Семечки
Павел
Павел 20:48

Each dependency line may be followed by a series of TAB indented command lines which define how to transform the components (usually source files) into the target (usually the "output"). If any of the prerequisites has a more recent modification time than the target, the command lines are run. The GNU Make documentation refers to the commands associated with a rule as a "recipe".
Павел
Павел 21:10

The first command may appear on the same line after the prerequisites, separated by a semicolon,

targets : prerequisites ; command

for example,

hello: ; @echo "hello"

Each command line must begin with a tab character to be recognized as a command
28 сентября
Павел
Павел 2:04

A rule may have no command lines defined. The dependency line can consist solely of components that refer to targets, for example:

realclean: clean distclean
30 сентября
Павел
Павел 1:28






Командные строки могут иметь один или несколько из следующих трех префиксов:

дефис-минус (-), указав , что ошибки игнорируются

в знак (@), указав , что команда не выводится на стандартный вывод перед его выполнением

знак плюс (+), команда выполняется , даже если Make вызывается в режиме «не выполнять»
Павел
Павел 10:34

Продолжение строки обозначается знаком обратной косой \черты в конце строки
Павел
Павел 10:43



. Правила преобразования задаются в скрипте с именем Makefile, который должен находиться в корне рабочей директории проекта. Сам скрипт состоит из набора правил, которые в свою очередь описываются:

1) целями (то, что данное правило делает);
2) реквизитами (то, что необходимо для выполнения правила и получения целей);
3) командами (выполняющими данные преобразования).

В общем виде синтаксис makefile можно представить так:

Индентация осуществляется исключительно при помощи символов табуляции, # каждой команде должен предшествовать отступ <цели>: <реквизиты> <команда #1> ... <команда #n>

Павел
Павел 11:10

правило make это ответы на три вопроса:

{Из чего делаем? (реквизиты)} —-> [Как делаем? (команды)] —-> {Что делаем? (цели)}
Несложно заметить что процессы трансляции и компиляции очень красиво ложатся на эту схему:

{исходные файлы} —-> [трансляция] —-> {объектные файлы}
{объектные файлы} —-> [линковка] —-> {исполнимые файлы}

Для его компиляции достаточно очень простого мэйкфайла:

hello: main.c gcc -o hello main.c

https://m.habr.com/ru/post/211751/
Просто о make
Просто о make
m.habr.com
Павел
Павел 11:58

Данный Makefile состоит из одного правила, которое в свою очередь состоит из цели — «hello», реквизита — «main.c», и команды — «gcc -o hello main.c». Теперь, для компиляции достаточно дать команду make в рабочем каталоге. По умолчанию make станет выполнять самое первое правило, если цель выполнения не была явно указана при вызове:

$ make <цель>

Makefile, выполняющий компиляцию этой программы может выглядеть так:

hello: main.c hello.c gcc -o hello main.c hello.c
Павел
Павел 12:08

make сопоставляет время изменения целей и их реквизитов (используя данные файловой системы), благодаря чему самостоятельно решает какие правила следует выполнить, а какие можно просто проигнорировать:

в качестве make целей могут выступать не только реальные файлы
Павел
Павел 12:21

фиктивные (phony) цели.

Печенье

краткий список стандартных целей:

all — является стандартной целью по умолчанию. При вызове make ее можно явно не указывать.

clean — очистить каталог от всех файлов полученных в результате компиляции.

install — произвести инсталляцию

uninstall — и деинсталляцию соответственно.
Павел
Павел 13:34

Для того чтобы make не искал файлы с такими именами, их следует определить в Makefile, при помощи директивы .PHONY. Далее показан пример Makefile с целями all, clean, install и uninstall:
Павел
Павел 13:57

в цели all не указаны команды; все что ей нужно — получить реквизит hello

, чтобы гарантированно полностью пересобрать проект, нужно предварительно очистить рабочий каталог:

$ make clean $ make
Павел
Павел 14:04

Переменные в make представляют собой именованные строки и определяются очень просто:

<VAR_NAME> = <value string>
Существует негласное правило, согласно которому следует именовать переменные в верхнем регистре, например:

SRC = main.c hello.c

Для использования значения переменной ее следует разименовать при помощи конструкции $(<VAR_NAME>)
4 октября
Павел
Павел 13:18

https://golang.org/doc/effective_go.html

The gofmt program (also available as go fmt, which operates at the package level rather than source file level) reads a Go program and emits the source in a standard style
7 октября
Павел
Павел 20:11

IndentationWe use tabs for indentation
Павел
Павел 20:16

Go has no line length limit
Павел
Павел 20:35

imply [ɪmˈplaɪ] гл

подразумевать, означать, предполагать, значить, иметь в виду, обозначать

mean, assume
Павел
Павел 20:42





to make it seem likely that something is true or existsSYNONYM suggest

x«8 + y«16 means what the spacing implies, unlike in the other languages.
10 октября
Павел
Павел 16:37

халва

шоколадка

2 или 3

баунти
Павел
Павел 17:02

Соус
Павел
Павел 17:48

Go provides C-style /* */ block comments and C++-style // line comments.
Павел
Павел 19:27

godocобрабатывают исходные файлы Go для извлечения документации о содержимом пакета
Павел
Павел 20:16

Every package should have a package comment, a block comment preceding the package clause. For multi-file packages, the package comment only needs to be present in one file, and any one will do. The package comment should introduce the package and provide information relevant to the package as a whole
Павел
Павел 20:28

/*
Павел
Павел 20:33

// Package path implements utility routines for // manipulating slash-separated filename paths.
Павел
Павел 20:51

Inside a package, any comment immediately preceding a top-level declaration serves as a doc comment for that declaration. Every exported (capitalized) name in a program should have a doc comment.
11 октября
Павел
Павел 13:46

cache.zip
4 КБ
12 октября
Павел
Павел 18:47

Барни 25 штук

Пластелин

Ряженка

Минералка

Шампунь для окрашенных волос
Павел
Павел 19:14

семечек
Павел
Павел 19:57

Орхидея
Павел
Павел 20:41

Фильтр
13 октября
Павел
Павел 8:24

visibility of a name outside a package is determined by whether its first character is upper case
14 октября
Павел
Павел 8:20

По соглашению, пакетам даются однословные имена в нижнем регистре; не должно быть необходимости в подчеркивании или смешанных заглавных буквах. Сделайте ошибку в сторону краткости, поскольку каждый, кто использует ваш пакет, будет вводить это имя
Павел
Павел 8:38

the package name is the base name of its source directory; the package in src/encoding/base64 is imported as "encoding/base64
Павел
Павел 8:43

, bufio.Reader не конфликтует с io.Reader
15 октября
Павел
Павел 8:33

. Если у вас есть поле с именем owner(нижний регистр, неэкспортированный), следует вызывать метод получения Owner(верхний регистр, экспортируется), а не GetOwner

Использование имен в верхнем регистре для экспорта обеспечивает ловушку, позволяющую отличить поле от метода. При необходимости, скорее всего, будет вызвана функция установки SetOwner
16 октября
Павел
Павел 8:15

один метод интерфейсы названы именем метода плюс суффикс -er или подобной модификации для построения агента существительное: Reader, Writer, Formatter, и CloseNotifierт.д.
Павел
Павел 8:35

. Read, Write, Close, Flush, String
17 октября
Павел
Павел 2:22

use MixedCaps or mixedCaps rather than underscores to write multiword names.
18 октября
Павел
Павел 2:45

lexer uses a simple rule to insert semicolons automatically

if the newline comes after a token that could end a statement, insert a semicolon”

A semicolon can also be omitted immediately before a closing brace, so a statement such as

go func() { for { dst <- <-src } }()

needs no semicolons. Idiomatic Go programs have semicolons only in places such as for loop clauses, to separate the initializer, condition, and continuation elements.
Павел
Павел 13:24

A semicolon can also be omitted immediately before a closing brace, so a statement such as

go func() { for { dst <- <-src } }()

needs no semicolons. Idiomatic Go programs have semicolons only in places such as for loop clauses, to separate the initializer, condition, and continuation elements
20 октября
Павел
Павел 2:24

you cannot put the opening brace of a control structure (if, for, switch, or select) on the next line. If you do, a semicolon will be inserted before the brace
21 октября
Павел
Павел 2:40

Control structures

no parentheses and the bodies must always be brace-delimited.
22 октября
Павел
Павел 2:48

ifвыглядит так:

if x> 0 { вернуть y }

Обязательные фигурные скобки

Since if and switch accept an initialization statement, it's common to see one used to set up a local variable.

if err := file.Chmod(0664); err != nil { log.Print(err) return err }
Павел
Павел 9:27

случаи ошибок обычно заканчиваются return операторами, результирующий код не требует elseоператоров
23 октября
Павел
Павел 10:22

f, err := os.Open(name)

This statement declares two variables, f and err
24 октября
Павел
Павел 2:26

err is declared by the first statement, but only re-assigned in the second.

In a := declaration a variable v may appear even if it has already been declared, provided:

this declaration is in the same scope as the existing declaration of v (if v is already declared in an outer scope, the declaration will create a new variable §),

the corresponding value in the initialization is assignable to v, and

there is at least one other variable that is created by the declaration.

it easy to use a single err value, for example, in a long if-else chain

scope of function parameters and return values is the same as the function body, even though they appear lexically outside the braces that enclose the body.
Павел
Павел 13:24

. There are three forms, only one of which has semicolons.

// Like a C for for init; condition; post { } // Like a C while for condition { } // Like a C for(;;) for { }
Павел
Павел 13:34

Short declarations make it easy to declare the index variable right in the loop.

sum := 0 for i := 0; i < 10; i++ { sum += i }

If you're looping over an array, slice, string, or map, or reading from a channel, a range clause can manage the loop.

for key, value := range oldMap { newMap[key] = value }

If you only need the first item in the range (the key or index), drop the second:

for key := range m { if key.expired() { delete(m, key) } }

If you only need the second item in the range (the value), use the blank identifier, an underscore, to discard the first:

sum := 0 for _, value := range array { sum += value }
Павел
Павел 13:45

The name (with associated builtin type) rune is Go terminology for a single Unicode code point.
Павел
Павел 20:38

Сырки гречка минералка
Павел
Павел 20:47

use parallel assignment (although that precludes ++ and —).

// Reverse a for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 { a[i], a[j] = a[j], a[i] }
25 октября
Павел
Павел 16:10

The expressions need not be constants or even integers,

write an if-else-if-else chain as a switch.
27 октября
Павел
Павел 2:35

There is no automatic fall through, but cases can be presented in comma-separated lists.

func shouldEscape(c byte) bool { switch c { case ' ', '?', '&', '=', '#', '+', '%':

break statements can be used to terminate a switch early
28 октября
Павел
Павел 2:36

Go that can be accomplished by putting a label on the loop and "breaking" to that label

continue statement also accepts an optional label but it applies only to loops.

Type switch
29 октября
Павел
Павел 2:45

discover the dynamic type of an interface variable. Such a type switch uses the syntax of a type assertion with the keyword type inside the parentheses.

One of Go's unusual features is that functions and methods can return multiple values. This form can be used to improve on a couple of clumsy idioms in C programs: in-band error returns such as -1 for EOF and modifying an argument passed by address
вчера
Павел
Павел 2:43

Write can return a count and an error:

it returns the number of bytes written and a non-nil error when n != len(b).



31 октября
Павел
Павел 2:08

The return or result "parameters" of a Go function can be given names and used as regular variables, just like the incoming parameters. When named, they are initialized to the zero values for their types when the function begins; if the function executes a return statement with no arguments, the current values of the result parameters are used as the returned values.

The names are not mandatory but they can make code shorter and clearer: they're documentation
Павел
Павел 13:18

Go's defer statement schedules a function call (the deferred function) to be run immediately before the function executing the defer returns. It's an unusual but effective way to deal with situations such as resources that must be released regardless of which path a function takes to return. The canonical examples are unlocking a mutex or closing a file.
Павел
Павел 14:50

Deferring a call to a function such as Close has two advantages. First, it guarantees that you will never forget to close the file, a mistake that's easy to make if you later edit the function to add a new return path. Second, it means that the close sits near the open, which is much clearer than placing it at the end of the function.
Павел
Павел 18:08

, it guarantees that you will never forget to close the file, a mistake that's easy to make if you later edit the function to add a new return path. Second, it means that the close sits near the open, which is much clearer than placing it at the end of the function.
Павел
Павел 18:25

The arguments to the deferred function (which include the receiver if the function is a method) are evaluated when the defer executes, not when the call executes
Павел
Павел 18:41

Deferred functions are executed in LIFO order, so this code will cause 4 3 2 1 0
Павел
Павел 19:11

Allocation with new ¶

Go has two allocation primitives, the built-in functions new and make.
1 ноября
Павел
Павел 10:38



new
Павел
Павел 11:33

, it returns a pointer to a newly allocated zero value of type T.
Павел
Павел 17:11

The zero-value-is-useful property works transitively

. In the next snippet, both p and v will work correctly without further arrangement.
Павел
Павел 17:25

Constructors and composite literals
2 ноября
Павел
Павел 2:12

initializing constructor

f := new(File) f.fd = fd f.name = name f.dirinfo = nil f.nepipe = 0

composite literal,

OK to return the address of a local variable;

the storage associated with the variable survives after the function returns. In fact, taking the address of a composite literal allocates a fresh instance each time it is evaluated, so we can combine these last two lines.

return &File{fd, name, nil, 0}

The fields of a composite literal are laid out in order and must all be present. However, by labeling the elements explicitly as field:value pairs, the initializers can appear in any order, with the missing ones left as their respective zero values
вчера
Павел
Павел 0:50



https://blog.golang.org/json

To encode JSON data we use the Marshal function.

func Marshal(v interface{}) ([]byte, error)

b, err := json.Marshal(m)

Если все в порядке, errбудет nilи bбудет []byteсодержать эти данные JSON:

b == []byte(`{"Name":"Alice","Body":"Hello","Time":1294706395881547000}`)

Пакет json получает доступ только к экспортированным полям структурных типов (начинающимся с заглавной буквы). Следовательно, в выводе JSON будут присутствовать только экспортированные поля структуры.
Павел
Павел 18:43

A map is an unordered group of elements of one type, called the element type, indexed by a set of unique keys of another type, called the key type. The value of an uninitialized map is nil.

MapType = "map" "[" KeyType "]" ElementType . KeyType = Type .
4 ноября
Павел
Павел 2:22

The number of map elements is called its length. For a map m, it can be discovered using the built-in function len and may change during execution. Elements may be added during execution using assignments and retrieved with index expressions; they may be removed with the delete built-in function.

A new, empty map value is made using the built-in function make, which takes the map type and an optional capacity hint as arguments:

make(map[string]int) make(map[string]int, 100)

The initial capacity does not bound its size: maps grow to accommodate the number of items stored in them, with the exception of nil maps. A nil map is equivalent to an empty map except that no elements may be added.
Павел
Павел 14:23

Maps are a convenient and powerful built-in data structure that associate values of one type (the key) with values of another type (the element or value). The key can be of any type for which the equality operator is defined, such as integers, floating point and complex numbers, strings, pointers, interfaces (as long as the dynamic type supports equality), structs and arrays. Slices cannot be used as map keys, because equality is not defined on them. Like slices, maps hold references to an underlying data structure. If you pass a map to a function that changes the contents of the map, the changes will be visible in the caller.

Maps can be constructed using the usual composite literal syntax with colon-separated key-value pairs, so it's easy to build them during initialization.

Maps are a convenient and powerful built-in data structure that associate values of one type (the key) with values of another type (the element or value). The key can be of any type for which the equality operator is defined, such as integers, floating point and complex numbers, strings, pointers, interfaces (as long as the dynamic type supports equality), structs and arrays. Slices cannot be used as map keys, because equality is not defined on them. Like slices, maps hold references to an underlying data structure. If you pass a map to a function that changes the contents of the map, the changes will be visible in the caller.

Maps can be constructed using the usual composite literal syntax with colon-separated key-value pairs, so it's easy to build them during initialization.

Maps are a convenient and powerful built-in data structure that associate values of one type (the key) with values of another type (the element or value). The key can be of any type for which the equality operator is defined, such as integers, floating point and complex numbers, strings, pointers, interfaces (as long as the dynamic type supports equality), structs and arrays. Slices cannot be used as map keys, because equality is not defined on them. Like slices, maps hold references to an underlying data structure. If you pass a map to a function that changes the contents of the map, the changes will be visible in the caller.

Maps can be constructed using the usual composite literal syntax with colon-separated key-value pairs, so it's easy to build them during initialization.

https://golang.org/doc/effective_go.html#maps
Effective Go - The Go Programming Language..
golang.org
Павел
Павел 16:42

https://sndeep.info/en

https://checkcoverage.apple.com/ru/ru/
Проверка права на сервисное обслуживание..
checkcoverage.apple.com
Павел
Павел 20:35

Assigning and fetching map values looks syntactically just like doing the same for arrays and slices except that the index doesn't need to be an integer.

offset := timeZone["EST"]
Павел
Павел 20:49

An attempt to fetch a map value with a key that is not present in the map will return the zero value for the type of the entries in the map. For instance, if the map contains integers, looking up a non-existent key will return 0. A set can be implemented as a map with value type bool. Set the map entry to true to put the value in the set, and then test it by simple indexing.
Павел
Павел 20:57

Sometimes you need to distinguish a missing entry from a zero value. Is there an entry for "UTC" or is that 0 because it's not in the map at all? You can discriminate with a form of multiple assignment.
5 ноября
Павел
Павел 2:26

var seconds int var ok bool seconds, ok = timeZone[tz]

this is called the “comma ok” idiom. In this example, if tz is present, seconds will be set appropriately and ok will be true; if not, seconds will be set to zero and ok will be false.

To test for presence in the map without worrying about the actual value, you can use the blank identifier (_) in place of the usual variable for the value.

_, present := timeZone[tz]
6 ноября
Павел
Павел 2:36

https://golang.org/ref/spec#Declarations_and_scope
7 ноября
Павел
Павел 20:08

no identifier may be declared in both the file and package block.
Павел
Павел 20:19

The scope of a declared identifier is the extent of source text in which the identifier denotes
Павел
Павел 20:30

The scope of an identifier denoting a constant, type, variable, or function (but not method) declared at top level (outside any function) is the package block.

The scope of the package name of an imported package is the file block of the file containing the import declaration.
8 ноября
Павел
Павел 2:27

The scope of a constant or variable identifier declared inside a function begins at the end of the ConstSpec or VarSpec (ShortVarDecl for short variable declarations) and ends at the end of the innermost containing block.

The scope of a type identifier declared inside a function begins at the identifier in the TypeSpec and ends at the end of the innermost containing block.
10 ноября
Павел
Павел 2:21

Thus we could say

return &File{fd: fd, name: name}

https://golang.org/doc/effective_go.html#allocation_new
Effective Go - The Go Programming Language..
golang.org

В качестве предельного случая, если составной литерал вообще не содержит полей, он создает нулевое значение для типа. Выражения new(File)и &File{}эквивалентны.

Composite literals can also be created for arrays, slices, and maps, with the field labels being indices or map keys as appropriate
11 ноября
Павел
Павел 2:25

The built-in function make(T, args) serves a purpose different from new(T). It creates slices, maps, and channels only, and it returns an initialized (not zeroed) value of type T (not *T)

. The built-in function make(T, args) serves a purpose different from new(T). It creates slices, maps, and channels only, and it returns an initialized (not zeroed) value of type T (not *T). The reason for the distinction is that these three types represent, under the covers, references to data structures that must be initialized before use. A slice, for example, is a three-item descriptor containing a pointer to the data (inside an array), the length, and the capacity, and until those items are initialized, the slice is nil. For slices, maps, and channels, make initializes the internal data structure and prepares the value for use.
13 ноября
Павел
Павел 20:32

allocates an array of 100 ints and then creates a slice structure with length 10 and a capacity of 100 pointing at the first 10 elements of the array. (When making a slice, the capacity can be omitted; see the section on slices for more information.) In contrast, new([]int) returns a pointer to a newly allocated, zeroed slice structure, that is, a pointer to a nil slice value.
Павел
Павел 20:43

. To obtain an explicit pointer allocate with new or take the address of a variable explicitly.
14 ноября
Павел
Павел 12:53

Arrays are useful when planning the detailed layout of memory and sometimes can help avoid allocation, but primarily they are a building block for slices,
Павел
Павел 13:08

differences between the ways arrays work in Go and C. In Go,

Arrays are values. Assigning one array to another copies all the elements.

In particular, if you pass an array to a function, it will receive a copy

The size of an array is part of its type. The types [10]int and [20]int are distinct.
Павел
Павел 13:14

The value property can be useful but also expensive; if you want C-like behavior and efficiency, you can pass a pointer to the array

, most array programming in Go is done with slices ra

Slices hold references to an underlying array, and if you assign one slice to another, both refer to the same array

If a function takes a slice argument, changes it makes to the elements of the slice will be visible to the caller, analogous to passing a pointer to the underlying array.
15 ноября
Павел
Павел 13:04

A Read function can therefore accept a slice argument rather than a pointer and a count;
Павел
Павел 13:11

To read into the first 32 bytes of a larger buffer buf, slice (here used as a verb) the buffer.
Павел
Павел 13:27

. If the data exceeds the capacity, the slice is reallocated
16 ноября
Павел
Павел 18:52

Мише сырки

Хлеб
Павел
Павел 20:36

The function uses the fact that len and cap are legal when applied to the nil slice, and return 0.
Павел
Павел 20:49

The length of a slice may be changed as long as it still fits within the limits of the underlying array
Павел
Павел 20:56

}

We must return the slice afterwards because, although Append can modify the elements of slice, the slice itself (the run-time data structure holding the pointer, length, and capacity) is passed by value
Павел
Павел 21:01

like this:

type Transform [3][3]float64 // A 3x3 array, really an array of arrays. type LinesOfText [][]byte // A slice of byte slices.
21 ноября
Павел
Павел 11:22

Because slices are variable-length, it is possible to have each inner slice be a different length. That can be a common situation, as in our LinesOfText example: each line has an independent length.
Павел
Павел 12:26

each line has an independent length.

text := LinesOfText{ []byte("Now is the time"), []byte("for all good gophers"), []byte("to bring some fun to the party."), }
Павел
Павел 12:40

allocate a 2D slice
Павел
Павел 13:45

to allocate each slice independently; the other is to allocate a single array and point the individual slices into it.
Павел
Павел 20:36

You don't need to provide a format string. For each of Printf, Fprintf and Sprintf there is another pair of functions, for instance Print and Println. These functions do not take a format string but instead generate a default format for each argument. The Println versions also insert a blank between arguments and append a newline to the output while the Print versions add blanks only if the operand on neither side is a string.
Павел
Павел 20:45

The formatted print functions fmt.Fprint and friends take as a first argument any object that implements the io.Writer interface
Павел
Павел 14:12

%d

use the type of the argument

default conversion, such as decimal for integers,

catchall format %v (for “value”); the result is exactly what Print and Println would produce
25 ноября
Павел
Павел 9:31

Посмотреть все изображения
26 ноября
Павел
Павел 9:44

, that format can print any value, even arrays, slices, structs, and maps

which gives output:

map[CST:-21600 EST:-18000 MST:-25200 PST:-28800 UTC:0]

For maps, Printf and friends sort the output lexicographically by key
29 ноября
Павел
Павел 12:20

modified format %+v annotates the fields of the structure with their names, and for any value the alternate format %#v prints the value in full Go syntax.
Павел
Павел 12:30

&{7 -2.35 abc def} &{a:7 b:-2.35 c:abc def} &main.T{a:7, b:-2.35, c:"abc\tdef"} map[string]int{"CST":-21600, "EST":-18000, "MST":-25200, "PST":-28800, "UTC":0}
Павел
Павел 20:30

prints the type of a value.

fmt.Printf("%T\n", timeZone)
1 декабря
Павел
Павел 20:01

If you want to control the default format for a custom type, all that's required is to define a method with the signature String() string on the type. For our simple type T, that might look like this.

print values of type T as well as pointers to T, the receiver for String must be of value type
Павел
Павел 20:13

calling Sprintf in a way that will recur into your String method indefinitely. This can happen if the Sprintf call attempts to print the receiver directly as a string, which in turn will invoke the method again
Павел
Павел 20:41

Within the function Printf, v acts like a variable of type []interface{} but if it is passed to another variadic function, it acts like a regular list of arguments.
Павел
Павел 20:47

We write ... after v in the nested call to Sprintln to tell the compiler to treat v as a list of arguments; otherwise it would just pass v as a single slice argument.
4 декабря
Павел
Павел 21:11

a ... parameter can be of a specific type

Append
5 декабря
Павел
Павел 12:39

func append(slice []T, elements ...T) []T

where T is a placeholder for any given type. You can't actually write a function in Go where the type T is determined by the caller.

it needs support from the compiler
Павел
Павел 16:23

https://dev-gang.ru/article/rabota-s-kontekstom-v-go-..
Работа с контекстом в Go
Работа с контекстом в Go
dev-gang.ru
Павел
Павел 19:55

чтобы поддерживать хороший контроль потока, понадобится библиотека контекста

стандартной библиотекой с Go

Контекст - это крайний срок, который вы можете передать в запущенный процесс в своем коде. Этот крайний срок может указывать на то, что процесс должен прекратить работу и вернуться после выполнения условия. Это становится полезным при обращении к внешним API, базам данных, как показано выше, или системным командам.

горутины - это легкие потоки, которые можно запускать для процессов, а каналы - это конвейеры, используемые для передачи данных между этими новыми процессами.

Библиотека контекста определяет новый интерфейс под названием Context. В интерфейсе контекста есть несколько интересных полей
Павел
Павел 20:40

Поле Done - это канал, который закрывается, когда работа, выполненная для контекста, должна быть отменена. Эта операция может происходить асинхронно. Канал может вернуться как nil, если связанный контекст никогда не может быть отменен. Различные типы контекстов организуют отмену работы в зависимости от обстоятельств, в которые мы попадем.

Err вернет nil, пока не будет закрыто Done. После чего Err либо вернет Cancelled, если контекст был отменен, либо DealineExceeded, если крайний срок контекста прошел.

Поле «Значение» представляет собой интерфейс «ключ-значение», который будет возвращать значение, связанное с контекстом, как ключ или ноль, если не было связанного значения. Значения следует использовать с осторожностью, поскольку они предназначены не для передачи параметров в функцию, а для процессов передачи данных в области запроса и границ API.
6 декабря
Павел
Павел 17:12

у библиотека контекста является частью параллелизма. На данный момент горутины - это легкие потоки, которые можно запускать для процессов, а каналы - это конвейеры, используемые для передачи данных между этими новыми процессами.

. Контекст должен принимать форму, необходимую для каждого использования. Он должен быть бесформенным, или, говоря словами Брюса Ли, быть похожим на воду. Ваш контекст должен течь через ваш код и развиваться в зависимости от потребности.
вчера
Павел
Павел 0:55

https://golang-blog.blogspot.com/2019/10/concurrency-..
Паттерны конкурентности в Golang: Context
Паттерны конкурентности в Golang: Context
golang-blog.blogspot.com

На Go серверах каждый входящий запрос обрабатывается в своей собственной goroutine. Обработчики запросов часто запускают дополнительные goroutine для доступа к бэкэндам, таким как базы данных и службы RPC. Множеству goroutine, работающих над запросом, обычно требуется доступ к специфическим для запроса значениям, таким как личность (identity) конечного пользователя, токены авторизации и крайний срок запроса (request's deadline). Когда запрос отменяется или истекает время ожидания, все goroutine, работающие над этим запросом, должны быстро завершиться, чтобы система могла вернуть любые ресурсы, которые они используют.

В Google разработали пакет context, который позволяет легко передавать значения в области видимости запроса, сигналы отмены и крайние сроки (deadlines) через границы API всем goroutine, участвующим в обработке запроса. Пакет общедоступен как context
Файл не выбран