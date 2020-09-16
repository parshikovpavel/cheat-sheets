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



## Сборка программы

При сборке программы можно указать *import path* пакета. В этом случае команду можно выдать из любой *directory*. *Package* будет автоматически найден внутри *workspace*.

```
$ go install github.com/user/hello
```

Если текущей является *package path*, то можно запускать без указания *package path*

```
$ cd $GOPATH/src/github.com/user/hello
$ go install
```



### `go build`

Сборка *package* по его *import path*, вместе с его dependency's, но без процесса *install* (размещения исполняемого файла в `bin`).

```bash
go build [-o output] [-i] [build flags] [packages]
```



Поведение:

- при компиляции `package main` – записывает полученный исполняемый файл в выходной файл с именем первого исходного файла (`go build ed.go rx.go` записывает в файл `ed`) или в файл с именем каталога исхожного кода (`go build unix/sam` пишет в файл `sam`)

- При компиляции нескольких *package*'s или одного не `package main` – компилирует пакеты, но отбрасывает получившийся файл, и може использоваться только для проверки возможности сборки *package*.

Флаги:

-  `-o` заставляет `build` писать результирующий исполняемый файл или объект в указанный *output file* или *directory* вместо поведения по умолчанию, описанного ранее. 

### `go install`

*Compile* и *install* программу. При этом программа помещается в каталог:

- по пути `GOBIN` *environment variable*
- если `GOBIN` не установлена, то по пути `$GOPATH/bin`. Если в `GOPATH` вписано несколько каталогов, то будет найден соответствующий каталог в `GOPATH` и программа будет сохранена в соответствующем каталоге `bin`.

Таким образом, различные скомпилированные программы помещаются в один и тот же каталог. Можно включить путь к этому каталогу в `PATH`:

 ```bash
export PATH=$PATH:$GOPATH/bin
 ```

### `go run`

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

# Типизация

Язык Go использует строгую типизацию. Это не позволяет, например, сложить значения типов `int32` и `int16` без явного их преобразования.

# Declarations и scope

## Predeclared identifiers

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

### `nil`

`nil` – *Predeclared identifiers* (!!!)

*Identifier* `nil` используется как значение для пустых *pointer*'ов и пустых значений.

`nil` может использоваться в операциях сравнения и присваивания. У `nil` нет методов.

`nil` играет почти ту же роль, что и `null` в PHP и Java.



## Объявление *variable*

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



## Объявление `type`

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





## Объявление *function*

