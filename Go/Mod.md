https://github.com/golang/go/wiki/Modules#daily-workflow

# Glossary

- *Main module* –  модуль, в котором `go` *command* вызывается. *Main module* определяется `go.mod` файлом в текущем или родительском каталоге.

*Dependency* модуля может быть двух видов

- *Direct* - это *dependency*, для которой *module* делает `import` напрямую.
- *Indirect* – это *dependency*, для которой `import` выполняется другими, *direct dependency*'s. 

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

- не задан или `GO111MODULE = auto` – включен ли *module-aware mode*, зависит от наличия файла `go.mod`  в текущем каталоге и родительских каталогах, как описано выше. 
- `GO111MODULE=on` – включен *module-aware mode*, `GOPATH` игнорируется
- `GO111MODULE=off` – включен *GOPATH mode*, возможности *Go modules* не используются. Зависимости ищет в каталоге `vendor` и `GOPATH`.

При *module-aware mode*:

- значение `GOPATH` *environment variable* не используется при поиске *dependency*'s
- загруженные *dependency*'s продолжают сохраняться в `GOPATH/pkg/mod`
- *command*'ы (*executable*'s) устанавливаются в `GOPATH/bin` (если `GOBIN` не установлена).

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

В одном каталоге может находиться только один *package*. Однако есть исключение для unit test'ов, которые должны находиться в файлах `XXX_test.go` и пакете `package XXX_test` 

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



# Файл `go.mod`

## `require` *directive*

`require` *directive* объявляет минимально необходимую версию для данного *module-dependency*. 

```
RequireDirective = "require" ( RequireSpec | "(" newline { RequireSpec } ")" newline ) .
RequireSpec = ModulePath Version newline .
```

### `indirect` комментарий

Определение *direct* и *indirect dependency* ([link](#glossary)).

`go` *command* автоматически добавляет комментарий `//indirect` к некоторым *dependency*:

- *Dependency*, для которой нет `import`  ни в одном из исходных файлов *main module*. И также для этих *dependency*'s нет `import` на нижележащих уровнях *dependency*'s.
- *Dependency*, для которой нет `import`  ни в одном из исходных файлов *main module*. Но для этих *dependency* ЕСТЬ (!) `import` на нижележащих уровнях *dependency*'s. Такой `require` может появиться, если *module-dependency* первого уровня импортирует *module-dependency* второго уровня, но не указывает это  в своем  `go.mod` файле, или вообще не имеет своего `go.mod` файла, поэтому эта *dependency* должна быть поднята на уровень выше.
- если указанная (в команде `go get`) версия *module* выше, чем та, которая требуется (транзитивно) другими *main module's dependencies* (зависимостями основного модуля). Т.е. (? точно indirect) *indirect dependency*  требуется в более высокой версии, чем подразумевается графом модулей. Обычно это происходит после явного *upgrade* для некоторых *direct* или *indirect dependency*'s через `go get -u ./...` .



Пример:

```go
require golang.org/x/net v1.2.3

require (
    golang.org/x/crypto v1.4.5 // indirect
    golang.org/x/text v1.6.7
)
```





# Module-aware commands

## `go get`

*Update* *dependency*'s в `go.mod` *file* для *main module*, а затем *build* и *install* их (соответственно install executable module в папку `GOPATH/bin`, если `GOBIN` не установлена [link](#поддержка-go-modules))

```bash
go get [-d] [-t] [-u] [-v] [-insecure] [build flags] [packages]
```

Флаги:

- `-u `– сделать *upgrade* для *module*'s, импортируемых *directly* или *indirectly by packages*, указанными в команде. *Upgrade* будет сделан к последней версии.





https://github.com/golang/go/wiki/Modules#daily-workflow

### Как сделать *upgrade* и *downgrade* для *dependency*

Чтобы сделать *upgrade* для *dependency* к ее *latest version*:

```bash
go get example.com/package
```

Чтобы сделать *upgrade* для *dependency* и всех ее *dependency*'s к *latest version*:

```bash
go get -u example.com/package
```



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



## `go mod tidy`

```bash
go mod tidy [-e] [-v]
```

`go mod tidy` проверяет , что `go.mod` файл соответствует исходному коду в *module*. Он добавляет отсутствующие `require`, необходимые для сборки *package*'s и *dependency*'s текущего *module*, и удаляет `require` для тех *module*'s, которые не требуются ни для каких *package*'s. Команда также добавляет любые недостающие записи в `go.sum` и удаляет ненужные записи.

Флаги:

- `-e` – продолжать , несмотря на возникающие ошибки во время загрузки *package*'s.

- `-v` – печатать информацию об удаленных *module*'s на *standard error*.

`go mod tidy` рекурсивно загружает все *package*'s в main module ([link](#glossary)) и все *package*'s, которые импортируют *package*'s первого уровня и т.д.. 

После выполнения `go mod tidy`, каждый *module*, который предоставляет (*package*'s которого используются прямо или косвенно) один или несколько *package*'s:

- (1, прямо) имеет соответствующую ему `require` директиву в `go.mod` файле для *main module*
- (2, косвенно) имеет соответствующую ему `require` директиву внутри другого module более высокого уровня, который также включается через `require`. 

 `go mod tidy` также указывает номер последней версии для каждого недостающего *module* (см [запросов Версия](https://golang.org/ref/mod#version-queries) для определения `latest` версии).

 `go mod tidy` удаляет `require` *directive*'s для *module*'s, package's которых не используются прямо или косвенно.

`go mod tidy `также добавляет или удаляет `// indirect `комментарии к `require` *directive*'s ([link](#indirect-комментарий)).










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