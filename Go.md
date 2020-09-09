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

Существует два способа передачи *parameter*'ов:

- *by value* (`T`). Внутри *function* мы используем копию оригинального значения. Любые изменения переменной, произведенные внутри функции, никак не отразятся на оригинальном значении.
- *by pointer* (`*T`). Внутри *function* мы можем изменить оригинальное значение, на которое ссылается *pointer*.



Существуют следующие причины передачи параметра *by pointer*:

- в Go не используется техника *copy-on-write* (????) и поэтому значение большого типа, намного дешевле передать в
  параметре *pointer* на это значение, чем копировать само значение.
- если необходимо в *function* измененить оригинальное значение фактического параметра. 



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

В других языках *receiver* обычно имеет имя `this` или `self`. В языке Go тоже можно использовать такие имена, но считается, что это не соот-
ветствует стилю Go.

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

*Pointer* – это переменная, хранящая адрес в памяти, где находится другое значение. *Pointer*'ы в Go практически ничем не отличаются от *pointer*'ов в
языках C и C++, за исключением того, что Go не поддерживает арифметику с указателями. *Pointer* объявляется добавлением символа звездочки `*` перед *type* `*T`. 

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

# Expression

Expression (выражение) позволяет вычислить значение путем применения *operator*'s и *function*'s к *operand*'s.



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

## Slice expression

Получение среза, содержащего с `n`-го по последний элемент.

```go
slice[n:]
```





## Инициализация программы и исполнения

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



# Организация кода

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

## Import path

*Import path* (путь импорта) – строка, используемая для импорта *package* (не файла, а *package*). *Import path* соответствует местоположению *package* внутри *workspace* (в папке `src`) или в *remote repository* (удаленном репозитории).

*Package*'s из *standard library* имеют короткие *import path*, такие как `"fmt"` и `"net/http"`.

Для *package* можно выбрать любой произвольный *path*. Принято, что если код хранится в некотором удаленном *repository*, то необходимо использовать корень этого удаленного *repository* в качестве *base path*. 

Например, если код размещается в *repository*  `github.com/user`:

- путь к каталогу с *repository* – `$GOPATH/src/github.com/user`
- путь к каталогу с `hello` *package* – `$GOPATH/src/github.com/user/hello`.  *Package import path* – `github.com/user/hello`

## Определение package

Любой фрагмент программного кода должен быть включен в *package* (пакет).

Первый оператор в исходном файле Go должен быть:

```go
package <name>
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

## Import package

Импортирование *package*: 

```go
import (
    "fmt"
    "os"
    "strings",
    "github.com/user/stringutil"
)
```

Здесь указывается *package import path* (не файла, а *package*).

Импортируемые пакеты можно не отделять друг от друга запятыми.

## Использование package

Обращение к `type`, `func`, `var` (переменным) и другим элементам *package* записывается в виде `<package>.<элемент>`, где `<package>` – это последний (или единственный) компонент в имени *package*. Например, обращение к функции `Reverse()` в *package* `github.com/user/stringutil` записывается так `stringutil.Reverse()` 

## Общий алгоритм импорта и использования package

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





Диапазон индексов указывается в формате
первый:последний. Если первый индекс опущен, как в данном случае,
вместо него используется значение 0, а если опущен последний индекс,
вместо него используется значение вызова функции len() для среза.
сегодня
Павел
Павел 2:02

для тяжелых пользовательских типов обычно
намного эффективнее в качестве приемников передавать указате-
ли
, потому что операция передачи указателя (который обычно яв-
ляется простым 32- или 64-битным значением) намного дешевле,
чем передача объемного значения, даже в методах, не изменяющих
значения приемника.

если метод применяется к значению,
но требует передать ему указатель на значение, компилятор Go пой-
мет, что методу нужно передать адрес значения (то есть адресуемое
значение – см. §6.2.1), а не его копию. Соответственно, если метод
применяется к указателю на значение, но требует передать ему само
значение, компилятор Go поймет, что нужно разыменовать указа-
тель и передать методу значение, на которое он указывает1.

поэтому в языке Go отсутствует оператор ->, используемый в
языках C и C++.
Павел
Павел 12:09

Пакет bufio предоставляет функции буферизованного ввода/
вывода, включая функции чтения и записи строк из и в текстовые
файлы в кодировке UTF-8. Пакет io предоставляет низкоуровневые
функции ввода/вывода, а также интерфейсы io.Reader и io.Writer,

Пакет io/ioutil – высокоуров-
невые функции для работы с файлами. Пакет regexp дает поддержку

регулярных выражений.