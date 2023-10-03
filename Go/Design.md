https://www.youtube.com/watch?v=B0lV7I3FO4E

https://github.com/golang-standards/project-layout

https://developpaper.com/how-to-write-elegant-golang-code/7

https://zhashkevych.medium.com/%D1%87%D0%B8%D1%81%D1%82%D0%B0%D1%8F-%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0-%D0%BD%D0%B0-golang-cccbfdc95eba

https://www.zhashkevych.com/clean-architecture

Варианты дизайна:

- flat структура – все файлы лежат вместе с `main.go` в одной папке. Удобна для – небольшой библиотеки с небольшим количеством файлов.
- 

По репозиторию https://github.com/golang-standards/project-layout

Пишут, что тут хорошо расписано про основные паттерны https://threedots.tech/post/common-anti-patterns-in-go-web-applications/

https://threedots.tech/post/introducing-clean-architecture/



Чистая архитектура Р. Мартин

# Архитектура проекта

https://www.youtube.com/watch?v=B0lV7I3FO4E&t=5s

## Flat

Все файлы, включая `main.go` лежат в одной папке и соответственно в одном `main` *package*.

Подходит:

- для небольших библиотек из нескольких файлов 

## Стандартная схема

https://github.com/golang-standards/project-layout



На верхнем уровне три директории:

- `/cmd`
- `/internal`
- `/pkg`



## Директория `/cmd`

Является точкой входа для приложения.

Эта директория хранит `main` *package*'s с `main()` *function*'s. На основании этих файлов собираются *binary*, которые будут использоваться для запуска *application*. Эти `main.go` файлы, как правило, небольшие.

Внутри `/cmd` директории часто делают еще поддиректории. Это необходимо, если проект состоит из нескольких приложений, использующих общую логику.Например, веб-приложение, которое состоит из:

- приложения на базе REST API
- крона
- воркера





# Design Pattern

https://github.com/tmrts/go-patterns

## Strategy

Интерфейс стретегии:

```go
type SyncStrategy interface {
	SyncData(ctx context.Context, cb func(ctx context.Context) error) error
}
```

Реализация интерфейса стратегии:

```go
type simpleSync struct {
}

func NewSimpleSyncStrategy() *simpleSync {
	return &simpleSync{}
}

// Просто синхронизируем словари при запуске сервиса и все
func (ts *simpleSync) SyncData(ctx context.Context, cb func(context.Context) error) error {
	err := cb(ctx)

	return err
}
```

Использование стратегии:

```go
type validator struct {}

func NewParamsValidator(
	ctx context.Context,
  geoSyncStrategy SyncStrategy,
) (*validator, error) {
	v := &validator{}

  err := geoSyncStrategy.SyncData(ctx, v.getGeoData)

  return v, nil
}

func (v *validator) getGeoData(ctx context.Context) error {
  ...
  return nil
}
```

## Функция с переменным числом аргументов

https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis

https://golang.cafe/blog/golang-functional-options-pattern.html

https://www.sohamkamani.com/golang/options-pattern/

https://medium.com/german-gorelkin/go-functional-options-727ec78e4a8f

Допустим, нам необходимо реализовать *package*, который описывает клиент для микросервиса

```go
package client

// Структура клиента сервиса
type Client struct {
	url        string
}

// Конструктор клиента
func New(url string) *Client {
	return &Client{url: url}
}
```

По мере развития проекта возникает проблема – как добавить дополнительные параметры в конструктор, чтобы настроить *Client*. 

Так как в Go отсутствуют опциональные аргументы со значением по умолчанию (как в PHP), мы не можем просто добавить дополнительные аргументы со значением по умолчанию. Т.к. у нас поломается код во всех точках, где `Client` использовался ранее.

### Добавляем новые отдельные функции-конструкторы

Для каждого сочетания возможных входных аргументов – пишем отдельную функцию-конструктор со своим именем. Например:

```go
package client

// Структура клиента сервиса
type Client struct {
	url        string
  timeout time.Duration
  maxIdleConns int
}

func NewWithTimeout(url string, timeout time.Duration) *Client {
   return &client{
      url: url,
      timeout: timeout,
   }
}

func NewWithTimeoutAndMaxIdleConns(url string, timeout time.Duration, maxIdleConns int) *Client {
  return &client{
     url: url,
     timeout: timeout,
     maxIdleConns: maxIdleConns
  }
}
```

Недостатки:

- негибко
- громоздко

### Используем аргумент-структуру `Config`

Создаем новый *exported type* `Config`, который будет содержать все возможные настройки для `Client`. Тип `Config`  можно без проблем расширять, не нарушая API конструктора `New()`.

