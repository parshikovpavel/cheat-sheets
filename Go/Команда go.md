# Параметр `packages`

Многие *comman*d'ы `go`  применяются к `packages` – списку *import path*'s

```go
go <action> [packages]
```

Виды *import path*:

- *Relative import path* ([link](#relative-import-path)) – *Import path*, который является корневым путем (наверное, вроде `/Users/...`) или начинаются с `./` или `../` () – интерпретируется как путь в файловой системе и обозначает *package* в этом каталоге.
- *Non-relative import path* – *import path* `path` обозначает *package*, найденный в каталоге `DIR/src/path`, где `DIR`–  одна из папок, перечисленных в `GOPATH` (это т.е. по сути когда )
- если *import path* не указан, `go <action>` применяется к *package* в текущем каталоге. Т.е. по умолчанию значение параметра `packages` – `.`

## *Pattern*

*Import path* называется *pattern*, если он включает один или несколько подстановочных знаков `...`, каждый из которых может соответствовать любой строке, включая пустую строку и строки, содержащие `/`. 

В зависимости от видов *import path*:

- Для *non-relative import path*. *Pattern* распространяется на все *package directories* внутри `GOPATH`, соответствующие *pattern*'у.

  Например, *pattern* `...` – соответствует всем *package directories* внутри `GOPATH`

- Для *relative import path*. *Pattern* распространяется на *package* в этом каталоге.

Две особенности применения *wildcard* `...`:

- в конце строки `/...` – также соответствует пустой строке. Например, `net/...` соответствовает `net` и *package*'s во всех подкаталогах `net`. 
- *wildcard* `...`  не сопоставляется с каталогом `vendor`. Так что `./...` не соответствует *package*'s в подкаталогах `./vendor` или `./mycode/vendor`. 

## Relative import path

*Import path*, который начинается с `./` и `../` , – называется *relative import path*. 

*Relative import path* поддерживаются в двух случаях:

- В *сommand* `go` вместо `packages` . 

  Примеры применения:

  - Находишься по *import path* `unicode`, хочешь протестировать по *import path* `unicode/utf8`:

    ```bash
    go test ./utf8
    ```

  - Находишься по *import path* `unicode/utf8`, хочешь протестировать по *import path* `unicode`:

    ```bash
    go test ..
    ```

  - Протестировать все нижележащие *import path*'s (поддиректории):

    ```go
    go test ./...
    ```

- ...



# `go env`

```bash
go env [-json] [-u] [-w] [var ...]
```

`env` выводит информацию о *Go environment*.

По умолчанию `env` выводит информацию в формате *shell script*. Если в качестве аргументов указано одно или несколько имен *variable*'s, `env` печатает значение каждой указанной `variable` в отдельной строке.

Флаг `-json` выводит *environment* в формате JSON, а не в формате *shell script*.

Флаг `-u` требует один или несколько аргументов и делает *unset* настроек по умолчанию для указанных *environment variable*'s, если они были установлены с помощью `go env -w`.

Флаг `-w` требует один или несколько аргументов в форме `NAME=VALUE` и изменяет настройки по умолчанию для указанных *environment variable*'s на заданные *value*'s.











# `go generate`

Команда `go generate` позволяет автоматизировать генерацию исходного кода перед компиляцией, используя инструменты самого Go. `go generate` позволяет избежать использования *Makefile* и *Bash script*. 

```bash
go generate [-n] [-v] [-x] [file.go... | packages]
```

Флаги:

- `-v` выводит имена пакетов и файлов по мере их обработки. 
- `-n` выводит команды, которые будут выполнены. 
- `-x` выводит команды по мере их выполнения.

`go generate` никогда не запускается автоматически при запуске команд `go build`, `go get`, `go test` и т.д. `go generate` нужно запускать явно. 

`go generate` сканирует файл на наличие директив вида:

```go
//go:generate command argument...
```

Эти директивы могут находиться в любом месте `.go` файла. Команду `//go:generate mockgen ...`  чаще всего размещают на первой строке или после строки с `package`.

`go generate` запускает эти команды, описанные директивой `//go:generate` внутри файлов. Без аргументов `go generate` сканирует файлы, относящиеся к текущему *package*.

Внутри `//go:generate` можно использовать любые команды, но изначально предполагается, что там размещаются команды для создания или обновления исходных файлов на Go.

Например `go generate`, для которой в качестве команды задано `echo`:

```go
package project

//go:generate echo Hello, Go Generate!

func Add(x, y int) int {
	return x + y
}
```

```bash
$ go generate
Hello, Go Generate!
```

Не должно быть пробелов:

- перед комментарием `//go:generate...`
- внутри `//go`

Параметры:

- `command` –  исполняемый файл, который может запускаться локально.
- `argument` – аргументы команды, разделенные пробелами. В качестве аргументов можно использовать строки в двойных кавычках, каждая строка – отдельных аргумент. Строки в кавычках вычисляются и используют синтаксис Go.

Чтобы сообщить людям и другим программам, что код выходного файла сгенерирован, сгенерированный файл должен включать строку, которая соответствует следующему регулярному выражению:

```go
^// Code generated .* DO NOT EDIT\.$
```

Эта строка должна располагаться перед любым не-комментарием или не-пробельным текстом в файле.

`go generate` устанавливает несколько *variable*'s, когда запускает команду:

- `$GOFILE` – имя файла

- `$GOPACKAGE` – имя *package* для файла, содержащего директиву. Можно использовать как часть имени *package*:

  ```go
  //go:generate mockgen -package ${GOPACKAGE}_test ...
  ```

  

Также в строку подставляются значения *environment variable*'s, определенных в ОС, – `$HOME`, `$NAME`, ...





# Форматирование (`gofmt`)

https://blog.golang.org/gofmt

https://pkg.go.dev/cmd/gofmt

https://pkg.go.dev/cmd/go#hdr-Gofmt__reformat__package_sources

Можно для форматирования использовать две команды:

-  `gofmt` – работает на *source file level*
-  `go fmt` – работает на *package level*

```
gofmt [flags] [path ...]
```

По умолчанию `gofmt` выводит отформатированные исходные файлы на *standard output*. Без аргументов команду можно использовать, чтобы узнать, как правильно форматировать код (т.к. исходный код не перезаписывается). 

- `path`:
  - Если не указан, то обрабатывается *standard input*
  - Если указан файл, то обрабрабатывает этот файл. 
  - Если указан каталог, то рекурсивно обрабатывает все файлы `.go` в этом каталоге (файлы, начинающиеся с точки `.`, игнорируются.) 

- `flags`:
  - `-s` – попытаться упростить код
  - `-w` – не выводить отформатированный код на *standard output*. Если найдены ошибки в форматировании, то файл перезаписывается с правильным форматом. 



<u>Пример:</u>

Отформатировать и упростить код, перезаписать файлы в правильном формате

```
gofmt -s -w [path]
```





## Правила форматирования

- Для отступов используется *tab*. 
- Для выравнивания используются пробелы.
- Количество символов в строке не ограничено. При желании можно разбить строку на несколько строк. 



# `go list`

```bash
go list [-f format] [-json] [-m] [list flags] [build flags] [packages]
```

`go list` выводит список *package*'s, которые перечислены в параметре `packages` (это не обязательно название *package*, [link](#параметр-packages)), один на строке. 

Примеры использования:

- вывести список *package*'s в нижележащих *import path*'s (поддиректориях):

  ```bash
  $ go list ./...
  github.com/parshikovpa/service/internal/ab
  github.com/parshikovpa/service/internal/api/handler/vertical_form
  github.com/parshikovpa/service/internal/clients/autosearch
  github.com/parshikovpa/service/internal/clients/breadcrumbs
  ```

  Используется relative import path ([link](#параметр-packages)).

- вывести список используемых (мое???) *package*'s стандартной библиотеки + внешних *package*'s.

  ```bash
  $ go list ...
  archive/tar
  archive/zip
  bufio
  ```

  





# `go imports`

```bash
goimports -w models.go
```







# Testing flag'и

[Здесь смотреть](Testing.md#testing-flagи)