Смотреть [function type и `Signature`](#function-тип)

Объявление *function* связывает *function name* с ее *signature* и *body*.

```
FunctionDecl = "func" FunctionName Signature [ FunctionBody ] .
FunctionName = identifier .
FunctionBody = Block .
```



```go
func min(x int, y int) int {
	if x < y {
		return x
	}
	return y
}
```

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

Смотреть [function type и `Signature`](#function-тип).

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



# Literal и constant

Literal (литерал) – это представление *constant*'ы (константы). 

*Constant*'e можно явно присвоить тип при объявлении *constant*, объявлении *variable*, как *operand* в *expression* и т.д. Если *constant*'а не может быть представлена с помощью указанного типа, то выбрасывается ошибка.

У *constant*'ы есть тип *by default*, в который *constant*'а неявно преобразуется, когда требуется значение с явно определенным типом, например *short variable declaration*  `i := 0`. 

Типы *constant* *by default*:

- *boolean* – `bool`
- *rune* – `rune`
- *integer* – `int`
- *floating-point* – `float64`
- *complex* – `complex128`
- *string* – `string`

### Rune literal

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





# Type's

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

## `struct`

`struct` (структура) – составной тип, последовательность именованных элементов, называемых *field* (поле), у каждого из которых есть *name* и *type*.

```go
// An empty struct.
struct {}

// A struct with 6 fields.
struct {
	x, y int
	u float32
	_ float32  // padding
	A *[]int
	F func()
}
```

## Коллекции

### `array`

`array` – это нумерованная последовательность элементов одного *type*. 

```
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

Длина является частью *type* массива. Поэтому в качества параметра функции может подставляться только массив определенного размера. Поэтому массивы практически никогда не используются напрямую. А используются через `slice`.

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

## `interface`

`interface`  – описывает некоторый *method set*.

`interface` является абстрактными типом и не позволяет создавать его экземпляры.

Для того чтобы конкретный `type` удовлетворял *interface*'у, достаточно (!!!) чтобы он имел реализацию *method*'ов, определяемых *interfacе*'ом. Т.е. не требуется формально определять связь между `interface` и конкретным `type`. `type` будет автоматически удовлетворять всем `interface`'s, методы которых он реализует требованиям
нескольких интерфейсов, реализуя методы всех этих интерфейсов.

Реализующий `interface` конкретный `type` может подставляться в те места кода, где требуется этот `interface`. 

В определении *interface*'а

```
InterfaceType      = "interface" "{" { ( MethodSpec | InterfaceTypeName ) ";" } "}" .
MethodSpec         = MethodName Signature .
MethodName         = identifier .
InterfaceTypeName  = TypeName .
```

 могут:

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

### Empty interface

Все `type`'s реализуют *empty interface* (пустой интерфейс), т.к. он не требует от `type` реализации ни каких методов:

```go
interface{}
```

*Empty interface* может использоваться для ссылки на значение любого `type` (подобно `void*` в C++)

Таким образом, объявление *slice*, который может хранит элементы любых `type`:

```go
type Stack []interface{}
```

## *Function* тип

Смотреть также [объявление function](#объявление-function)

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

- либо все указаны (отдельно в списке *parameter*'ов и отдельно в списке *parameter*'ов и *result*'ов)
- либо ни одно из имен не указано (отдельно в списке *parameter*'ов и отдельно в списке *parameter*'ов и *result*'ов):

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

TODO!!! https://golang.org/ref/spec#Operators

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



# Statement

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

При разделении операторов:

- можно использовать точку с запятой `;`. Обязательно, если в одной строке располагается несколько инструкций

  ```go
  a := 1;
  ```

- можно не использовать точку с запятой `;`. Тогда они автоматически будут добавлена компилятором.

  ```go
  a := 1
  ```

## `if`

Условное выражение в инструкции `if` можно не заключать в круглые скобки:

```go
if a > 1 { 
	// ...    
}
```



## `for`

Существует три формы цикла `for`:

- с одним условием (в других языках используется `while`). Цикл выполняется до тех пор, пока условие – `true`. 

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

  

- c `range` *clause*. Итерирует все элементы *array*, *slice*, *string* или *map* или значений, полученных по *channel*. Для каждого элемента присваивает переменным (может быть несколько переменных в `IdentifierList`) некоторые значения, зависящие от типа `Expression` (см. таблицу ниже), а затем выполняет блок.

  ```go
  RangeClause = [ ExpressionList "=" | IdentifierList ":=" ] "range" Expression .
  for a := range slice {
      // ...
  }
  ```

  Значения *value*'s зависящие от типа `Expression` следующим образом:

  ```
  Range expression                          1st value          2nd value
  
  array or slice  a  [n]E, *[n]E, or []E    index    i  int    a[i]       E
  string          s  string type            index    i  int    see below  rune
  map             m  map[K]V                key      k  K      m[k]       V
  channel         c  chan E, <-chan E       element  e  E
  ```

  

- с *for clause*. Стандартная форма для C++ и PHP:

  ```go
  ForClause = [ InitStmt ] ";" [ Условие ] ";" [ PostStmt ].
  for i := 0; i < 10; i++ {
  	// ...
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

## *Index expressions*

Получение значения по индексу для `array`, `*array`, `slice`, `string` и `map`:

```go
a[x]
```

Для `string` , `a[x]` – значение байта по индексу `x`, тип `a[x]` – `byte`

Для `map` : 

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

- так и после ее использования:

  ```go
  func main() {
      fmt.Print(a)
  }
  
  var a = 1
  ```

  



# Garbage collector

 Garbage collector (сборщик мусора) съедает в момент работы 1мс.



# Go modules

Основные принципы:

- Программы размещаются внутри *package*. 

- *Package* – набор из *source* файлов, размещенных в одном каталоге, которые компилируются вместе. `func`, `type`, `var` и `constant`, определенные в любом файле некоторого *package* видны внутри всех других файлов этого же *package*.

- *Repository*, как правило, содержит только один *module*, расположенный в корне репозитория (но может и несколько *module*'s). 

- *Module* – набор связанных *package*'s, которые *released* (релизятся) вместе. *Module* содержит *package*'s в каталоге, содержащем файл `go.mod`, а также подкаталогах этого каталога, вплоть до следующего подкаталога, содержащего другой файл `go.mod` (если есть).

- *Module path* (путь к модулю) – префикс для *import path* для всех *package*'s внутри *module*, объявляется в файле `go.mod`. 

  ```
  module example.com/service
  ```

  *Module path* позволяет указать, где `go` *command* будет искать его его для загрузки. Например, чтобы *module* `golang.org/x/tools` будет извлекаться из *repository* `https://golang.org/x/tools`.

- Даже если *module* изначально не опубликован в *repository* (*github*), желательно его сразу структурировать так, как будто он будет опубликован в *repository* (*github*).

- Путь к каждому модулю не только служит префиксом пути импорта для его пакетов, но также указывает, где `go`команда должна искать его для его загрузки. Например, чтобы загрузить модуль `golang.org/x/tools`, `go`команда обращается к репозиторию, указанному `https://golang.org/x/tools`(подробнее описано [здесь](https://golang.org/cmd/go/#hdr-Relative_import_paths) ).

## Import path

*Import path* (путь импорта) – строка, используемая для импорта *package* (не файла, а *package*). *Import path* = *module path* + подкаталог внутри модуля.

Если в *module* с *module path* `github.com/google/go-cmp` находится *package* в каталоге `cmp/`, то *import path* этого *package* - `github.com/google/go-cmp/cmp`. 

*Package*'s из *standard library* имеют короткие *import path* (нет *module path*), такие как `"fmt"` и `"net/http"`.

Принято, что если код хранится в некотором удаленном *repository*, то необходимо использовать корень этого удаленного *repository* в качестве *base path*. 

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

ТУТ!!!!



## `go get`

# Структура файла с кодом

Общая структура любого файла с кодом:

```go
SourceFile = 
		PackageClause ";"     // package ..., определяет package, к которому принадлежит файл
		{ ImportDecl ";" }    // import ..., определяется package's, которые будут использованы
		{ TopLevelDecl ";" } . // основное содержимое, определение var, func, constant, type
```





## `package`

Любой фрагмент программного кода должен быть включен в *package* (пакет).

Объявление `package` указывает, к какому *package* принадлежит файл. Все файла с одним и тем же `PackageName` составляют *package*. `PackageName` не содержит `/`, он содержит только последнюю часть *import path*.

Первый оператор в исходном файле Go должен быть:

```go
package <name>
package stack
```

```
PackageClause  = "package" PackageName .
PackageName    = identifier .
```



Каждая программа должна иметь `main` *package* с функцией `main()` , которая является точкой входа в программу:

```go
package main

func main() {
	// ...    
}
```

Функция `main()` всегда не имеет аргументов и ничего не возвращает. Когда функция `main.main()` завершается, одновременно с ней
завершается выполнение программы, и она возвращает операционной системе значение 0.

Также можно использовать функцию `init()`, которая выполняется перед функцией `main()`.

Язык Go оперирует в терминах *package*'s, а не файлов. То есть *package* можно разбить на любое количество файлов, и если все они будут иметь одинаковое объявление `package`, то все они будут являться частями одного и того же *package*, как если бы все их содержимое находилось в единственном файле.

## `import`

`import` позволяет импортировать *package* и использовать *exported identifier*'s из этого *package*.

```
ImportDecl       = "import" ( ImportSpec | "(" { ImportSpec ";" } ")" ) .
ImportSpec       = [ "." | PackageName ] ImportPath .
ImportPath       = string_lit .
```

`PackageName` в файле может использоваться в *qualified identifier*'s для обращения к *exported identifier*'s из импортированного *package*. Если `PackageName` опущен, то используется `PackageName` из импортированного *package*.



ТУТ!!!

Импортирование *package*: 

```go
import (
    "fmt"
    "os"
    "strings"
    "github.com/user/stringutil"
)
```

Здесь указывается *package import path* (не файла, а *package*).

Импортируемые пакеты можно не отделять друг от друга запятыми.

## Использование package

Обращение к `type`, `func`, `var` (переменным) и другим элементам *package* записывается в виде `<package>.<элемент>`, где `<package>` – это последний (или единственный) компонент в имени *package*. Например, обращение к функции `Reverse()` в *package* `github.com/user/stringutil` записывается так `stringutil.Reverse()` 



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

# Go modules

https://golang.org/cmd/go/#hdr-GOPATH_and_Modules





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

Map:        Если size не указан - выделяется пустой map небольшого размера
            Если size указан - выделяется пустой map для хранения указанного количества элементов

Channel:    Если size не указан - channel небуферизованный
            Если size указан, то buffer capacity = size
```

### `error`

`error` *type* – встроенный *interface* тип для представления состояния ошибки, значение `nil` – отсутствие ошибки.

```go
type error interface {
    Error() string
}
```



## `fmt`

`fmt` – *formatted* (форматированный). 

*Package* реализует форматированный I/O по аналогии с функциями `printf()` и `scanf()` в Си. 

Некоторые функции принимают `format`, в котором можно использовать спецификаторы формата `%`, напоминающие спецификаторы функций `printf()` и `scanf()` в Си.



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



## `strings`

Содержит функции для работы со строками.





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

## `error`

### `New()`

```go
func New(text string) error
```

Возвращает ошибку с текстом `text`.

```go
func ... (int, error) {
    if ... {
        return nil, errors.New("...")
    }
    ...
    return x, nil
}
```






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

kjernmvck454#&
Павел
Павел 19:25

2 встречи онбординг
Павел
Павел 19:56

Начал заполнять онбординг чеклист

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

Backlog
Состоит из задач:

которые готовы для груминга (требования описаны в пункте DoR for Grooming)
которые готовы для разработки (отмечены зеленой галочкой в заголовке задачи, требования зафиксированы в пункте DoR for Development)
заглушек, которые грубо описывают, что потребуется сделать в будущем
баги
Для удобной разбивки бэклога заведены два блока (фактически на базе спринтов сделаны) — "Продуктовые задачи" и "Технические задачи".Необходимо новые задачи добавлять сразу в эти блоки

Груминг
Мы берем в спринт только прогрумленные задачи, которые отмечены зеленой галочкой .
Задача считается прогрумленной, если удовлетворяет требованиям из пункта DoR for Development. Прогрумленная задача помечается зеленой галочкой в заголовке задачи
В рамках спринта работа с бэклогом выглядит следующим образом:

Product Owner, Team Lead или инженер готовит PBI, который планирует реализовывать, согласно требованиям из DoR for груминг. Фактически получается так, что PO готовит продуктовые задачи, TL и инженеры - технические задачи
Задачи презентуются овнером этой задачи на 1-м груминге спринте. Owner знакомит команду с задачей, что требуется сделать. Команда задает любые вопросы, подсвечивает риски. Все вопросы фиксируются в разделе "Вопросы" самой задачи. У каждого вопроса назначется ответственный, который будет его закрывать
К следующему грумингу ответственный решает этот вопрос и фиксирует ответ рядом с вопросом. Если требуется, то какие-то моменты можно и нужно добавлять в описание самой задачи. Если больше открытых вопросов не остается, то задача помечается зеленой галочкой в заголовке задачи таким образом Если вопросы остаются, то задача уходить на следующий круг по такому же сценарию
P.S. Описан идеальный процесс, возможно, когда задача приносится не на 1-й груминг спринта, а позже. Алгоритм работы такой же.

ВАЖНО

Есть кейсы, когда договоренность может быть нарушена. Фактически задача может быть взята в спринт с открытыми вопросами, которые будут закрываться в момент реализации. Такая же задача может взята в работу в рамках текущего спринта. Команда учитывает риски, когда берет такую задачу в спринт. Случаи, в которых договоренность может быть нарушена, следующие: репутационные, юридические, финансовые риски для компании.

Definition of Ready for Grooming
В качестве списка Definition of Ready for Grooming необходимо использовать следующий набор критериев:

№ Критерии готовности задачи к груммингу
Кратко Подробно
1
Продуктовая проработка

Заполнены поля "Текущая ситуация", "Что надо сделать", "Критерии приемки" в Jira.
2
Дизайн

Для задач с дизайном готов концепт дизайна, значительных изменений в нём не будет. Для сложных состояний должен быть предоставлен прототип либо анимация.
3 Зависимости Определено наличие внешних зависимостей.
4 Логирование Определен объём необходимого логирования.

Quality gate 1:

Independent – завершённость, выполнимость, независимость – история должна быть абсолютно завершенной, осуществимой на практике, независимой от разных обстоятельств
Negotiable – открытость к ее обсуждению – история до своего завершения должна быть быть открыта для общего обсуждения, любой участник группы должен иметь возможность внести в нее свои поправки
Valuable – ценность – история должна приносить реальную пользу и увеличивать ценность проекта для заказчиков, пользователей и любых заинтересованных лиц
Estimable – оценочность – история должна быть удобной для оценки объёма работ
Small – лаконичность – история должна быть краткой, компактно изложенной и конкретной, чтобы можно было просто и быстро планировать работы. Если история получается слишком расплывчатой и абстрактной, перепишите ее, а лучше разбейте на меньшие фрагменты
Testable – тестируемость – история должна пройти проверку на практике, чтобы считаться завершенной. Составьте заранее список критериев, которым должна соответствовать законченная история.

Definition of Ready for Development

Считаем что задача была успешна прогруммлена и может быть взята на реализацию в спринт, если она соответствует нижеописанным требованиям:

User story разбита на Product

Backlog Item-ы (PBI)
Каждый PBI по времени должен быть выполним, учитывая последовательные работы по разработке и тестированию
PBI должен быть с заполненным актуальным описанием, иметь исчерпывающие (необходимые для оценки) продуктовые требования, заведен в Jira в пространстве BX.
Имеется дизайн; возможно прототип приложения/фичи
Спроектировано API (определены методы и входные/выходные параметры)
Нет технических блокеров для реализации
Имеются необходимые согласования с внешними заинтересованными лицами
Задача получила оценку в стори-поинтах от команды разработки
QG1 пройдены
Безопасность

Выявлены и указаны потенциальные риски безопасности. Если требуется, необходимо обратиться к коллегам из Security для однозначной оценки риска

Performance

Оценены риски просадки производительности (серверной или клиентской).
Если есть техническая возможность, то произведены замеры в синтетических условиях для оценки влияния (для клиентских приложений сейчас есть такая возможность).
Если вводится в эксплуатацию новый сервис, то предварительно в обязательном порядке требуется проведение нагрузочного тестирования. См. алгоритм Нагрузочное тестирование
Если A/B, то в конфигурацию теста добавить preset perf-метрик, если существует риск просадки перформанса

Definition of Done
В качестве списка предлагается использовать следующий набор критериев:

№ Критерии готовности
Кратко Подробно
1 Разработка Завершена разработка всех технических тасков (важно, что учитывается раскатка и удаление (если эксперимент не успешен) тестовой группы).
2 Тесткейсы Все тесткейсы написаны и вмёржены в мастер(звезда).
3 Design Review Все вопросы касательно дизайна решены, пройдено дизайн ревью.
4 Тестирование Завершены все итерации тестирования, P0, P1 и P2 баги исправлены, не блокирующие P3 и P4 баги чинятся в след спринте или закрываются со статусом "Won't fix", определяется через Product Owner.
5
Логирование

Данные по заранее обговорённым события записываются в хранилища (clickstream, mongo и т.д.).

6
Автотесты

Реализованный в задаче функционал покрыт автотестами(звезда).

7 Включен Feature-toggle Задача на включение фичатоггла находится в статусе "Waiting for release".
8 Код на бою Код находится на бою. Для случаев, в которых PBI не предусматривает вливания кода в бой, пункт звучит как: "Код в мастере"/"Код в develop".
9 Performance
Если по результатам A/B выявлена просадка производительности, то необходимо:

Найти возможность оптимизировать
Если получен рост продуктовых метрик, то раскатывать после согласования с PM, TUL, PUL
Остальные случаи необходимости раскатки тестовой группы с просадкой перформанса также необходимо обсудить TL, PM, TUL, PUL
(звезда) Отдельные правила для A/B.
Если функциональность раскатывается под A/B, тесткейсы и автотетесты писать не нужно.
Функциональность покрывается тесткейсами и автотетестами при раскатке на 100%.

Как считаем Scope Drop

Задача "идёт" в Scope Drop в случае, если какая-то часть из декомпозиции в итоге не будет выполнена в спринте. Например, разработку закончили, но не успели протестировать. В таком случае задача считается не выполненной и весь её объем попадает в scope drop. Отдельно отмечу, что демо юнита проводится в начале последнего дня спринта, поэтому если команда понимает, что успеет доделать задачу до конца дня и смержить в master/develop, то тогда можно засчитать как сделанную авансом и показать на демо velocity с учетом аванса. Но здесь рекомендуется оценивать свои шансы ответственно и честно. Если в итоге окажется, что задачу так и не успели закончить, то в Agile Report необходимо занести актуальную информацию.
Павел
Павел 0:57

Swap задач в спринте
Описание процесса swap'а задач в спринте и как считать Scope Drop спринта.

Если свапаются равноценные по объему PBI, то всё прозрачно. Примеры:
В спринте был PBI на 10SP, его заменили на один с таким же объемом или на несколько, сумма которых равна 10SP.
Если свапаются не равноценные задачи. Примеры:
Случай №1 — вместо PBI на 10SP приходит один PBI или несколько на сумму больше 10SP.
Scope Drop считать от объема, который был запланирован на старте спринта.
Пример: в ходе спринте свапают PBI с объёмом 10SP на два PBI с объемом 5SP и 7SP, по окончанию спринта:
а) если PBI с объемом 7SP не был выполнен, а с объемом 5SP – выполнен, то Scope Drop = 10SP - 5SP(выполненный PBI) = 5SP
б) если PBI с объемом 5SP не был выполнен, а с объемом 7SP – выполнен. Scope Drop = 10SP - 7SP(выполненный PBI) = 3SP
в) соответственно, если выполнили оба PBI, то Scope Drop = 0
Случай №2 — вместо PBI на 10SP приходит один PBI или несколько на сумму меньше 10SP
Scope Drop считать от объема, который был запланирован на старте спринта
Пример: в ходе спринте свапают PBI с объёмом 10SP на два PBI с объемом 5SP и 3SP, по окончанию спринта:
а) если PBI с объемом 3SP не был выполнен, а с объемом 5SP – выполнен, то Scope Drop = 10SP - 5SP(выполненный PBI) = 5SP
б) если PBI с объемом 5SP не был выполнен, а с объемом 3SP – выполнен. Scope Drop = 10SP - 3SP(выполненный PBI) = 7SP
в) если оба PBI выполнены, то Scope Drop = 10SP - 3SP - 5SP = 2SP
Павел
Павел 1:34

Может спросить у камиля
Павел
Павел 8:30

Первый небольшой кормит в монолит
Павел
Павел 9:21

Конфигурирование php
29 августа
Павел
Павел 2:30

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