```go
package client

// Структура клиента сервиса
type Client struct {
	url        string
  timeout time.Duration
  maxIdleConns int
}

// Структура конфига клиента
type Config struct {
  timeout time.Duration
  maxIdleConns int
}

func New(url string, config Config) *Client {
   return &client{
      url: url,
      timeout: config.timeout,
      maxIdleConns: config.maxIdleConns,
   }
}
```

Преимущество:

- более гибко, можно просто расширять `Config` не меняя контракт `New()`

Недостатки:

- https://medium.com/german-gorelkin/go-functional-options-727ec78e4a8f



### Functional options









## DI

Внедрение зависимости выполняется в фабричном методе (конструкторе, роль которого выполняет функция `New()`). 

Правила:

- Зависимость передается в `New()`  через *interface*
- `return &object{...}`
- возвращается созданный объект также через *interface*

Пример в разделе [Структура тестируемого модуля с внешней зависимостью]()





### Описание interface's внешних библиотек

в Go интерфейсы вынесены из библиотеки, реализующей некоторый функционал. Т.к. описывать *interface* вместе с классом не обязательно и есть *duck typing*. 

Например, в библиотеке реализация тоглов:

```go
package toggle

type Toggles interface {
  GetBool(key string, defaultValue bool) bool
  GetPercent()
}

type toggles struct {
	// ...
}

func (t *toggles) GetBool(key string, defaultValue bool) bool {
  // ...
}
```

 В подключающем *package*:

- Явно описывается *interface* для внешней зависимости. При этом можно описать именно такой *inteface*, который нужен в подключающем коде, исключить из *inteface* все ненужное.

  Все *interface*'s в *package* собираются в один файл `contract.go`. Это позволяется сгененировать все *mock*'и за одну `//go:generate` команду:

  ```go
  // Имя файла – contract.go
  
  //go:generate mockery --name=InfomodelClient --name=GeoClient --case=underscore
  package handler
  
  type Toggles interface {
  	GetBool(key string, defaultValue bool) bool
  }
  
  type GeoClient interface {
    getService()
  }
  ```

