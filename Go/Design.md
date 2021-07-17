https://www.youtube.com/watch?v=B0lV7I3FO4E

https://github.com/golang-standards/project-layout

https://developpaper.com/how-to-write-elegant-golang-code/7

Варианты дизайна:

- flat структура – все файлы лежат вместе с `main.go` в одной папке. Удобна для – небольшой библиотеки с небольшим количеством файлов.
- 

По репозиторию https://github.com/golang-standards/project-layout

## Директория `/cmd`

Является точкой входа для приложения



# Design Pattern

## Общее

Все *interface*'s в *package* собираются в один файл. Это позволяется сгененировать все *mock*'и за одну `//go:generate` команду:

```go
// interfaces.go

package params_validator

//go:generate mockery --name=InfomodelClient --name=GeoClient --case=underscore

type DocumentNames struct {}

type Metro interface {}

type GeoClient interface {}

type InfomodelClient interface {}
```



# Strategy

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

# Общая структура тестируемого модуля с внешней зависимостью

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
4. Для `ExternalService` выделить *interface* `Service` и включить в него только метод `DoSomething()`, который используется в `Handler.Handle()`.
5. Передавать `ExternalService` не *struct by pointer*, а *interface* *by value*

```go
package handler

type Handler interface {
	Handle() error
}

type handler struct {
	s Service
}

type Service interface {
	DoSomething(a int) error
}

func New(s Service) Handler {
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



# Go

Поинты:

- в Go интерфейсы вынесены из библиотеки, реализующей некоторый функционал. Т.к. описывать interface в классе не обязательно и есть *duck typing*. 

  Например, в библиотеке реализация тоглов:

  ```go
  type Toggles struct {
  	// ...
  }
  
  func (t *Toggles) GetBool(key string, defaultValue bool) bool {
    // ...
  }
  ```

   В подключающем коде:

  - описание интерфейса:

    ```go
    type Toggles interface {
    	GetBool(key string, defaultValue bool) bool
    }
    ```

  - описание своей mock-реализации:

    ```go
    type DummyToggles struct {
    	// ...
    }
    
    func (c *DummyToggles) GetBool(key string, defaultValue bool) bool {
    	// ...
    }
    ```

Преимущества:

можно описать именно такой интерфейс, который нужен в подключающем коде, исключить из интерфейса все ненужное.