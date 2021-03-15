https://github.com/golang/mock

https://pkg.go.dev/github.com/golang/mock/gomock

https://blog.codecentric.de/en/2017/08/gomock-tutorial/

https://golang.org/pkg/testing/

https://www.philosophicalhacker.com/post/integration-tests-in-go/



# `package testing`

## Общая схема

<u>Имя файла</u>

Имя файла с тестами должно иметь вид:

```
<name>_test.go
```

<u>Местоположения файла</u>

Файл должен быть в том же *package* (!!!), что и тестируемый файл. (В той же самой папке???). Этот файл будет исключен из обычных сборок пакета, но будет включен в сборку при запуске команды `go test`. 



<u>Имя функции</u>

В файле с тестами размещаются функции вида (где `<FuncName>` – начинается с заглавной буквы):

```go
func Test<FuncName>(t *testing.T) {
 	// ... 
}
```

<u>Структура функции с тестом</u>

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

В функции с тестом необходимо использовать методы `t.Error`, `t.Errorf`, `t.Fail`,..., чтобы оповестить об ошибке. В этом случае тест считается неудачным.

### Запуск тестов

https://golang.org/cmd/go/#hdr-Test_packages

Общий формат команды:

```
go test [build/test flags] [packages] [build/test flags & test binary flags]
```

Команда автоматизирует тестирование *package*'s, указанных с помощью *import path*'s. 

В обычном режиме выводится краткое описание:

```
$ go test
PASS
ok      example.com/greetings   0.364s
```

Для подробного вывода необходимо добавить флаг `-v`:

```bash
$ go test -v
=== RUN   TestHelloName
--- PASS: TestHelloName (0.00s)
=== RUN   TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
PASS
ok      example.com/greetings   0.372s
```





Команда может быть запущена в двух *mode*'s:

- *local directory mode*. Команда вызывается без `[packages]` в виде:

  ```bash
  go test
  ```

  Команда компилирует исходные коды *package*'s и *test*'s, найденные в текущем каталоге, а затем запускает полученный получившийся бинарник. 

  В этом режиме кэширование отключено.

- *package list mode* (режим списка пакетов). Включается, когда указан аргумент `[packages]`:

  ```bash
  go test [packages]
  go test github.com/user/stringutil
  go test math
  go test ./...  # ???Вложенный каталог
  go test .      # Текущий каталог
  ```

  

В этом режиме выполняется компиляция и тестирование всех перечисленных *package*'s. 

В этом режиме успешные результаты тестирования *package*'s кешируются, чтобы избежать ненужного повторного запуска тестов. 

  При этом будут запущены:

  - из package `<import_path>` 
  - из файлов вида `<name>_test.go`
  - каждая функция с сигнатурой вида `Test<FuncName>` 



# `package testing`

## `type T`

Тип, который передается в функцию `Test<FuncName>(t *testing.T)` для управления состоянием теста.

Тест заканчивается, когда его функция `Test<FuncName>(t *testing.T)`  вызывает один из следующих методов:

- `FailNow`
- `Fatal`
- `Fatalf`
- `SkipNow`
- `Skip`
- `Skipf`

Эти методы, а также метод `Parallel`, должны вызываться только из *goroutine*, выполняющей функцию `Test<FuncName>()`.

Другие методы (например, `Log`, `Error`), могут быть вызваны одновременно из нескольких *goroutine*'s.

```
type T struct {
     // contains filtered or unexported fields
}
```

### `Errorf()`

```
func (c *T) Errorf(format string, args ...interface{})
```

Функция, эквивалентна `Logf()`, за которым следует `Fail()`.

### `Fail()`

```
func (c *T) Fail()
```

`Fail()` отмечает, что произошла ошибка, но продолжает выполнение.

### `FailNow()`

```go
func (c *T) FailNow()
```