- В том случае, если необходимо целиком использовать *interface* внешней зависимости (т.е. явно описать *interface*, который включает в себя все методы внешнего *interface*), то нужно использовать *embedded interface* ([link](lang.go#embedded-interface)):

  ```go
  package handler
  
  import (
  	schema ".../toggles"
  )
  
  type Toggles interface {
  	schema.Toggles
  }
  ```

  

- для тестов генерируем *mock*'и зависимостей:

  ```go
  package handler_test
  
  // MockLog is a mock of Log interface.
  type MockToggles struct {
  	ctrl     *gomock.Controller
  	recorder *MockLogMockRecorder
  }
  
  //...
  ```

  можно описать mock'и вручную:

  ```go
  package handler_test
  
  type DummyToggles struct {
  	// ...
  }
  
  func (c *DummyToggles) GetBool(key string, defaultValue bool) bool {
  	// ...
  }
  ```

  


Преимущества:

- 

### Размещение interface's зависимостей







### Cтруктура тестируемого модуля с внешней зависимостью

Допустим у нас есть *struct* для обработчика `Handler`, который в методе `Handle()` вызывает метод `DoSomething()` у `ExternalService` (напрямую, или через интерфейс):

```go
package handler

import "github.com/foo/external"

type Handler struct {
	s *external.ExternalService
}

func New(s *external.ExternalService) *Handler {
	return &Handler{s}
}

func (h *Handler) Handle() error {
	return h.s.DoSomething(123)
}
```

Чтобы сделать такой модуль тестируемым нужно:

1. Вместо *exported struct* `Handler`  сделать *non-exported struct* `handler`

2. Для этой *struct* выделить *interface* `Handler` (DIP для `Handler`)

3. Возвращать из конструктора (`New()`) не *struct by pointer*, а *interface* *by value*

4. Для `ExternalService` выделить *interface* ` ExternalService` НО!! в текущем *package* и включить в него только метод `DoSomething()`, который используется в `Handler.Handle()`. Файл с *interface* называем `contract.go`. Тут же размещаем `//go:generate` для генерации mock'ов.

   В файле `contract.go` размещаем ВСЕ!!! *interface*'s к внешним зависимостям. Это позволяет их держать вместе и генерировать все *mock*'и за одну команду ([смотри ниже](#описание-interfaces-внешних-библиотек)).

   <u>contract.go</u>

   ```go
   //go:generate mockgen -source $GOFILE -destination=mock_contract_test.go -package=handler_test
   
   package handler
   
   type Service interface {
   	DoSomething(a int) error
   }
   
   ```

   

5. Передавать `ExternalService` не *struct by pointer*, а *interface* *by value*

```go
package handler

type Handler interface {
	Handle() error
}

type handler struct {
	s ExternalService
}


func New(s ExternalService) Handler {
	return &handler{s}
}

func (h *handler) Handle() error {
	return h.s.DoSomething(123)
}
```

Преимущества:

- Полный DIP (через *interface*'s)
- Легко создать *mock* для `Service`. 
- *interface* `Service` описан в текущем *package* и мы вообще нигде никак не завязываемся на `external.externalService`. А зависим через наш *interface*, в котором описаны только нужные нам методы (используем преимущества *duck typing*).



Если какая-то структура используется только в тестах, то поступают следующим образом:

- создают в package с исходным кодом (не тестами) файл `contract.go`, где описывают интерфейсы ко всем зависимостям-структурам. К этим же интерфейсам привязывают структуры в этом *package*.

# Code style

## panic

Можно выбрасывать `panic()` только на этапе инициализации приложения/воркера. Хотя можно и просто делать `return` из `main()`. Если приложение действительно не может настроить себя, то имеет смысл.

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```



В других местах кода выбрасывать `panic()`  нельзя.  Если проблему можно замаскировать или обойти, всегда лучше позволить чему-то работать, а не останавливать всю программу.

- [effective-go#panic](https://golang.org/doc/effective_go.html#panic)

# Preemptive Interface

*Preemptive Interface* (превентивный, упреждающий интерфейс) – *interface*, который написан до того, как возникает реальная потребность в его использовании.

Пример *preemptive Interface*, который идет в комплекте с типом, который его реализует и возвращает *interface* в конструкторе:

```go
type Auth interface {
  GetUser() (User, error)
}
type authImpl struct {
  // ...
}
func NewAuth() Auth {
  return &authImpl
}
```

Для языков без *duck typing* (вроде PHP, Java) использование *preemptive Interface* – стандартная практика, т.к. реализация *interface* явно прописывается в определении класса. Если класс изначально не реализует нужный *interface*, то это потом, возможно, будет сложно обеспечить – класс может лежать во внешней библиотеке, он может быть недоступен для изменения, его могут использовать другие разработчики. 



## Preemptive interface и mock

[смотри](Design.md#Cтруктура-тестируемого-модуля-с-внешней-зависимостью) 





https://medium.com/@cep21/preemptive-interface-anti-pattern-in-go-54c18ac0668a

https://medium.com/@cep21/what-accept-interfaces-return-structs-means-in-go-2fe879e25ee8

https://mycodesmells.com/post/accept-interfaces-return-struct-in-go







# Короткие *name* переменных

*Variable name*'s в Go должны быть короткими, а не длинными. 

Правило большого пальца (rule of thumb) – чем больше расстояние между *name's declaration* и его использованием, тем более длинным и информативным должно быть *name*. Информативность *variable name* должна быть напрямую связана с размером *scope*, в которой это *name* может использоваться. 

Для *local variable* с ограниченным *scope* необходимо использовать короткое *name*. *Name* `i` передает столько же информации, сколько `index` или `idx` и его легче читать. Точно так же `i` и `j` – лучшая пара *name*'s для *index variable*'s, чем `i1` и `i2` (или, что еще хуже, `index1` и `index2`), потому что их легче отличить при беглом просмотре программы.  Для *method receiver* достаточно одной или двух букв. 

*Global variable name*'s и *field* из широко используемой *struct* должны передавать относительно больше информации, потому что они появляются в большем разнообразии контекстов. Даже в этом случае короткое точное *name* может сказать больше, чем длинное: например, сравните `acquire` и `take_ownership`. Сделайте так, чтобы каждое *name* было говорящим ([Russ Cox](https://research.swtch.com/names))



# Имена файлов из нескольких слов

Имена файлов, package из нескольких слов разделяются подчеркиваниями и пишутся строчными буквами:

```
service-abc/internal/rpc/create_hello_world
```



# Именование interface и struct, которая его реализует

Правила:

- Для *interface* используется имя с заглавной буквы (*exported*)
- Для *struct* используется имя со строчной буквы (*non-exported*)

Используется структура через *interface*

Пример:

```go
type Service interface {
	// ...
}

type service struct {
	// ...
}

func New() *service) {
	return &service{
    //...
  }
}
```







TODO!!!

















https://www.reddit.com/r/golang/comments/8wxwgv/why_does_go_encourage_short_variable_names/

https://talks.golang.org/2014/names.slide#1

https://research.swtch.com/names

https://yandex.ru/search/?text=%D0%B7%D0%B0%D1%80%D0%BF%D0%BB%D0%B0%D1%82%D0%B0+net&&lr=191