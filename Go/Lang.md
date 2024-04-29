https://go101.org/article/concurrent-atomic-operation.html

https://go101.org/article/concurrent-synchronization-more.html



https://yourbasic.org/golang/time-reset-wait-stop-timeout-cancel-interval/

https://medium.com/@cep21/preemptive-interface-anti-pattern-in-go-54c18ac0668a

https://github.com/spy16/droplets/blob/master/docs/interfaces.md



Курс по Go: https://cf.avito.ru/pages/viewpage.action?pageId=74139827

https://medium.com/german-gorelkin

Супер блог:

- circuit breaker
- retry (можно запилить проект на гитхаб)





Правила написания кода на Go в репозитории Go

https://github.com/golang/go/wiki/CodeReviewComments



«[Network programming with Go](https://click.sender.yandex.ru/l/626484/633447/6/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://www.amazon.com/Network-Programming-Go-Adam-Woodbeck/dp/1718500882?utm_source=email&utm_medium=email&utm_campaign=sendr-626484)», Adam Woodbeck — ещё одна книга по Go. Особенно советуем раздел № 3 про HTTP-сервер.

![img](https://resize.yandex.net/mailservice?url=https%3A%2F%2Fcdn.letteros.com%2Fuploads%2F4699%2Fmf51n_mf25g_mf86q_i1.png&proxy=yes&key=d790a50f855d3c7bde9b371b60f8efb8)«[Cloud Native Go: Building Web Applications and Microservices for the Cloud with Go and React](https://click.sender.yandex.ru/l/626484/633447/7/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://www.amazon.com/Cloud-Native-Applications-Microservices-Developers/dp/0672337797?utm_source=email&utm_medium=email&utm_campaign=sendr-626484)», Dan Nemeth, Kevin Hoffman — хорошее руководство, как создавать веб-приложения на Go с использованием микросервисной архитектуры и облачных технологий.

![img](https://resize.yandex.net/mailservice?url=https%3A%2F%2Fcdn.letteros.com%2Fuploads%2F4699%2Fmf51n_mf25g_mf86q_i1.png&proxy=yes&key=d790a50f855d3c7bde9b371b60f8efb8)[Унифицируй это: как Lamoda делает единообразными свои Go сервисы](https://click.sender.yandex.ru/l/626484/633447/8/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://habr.com/ru/company/lamoda/blog/495344/?utm_source=email&utm_medium=email&utm_campaign=sendr-626484) — как в Lamoda справляются с разношерстностью около 40 микросервисов на Go.

![img](https://resize.yandex.net/mailservice?url=https%3A%2F%2Fcdn.letteros.com%2Fuploads%2F4699%2Fmf51n_mf25g_mf86q_i1.png&proxy=yes&key=d790a50f855d3c7bde9b371b60f8efb8)[Go Does Not Need a Java Style GC](https://click.sender.yandex.ru/l/626484/633447/9/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://itnext.io/go-does-not-need-a-java-style-gc-ac99b8d26c60?utm_source=email&utm_medium=email&utm_campaign=sendr-626484) — автор статьи сравнивает Garbage Collector в Go и Java и объясняет, почему в Go лучше.

![img](https://resize.yandex.net/mailservice?url=https%3A%2F%2Fcdn.letteros.com%2Fuploads%2F4699%2Fmf51n_mf25g_mf86q_i1.png&proxy=yes&key=d790a50f855d3c7bde9b371b60f8efb8)[Language Mechanics On Escape Analysis](https://click.sender.yandex.ru/l/626484/633447/10/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html?utm_source=email&utm_medium=email&utm_campaign=sendr-626484) — подробно про Escape-анализ в Go.

![img](https://resize.yandex.net/mailservice?url=https%3A%2F%2Fcdn.letteros.com%2Fuploads%2F4699%2Fmf51n_mf25g_mf86q_i1.png&proxy=yes&key=d790a50f855d3c7bde9b371b60f8efb8)[Под капотом Golang — как работают каналы](https://click.sender.yandex.ru/l/626484/633447/11/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://medium.com/@victor_nerd/под-капотом-golang-как-работают-каналы-часть-1-e1da9e3e104d?utm_source=email&utm_medium=email&utm_campaign=sendr-626484) — статья Игоря Угольникова про устройство каналов в Go.





[Just For Func](https://click.sender.yandex.ru/l/626484/633447/12/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://www.youtube.com/watch?v=H_4eRD8aegk&list=PL64wiCrrxh4Jisi7OcCJIUpguV_f5jGnZ&utm_source=email&utm_medium=email&utm_campaign=sendr-626484) — познавательные лекции по программированию.

![img](https://resize.yandex.net/mailservice?url=https%3A%2F%2Fcdn.letteros.com%2Fuploads%2F4699%2Fmf51n_mf25g_mf86q_i1.png&proxy=yes&key=d790a50f855d3c7bde9b371b60f8efb8)[Эффективное использование структур данных в Go](https://click.sender.yandex.ru/l/626484/633447/13/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://www.youtube.com/watch?v=kClQ7rM5uBY&utm_source=email&utm_medium=email&utm_campaign=sendr-626484) — доклад Александра Шарипова.



![img](https://resize.yandex.net/mailservice?url=https%3A%2F%2Fcdn.letteros.com%2Fuploads%2F4699%2Fmf51n_mf25g_mf86q_i1.png&proxy=yes&key=d790a50f855d3c7bde9b371b60f8efb8)[Standard Go Project Layout](https://click.sender.yandex.ru/l/626484/633447/14/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://github.com/golang-standards/project-layout/blob/master/README_ru.md?utm_source=email&utm_medium=email&utm_campaign=sendr-626484) — базовый макет организации проектов, разработанных на Golang.

![img](https://resize.yandex.net/mailservice?url=https%3A%2F%2Fcdn.letteros.com%2Fuploads%2F4699%2Fmf51n_mf25g_mf86q_i1.png&proxy=yes&key=d790a50f855d3c7bde9b371b60f8efb8)[Codewars](https://click.sender.yandex.ru/l/626484/633447/15/L/UEFNVFNRQnlGRmcrRGlRclN4VWdmRUorZm0wTFdXSlpWVWRBWG5KTmRXUkhRVlZaV0g5cVoyVmxWM1I2Wm1KVkFITUlYRXRjYlhvRgpkWFpnVVZ3TGFsQm1kU0l1Sm14SUlUeFFVQ0pSVHdGVGNYSlhmRUliRnlJaFhneHFUaGNhTFNkTGZsOEtBWFk9OjIwNzU6MA%3D%3D/*https://www.codewars.com/?language=go&utm_source=email&utm_medium=email&utm_campaign=sendr-626484) — сайт, который поможет «набить руку» при решении задачек на Go.

Graceful shutdown https://rudderstack.com/blog/implementing-graceful-shutdown-in-go/

http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/index.html#close_http_resp_body



# Общие особенности

* компилируемый язык
* имеет сборщик мусора
* CSP-style, асинхронная модель, возможность в процессе выделять потоки и писать параллельные программы (аналог OpenMP)
* на выходе получаем готовый бинарник без зависимостей, которые надо было бы скачивать менеджером зависимостей
* строгая статическая типизация, типы переменных не динамические, и поэтому нет оверхеда по памяти на хранение типа.

Цель создания – быстрая разработка больших распределенных систем

Команда разработки:

- Кен Томпсон (Unix, C, UTF-8)
- Роб Пайк (UTF-8)
- Роберт Гризмер
- Брэд Фицпатрик (livejournal, memcached)

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

  ```
  cd $HOME
  go clean -modcache
  cd твоя-папка-проекта
  rm -rf vendor
  go mod tidy && go mod vendor
  ```

  

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

Literal (литерал) является описанием *constant*'ы (константы). По сути, это *constant* без имени. 



## Integer literal

*Integer literal*  - это последовательность цифр, представляющая [integer constant](#constant). Опциональный префикс устанавливает *non-decimal base*: 

- `0b` или `0B` для *binary*
- `0`, `0o` или `0O` для *octal*
- `0x`или `0X` для *hexadecimal*. 

В *hexadecimal literal*, буквы `a` - `f` и `A` - `F` представляет значения от 10 до 15.

Для удобства чтения символ подчеркивания `_` может стоять после префикса или между последовательными цифрами (любое количество и в любом месте); такие подчеркивания не изменяют значение *literal*'а.

```
int_lit        = decimal_lit | binary_lit | octal_lit | hex_lit .
decimal_lit    = "0" | ( "1" … "9" ) [ [ "_" ] decimal_digits ] .
binary_lit     = "0" ( "b" | "B" ) [ "_" ] binary_digits .
octal_lit      = "0" [ "o" | "O" ] [ "_" ] octal_digits .
hex_lit        = "0" ( "x" | "X" ) [ "_" ] hex_digits .

decimal_digits = decimal_digit { [ "_" ] decimal_digit } .
binary_digits  = binary_digit { [ "_" ] binary_digit } .
octal_digits   = octal_digit { [ "_" ] octal_digit } .
hex_digits     = hex_digit { [ "_" ] hex_digit } .
```



```go
// это правильные literal'ы
42
4_2
0600
0_600
0o600
0O600       // второй символ - заглавная буква 'O'
0xBadFace
0xBad_Face
0x_67_7a_2f_cc_40_c6
170141183460469231731687303715884105727
170_141183_460469_231731_687303_715884_105727

// это неправильные literal'ы
_42         // identifier, не integer literal
42_         // invalid: _ должен разделять последовательные цифры
4__2        // invalid: только один _ между двумя цифрами
0_xBadFace  // invalid: _ должен разделять последовательные цифры
```







## Rune literal

Rune literal (строковый литерал) – *integer value*, которое задает *Unicode code point*. Записывается в *single quote*'s. 

Может состоять из:

- одного символа (`'x'`). Может использоваться любой символ, кроме *newline* и неэкранированной *single quote*. Такой символ описывает *Unicode value* этого символа.
- нескольких символов, начинающихся с *backslash* ( `'\n'`). Такие многосимвольные последовательности кодируют *Unicode value*, используя специально определенные форматы. TODO!!!

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

[link](types/string.md#string-literal)



# Constant

[link](constant.md#constant)





# Type

*Type* может задаваться:

- именем типа (`TypeName`). Возможны варианты:
  - Есть набор предопределенных *type name*'s (`int`, `bool`, ...). 
  - Другие вводятся с помощью [type declaration](#type-declaration). В этом случае у этого типа `T` есть *underlying type* (базовый тип)
- Литералом типа (`TypeLit`). Так задаются *composite types* (составные типы) – `array`, `struct`, *pointer*, `func`, `interface`, `slice`, `map` и `channel` .

```
Type      = TypeName | TypeLit | "(" Type ")" .
TypeName  = identifier | QualifiedIdent .
TypeLit   = ArrayType | StructType | PointerType | FunctionType | InterfaceType |
	    SliceType | MapType | ChannelType .
```

Каждый type имеет некоторое [zero value](#zero-value).

## Виды type's

https://go101.org/article/value-part.html

### Reference type

Есть ссылочные (*reference*) типы, которые содержат внутри себя *reference* на другую структуру:

- *map*
- *slice*
- *channels*
- ...

Поэтому при присвоении – присваивается *value*, которое внутри содержит *reference*. Поэтому, как-бы, они присваиваются *by reference*. 

Эти *type*'s аллоцируются с помощью `make()`, которая возвращает *value* (не *pointer*). Но т.к. они уже внутри содержат *reference*, их всегда и используют *by value* и получать отдельно *pointer* не нужно. 

### Non-reference type

Остальные *type*'s содержат *value* и при присвоении – копируется *value*.

## Type width

Мое: кажется, что это тоже самое что *size*.

Термин *width*, как и большинство терминов, заимствован из компилятора `gc`.

*Width* описывает количество байтов хранилища, которое занимает *type instance*. 

*Width* – это свойство *type*. Поскольку каждое *value* в программе имеет *type*, *value width* определяется его *type* и всегда кратна 8 битам.

Мы можем определить любое *value width* и, следовательно, его *type width* с помощью функции [`unsafe.Sizeof()`](packages/unsafe.md#sizeof).

```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	var a int8
	b := make([]int, 1)
	fmt.Println (unsafe.Sizeof(a)) // выводит 1
  fmt.Println (unsafe.Sizeof(b)) // выводит 24 (pointer+size+capacity)
}
```

*Width* для *array type* – это сумма *width* всех его элементов:

```go
func main() {
	var a [10]int64
	fmt.Println (unsafe.Sizeof(a)) // выводит 80
}
```

Для *struct type* – *width* является суммой *field type width*'s плюс [padding](Internal.md#alignment-padding).

```go
type S struct {
        a uint16
        b uint32
}
var s S
fmt.Println(unsafe.Sizeof(s)) // prints 8, not 6
```





## *Method set*

*Type* имеет *method set*, связанный с ним. *Method set* для *type* определяет *interface*'s, которые этот *type* реализует. 

*Method set* для *interface type* – его *interface*. 

- *Method set* для любого типа `T` содержит все *method*'ы, объявленные с *receiver type* `T` (но *method*'ы, объявленные с *receiver type* `*T` также можно вызывать через *value*, т.к. выполняется автоматическое приведение *value* → *pointer*). 
- *Method set* любого *pointer type*  `*T` – множество всех его *method*'s, объявленных с *receiver type* `*T` или `T`  (т.е. он также содержит в себе *method set* для *type* `T`)
- К *struct type*, содержащим *embedded field*'s, применяются дополнительные правила по формированию *method set* (описано в разделе [struct type](#struct-type))

Связано с [link](#приведение-value-и-pointerа-к-interfacу)

TODO!!!! Почитать еще тут [link](https://stackoverflow.com/questions/23044855/what-is-the-reason-golang-discriminates-method-sets-on-t-and-t/23046811#23046811)

## *String type*

[link](types/string.md)

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
uintptr  unsigned integer достаточно большой, чтобы хранить неинтерпретированные bits для pointer value (любого адреса в памяти)
```

- `rune` – псевдоним для `int32`

## *struct type*

[смотреть](types/struct.md)





 

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

### *Slice type*

[смотреть тут](types/slice.md)

## *Pointer type*

*Pointer type* (`*T`) – *type*, в переменные которого можно сохранит адрес (указатель) на значение *variable* ([1](#variable)) некоторого конкретного *type* `T`. Тип `T` называется *base type* (базовым типом) *pointer*'а. 

*Pointer*'ы в Go практически ничем не отличаются от *pointer*'ов в языках C и C++, за исключением того, что Go не поддерживает арифметику с указателями. *Pointer* объявляется добавлением символа звездочки `*` перед *type* `*T`. 

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

*Function type* описывает множество всех *function*'s с теми же самыми *parameter type*'s и *result type*'s. Значение неинициализированной *variable* с *function type* – `nil` (т.к. *function type* – *reference type*).

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

`ParameterDecl` может включать список *identifier*'s (`IdentifierList`), для которых один раз указан *type*:

```go
func(a, b int)
```

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

К *type* последнего *parameter*'а можно добавить префикс  `...` и тогда функция может быть вызвана с нулем или более аргументов для этого параметра. Такая функция называется *variadic* (вариативной). 

```go
func(prefix string, values ...int)
```

Если в *variadic function* требуются параметры разных *type*'s, можно использовать `interface{}`:

```go
func(a, b int, z float64, opt ...interface{}) (success bool)
```

Про передачу параметров в *variadic function* написано [здесь](#passing-arguments-to-parameters)



В Go нет опциональных параметров, в отличии от PHP. Т.е. все формальным параметрам должны быть назначены фактические параметры при вызове.

## *Interface* *type*

[в отдельном файле](types/interface.md)

## `map` *type*

[в отдельном файле](types/map.md)



 

## *Channel type*

[link](types/channel.md)

# Property of types and values

## Type identity 

Type identity (тождество, идентичность типов)

Два type's – или *identical*, или *different*.





- Два *function type*'s – *identical*, если у них одинаковое количество и одинаковые типы соответствующих *parameter*'s и *result value*'s, и либо обе *function*'s – *variadic*, либо обе – *non-variadic*. Имена для *parameter*'s и *result*'s могут не совпадать.



## Assignability

**✔**

Значение `x`  – *assignable* c *variable* типа `T` (говорят, `x`  – *assignable* с типом `T`) если выполняется одно из следующих условий:

- Тип `x` – это тип `T`.
- Тип `V` значения `x` и тип `T` имеют один и тот же *underlying type* и хотя бы один из `V` или `T` НЕ является [defined type](#определение-type).
- `T` – это *interface type* и значение `x` [implement](#interface-type) тип `T`.
- `x` – это *bidirectional channel value*, `T` – это *channel type*, тип `V` значения `x`  и тип `T`  имеют одни и те же *element type*, и хотя бы один из `V` или `T` НЕ является [defined type](#определение-type).
- `x` – это `nil` и `T` является *pointer, function, slice, map, channel*, или *interface type*.
- `x` – это [untyped constant](constant.md), которая [representable](#representability) значением типа `T`.



## Representability

A [constant](constant.md#constant) `x` (*untyped*???) – *representable* значением типа `T`, если выполняется одно из следующих условий:

- `x` находится во множестве значений, определяемом типом `T`

- TODO!!!







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

*Identifier*, объявленный внутри *block*'а, может быть переобъявлен в любом внутреннем *block*'е. До тех пора пока внутренний *identifier* находится в *scope*, он перекрывает (*shadow*) внешние *identifier*'s. Внешние *identifier*'s называются в этом случае – *shadowed*.

## Примеры *variable shadowing*

Т.е. в разных *block*'ах можно объявлять переменные с одинаковым названием.

- Можно случайно объявить новый *identifier* во внутреннем *block*'е, думая что используется *identifier* из внешнего *block*'а:

  ```go
  err := f()
  
  if ... {
    resp, err := d() // Здесь объявляется новый identifier `err`, а не используется identifier из внешнего block'а
  }
  ```

- *Shadowing* переменной в заголовке блоков `if` и `for`:

  ```go
  package main
  import "fmt"
  func main() {
      i := 1 //scope: main
      for i := 2; i < 3; i++ {
          // i shadowed inside this block
          fmt.Println(i) // 2
      }
  
      if i := "test"; true {
          // i shadowed inside this block
          fmt.Println(i) // test
      } else { // в случае условия false
          // i shadowed inside this block
          fmt.Println(i) //test
      }
  }
  ```

- использование явно объявленных блоков с помощью скобок `{` и `}`:

  ```go
  package main
  import "fmt"
  func main() {
      i := 1
      //new scope:
      {
          i := "hi" // new local var
          fmt.Println(i) // hi
      }
  }
  ```

- можно даже инициализировать переменную во внутреннем *block*'е, значением переменной из внешнего *block*'а:

  ```go
  func main() {
  	a := 1
  
  	{
  		a := a // Переменную во внутреннем block'е инициализируем значением переменной из внешнего block'а
  		a++
  		println(a) // 2
  	}
  
  	println(a) // 1
  }
  ```

  







 



## Identifier

### Blank identifier

*Empty identifier* (??? возможно и такое название)

*Blank identifier* записывается символом подчеркивания `_`. Он используется как анонимный заполнитель вместо обычного *non-blank identifier* и имеет особое значение в [declaration](#declarations-и-scope) , в качестве [operand'а](#operand) и в [assignment](#assignment) .



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

Когда *identifier* – *exported* в *package*, это означает, что к нему можно получить прямой доступ из любого другого *package*. Когда *identifier* – *non-exported* в *package*, к нему нельзя получить прямой (!!!) доступ из любого другого *package*. Но это также значит, что к *non-exported identifier* можно получить доступ НЕ напрямую.

*Identifier* является *exported*, если выполняется два условия:

1. первый символ *identifier*'а – это буква в *Unicode upper case*; И!!!!
2. *identifier* объявлен в *package block* ([1](#block)) или является *field* ([1](#struct-type)) или *method* ([1](#method-declaration)) (т.е. *field* и *method* могут быть *exported*, и должны в этом случае начинаться с буквы в *Unicode upper case*!!!)

Все остальные *identifier*'s – *non-exported*.

#### *Exported field* и *method*

*Field* и *method* могут быть *exported*.

Варианты:

- *struct* – *exported*, *field* и *method* – *non-exported*, мы не можем обратиться к *field* и *method* через *struct* из другого *package*.

  ```go
  package model
  
  type Person struct {
  	age  int
  }
  
  func (p *Person) getAge() string {
      return p.Name
  }
  ```

  ```go
  package main
  
  import "fmt"
  
  func main() {
      p := &Person{
          age:  21,
      }
  
      fmt.Println(p.age) // ошибка
      fmt.Println(p.getName()) // ошибка
  }
  ```

- *struct* – *non-exported*:

  - *method* – *exported*. Имеет смысл, когда тип реализует *interface*. И из другого *package* мы получаем доступ к *method* через интерфейс. Но при этом сам *struct type* остается *non-exported*.

    ```go
    type Client interface {
    	GetUrls()
    }
    
    type client struct {}
    
    func New() Client {
    	return &client{}
    }
    
    func (c *client) GetUrls() {}
    ```

  - *field* – *exported*. Пример – кодирование в json через пакет `encoding/json`.

    ```go
    type response struct {
        Success bool   `json:"success"`
    }
    
    func main(w http.ResponseWriter, r *http.Request) {
        resp := &response{
            Success: true,
        }
        json.NewEncoder(w).Encode(resp); 
    }
    ```

    У `Encode()` аргумент – `interface{}`. Наверное, там как то используется *reflection* для доступа к *exported field*.

#### *Non-exported type*

Можно возвращать *non-exported type* из *exported function*:

```go
package params

type validator struct {}

func NewValidator() *validator {
  return &validator{}
```

golint при этом выдает ошибку (может уже не выдает?) `exported func XXX returns unexported type *xxx, which can be annoying to use (golint)"` ([1](https://github.com/golang/lint/issues/210))

Таким способом можно реализовать различные фабричные (factory, создающие) *design pattern*. Например, singleton ([1](http://blog.ralch.com/tutorial/design-patterns/golang-singleton/)). Это полезно в Go, если вы не хотите, чтобы сторонний разработчик напрямую создавал объекты `validator`. Таким образом, «конструктор» или «фабрика» `NewValidator()` - единственное место, где можно получить новые экземпляры.

При этом код, который получает *non-exported type*, не может использовать его для различных «сигнатур» (например, типов, функций):

```go
type eff struct {
    m *params.validator  // <-- cant do this
}

func main() {
   var m params.validator // <-- cant do this
   m2 := params.NewValidator() // can do this
}
```

Лучшим вариантом, будет создание *exported interface* с возвращаемым в нем *non-exported type*:

```go
package params

type Valid interface {}

type validator struct {}

func NewValid() *Valid {
  return &validator{}
}
```





## *Constant declaration*

[link](constant.md#constant-declaration)



## Iota

[link](constant.md#iota)



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



## *Variable declaration*

*Variable declaration* создает одну или более *variable*'s, привязывает к ним *identifier*'s и дает им *type* и *initial value*.

<pre>
VarDecl     = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
VarSpec     = IdentifierList ( Type [ "=" <a href="#constant-declaration">ExpressionList</a> ] | "=" <a href="#constant-declaration">ExpressionList</a> ) .  
</pre>
После объявления переменной (с указанием типа или автоматическим определением типа), в следствие строгой типизации, ей могут присваиваться значения только того же типа.

Эта форма объявления называется – полная форма объявления (*regular variable declaration*).

```go
var a = "b"; 
var a string = "b" // С указанием типа

var i int
var U, V, W float64
var k = 0
var x, y float32 = -1, -2
var c uint64 = b
var (
	i       int
	u, v, s = 2.0, 3.0, "bar"
)

var _, ok = entries[name]  // получение данных из map; нас интересует только есть элемент `ok`, или нет

var b = []int{2, 3, 5}      
```





Если `ExpressionList` не указан, *variable* инициализируется *zero value* ([link](#zero-value)).

### Short variable declaration

*Short variable declaration* (сокращенное объявление переменной). Одновременно объявляет и инициализирует переменные. 

<pre>
ShortVarDecl = IdentifierList ":=" ExpressionList .  
</pre>

Это сокращение для *regular variable declaration* с `ExpressionList`, но без `Type`:

```
"var" IdentifierList = ExpressionList .
```



```go
a := "b"; // Без указания типа
a := string("b") // С указанием типа
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



*Short variable declaration* может появляться только внутри *function*. В некоторых контекстах, таких как *initializer* (??? наверно, присвоение в заголовке) для `if` , `for` или `switch` *statement*'s , *short variable declaration* может использоваться для объявления локальных временных *variable*.



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



### Передача параметров *by value* VS *by pointer*

Существует два способа передачи *parameter*'ов (и *receiver*'а):

- *by value* (`T`). Внутри *function* мы используем копию оригинального значения. Любые изменения переменной, произведенные внутри функции, никак не отразятся на оригинальном значении.
- *by pointer* (`*T`). Внутри *function* мы можем изменить оригинальное значение, на которое ссылается *pointer*.



Существуют следующие причины передачи *parameter*'ов (и *receiver*'а) *by pointer*:

- если необходимо в *function* измененить оригинальное значение фактического параметра. 
- в Go не используется техника *copy-on-write* (????). Для маленьких по размеру *type* (например, для которых *underlying type* – `struct` из нескольких значений `int` или `string`) – можно использовать передачу *by value*. Для значений большого типа, намного дешевле передать в параметре *pointer* на это значение, чем копировать само значение. Потому что операция передачи указателя (обычно 32- или 64-битного значения) намного дешевле. И это имеет смысл даже для методов, не изменяющих значения *parameter*'ов (или *receiver*'а). 

Связано с [link](#передача-by-value-vs-передача-by-pointer)

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
-  или быть *pointer*'ом на [defined type](#определение-type) (`*T`). В этом случае *method* может изменять значение *receiver*'а. Т.к. это часто требуется, *pointer receiver*'ы встречаются чаще, чем *value receiver*'ы.

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
```

В качестве *underlying type* может выступать не только *struct type*, но и любой (точно любой) другой *type*. Например, `int`:

```go
type Implementation int

func (v Implementation) String() string {
    return fmt.Sprintf("Hello %d", v)
}
```

### nil receiver

https://go101.org/article/nil.html

https://stackoverflow.com/questions/69384852/behaviour-of-nil-receiver-in-method-in-golang

https://medium.com/@reetas/nil-receiver-in-golang-9d61ed8fd230

https://stackoverflow.com/questions/42238624/calling-a-method-on-a-nil-struct-pointer-doesnt-panic-why-not

https://groups.google.com/access-error?continue=https://groups.google.com/g/golang-nuts/c/wcrZ3P1zeAk/m/WI88iQgFMvwJ?pli%3D1





### Reciever By value vs By pointer

Для одного *type* можно смешивать *method*'s с *reciever by value* и c *receiver by pointer*:

```
type MyType struct {}

func (m *MyType) SetVal() {}
func (m MyType) GetVal() {}
```

TODO??? Как работает вызов методов у reciever'а by value, влияет на исходный reciever или на копию?

https://stackoverflow.com/questions/27775376/value-receiver-vs-pointer-receiver/27775558#27775558

Best practices  тут (от создателей Go): https://github.com/golang/go/wiki/CodeReviewComments#receiver-type

## Конструктор

Go не является традиционным ООП языком, `struct` не обладают всеми свойствами объектов и и не могут иметь конструктор.

Однако *variable* любого типа (в том числе `struct`) инициализируются *zero value*. А *composite type*'s инициализируются рекурсивно. 

Если нужно получать *variable*, инициализированную некоторым значением, можно вручную написать функцию-конструктор и вызывать ее явно. 

<u>Примеры:</u>

```go
package debug

type Handler struct {
	storage Redis
}

func New(storage Redis) *Handler {
	return &Handler{
    storage: storage
  }
}
```

```go
d := debug.New(storage)
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

## Operand

*Operand* описывает элементарное значение в *expression*. 

Operand может быть представлен как:

- *literal*
- (*qualified* или *non-qualified*) *non-blank identifier* обозначающий
  - *constant*
  - *variable*
  - *function*
  - *expression* в круглых скобках `()`.

*Blank identifier* может появляться в качестве operand только слева от знака присваивания (*assignment*).

<pre>
Operand     = Literal | OperandName | "(" Expression ")" .
Literal     = BasicLit | CompositeLit | FunctionLit .
BasicLit    = int_lit | float_lit | imaginary_lit | rune_lit | string_lit .
OperandName = identifier | QualifiedIdent .  
</pre>



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

- *struct type*. `Key` – *field name*.

- *array type*. `Key` – *index*.

  ```go
  a := [...]int {1, 2}
  ```

- *slice type*. `Key` – *index*.

  ```go
  a := []int {1, 2}
  ```

  

- *map type*. `Key` – *key* (ключ). Для *map type*, все элементы должны иметь `Key`.

  ```go
  a := map[int]string {1: "a", 2: "b"}
  ```

  

Пусть объявлено:

```go
type S struct {
  a int
  b int
}
```

Тогда *struct literal* должен:

- Если `ElementList` не содержит `Key`'s, то должны быть указаны элементы для каждого *field* в *struct type* и в том порядке, в каком *field*'s перечислены при описании *struct type*. 

  ```go
  s := S{1, 2}
  ```

- Если у одного элемента есть `Key`, у всех элементов должны быть `Key`.

  ```go
  s := {
    a: 1
    b: 2
  }
  ```

  

- Если у элементов указаны `Key`'s, то не обязательно должны быть указаны элементы для всех *struct field*'s. Для *field*'s, которые не указаны, устанавливается *zero value*. 

  ```go
  s := S{
    a: 1 // zero value для b
  }
  ```

- Может полностью отсутствовать `ElementList`. В этом случае для всех *field*'s устанавливается *zero value*.

  ```go
  s := S{} // zero value для a и b
  ```

  

  



TODO!!! Zero value

Можно использовать *address operator* ([1](#address-operator)) для *composite literal*, чтобы получить *pointer* на новое значение:

```go
var s *S = &S{a: 1}
```

Если используется *address operator* для пустого *composite literal* для *struct type* (!!!!) – то это аналогично `new()` :

```go
s := new(S)
s := &S{}
```

Однако использование *address operator* для пустого *composite literal* для *slice type* и *map type* – это не то же  самое, что и `new()`:

```go
p1 := &[]int{}    // p1 указывает на инициализированный, пустой slice со значением []int{} и длиной 0
p2 := new([]int)  // p2 указывает на неинициализированный slice со значением nil и длиной 0
```



В composite literal для типов: *array type*, *slice type*, *map type* (типа `T`) –  у *element*'ов (для *array*, *slice*, *map*)  и *key*'s (для *map*), которые сами по себе являются *composite literal*, может опускать соответствующий *literal type*, если он идентичен *element type* или *key type* `T`. 

```go
type Point struct{ x, y float64 }

[...]Point{{1.5, -3.5}, {0, 0}}     // same as [...]Point{Point{1.5, -3.5}, Point{0, 0}}
[][]int{{1, 2, 3}, {4, 5}}          // same as [][]int{[]int{1, 2, 3}, []int{4, 5}}
[][]Point{{{0, 1}, {1, 2}}}         // same as [][]Point{[]Point{Point{0, 1}, Point{1, 2}}}
map[string]Point{"orig": {0, 0}}    // same as map[string]Point{"orig": Point{0, 0}}
map[Point]string{{0, 0}: "orig"}    // same as map[Point]string{Point{0, 0}: "orig"}
```



Аналогично, внутри *pointer* на *composite literal* (`*T`) можно опускать *address operator* (`&T`), если *element*'s или *key*'s являются *pointer*'s этого же типа (`*T`).

```go
type PPoint *Point
[2]*Point{{1.5, -3.5}, {}}          // same as [2]*Point{&Point{1.5, -3.5}, &Point{}}
[2]PPoint{{1.5, -3.5}, {}}          // same as [2]PPoint{PPoint(&Point{1.5, -3.5}), PPoint(&Point{})}
```





### Array literal

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



### Slice literal

Slice literal описывается следующим образом: 

```
[]T{x1, x2, … xn}
```

Его length и capacity – максимальный индекс + 1.

Каждая строка *composite literal* должна заканчиваться запятой `,`, даже если это последняя или единственная строка в expression. Это результат  автоматической простановки точек с запятой ([1](#точка-с-запятой)).

```go
mapa := map[string]string{
    "jedan": "one",
    "dva":   "two"  // ошибка, здесь будет поставлена ;
}
```




## *Function literal*

*Function literal* представляет анонимную *function*.

<pre>
FunctionLit = "func" Signature FunctionBody .  
</pre>

```go
func (b int) int { return b }
```

*Function literal* может:

- присваиваться переменной:

  ```go
  f := func (b int) int { return b }
  ```

- вызываться сразу при объявлении:

  ```go
  func (b int) int { return b } (1)
  ```

*Function literal* – *closure* (замыкание): они могут использовать *variable*'s, определенные во внешних *function*'s (??? *block*'s). Эти *variable*'s используется совместно внешними *function*'s и *function literal*, и не вычищаются из памяти до тех пор пока доступны в каком-то месте.



## *Primary expression*

Primary expression – операнд для *unary* и *binary expression*.

<pre>
PrimaryExpr =
   Operand |
   Conversion |
   MethodExpr |
   PrimaryExpr Selector |
   PrimaryExpr Index |
   PrimaryExpr Slice |
   PrimaryExpr TypeAssertion |
   PrimaryExpr Arguments .
Selector       = "." identifier .
Index          = "[" Expression "]" .
Slice          = "[" [ Expression ] ":" [ Expression ] "]" |
                 "[" [ Expression ] ":" Expression ":" Expression "]" .
TypeAssertion  = "." "(" Type ")" .
Arguments      = "(" [ ( ExpressionList | Type [ "," ExpressionList ] ) [ "..." ] [ "," ] ] ")" .  
</pre>

Примеры:

```
x
2
(s + ".txt")
f(3.1415, true)
Point{1, 2}
m["foo"]
s[i : j + 1]
obj.color
f.p[i].x()
```



## *Selector expression*

Для *primary expression* `x` (за исключением случая, когда `x` – *package name*, и используется конструкция `<packageName>.<identifier>`), *selector expression*

```
x.f
```

описывает *field* или *method* `f` значения `x` (или иногда `*x`). Идентификатор `f` называется *(field or method) selector*. Тип *selector expression* - это тип `f`. 

*Selector* `f`  может:

- обозначать *field* или *method* `f` самого типа `T`, 
- или может ссылаться на *field* или *method* `f` вложенного в `T` *embedded field* ([1](#embedded field)).

Пример:

```go
type T0 struct {
	x int
}

func (*T0) M0()

type T1 struct {
	y int
}

func (T1) M1()

type T2 struct {
	z int
	T1
	*T0
}

func (*T2) M2()

type Q *T2

var t T2     // with t.T0 != nil (что это за коммент?)
var p *T2    // with p != nil and (*p).T0 != nil (что это за коммент?)
var q Q = p
```



К *selector*'ам применяются следующие правила (эти правила фокусируются в основном на embedded и promoted fields, более простые варианты одноуровневого доступа подробно описал здесь [1](#автоматическое-приведение-pointer-value-и-value-pointer*)) :

- Для значения `x` типа `T` или `*T`, где `T` – НЕ(!!!) *pointer* или *interface type*, `x.f` обозначает  *field* или *method* на самой мелкой (высшей) глубине в `T`.
- TODO:
- 

## *Method expression*

Также [method value](#method-value)

Если `M` находится в [method set](https://go.dev/ref/spec#Method_sets) типа `T`, `T.M` – это *function*, которую можно вызывать как обычную *function* с теми же аргументами как и `M`, но с префиксом в виде дополнительного аргумента, который является *receiver*'ом *method*'а. Такой *expression* называется *method expression*.

```
MethodExpr    = ReceiverType "." MethodName .
ReceiverType  = Type .
```

Рассмотрим *struct type* `T` с двумя методами, `Mv ` – *receiver* которого имеет тип `T`, и `Mp` – *receiver* которого имеет тип `*T`.

```go
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
```

*Expression* вида

```go
T.Mv
```

возвращает *function value* (*function value* – это *value* с *function type*), представляющее `Mv`, но с явным *receiver*'ом в качестве первого аргумента; оно имеет сигнатуру

```go
func(tv T, a int) int
```

Пример:

```go
type T struct {
  a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver

func main() {
  a := T.Mv

  fmt.Printf("%T", a)  // func(main.T, int) int
}
```

Эта *function* может быть вызвана обычным образом с явным *receiver*'ом, поэтому эти пять вызовов эквивалентны:

```go
t.Mv(7)
T.Mv(t, 7)
(T).Mv(t, 7)
f1 := T.Mv; f1(t, 7)
f2 := (T).Mv; f2(t, 7)
```

Точно так же *expression*

```go
(*T).Mp
```

возвращает *function value*, представляющее `Mp` с сигнатурой

```go
func(tp *T, f float32) float32
```

Для метода с *value receiver* с помощью *method expression* можно получить *function* с явным *pointer receiver*, поэтому

```
(*T).Mv
```

возвращает *function value*, представляющее `Mv` с сигнатурой

```go
func(tv *T, a int) int
```

Такая *function* получает *pointer* `*T` в качестве *receiver*, создает *value*, который передает в качестве *receiver*'а в *underlying method* (т.е. в исходный метод); метод не перезаписывает *value*, адрес которого передается при вызове *function*.

Последний случай, *value-receiver function* для *pointer-receiver method*, является недопустимым, поскольку *pointer-receiver method* не входят в [method set](#method-set) для *value type*.

*Function value*, полученные через *method expression*, вызываются с использованием синтаксиса [function call]() (вызов функции); *receiver* выступает в качестве первого аргумента *call* (вызова). То есть для `f := T.Mv`, `f()` вызывается как `f(t, 7)` а не как `t.f(7)`. Чтобы создать *function*, которая привязана к *receiver*'у, используйте [function literal](#function-literal) или [method value](#method-value).

Допустимо получать *function value* по методу из *interface type*. Результирующая *function* принимает явный *receiver* этого *interface type*.

## *Method value*

Также [method expression](#method-expression)

Если *expression* `x` имеет тип `T` и `M` находится в [method set](#method-set) типа `T`, `x.M` называется *method value*. *Method value* `x.M` — это *function value*, которое можно вызывать с теми же аргументами, что *method call* (вызов метода) `x.M`. *Expression* `x` вычисляется и сохраняется во время вычисления *method value*; сохраненная копия затем используется в качестве *receiver*'а в любых *call* (вызовах), которые могут быть выполнены позже.

*Type* `T ` может быть *interface* или *non-interface type*.

Как и при обсуждении [method expression](#method-expression) выше, рассмотрим *struct type* `T` с двумя *method*'ами, *receiver* `Mv` имеет тип `T`, и *receiver* `Mp` имеет тип `*T`.

```go
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
var pt *T
func makeT() T
```

*Expression* вида:

```go
t.Mv
```

возвращает *function value* типа

```go
func(int) int
```

Эти два вызова эквивалентны:

```
t.Mv(7)
f := t.Mv; f(7)
```

Точно так же *expression*

```go
pt.Mp
```

возвращает *function value* типа

```go
func(float32) float32
```

Как и в случае с [selector expression](#selector-expression), обращение к *non-interface method* с *value receiver*'ом, используя *pointer*, будет автоматически разыменовывать этот *pointer*: `pt.Mv` – эквивалентно `(*pt).Mv`.

Как и в случае с  [method call](https://go.dev/ref/spec#Calls), обращение к *non-interface method*'у с *pointer receiver*'ом, используя *addressable value* (???), будет автоматически брать адрес этого *value*: `t.Mp` эквивалентно `(&t).Mp`.

```go
f := t.Mv; f(7)   // like t.Mv(7)
f := pt.Mp; f(7)  // like pt.Mp(7)
f := pt.Mv; f(7)  // like (*pt).Mv(7)
f := t.Mp; f(7)   // like (&t).Mp(7)
f := makeT().Mp   // invalid: result of makeT() is not addressable
```

Хотя в приведенных выше примерах используются *non-interface type*'s, также разрешено создавать *method value* для *value*, имеющего *interface type*.

```go
var i interface { M(int) } = myVal
f := i.M; f(7)  // like i.M(7)
```

## *Index expression*

Получение значения по индексу для `array`, `*array`, `slice`, `string` и `map`:

```go
a[x]
```

Если `a` – НЕ `map`:

- индекс  `x` должен быть *integer type* или *untyped constant*
- индекс `x` находится *in range*, если `0 <= x < len(a)`, иначе он *out of range*

Для `slice S`:

- если индекс `x` выходит *out of range*, возникает [run-time panic](https://go.dev/ref/spec#Run_time_panics)
- `a[x]` — это *slice element* по индексу `x`, а тип `a[x]` — это тип элемента `S`



Для `string`:

-  `a[x]` – значение байта по индексу `x`, тип `a[x]` – `byte`

Для `map` :

- тип `x` должен быть *assignable* c *key type*
- если `map` – `nil` или не содержит ключ `x`, то `a[x]` – [*zero value*](#zero-value) для *element type*.
-  также с *index expression* при [assignment](https://go.dev/ref/spec#Assignments) можно использовать специальную форму с дополнительным [boolean value](types/map.md#частые-операции)

TODO!!!

## *Slice expression*

Смотреть [здесь](types/slice.md#slice-expression)

## Type assertion

Есть два вида преобразований:

- [conversion](#conversion) (приведение типа)
- *type assertion*

Если:

- `x` – *expression of interface type*, но не [type parameter](https://go.dev/ref/spec#Type_parameter_declarations)
- `T` – *type*

то 

```go
x.(T)
```

утверждает, что:

-  `x != nil` 
- и что значение в `x`  имеет тип `T`.

Если *type assertions* выполняется, возвращает значение, хранящееся в `x` с типом `T`. 

TODO!!!

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

  `ok` – это *untyped boolean value*.

  Если *type assertions* выполняется – `ok == true`, иначе – `ok == false` и значение `v` – *zero value* для типа `T`. *Run-time panic* не происходит.

  Аналог *type switch* ([link](#type-switch)):
  
  ```go
  if v, ok := any.(Stringer); ok {
    return v.String()
}
  ```
  
  





## Calls (вызовы *function* и *method*)

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

Про передачу параметров *by pointer* здесь ([1](#pointer-internal))

### Passing arguments to `...` parameters

Если *function* `f` является [variadic](#function-type) с последним параметром `p` типа `...T`, то внутри `f`  тип переменной  `p` – тип `[]T`. 

```go
func f (p ...int) {
	// p имеет тип []int
}
```

Если `f` вызывается без фактических параметров для  `p`, то в функции `p == nil`. 

```go
func f (p ...int) {
  // p = {[]int} nil
}

func main() {
	f()
}
```

Если `f` вызывается с фактическими параметрами для  `p`, то внутри функции `p` – это новый *slice* типа `[]T ` с новым *underlying array*, элемент которого – это фактические аргументы `f`, которые все должны быть *assignable* с `T`. 

```go
func f (p ...int) {
  // p = {[]int} len:2, cap:2
  //    0 = {int} 1
  //    1 = {int} 2
}

func main() {
	f(1, 2)
}

```

Если последний параметр `p` –  *assignable* к *slice type* `[]T` и после него указано троеточие `...`., то параметр передается без изменений. В этом случае новый *slice* не создается.

```go
func f (p ...int) {
  // p = {[]int} len:2, cap:2
  //    0 = {int} 1
  //    1 = {int} 2
}

func main() {
	array := []int{1,2}
	f(array...)
}
```

Внутри функции `f` формальный параметр `p` будет иметь будет иметь то же значение что и `array`, с тем же *underlying array*.



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
*Comparison operator* обсуждаются [в соответствующем разделе](#comparison-operator). Для других *binary operator*, *operand type*'s должны быть (!!!!) [identical](#type-identity), если *operation* не включает *shift operator* или [untyped constants](#constant). Для подробного описания *operation*, включающих только *constant*'s, смотрите раздел [constant expressions](constant.md#constant-expression).

TODO!!!!

### Arithmetic operator

*Arithmetic operator* применяются к *numeric value*'s и возвращают результат того же самого *type*, что и первый *operand*.

Ниже таблица с указанием *type*'s, к которым они могут применяться:

```go
+    sum                    integers, floats, complex values, strings
-    difference             integers, floats, complex values
*    product                integers, floats, complex values
/    quotient               integers, floats, complex values
%    remainder              integers

&    bitwise AND            integers
|    bitwise OR             integers
^    bitwise XOR            integers
&^   bit clear (AND NOT)    integers

<<   left shift             integer << integer >= 0
>>   right shift            integer >> integer >= 0
```

#### Integer operator

Для двух *integer value*'s `x` и `y` целое частное `q = x / y` и остаток от деления `r = x % y` удовлетворяют следующим отношениям:

```
x = q*y + r  and  |r| < |y|
```

при этом для `x / y` выполняется *truncate*:

```
 x     y     x / y     x % y
 5     3       1         2
-5     3      -1        -2
 5    -3      -1         2
-5    -3       1        -2
```

Как сказано в [operator](#operator):

> Для других *binary operator*, *operand type*'s должны быть (!!!!) [identical](#type-identity)

Поэтому здесь будет выдана ошибка:

```go
package main
 
func main() {
    a := 3.14
    b := 2
    result := a / b  // invalid operation: a / b (mismatched types int and int64)
}
```



#### Integer overflow

Для *unsigned integer value* операции `+`, `-`, `*` и `<<` вычисляются по модулю `2^n`, где `n` — разрядность типа *[unsigned integer](https://go.dev/ref/spec#Numeric_types)'s type*. Грубо говоря, эти операции с *unsigned integer* отбрасывают старшие *bit*'ы при переполнении, и программы могут полагаться на «циклический переход».

Для *signed integer* операции `+`, `-`, `*`, `/` и `<<` могут законно переполняться, а результирующее значение существует и детерминистически определяется представлением для *signed integer*, операцией и ее операндами. Переполнение не вызывает [run-time panic](https://go.dev/ref/spec#Run_time_panics). Компилятор не может оптимизировать код, предполагая, что переполнения не происходит. Например, он не может заранее предполагать, что `x < x + 1` – это всегда верно.



#### String concatenation

`string` можно конкатенировать с помощью оператора `+` или оператора присваивания `+=`:

```
s := "hi" + " "
s += "and good bye"
```

В результате будет создана новая строка (т.к. строки *immutable*).



### Comparison operator

*Comparison operator*'ы (операторы сравнения) сравнивают два *operand*'а и выбрасывают *untyped* значение типа `bool`.

```
==    equal
!=    not equal
<     less
<=    less or equal
>     greater
>=    greater or equal
```

```go
const c = 3 < 4            // c – untyped boolean constant со значением `true`

type MyBool bool
var x, y int
var (
  // Результат сравнения — untyped boolean.
	// Применяются обычные правила assignment.
	b3        = x == y // b3 has type bool
	b4 bool   = x == y // b4 has type bool
	b5 MyBool = x == y // b5 has type MyBool
)
```



Первый *operand* должен быть [assignable]() с типом второго operand'а, или наоборот.

Правила применения этих *operator*'ов:

- Операторы равенства `==` и `!= ` применяются к *comparable* операндам. 
- Операторы порядка  `<`, `<=`, `>` и `>=` применяются к *ordered* операндам . 

Отношение разных *type*'s к понятиям *comparable* и *ordered* указано ниже:

- Значения *boolean type* – *comparable*. Два *boolean value*'s равны, если они или оба `true`, или оба `false`.
- Значения *string type* – *comparable* и *ordered*, побайтово в лексическом порядке (*lexically byte-wise*) (??? побайтово значит сравниваются).
- Значения *pointer type* – *comparable*. Два *pointer value*'s равны, если они указывают на одну и ту же *variable* или если оба имеют значение `nil`. *Pointer*'ы на разные *variable*'s, которые имеют [zero-size](https://go.dev/ref/spec#Size_and_alignment_guarantees) ([struct{}](types/struct.md#empty-struct)) могут быть равны или не равны.

- Значения *channel type* – *comparable*. Два *channel value*'s равны, если они были созданы одним и тем же вызовом [`make()`](https://go.dev/ref/spec#Making_slices_maps_and_channels) или если оба имеют значение `nil`.
- Значения *interface type*. Два *interface value*'s равны, если они (1) имеют [identical](#type-identity) *dynamic type*'s и одинаковые *dynamic value*'s или (2) если оба имеют значение `nil`.
- Значение `x` с *non-interface type* `X` и значение `t` c *interface type* `T` – *comparable*, когда значения c *type* `X` – *comparable* и `X` *implement* `T`. Они равны, если *dynamic type* `t` идентичен `X`, а *dynamic value* `t` равно `x`.
- Значения *struct type* – *comparable*, если все их *field*'s – *comparable*. Два *struct value*'s равны, если равны их соответствующие non-[blank](blank-identifier) *field*'s
- Значения *array type* – *comparable*, если значения *array element type* – *comparable*. Два *array value*'s равны, если равны их соответствующие *element*'ы.

Сравнение двух *interface value*'s с одинаковыми *dynamic type*'s вызывает [run-time panic](https://go.dev/ref/spec#Run_time_panics), если *value*'s этого *type* – не *comparable*. Это поведение применимо не только к прямому сравнению *interface value*'s, но и при сравнении (1) *array*'s с *interface value*'s или (2) *struct*'s с *interface field*'s.

Значения *slice*, *map* и *function type* – *not comparable*. Однако, как особый случай, значения *slice*, *map* или *function type* функции могут сравниваться с `nil`.





### Address operator

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

Для *pointer*'а  `x` типа `*T `, разыменование (*dereferencing*, косвенное обращение, *inderection*) *pointer*'а `*x` обозначает переменную типа `T`, на которую указывает `x`. Разыменование выполняется добавлением символа звездочки `*` перед именем переменной.

```go
*p
*pf(x)
```

С помощью *dereferencing* можно изменить *value*, на которое указывает *pointer*.

- например, можно изменить *value* через *pointer* в функции:

  ```go
  func f(a *int) {
    *a = 5
  }
  
  func main() {
    a := 1
  
    f(&a)
  
    fmt.Printf("%v", a)  // 5
  }
  ```

  

- или изменить значение *receiver*'а в методе:

  ```go
  type A int
  
  func (a *A) f() {
    *a = A(5)
  }
  
  func main() {
    var a A
  
    a.f()
  
    fmt.Printf("%v", a)  // 5
  }
  ```

  



### Receive operator

смотреть [Receive operator](types/channel.md#receive-operator)

### Ternary operator `?:`

В Go нет *ternary operator* `?:`. Для достижения того же результата необходимо использовать `if-else`:

```go
if expr {
    n = trueVal
} else {
    n = falseVal
}
```

Если нужно объявить переменную, то короче будет так:

```go
n := falseVal

if expr {
    n = trueVal
} 
```





Причина отсутствия *ternary operator* `?:` в Go заключается в том, что разработчики языка слишком часто видели *ternary operator* `?:`, используемый для создания непостижимо сложных выражений. `if-else` *statement*, хотя длиннее по размеру, но бесспорно яснее. Для языка требуется только одна конструкция *conditional control flow*.



## Conversion

Есть два вида преобразований:

- conversion (приведение типа)
- [type assertion](#type-assertion)

(подробнее https://stackoverflow.com/a/19579058)

*Conversion* (приведение типа) изменяет тип *expression* на тип, указанный в *conversion*. 

*Conversion* может быть:

- *explicit conversion* – явное приведение типа
- *implied conversion* – неявное приведение типа, определяемое контекстом для *expression*.

Преобразование может появиться буквально в источнике или *подразумеваться* (???) контекстом, в котором появляется выражение.

*Explicit conversion* записывается в виде:

<pre>
Conversion = Type "(" Expression [ "," ] ")" .  
</pre>
Если тип начинается с оператора `*`или `<-`, или если тип начинается с ключевого слова `func` и не имеет списка результатов, он должен быть заключен в круглые скобки `()`, во избежание двусмысленности:

```go
*Point(p)        // same as *(Point(p))
(*Point)(p)      // p is converted to *Point
(*Point)(nil)    // nil представляется в виде типа *Point
<-chan int(c)    // same as <-(chan int(c))
(<-chan int)(c)  // c is converted to <-chan int
func()(x)        // function signature func() x
(func())(x)      // x is converted to func()
(func() int)(x)  // x is converted to func() int
func() int(x)    // x is converted to func() int (unambiguous)
```



При этом `Expression` должно быть *converted* (!!!) в тип `Type`. Правила для *converted*:

- *[Constant](https://golang.org/ref/spec#Constants) value* `x` может быть *converted* в тип `T`, если `x` – [representable]() значением типа `T`.

- *Non-constant value* `x` может быть *converted* в тип `T`, если:

  - `x` – [*assignable*](#assignability) к `T`.

    Пример:

    ```go
    type MyInt int
    var a int = 10
    var b MyInt = MyInt(a)
    ```
    
  - игнорируя *struct tag*'s (см. ниже), `x`'s *type* и `T` не являются  [type parameter](https://go.dev/ref/spec#Type_parameter_declarations)'ами, но имеют [identical](https://go.dev/ref/spec#Type_identity) [underlying types](https://go.dev/ref/spec#Types).
  
  - игнорируя *struct tag*'s (см. ниже), `x`'s *type* и `T` являются *pointer type*'s, которые не являются  [named type's](https://go.dev/ref/spec#Types), а их *pointer base type*'s не являются *type parameter*'s, но имеют идентичные *underlying type*'s.
  
  - `x`'s *type* и `T`  являются *integer* или *floating point (*число с плавающей запятой) типами.



Т.е. при *conversion* есть разница между *constant* и *non-constant value*:

```go
  const a float64 = 5.7
  b := int(a)            // ошибка, т.к. float64 – не representable типом int

  var c float64 = 5.7
  d := int(c)            // ок, т.к. типы int и float64 – оба integer или floating point типы.
```



### Conversion между numeric type's

При *conversion* для *non-constant numeric value* применяются следующие правила:

1. При *conversion* между [integer types](#числовые-типы), если *value* является *signed integer*, (TODO!!! надо разбираться)
2. При *conversion* *floating-point number* в *integer* – дробная часть отбрасывается (truncation, усечение до нуля).
3. TODO!!!






### Conversion в и из *string type*

4. *Conversion* значения *string type* в *byte slice type* – дает *slice*, элементы которого – байты строки.

   ```go
   []byte("hellø")   // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
   MyBytes("hellø")  // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}  // defined type
   ```




## Constant expression

[link](constant.md#constant-expression)






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



## Send statement

смотреть [Send statement](types/channel.md#send-statement)





## IncDec statement `++`, `--`

В отличие от C++, операторы `++` и `—` могут использоваться только как оператор, но не как выражение. 

```go
a++
```

Они могут применяться лишь в постфиксной форме. Это предотвращает появление проблем, связанных с неправильным порядком вычислений.



## Assignment

TODO







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

Алгоритм:

- Вычисляется *expression* в *switch*
- вычисляются *expression* в *case*. Они не обязательно должны быть константами. Первое *expression* равное *switch expression* запускает выполнение *statement*'s соответствующего блока; остальные *case*'s пропускаются. 
- Если ни для одного *case* не выполняется равенство, выполняются *statement*'s из *default*. 

Может быть не более одного *default case*, и он может появляться в любом месте *switch statement*. Отсутствующее *switch expression* эквивалентно логическому значению `true`.

<pre>
ExprSwitchStmt = "switch" [ SimpleStmt ";" ] [ Expression ] "{" { ExprCaseClause } "}" .
ExprCaseClause = ExprSwitchCase ":" StatementList .
ExprSwitchCase = "case" <a href="#constant-declaration">ExpressionList</a> | "default" .
</pre>
*Simple statement* может предшествовать *switch expression*, который выполняется перед тем как будет вычислено *switch expression*.

Компилятор запрещает использовать несколько *case expression*'s, которые содержат одну и ту же *integer, floating point* или *string constant*. 

<u>Примеры:</u>

```go
switch tag {
default: s3()
case 0, 1, 2, 3: s1()
case 4, 5, 6, 7: s2()
}

switch x := f(); {  // missing switch expression means "true"
case x < 0: return -x
default: return x
}

switch {
case x < y: f1()
case x < z: f2()
case x == 4: f3()
}
```



```go
switch lookup {
case
    "900898296620",
    "900898296636":
    return true
}
```



```go
switch doc.Key.Type {
		case internal.CountryDocumentType, internal.RegionDocumentType, internal.CityDocumentType:
			locs.Set(doc.Key.AvitoId, *geo)
		case internal.DistrictDocumentType:
			distr.Set(doc.Key.AvitoId, *geo)
		default:
			return UnknownLocationType
		}
```



Когда необходимо указать несколько *case expression*'s для одного `StatementList`, необходимо их перечислять через запятую `,`, а не как в PHP ставить после каждого *case expression* двоеточие `:`:

```go
// Правильно
case errors.Is(err, form.ErrFormNotFound),
		errors.Is(err, form.ErrNoTransactions):
		return rpcprotocol.NewValidationError(err.Error())
```

```go
// Неправильно
case errors.Is(err, form.ErrFormNotFound):   // здесь как будто пустой StatementList
case errors.Is(err, form.ErrNoTransactions):
		return rpcprotocol.NewValidationError(err.Error())
```







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
---

- `ExpressionList` – уже объявленные переменные
- `IdentifierList` – новые переменные, идентификаторы

Итерирует *entry*'s для *array*, *slice*, *string* или *map* или значений, полученных по *channel*. 

`Expression`  может быть:

- array
- pointer to array
- slice
- string
- map
- *bidirectional* или *receive-only channel*

Для каждого элемента присваивает *iteration variable*'s (может быть несколько переменных в `IdentifierList`) некоторые *iteration value*'s, зависящие от типа `Expression` (см. таблицу ниже), а затем выполняет блок.


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

1. Для *array*, *pointer to array* и *slice*, выполняет итерации по индексам в порядке возрастания, начиная с индекса 0. Для `nil` *slice* количество итераций равно 0.
2. Для *string*, выполняет итерации по *Unicode code point*'s в *string*, начиная с байта с индексом 0. При последующих итерациях *index value* будет равно индексу первого байта последовательных *UTF-8-encoded code point*'s в *string* и второе *value* с типом `rune` будет значением соответствующей *code point*. 
3. Порядок итераций по *map* не определен и не гарантируется, что он будет одинаковым от одной итерации к другой. Если еще не достигнутая *map entry* будет удалена во время итерации, соответствующее *iteration value* не будет выдано. Если *map entry* создается во время итерации, эта запись может быть выдана во время итерации или может быть пропущена. Если *map* есть `nil`, количество итераций равно 0.
4. Для *channel*, выдаваемые *iteration value*'s являются последовательными значениями, отправляемыми по *channel* до тех пор, пока *channel* не будет *close*. Если *channel* есть `nil`, *range expression* блокируется навсегда.

`Expression` вычисляется один раз перед началом цикла, за одним исключением: если присутствует не более одной *iteration variable* и `len(Expression)` является [constant](https://go.dev/ref/spec#Length_and_capacity)), `Expression` вообще не вычисляется. Пример функции внутри `Expression`:

```go
func main() {
  gen := func() <-chan int { // функция-generator, аналог как в PHP генераторы
    dst := make(chan int)
    go func() {
      for n := 1; n < 5; n++ {
        dst <- n
      }
      close(dst)
    }()
    return dst
  }

  for n := range gen() { // функция gen() будет вызвана один раз перед началом цикла
    fmt.Println(n)
  }
}
```





*Iteration variable*'s могут быть объявлены с формы [short variable declaration](#short-variable-declarations) ( `:=`).  Их [scope](#declarations-and-scope) в этом случае - block для `for` *statement*; они используются повторно в каждой итерации. Если *iteration variable*'s объявлены вне `for` *statement*, после выполнения их значения будут такими же, как и на последней итерации.



```go
var testdata *struct {
	a *[7]int
}
for i, _ := range testdata.a {
	// testdata.a is never evaluated; len(testdata.a) is constant
	// i ranges from 0 to 6
	f(i)
}

var a [10]string
for i, s := range a {
	// type of i is int
	// type of s is string
	// s == a[i]
	g(i, s)
}

var key string
var val interface{}  // element type of m is assignable to val
m := map[string]int{"mon":0, "tue":1, "wed":2, "thu":3, "fri":4, "sat":5, "sun":6}
for key, val = range m {
	h(key, val)
}
// key == last map key encountered in iteration
// val == map[key]

var ch chan Work = producer()
for w := range ch {
	doWork(w)
}
```

Можно не указывать `ExpressionList` и `IdentifierList`:

```go
// очистить channel, receive все value's
for range ch {}

// tick каждую 1 секунду
for range time.Tick(time.Second) {
  // ..
}
```



Т.к. выполняется присваивание значения для *iteration variable*, то (как и все присваивания в Go) происходит копирование значения, присваивание *by value*.

Если необходимо изменить значение в исходном значении, то нужно обращаться к его элементам по индексам и изменять их:

```go
type MyType struct {
    field string
}

func main() {
    var array [10]MyType

    for idx, _ := range array {
        array[idx].field = "foo"
    }
}
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

## `return` *statement*

<pre>
ReturnStmt = "return" [ <a href="#constant-declaration">ExpressionList</a> ] .  
</pre>


В функции без [*result parameters*](#function-type) используется пустой `return`:

```go
func noResult() {
	return
}
```

Если у функции есть [*result parameters*](#function-type):

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

3. Если для [*result parameters*](#function-type) указаны имена, то можно использовать пустой `return`. `return` просто возвращает значения этих переменных.

   ```go
   func complexF3() (re float64, im float64) {
   	re = 7.0
   	im = 4.0
   	return
   }
   ```

*Result parameter*'ы инициализируются *zero value* при входе в функцию. `return` *statement* устанавливает значения *result parameter*'ов перед выполнением любых *deferred function*'s.







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



## `go` *statement*

[смотреть тут](goroutine.md#go-statement)



## `select` *statement*

[смотреть тут](types/channel.md#selecr-statement)



## `defer` *statement*

`defer` *statement* вызывает *function*,  выполнение которой откладывается (*defer*) до момента, когда окружающая *function* завершится (*return*). 

```
DeferStmt = "defer" Expression .
```

`Expression` должен быть *function call* или *method call*; его нельзя заключать в круглые скобки `()`. Вызов *built-in function* не разрешен.

Когда исполняется `defer` *statement*, *function value* (сама функция; *value* имеющее *function type*) и фактические *parameter*'ы вычисляют сразу (в порядке [order of evaluation](#order-of-evaluation)), и их значение сохраняется, но *function* сразу не вызывается. Вместо этого, *deferred function* вызывается перед тем моментом, когда окружающая *function* завершится (*return*).

*Deferred function*'s вызываются в порядке, обратном тому, в котором был сделан `defer`. 

```go
func main() {
	// печатает 3 2 1 0 перед тем как, окружающая function делает return
	for i := 0; i <= 3; i++ {
		defer fmt.Print(i)
	}
	
	return
}
```



Возможные варианты завершения:

- окружающая *function* выполнила оператор [`return` *statement*](#return-statement). *Deferred function*'s выполняются *после того, как* `return` *statement* установил все *result parameter*'s, но *до того, как* *function* вернется к своей *caller* (вызывающей функции).
- достигла конца своего *body*
- соответствующая *goroutine* бросает *panic*.

 Если *value* для *deferred function* равно `nil`, бросается *panic* во время вызова *function*, а не во время выполнения `defer` *statement*.

```go
func main() {

	var f func() // f == nil

	defer f()
  
  return // после  этого бросается panic
}
```

### Возврат значения из defer

Если *deferred function* является [*function literal*](#function-literal), а окружающая *function* имеет [именованные *result parameter*'s](#function-type) которые находятся в *scope* этого *function literal*, *deferred function* может получить доступ и изменить эти *result parameter*'s  до того, как они будут возвращены. 

```go
func f() (result int) {
	defer func() {
		// получаем доступ к переменной result после того, 
		// как ей будет установлено значение 1 в return statement
		result = 2
	}()

	return 1
}

func main() {
	fmt.Println(f()) // 2
}
```

А вот так, при этом, не работает. Т.к. *result parameter* – неименованный:

```go
func f() int {
  result := 1

  defer func() {
    // это присвоение ничего не дает,
    // т.к. result - это просто локальная переменная,
    // а не именованный параметр
    result = 2
  }()

  result = 1

  return result
}

func main() {
  fmt.Println(f()) // 1
}
```

 



Если *deferred function* имеет какие-либо *return value*'s, эти значения отбрасываются (не используются).

# Built-in functions

[смотреть тут](packages/builtin.md)

## Handling panics

Go не поддерживает *exception* и операции `throw` и `catch`. Вместо этого в Go предпочтительнее использовать явную обработку ошибок. 

Но в Go есть механизм аналогичный `throw/catch`, который называется `panic/recover`.

TODO!!!

https://go101.org/article/control-flows-more.html#panic-recover



### `panic()`

```go
func panic(v interface{})
```

- `v` – *error value*

`panic()` переводит текущую *goroutine* в *panicking status*. `panic()` немедленно останавливает нормальное выполнение текущей функции `F()`, функция *return* и входит в *exiting phase* (фаза завершения). Любые *function*'s, которые были *defer* функцией `F()` и за'*push*'ены в *defer-call stack*, запускаются в обратном порядке, затем функция `F()` делает возврат в вызывающую *function*. В вызывающей функции `G()`, вызов `F()` ведет себя как вызов *panic*, завершая выполнение `G()` и выполняя любые *defer* функции. И так далее до функции самого верхнего уровня в *goroutine*. 

```go
panic(42)
panic("unreachable")
panic(Error("cannot parse"))
```



Пока *panicking goroutine* не завершит работу, она никак не влияет другие *goroutine*'s. 

Последовательностью *panicking* можно управляться функцией [`recover()`](#recover).

Если *panicking goroutine* завершится без *recover*, это приведет к заверению всей *program* с *non-zero exit code*. Сообщается об ошибке, включая значение аргумента функции `panic()`. Пример завершения всей *program*:

```go
func main() {
	fmt.Println("hi!")

	go func() {
		time.Sleep(time.Second)
		panic(123)
	}()

	for {
		time.Sleep(time.Second)
	}
}

// Вывод:
// hi!
// panic: 123

// Process finished with exit code 2
```





### `recover()`

```go
func recover() interface{}
```

`recover()` позволяет программе управлять поведением *panicking goroutine*. 

Возвращаемое значение:

- *value* `v`, переданное при вызове `panic()`.
- `nil`:
  - если *value* `v`, переданное при вызове `panic()`, – `nil`. 
  - когда *goroutine* не *panicking*. Это позволяет понять – *goroutine* выполняет *panicking* или нет.
  - `recover()` не было вызван напрямую в *defer function* для *panicking function*.

Предположим, что у нас объявлено:

```go
func G() {
  defer func D()
  
  panic(/* ... */)
  
  // ...
}

func D() {
  recover()
}
```

В функции `G()` выбрасывается *panic*.  Любые *function*'s, которые были *defer* функцией `G()` и за'*push*'ены в *defer-call stack* (в том числе функция `D()`), запускаются в обратном порядке,. Вызов `recover()` внутри *defer function* `D()` (но не *function*, которую она вызывает) удаляет *panicking status*, останавливает последовательность *panicking*, восстанавливая нормальное состояние (за исключением случая, если в `D()` бросается новая *panic*). Это означает, что исполняются все *defer function*'s в `G()` и выполнение `G()` прерывается возвратом в вызвавшую функцию (здесь `main()`)

```go
func main() {
	fmt.Println("start main()")
	G()
	fmt.Println("end main()")
}


func G() {
	defer func() {
		recover()
	}()

	fmt.Println("start G()")
	D()
	fmt.Println("end G()")
}

func D() {
	fmt.Println("start D()")
	panic(123)
	fmt.Println("end D()")
}

// Вывод:
// start main()
// start G()
// start D()
// end main()
```

Если `recover()` вызывается вне *defer function*, это не остановит последовательность *panicking*.

Пример функции `protect()`, которая принимает функцию-аргумент `g()` и при необходимости останавливает последовательность *panicking*, восстанавливая нормальное выполнение:

```go
func protect(g func()) {
	defer func() {
		log.Println("done")  // Println исполняется как обычно, даже если брошена panic
		if x := recover(); x != nil {
			log.Printf("run time panic: %v", x)
		}
	}()
	log.Println("start")
	g()
}
```

### Run-time panic

Ошибки выполнения, такие как попытка обратится в *array* по индексу за пределами границ, вызывают *run-time panic*, эквивалентную вызову функции [`panic`](#panic) со значением, которое реализует *interface type* `runtime.Error`. Этот тип удовлетворяет *interface type* [`error`](packages/errors.md). 

```go
package runtime

type Error interface {
	error
	// and perhaps other methods
}
```

Пример *run-time panic* при деления на 0:

```go
func main() {
	a, b := 1, 0
	_ = a/b
}

// panic: runtime error: integer divide by zero
// Process finished with exit code 2
```



TODO!!!
https://go101.org/article/panic-and-recover-use-cases.html

https://go101.org/article/panic-and-recover-more.html



# Автоматическое приведение *pointer→value* и *value→pointer*

<u>*Index* и *slice expressions*</u>

- Если `pa` – *pointer* `a` типа, то:
  - для *index expression* можно писать сокращенно `pa[x]` вместо `(*pa)[x]`
  - для простого *slice expression* можно писать сокращенно `pa[<low> : <high>]` вместо `(*pa)[<low> : <high>]`
  - для полного *slice expression* можно писать сокращенно `pa[<low> : <high> : <max>]` вместо `(*pa)[<low> : <high> : <max>]`

<u>Field и methods</u>

Если `x` – *defined type*, возможны абсолютно любые комбинации вызовов среди:

- `t` – *value* и `p` – *pointer*
- `methodValue()` – метод принимающие *value reciever*, `methodPointer()` – метод принимающий *pointer reciever*
- `field` – поле.

Возможно это также связано с тем, что *method set* любого *pointer type*  `*T` – множество всех его *method*'s, объявленных с *receiver type* `*T` or `T`  (т.е. он также содержит в себе *method set* для *type* `T`) ([1](#method-set))

Поэтому в Go отсутствует оператор `->` из C и C++, т.к. Go самостоятельно автоматически приводит *pointer→value* и *value→pointer*

```go
type T struct {
	field int
}

func (T) methodValue() {}
func (*T) methodPointer() {}


func main() {
	t := T{}
	p := &T{}

	_ = t.field
	_ = p.field       // (*p).field
	t.methodValue()
	t.methodPointer() // (&t).methodPointer()
	p.methodValue()   // (*p).methodValue()
	p.methodPointer()
}
```





# Program initialization and execution

## Zero value

Когда для *variable* выделяется память, через:

- объявление *variable*
- вызов `new`
- когда создается новое значение (???)
- создание *composite literal*
- вызов `make`

и не обеспечивается явная инициализация, переменной присваивается *zero value* (нулевое значение, значение *by default*). Поэтому в языке
Go не может быть неинициализированных переменных.

Т.е. пользователь может *allocate* память и сразу использовать ее без явной инициализации. Поэтому многие встроенные типы (`bytes.Buffer`, `sync.Mutex`) не нужно дополнительно настраивать после *allocation*, они сразу настроены *by default*. 

*Zero value* зависит от типа:

- для `bool` – `false`

- для `numeric` – `0`

- для `string` – `""`

- для *pointer*, *function*, `interface`, `slice`, *channel* и *map* – `nil`

- для *array type* – он не может быть нулевого размера, т.к. размер указывается при его создании. Поэтому, если *array type* не инициализирован, каждому его элементу присваивается *zero value*:

  ```go
  var myIntArray [5]int     // инициализируется [0, 0, 0, 0, 0]
  ```

   

Для *composite type*'s эта инициализация выполняется рекурсивно, поэтому, например, поля каждого элемента `struct` внутри `array` будут установлены в *zero value*.

После

```go
type T struct { 
  i int; 
  f float64; 
  next *T 
}

var t T
```

Получаем

```
t.i == 0
t.f == 0.0
t.next == nil
```

Если нужно получать *variable*, инициализированную некоторым значением, можно вручную написать [функцию-конструктор](#конструктор) и вызывать ее явно.

## Package initialization

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

Подробнее про [`init()` *function*](init.go).



## Program execution

Программа – один неимпортируемый (*unimported*) *package*, называемый *main package*, связанный со всеми остальными *package*'s, которые он импортирует, транзитивно (импортирует через другие *package*'s). *Main package* должен называться `main` и в нем должны быть объявлена `main` *function*, которая не имеет *argument*'s и *return value*.

```go
func main () {…}
```

Исполнение программы начинается с инициализации *main package* и последующего вызова  `main` *function*. Когда  `main` *function* завершает исполнение, программа также завершается. Он не ждет завершения других (*non-main*) *goroutine*'s.



# Garbage collector

 Garbage collector (сборщик мусора) съедает в момент работы 1мс.

Mark & sweep. Зависимости помечаются серым, остается белый элемент, их удаляют

# Мое

## Function

К *function* относятся:

- *function type* ([link](#function-type))
- *function declaration* ([link](#function-declaration))
- *function literal* ([link](#function-literal))



Пример:

```go
// defined function type
type f func (a int) int

// присвоение с function literal
var a = func (a int) int { return a }

// Принимает в качестве параметра defined function type
func do(param f) {}

func main() {
	// передача в качестве параметра function literal
	do(a)
}
```

При этом невозможно описать *functional literal* с использованием *identifier*'а для *defined function type*:

```go
var a = f { return a } // ERROR!!!
```

Т.е. когда выполняется *variable declaration* для *function type* нужно польностью описать всю сигнатуру.

Это связано с тем, что и для *function declaration* ([link](#function-declaration)) и для *function literal* ([link](#function-literal)) нужно указывать `Signature` (и нельзя указывать *defined type*). Причина этого – *function type* не включает имена параметров, а определяет только их порядок и типы. Поэтому в  *function declaration* и *function literal* нужно описать *signature* целиком. Поэтому в *type identity* ([link](#type-identity)) указано, что две *function type*'s – *identical*, если у них одинаковое количество и одинаковые типы соответствующих *parameter*'s и *result value*'s, и либо обе *function*'s – *variadic*, либо обе – *non-variadic*. Имена для *parameter*'s и *result*'s могут не совпадать.



## Allocation

Есть следующие примитивы для *allocation*. Каждый из них применяется в определенных условиях.

- *built-in functions* `new` ([link](#new))
- *built-in functions* `make` ([link](#make))
- *variable declaration* ([link](#variable declaration))
- *composite literal* ([link](#composite-literal))



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







# Internal

## Pointer (internal)

Параметры в Go всегда передаются *by value*, но это *value* может представлять собой *pointer* (в принципе, как и в других языках. Например, в PHP тоже происходит передача *reference by value* , но по факту они все ссылаются на одно значение через *zreference*).

```go
	valueA := "a"
	valueB := "b"
	pointerA := &valueA
	println(pointerA)  // Исходный адрес: 0xc000044768

	pointerB := pointerA
	println(pointerB)  // Присвоили переменной тот же адрес: 0xc000044768

	pointerB = &valueB  // Присваиваем pointerB адрес другой переменной (valueB)

	println(pointerA)  // В pointerA остался тот же адрес: 0xc000044768
	println(valueA)  // В valueA осталось то же значение: "a"

	println(pointerB)  // В pointerB изменился адрес: 0xc000044758
	println(*pointerB) //  По pointerB хранится значение: "b"
```

## Передача by value VS Передача by pointer

https://medium.com/a-journey-with-go/go-should-i-use-a-pointer-instead-of-a-copy-of-my-struct-44b43b104963

## Копирование array и slice

https://stackoverflow.com/questions/51183003/arrays-in-go-are-by-value

# Отладка

Чтобы распечатать значение *pointer*'а и проверить на какой адрес указывает *pointer*, можно его просто распечатать через `println()`:

```go
valueA := "a"
pointerA := &valueA
println(pointerA)  // 0xc000044768
```







# Standard library

Фундаментальные типы данных в языке Go поддерживают привычные операторы (`+`, `-`,...), а стандартная библиотека Go добавляет дополнительные пакеты функций для работы с фундаментальными типами (например, `string`). 

При использовании в коде *package*, который логически включен в другой *package* (например, `path/filepath`) необходимо указывать только последний компонент их имени (например, `filepath`).

## `bytes`

*Package*, реализует функции для управления *byte slice*. Реализует возможности, аналогичные `package strings`.

###  `Contains()`

```go
func Contains(b, subslice []byte) bool
```

Функция, определяет содержится ли `subslice` внутри `b`.

```go
bytes.Contains([]byte("seafood"), []byte("foo"))
```





## `context`

https://www.youtube.com/watch?v=U5Y_St7lESk











## `io`

Обеспечивает базовый интерфейс для I/O примитивов.

### `ioutil`

Реализует I/O utility functions.

#### `ioutil.ReadAll()` 

```go
func ReadAll(r io.Reader) ([]byte, error)
```

Читает из `r` до *error* или *EOF* и возвращает прочитанные данные. При успехе возвращет `err == nil`. Если поток сразу начинается с EOF, никакой `error` не возвращается.

```go
  b, err := ioutil.ReadAll(r)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%s", b)
```

#### `ioutil.WriteFile()`

```go
func WriteFile(filename string, data []byte, perm os.FileMode) error
```

`WriteFile()` записывает `data` в файл с именем `filename`. Если файл не существует, `WriteFile()` создает его с разрешениями `perm` (перед `umask`); в противном случае `WriteFile()` *truncate* его перед записью без изменения разрешений.

```go
import (
	"io/ioutil"
	"log"
)

func main() {
	message := []byte("Hello, Gophers!")
	err := ioutil.WriteFile("testdata/hello", message, 0644)
	if err != nil {
		log.Fatal(err)
	}
}
```





## `net`

Предоставляет интерфейс для *network I/O*, включая *TCP/IP*, *UDP*, *domain name resolution* и *Unix domain sockets*.

### `http`

Предоставляет реализации HTTP client и HTTP server.

#### `type Client`

Представляет HTTP клиент. 

##### `Client.Do()`

```go
func (c *Client) Do(req *Request) (*Response, error)
```

Отправляет HTTP request и возвращает HTTP response, следуя политике (например, redirects, cookies, auth), настроенной на client.

Ошибка возвращается, если вызвана политикой *client*'а или неспособностью говорить по протоколу HTTP (например, проблемой сетевого подключения). Код состояния, отличный от 2xx, не вызывает ошибки.

Если возвращенная `error = nil`, ответ будет содержать *non-nil Body*, которое пользователь должен закрыть. Если Body не читается до EOF и не закрывается, базовый RoundTripper клиента (обычно Transport) может не иметь возможности повторно использовать persistent TCP connection к серверу для последующего *"keep-alive" request*.

Request Body, если оно *non-nil*, будет закрыто базовым транспортом даже при ошибках.

В случае ошибки любой Response можно игнорировать.





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





# Стандартные приемы

## `in_array()`

В Go 1.18+ можно:

- использовать уже готовую реализацию из экспериментального пакета `slices` (`golang.org/x/exp/slices`)

  ```go
  if slices.Contains(arr, "x") {
      // do something
  }
  ```

-  самому реализовать ее через дженерики (работает для любого *comparable type*) 

  ```go
  func Contains[T comparable](arr []T, x T) bool {
      for _, v := range arr {
          if v == x {
              return true
          }
      }
      return false
  }
  
  if Contains(arr, "x") {
      // do something
  }
  ```

  

До Go 1.18 не было такой встроенной функции для *array type* и *slice type*.

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