`FailNow()` отмечает, что произошла ошибка и останавливает выполнение функции, вызывая `runtime.Goexit` (который затем запускает все `defer` вызовы в текущей *goroutine*). Выполнение будет продолжено со следующего *test* или *benchmark*. `FailNow()` должен вызываться из *goroutine*, выполняющей *test* или *benchmark function*, а не из других *goroutine*'s, созданных во время теста. Вызов `FailNow()` не останавливает эти другие *goroutine*'s.

### `Fatalf()`

```go
func (c *T) Fatalf(format string, args ...interface{})
```

Функция, эквивалентна `Logf()`, за которым следует `FailNow()`.

### `Logf()`

```
func (c *T) Logf(format string, args ...interface{})
```

Функция форматирует `args` в соответствии с `format`, аналогично `Printf()`, и записывает текст в *error log*. Если в конце не указана *newline*, то она добавляется. 

- Для *test*, текст будет напечатан только в том случае, если тест терпит неудачу (*fail*) или установлен флаг `-test.v`. 
- Для *benchmark*, текст всегда печатается



# `gomock`

`gomock` – *mock framework* для Go.

 `mockgen` – инструмент генерации кода. Его использование не обязательно, но он позволяет автоматически сгенерировать *mock*'s и избежать ошибок. 

Особенности:

- имеет официальный статус в составе `github.com/golang`
- интегрирован со встроенным `testing` *package*,

## Подключение

- подключить `gomock`:

  ```bash
  go get github.com/golang/mock/gomock
  ```
  
  
    При этом в `go.mod` будет добавлено:

  ```
  require github.com/golang/mock v1.5.0
  ```

- установить `mockgen`:

  ```bash
  go get github.com/golang/mock/mockgen
  ```

  При этом `mockgen` будет установлен в папку `$GOPATH/bin/mockgen`. И может быть запущен:

  ```bash
  $ mockgen
  ```

## `mockgen`

Для генерации mock'а используется `mockgen`.

`mockgen` имеет два *mode*'s: 

- *source mode*
- *reflect mode*.

Результат при использовании *source mode* и *reflect mode* – одинаковый. 

### Source mode

В *source mode* происходит генерация *mock*'ов из указанного файла с кодом. Используется флаг `-source`.

```
mockgen -source=<source file> [other options]
mockgen -source=foo.go [other options]
```

### Reflect mode

В *reflect mode* для создания *mock*'ов используется *reflection*. С ее помощью анализируются *interface*'s в *package*. 

Для использования *reflection mode* необходимо передать два аргумента без флагов:

- *import path*. Можно использовать `.` чтобы указать *import path* для текущего *package* (удобно для `go:generate`)
- *interface*'s. В *reflect mode* нужно явно указать, для каких *interface*'s необходимо генерировать *mock*'s. Можно передать несколько несколько *interface*'s, перечисленных через запятую `,` (например, `Service1`, `Service2`).

```b
mockgen <import path> <interface's>

mockgen database/sql/driver Conn,Driver
mockgen . Conn,Driver
```

### Флаги

- `-source` – файл, содержащий *interface*'s, которые будут *mock*'аться
- `-destination` – файл, в который будет записан результирующий *source code*. Если этот флаг не указан, то *source code* будет выведен на *standard output*.
- `-package` – *package name*, в который будет помещен *source code* для *mock*'а. Если этот флаг не указан, то к *package name* исходного файла спереди будет присоединено `mock_` (`mock_<package_name>`).



### Структура файла с mock'ами `mock_XXX.go`

