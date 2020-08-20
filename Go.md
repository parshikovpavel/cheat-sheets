# Конфигурирование

Go – компилирующий язык.

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
export PATH=$PATH:$GOROOT/bin           # 2. В переменную PATH включить путь $GOROOT/bin
```

### `GOPATH`

Чтобы обеспечить сборку локальных программ и пакетов с помощью инструмента `go`, необходимо:

- каталог `bin` с инструментами языка Go (`$GOROOT/bin`) должен находиться в `PATH`.
- в дереве каталогов должен существовать каталог `src` для хранения исходных текстов локальных программ и пакетов. Например, `goeg/src/hello`. 
- Путь к каталогу, вмещающему каталог `src`, должен быть включен в `GOPATH` *environment variable* .

```bash
export GOPATH=$HOME/repos/goeg
```

Если нужно хранить программы в нескольких каталогах, то в `GOPATH` *environment variable* можно вписать несколько путей к каталогам, разделенных двоеточием:

```bash
export GOPATH=$HOME/app/go:$HOME/goeg
```



## Сборка программы

### `go build`

Сборка программы и размещение исполняемого файла в каталоге с исходным кодом. По умолчанию, выполняемый файл получает имя каталога, в который он помещается (например, `hello`)

```bash
$ cd ~/repos/goeg
$ go build
```

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
  - *import path* (?)
  - путь в файловой системе
  - шаблон, соответствующий одному известному пакету (?). Например, , как в случае «иди, беги». или «запустите мой / cmd».

## Запуск программы

Запуск программы:

```bash
./hello
```

# Синтаксис

## Объявление переменных

Язык Go использует строгую типизацию.

Виды объявлений:

- Сокращенное объявление переменной. Одновременно объявляет и инициализирует переменные. 

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

## Операторы

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

### `if`

Условное выражение в инструкции `if` можно не заключать в круглые скобки:

```go
if a > 1 { 
	// ...    
}
```

### `++`, `--`

В отличие от C++, операторы `++` и `—` могут использоваться только как оператор, но не как выражение. 

```go
a++
```

Они могут применяться лишь в постфиксной форме. Это предотвращает появление проблем, связанных с неправильным порядком вычислений.

### `for`

Формы цикла `for`:

- c `range` *clause*. Итерирует все элементы *array*, *slice*, *string* или *map* или значений, полученных по *channel*. Для каждого элемента *value* 's соответствующим *variable*'s, а затем выполняет блок.

  ```go
  for a := range slice {
      // ...
  }
  ```

  

Цикл for в языке Go может иметь различные формы
циклы for ...range
, возвращающие индекс
каждого элемента в указанном срезе.
for row := 0; row < len(bigDigits[0]); row++ {





## Область видимости

Область видимости при использовании оператора сокращенного объявления переменной `:=`  – ограничена блоком. 

Например, здесь область видимости переменной ограничена телом инструкции `if`:

```go
if <condition> {
    variable := "value"
}
```





## Комментарии

Комментарии оформляются в стиле языка C++: 

```go
// Однострочные комментарии, заканчивающиеся в конце строки

/* Блочные комментарии, занимающие несколько
строк 
*/
```

## Package

### Определение package

Любой фрагмент программного кода должен быть включен в *package* (пакет).

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

### Import package

Импортирование package: 

```go
import (
    "fmt"
    "os"
    "strings"
)
```

Импортируемые пакеты можно не отделять друг от друга запятыми.





## Пользовательские типы данных

Имеется возможность определять пользовательские типы данных, опираясь на фундаментальные типы. А также определять функции для работы с ними.



## Функции

Функции и методы в языке Go определяются с помощью ключевого слова `func`. 

```go
func a() {
	//...
}
```



## Коллекции

Типы коллекций:

- *array* (массив)
- *slice* (срез)



### Slice

Определение одномерного *slice*:

```go
[]Тип        // без инициализации
[]Тип{1,2,3} // с инициализацией
```

```go
a := []string{"1", "2", "3"} // сокращенная форма
var b = []int{2, 3, 5}       // полная форма
```

Размер *slice* указывать не нужно, компилятор сам может подсчитать количество элементов.

Чтобы определить многомерную коллекцию, необходимо в качестве `Тип` указать коллекцию:

```go
[][]Тип
```

```go
var bigDigits = [][]string{
    {"1","2","3"},
    {"4","5","6"},
}
```







### Функции

#### `len()`

Возвращает длину среза:

```go
len(os.Args)
```

### Обращение к элементам

Аналогично другим языкам, получение n-го элемента среза:

```go
slice[n]
```

Получение среза, содержащего с `n`-го по последний элемент.

```go
slice[n:]
```





# Standard library

Фундаментальные типы данных в языке Go поддерживают привычные операторы (`+`, `-`,...), а стандартная библиотека Go добавляет дополнительные пакеты функций для работы с фундаментальными типами (например, `string`). 

При использовании в коде *package*, который логически включен в другой *package* (например, `path/filepath`) необходимо указывать только последний компонент их имени (например, `filepath`).

## `builtin`

Встроенные функции, которые являются частью языка.

### `len`

Возвращает длину `v`:

```go
func len(v Type) int
```

Результат:

```
Array: число элементов в v.
Pointer to array: число элементов в *v (даже если v = nil).
Slice, or map: число элементов в v; если v = nil, len(v) - 0.
String: число byte's в v.
Channel: the number of elements queued (unread) in the channel buffer;
         if v is nil, len(v) is zero.
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