```go
// Code generated by MockGen. DO NOT EDIT.

// Файл, из которого был создан mock: 
// Source: service.go

// Package, который был указан при создании mock'а
package service

import (
	reflect "reflect"

	gomock "github.com/golang/mock/gomock"
)

// Сам mock для интерфейса Service
type MockService struct {
	ctrl     *gomock.Controller
	recorder *MockServiceMockRecorder
}

// Mock recorder (??? рекордер ожиданий mock'а), который позволяет описать expectation's
type MockServiceMockRecorder struct {
	mock *MockService
}

// Метод для создания нового mock instance (`MockService`)
func NewMockService(ctrl *gomock.Controller) *MockService {
	mock := &MockService{ctrl: ctrl}
	mock.recorder = &MockServiceMockRecorder{mock}
	return mock
}

// Метод EXPECT – единственный лишний метод, который добавляется к mock'у сервиса (`MockService`).
// Т.е такого метода нет в исходном interface. 
// Поэтому, вероятно, чтобы имя метода (expect) не конфликтовало с именами в исходном interface, 
// его записали прописными буквами (EXPECT).
// Вызывается на mock'е и возвращает object типа MockServiceMockRecorder, 
// который позволяет описать expectation's
func (m *MockService) EXPECT() *MockServiceMockRecorder {
	return m.recorder
}

// Ниже методы mock'а, которые реализуют методы исходного interface
// Т.е. у исходного interface есть единственный метод `Set(int) error`
func (m *MockService) Set(arg0 int) error {
	m.ctrl.T.Helper()
	ret := m.ctrl.Call(m, "Set", arg0)
	ret0, _ := ret[0].(error)
	return ret0
}

// Ниже методы, которые установлены на рекордере ожиданий mock'а
// Методы позволяют описать expectation's для методов исходного interface
// Эти методы - одноименные с методами mock'а (и исходного interface). 
// Т.е. у исходного interface есть единственный метод `Set()`
func (mr *MockServiceMockRecorder) Set(arg0 interface{}) *gomock.Call {
	mr.mock.ctrl.T.Helper()
	return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "Set", reflect.TypeOf((*MockService)(nil).Set), arg0)
}

```

Есть три класса:

- `MockService` – *mock* исходного *interface*
- `MockServiceMockRecorder` – рекордер ожиданий *mock*'а. Позволяет описать *expectation*'s для *mock*'а. Для этого на *recoreder*'е определены методы, одноименные методам *mock*'а (и исходного *interface*) (например, `Set()`) 

- `gomock.Controller` – *mock controller*, он отвечает за отслеживание и утверждение *expectation*'s, связанных с ним *mock*'ов.





## Пример

Возьмем для примера два файла:

- *interface* для сервиса `Service` в файле `service/service.go`:

  ```go
  package service
  
  type Service interface {
  	Set(int) error
  }
  ```

- *struct* для обработчика `Handler`, который в методе `Handle()` использует *interface* `Service`. Находится в файле `handler/handler.go`:

  ```go
  package handler
  
  import "github.com/parshikovpavel/hello/service"
  
  type Handler struct {
  	Service service.Service
  }
  
  func (h *Handler) Handle() error {
  	return h.Service.Set(123)
  }
  ```



Итак *module* будет иметь структуру:

```
'-- service
    '-- service.go
    '-- mock_service.go
'-- handler
    '-- handler.go
    '-- handler_test.go
```

(1) Разместим *mock* для *interface* `Service`  в том же самом `package service` в файле `service/mock_service.go`. 

Для генерации *mock*'а нужно выполнить:

- для генерации в *source mode* ([link](#source-mode)):

  ```bash
  mockgen -package service -destination mock_service.go -source service.go
  ```

- для генерации в *reflect mode* ([link](#reflect-mode)):

  ```bash
  mockgen -package service -destination mock_service.go github.com/parshikovpavel/hello/service Service
  ```

(2) Размеcтим тест для `Handler` в файле `handler/handler_test.go`.

```go
package handler_test

import (
  "github.com/sgreben/testing-with-gomock/mocks"
  "github.com/sgreben/testing-with-gomock/user"
)

func TestUse(t *testing.T) {
    mockCtrl := gomock.NewController(t)
    defer mockCtrl.Finish()

    mockDoer := mocks.NewMockDoer(mockCtrl)
    testUser := &user.User{Doer:mockDoer}

    // Expect Do to be called once with 123 and "Hello GoMock" as parameters, and return nil from the mocked call.
    mockDoer.EXPECT().DoSomething(123, "Hello GoMock").Return(nil).Times(1)

    testUser.Use()
}
```











# Запуск в GoLand

???

GoLand позволяет запустить под отладкой тест

кроме того в makefile есть цель tests или test