Павел
Павел
вчера в 12:42
Строки в языке Go хранят символы в кодировке UTF-8, поэтому
теоретически символы могут быть представлены двумя и более бай-
тами.
При обращении к определенной позиции в строке по индексу
возвращается байт, хранящийся в этой позиции. (В языке Go тип
byte является синонимом типа uint8.) После извлечения байта из
аргумента командной строки из него вычитается значение байта,
соответствующего символу 0,

Для определения литералов символов в языке Go используются
апострофы (одиночные кавычки), при этом литералы символов ин-
терпретируются как целочисленные значения, совместимые со все-
ми целочисленными типами в языке Go. Строгое соблюдение типов
в языке Go не позволяет, например, сложить значения типов int32
и int16 без явного их преобразования, но числовые константы и
литералы обретают тип в зависимости от контекста их использо-
вания, поэтому в данном случае литерал ‘0’ интерпретируется как
значение типа byte.

(В инструкции if константы 0 и 9
интерпретируются как значения типа byte в соответствии с типом
переменной digit, но если бы переменная digit имела другой тип,
например int, константы интерпретировались бы как значения это-
го типа.) Строки в языке Go являются неизменяемыми значениями
(то есть они не могут изменяться),

оператор конкатенации +
, возвращающий новую строку, являющу-
юся результатом объединения строковых операндов слева и справа.
Павел
Павел 18:16

функция log.Fatal() с текстом сообщения об
ошибке. Эта функция выводит дату, время и сообщение в os.Stderr,
если явно не было указано другое место для вывода,
Павел
Павел 18:22

log.Fatalf()
, которая делает то же самое, но принимает специфика-
торы формата %.
Павел
Павел 18:42

порядок объявлений и опре-
делений в общем случае не имеет большого значения.
Павел
Павел 18:47

можно было бы объявить переменную
bigDigits как до, так и после функции main().
Павел
Павел 18:54

язык Go и поддерживает объектно-ориентированное програм-
мирование, в нем отсутствуют такие понятия, как классы и насле-
дование (отношения типа «является»). В языке Go поддерживается
возможность определения пользовательских типов и чрезвычайно
простой способ агрегации типов (отношения типа «имеет»).
сегодня
Павел
Павел 13:13

Go

* компилируемый язык
Павел
Павел 13:20

* имеет сборщик мусора

но он съедает в момент работы 1мс

* CSP-style, асинхронная модель, возможность в процессе выделять потоки и писать параллельные программы (аналог OpenMP)
Павел
Павел 19:26

* на выходе получаем готовый бинарник без зависимостей, которые надо было бы скачивать менеджером зависимостей


Павел
Павел 19:54

* указывается тип переменной, типы переменных не динамические, и поэтому нет оверхеда по памяти на хранение типа.
Павел
Павел 20:32

обеспечивается возможность полного отделения
типов данных от их поведения и поддерживается динамическая
(или так называемая утиная) типизация. Динамическая (утиная)
типизация – мощный механизм абстракций, позволяющий обраба-
тывать значения (например, передаваемые функциям) с помощью
предоставляемых ими методов, независимо от их фактических ти-
пов.

встроен-
ных фундаментальных типов, обозначаемых такими ключевыми
словами, как bool, int и string, или составных типов, обозначаемых
ключевым словом struct1.

именованные, так и неименованные поль-
зовательские типы
. Неименованные типы с одинаковой структурой
можно использовать взаимозаменяемо, однако они не могут иметь
методы. (Более подробно эта тема обсуждается в §6.4.) Любой име-
нованный пользовательский тип может иметь методы, и эти методы
составляют интерфейс типа. Именованные пользовательские типы,
даже с одинаковой структурой, не могут использоваться взаимо-
заменяемо. (Далее в этой книге под термином «пользовательские
типы» будут подразумеваться именованные пользовательские типы,
если явно не будет указано иное.)