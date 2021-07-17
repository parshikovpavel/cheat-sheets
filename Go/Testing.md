https://github.com/golang/mock

https://pkg.go.dev/github.com/golang/mock/gomock

https://golang.org/pkg/testing/

https://www.philosophicalhacker.com/post/integration-tests-in-go/

https://github.com/stretchr/testify#assert-package

https://pkg.go.dev/github.com/stretchr/testify/assert#Assertions





# `package testing`

## Общая схема

### Имя файла

Имя файла с тестами должно иметь вид:

```
<name>_test.go
```

### Местоположения тестов

[link](https://stackoverflow.com/questions/19998250/proper-package-naming-for-testing-with-the-go-language)

Существуют следующие стратегии размещения тестов. Они отличаются тем, находится ли тестовый код в том же *package*, что и тестируемый код, или нет

**Стратегия 1. Тестирование белого ящика**

Имя файла: github.com/user/myfunc.go

```golang
package myfunc
```

Имя тестового файла: github.com/user/myfunc_test.go

```golang
package myfunc
```

При этом есть доступ к non-exported identifier's и их можно использовать в тесте. 

**Стратегия 2. Тестирование черного ящика**

Имя файла: github.com/user/myfunc.go

```golang
package myfunc
```

Имя тестового файла: github.com/user/myfunc_test.go

```golang
package myfunc_test

import (
    "github.com/user/myfunc"
)
```

В тесте можно использовать только *exported identifier*'s.

Эта стратегия ДОЛЖНА удовлетворять правилам

1. Имена package's:
   - для основного package – `NameOfDirectory`.
   - для тестового package – `NameOfDirectory` + `_test`.
2. Имена файлов в `_test` *package* заканчиваются на`_test.go`

Это более предпочтительная стратегия, т.к. тестировать нужно интерфейс пакета, т.е. его *exported identifiers* и не тестировать его внутренности. 

Тестовые файлы с `package XXX_test` будут скомпилированы как отдельный *package*, а затем связаны и запущены с основным кодом.

Файл `XXX_test.go` будет исключен из обычных сборок пакета, но будет включен в сборку при запуске команды `go test`. 



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
  go test ./...  # Вложенный каталог (см. сommandGo.go#relative-import-path)
  go test .      # Текущий каталог
  ```

  

В этом режиме выполняется компиляция и тестирование всех перечисленных *package*'s. 

В этом режиме успешные результаты тестирования *package*'s кешируются, чтобы избежать ненужного повторного запуска тестов. 

При этом будут запущены:

  - из package `<import_path>` 
  - из файлов вида `<name>_test.go`
  - каждая функция с сигнатурой вида `Test<FuncName>` 

Необходимо размещать 



# `package testing`

## `type T` и `type B`

### `Run()`

```go
func (t *T) Run(name string, f func(t *T)) bool
func (b *B) Run(name string, f func(b *B)) bool
```

`Run()` запускает `f()` как *subtest* для `t` с именем `name`. Он запускает `f()` в отдельной *goroutine* и блокируется (т.е. по умолчанию тесты не выполняются параллельно) до тех пор, пока `f()` не вернется или не вызовет `t.Parallel()`, чтобы стать *parallel test*. Тесты по умолчанию выполняются последовательно, для параллельного выполнения необходимо использовать `t.Parallel()`.

`Run()` позволяет создавать *subtest*'s и *sub-benchmark*'s без необходимости определять отдельные функции для каждого. Это позволяет использовать *table-driven benchmarks* и создавать *hierarchical tests*. Он также предоставляет возможность поделиться общим кодом *setup* и *tear-down*:

```go
func TestFoo(t *testing.T) {
    // <setup code>
    t.Run("A=1", func(t *testing.T) { ... })
    t.Run("A=2", func(t *testing.T) { ... })
    t.Run("B=1", func(t *testing.T) { ... })
    // <tear-down code>
}
```

```go
func TestParse_Success(t *testing.T) {
	cases := []struct {
		name       string
		expected   int
	}{
		{"grid", 1},
    {"list", 2},
	}

	for _, tc := range cases {
		test := tc
		t.Run(test.name, func(t *testing.T) {
			// ...
			require.Equal(t, test.expected, res)
		})
	}
}
```

Каждый *subtest* и *sub-benchmark* имеет уникальное *name*: оно получается как комбинация *name* теста самого верхнего уровня и последовательности *name*'s, переданных в `Run()`, разделенных *slash*'s (`/`), с необязательным завершающим порядковым номером (если несколько тестов с одним именем).





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

`gomock` – это:

- *package*
- реализует *mock framework* для Go.

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

### Местоположение mock'ов

Встречал 3 стратегии:

- файлы с моками размещаются в отдельном каталоге  `mocks` и в `package mocks`. Примеры [1](https://blog.codecentric.de/en/2017/08/gomock-tutorial/) и [2](https://medium.com/@benbjohnson/standard-package-layout-7cdbc8391fc1). 

  Наиболее распространненый вариант, описывается во всех статьях. Позволяет переиспользовать mock'и из любых других *package*'s.

- файл с моками размещается в том же *package*, что и исходный *interface*. В этом случае и тесты также размещаются в том же *package* ([стратегия 1](#местоположение-тестов)). Пример: service-search.

  Наиболее простой вариант, все в одном *package*. стратегия белого ящика.

- файл с моками размещается вместе с тестами в отдельном `package XXX_test`  ([стратегия 2](#местоположение-тестов)). В конце имени файлов с *mock*'ами нужно писать `_test` – `mock_XXX_test.go` (т.к. именование файлов должно удовлетворять правилам из [стратегия 2](#местоположение-тестов)) Пример: service-search-filters. 

  Стратегия черного ящика.

### Режимы `mockgen`

`mockgen` имеет два *mode*'s: 

- *source mode*
- *reflect mode*.

Результат при использовании *source mode* и *reflect mode* – одинаковый. 

#### Source mode

В *source mode* происходит генерация *mock*'ов для всех *interface*'s (!!!) из указанного файла с кодом. Используется флаг `-source`.

```
mockgen -source=<source file> [other options]
mockgen -source=foo.go [other options]
```

#### Reflect mode

В *reflect mode* для создания *mock*'ов используется *reflection*. С ее помощью анализируются *interface*'s в *package*. 

Для использования *reflection mode* необходимо передать два аргумента без флагов:

- *import path*. Можно использовать `.` чтобы указать *import path* для текущего *package* (удобно для `go:generate`)
- *interface*'s. В *reflect mode* нужно явно указать, для каких *interface*'s необходимо генерировать *mock*'s. Можно передать несколько несколько *interface*'s, перечисленных через запятую `,` (например, `Service1,Service2`).

```b
mockgen <import path> <interface's>

mockgen database/sql/driver Conn,Driver
mockgen . Conn,Driver
```

#### Флаги

- `-source` – файл, содержащий *interface*'s, которые будут *mock*'аться
- `-destination` – файл, в который будет записан результирующий *source code*. Если этот флаг не указан, то *source code* будет выведен на *standard output*.
- `-package` – *package name*, в который будет помещен *source code* для *mock*'а. Если этот флаг не указан, то к *package name* исходного файла спереди будет присоединено `mock_` (`mock_<package_name>`).
- `-mock_names`: Список кастомных имен для сгенерированных *mock*'ов. Указывается в виде списка разделенных запятыми `,` элементов, например  `Repository=MockSensorRepository,Endpoint=MockSensorEndpoint`, где `Repository` – имя интерфейса и `MockSensorRepository` – желаемое имя mock'а. Если у какого-то интерфейса не указано кастомное имя, то будет использоваться соглашение об именах по умолчанию.







#### Структура файла с mock'ами `mock_XXX.go`

```go
// Code generated by MockGen. DO NOT EDIT.

// Файл, из которого был создан mock: 
// Source: service.go

// Package, который был указан при создании mock'а
package service_test

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
// Т.е. у исходного interface есть единственный метод `DoSomething(int) error`
func (m *MockService) DoSomething(arg0 int) error {
	m.ctrl.T.Helper()
	ret := m.ctrl.Call(m, "DoSomething", arg0)
	ret0, _ := ret[0].(error)
	return ret0
}

// Ниже методы, которые установлены на рекордере ожиданий mock'а
// Эти методы - одноименные с методами mock'а (и исходного interface). 
// Т.е. у исходного interface есть единственный метод `DoSomething()`
// У этих методов отличается тип аргументов на interface{}. В данном случае вместо int - interface{} 
// (наверняка, т.к. методам также можно передавать matcher)
// Методы возвращают объект gomock.Call, относящийся к методу исходного interface 
// Объект gomock.Call позволяет описать ожидаемый вызов к mock'у (parameter values, return value) 
func (mr *MockServiceMockRecorder) DoSomething(arg0 interface{}) *gomock.Call {
	mr.mock.ctrl.T.Helper()
	return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "DoSomething", reflect.TypeOf((*MockService)(nil).DoSomething), arg0)
}

```

Есть следующие классы:

- `MockService` – *mock* исходного *interface*
- `MockServiceMockRecorder` – рекордер ожиданий *mock*'а. Позволяет описать *expectation*'s для *mock*'а. Для этого на *recoreder*'е определены методы, одноименные методам *mock*'а (и исходного *interface*) (например, `DoSomething()`) 
- `gomock.Controller` – *mock controller*, он отвечает за отслеживание и утверждение *expectation*'s, связанных с ним *mock*'ов.
- `gomock.Call` – этот объект относится к некоторому методу исходного *interface*. Позволяет описать ожидаемый вызов к *mock*'у (*parameter values, return value*) ([link](#gomockcall))





## Пример

### Структура файлов

Возьмем для примера два файла:

- *interface* для сервиса `Service` в файле `service/service.go`:

  ```go
  package service
  
  type Service interface {
  	DoSomething(int) error
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
  	return h.Service.DoSomething(123)
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

[Общая структура тестируемого модуля с внешней зависимостью](Design.md#общая-структура-тестируемого-модуля-с-внешней-зависимостью)



### Генерация *mock*'а

Разместим *mock* для *interface* `Service`  в том же самом `package service` в файле `service/mock_service.go`. 

Для генерации *mock*'а нужно выполнить.

#### Ручная генерация

- для генерации в *source mode* ([link](#source-mode)):

  ```bash
  mockgen -package service_test -destination mock_service.go -source service.go
  ```

- для генерации в *reflect mode* ([link](#reflect-mode)):

  ```bash
  mockgen -package service_test -destination mock_service.go github.com/parshikovpavel/hello/service Service
  ```

#### Автоматическая генерация через `go generate`

Вручную запускать команду `mockgen` для каждого *interface* (или *file*) слишком неудобно. Поэтому запуск `mockgen` можно автомазировать с помощью [`go generate`](Команда go#go-generate) .

Общие правила построения `//go:generate mockgen`:

- В файле, где находятся интерфейсы, которые нужно замокать, размещается только одна команда. 

- Команда  может находиться в любом месте `.go` файла. Но ее чаще всего размещают на первой строке или после строки с `package`.

- Команда генерирует *mock*'и для всех *interface*'s в файле ([source mode](#source-mode)). Проще использовать *source mode*.

- *Mock* для *interface* размещаются в том же самом *package* и в той же самой directory. *Mock*'и для файла `xxx.go` помещаются в файл `mock_xxx.go`.

- Можно генерировать *mock*'и для любого файла, не обязательно для того, где размещен `//go:generate`. Например, можно сгенерировать *mock*'и для библиотечных *interface*'s.

  ```go
  //go:generate mockgen -destination mock_metrics_test.go -package $GOPACKAGE -mock_names Client=MockMetrics go.xxx.ru/gl/metrics Client
  ```

Пример файл `service/service.go`:

```go
//go:generate mockgen -package ${GOPACKAGE} -destination mock_service.go -source $GOFILE
package service

type Service interface {
	DoSomething(int) error
}
```

*Current working directory* для параметров команды `mockgen` – это *directory*, где находится файл с `//go generate`.

Для того чтобы сгенерировать все *mock*'и в проекте, необходимо из *project’s root directory* выполнить команду ([link](Команда go.md#go-generate)):

```
go generate ./...
```

### Написание теста

Размеcтим тест для `Handler` в файле `handler/handler_test.go`.

```go
package handler_test

import (
	"github.com/golang/mock/gomock"
	"github.com/parshikovpavel/hello/handler"
	"github.com/parshikovpavel/hello/service"

	"testing"
)

func TestUse(t *testing.T) {
	ctrl := gomock.NewController(t) // Получение mock controller'а
	defer ctrl.Finish()             // Не обязательно для Go 1.4+

	mockService := service.NewMockService(ctrl) // Создание mock'а для interface Service

  // Вызов mockService.EXPECT() для настройки expectation's (подробнее в листинге mock'а)
	// Ожидает, что метод DoSomething будет вызван один раз со значением 123, вернет из метода nil
	mockService.EXPECT().DoSomething(123).Return(nil).Times(1)

	// Создаем экземпляр реального handler'а и инджектим в него mockService
	testHandler := handler.Handler{
		Service: mockService,
	}

	// Тестовый вызов handler'а
	testHandler.Handle()
}
```

#### Порядок описания *expectation*

```go
	mockService.EXPECT().DoSomething(123).Return(nil).Times(1)
```

- нужно вызвать метод `EXPECT()`, который возвращает *recorder* ожиданий. 
- На *recoreder*'е определены методы, одноименные методам *mock*'а (и исходного *interface*) (например, `DoSomething()`). Аргументы этих методов описывает  аргументы для ожидаемого вызова этого метода. В качестве аргумента можно использовать:
  - конкретное значение
  - *matcher*.
- Эти методы возвращают объект с типом  `type Call` ([link](#call)). Уже на этом объекте можно описывать ожидания. Возможные варианты:

  - количество вызовов:
    - `MaxTimes()`
    - `MinTimes()`
    - `Times()`
  - Возвращаемое значение
    - `Return()`

#### Matcher на аргументы и возвращаемое значение для метода

##### Matcher на один аргумент

В качестве аргументов при описании ожиданий можно использовать:

- конкретное значение

  ```go
  mockDoer.EXPECT().DoSomething(123)
  ```

- *matcher* (`type Matcher`, [link](#type-matcher)) – задает диапазон значений для ожидаемого аргумента.

  ```go
  mockDoer.EXPECT().DoSomething(gomock.Any())
  ```

##### Matcher на несколько аргументов

Обычный matcher имеет доступ только к одному аргементу. 

Если нужен сложный *assertion* сразу нескольких аргументов, следует использовать `Call.Do()` ([link](#do)).

Пример: проверить, что первый аргумент (`int`) меньше или равен длине второго аргумента (`string`):

```go
mockDoer.EXPECT().
    DoSomething(gomock.Any(), gomock.Any()).
    Return(nil).
    Do(func(x int, y string) {
        if x > len(y) {
            t.Fail()
        }
    })
```

##### Возвращаемое значение

Для статического возвращаемого значения необходимо использовать `Call.Return()` ([link](#return))

##### Matcher на аргументы и возвращаемое значение (все сразу)

Чтобы проверить аргументы метода и вернуть некоторое значение из метода mock'а – небходимо использовать `Call.DoAndReturn()` ([link](#doandreturn))

#### Сложные имитации методов mock'а

Примеры:

- Сохранение значений аргументов, с которыми были вызваны методы *mock*'а:

  ```go
  var s string
  
  mockIndex.EXPECT().Bar(gomock.Any()).DoAndReturn(
  
  	func(arg string) interface{} {
  		s = arg
  		return "bar"
  	},
  )
  
  r := mockIndex.Bar("foo")
  fmt.Printf("%s %s", r, s) // foo bar
  ```

- Имитация задержки в 1мс внутри метода:

  ```go
  mockIndex := NewMockFoo(ctrl)
  
  mockIndex.EXPECT().Bar().DoAndReturn(
  	func(arg string) string {
  		time.Sleep(1 * time.Millisecond)
  		return "foo"
  	},
  )
  
  r := mockIndex.Bar()
  fmt.Println(r) // foo
  ```

  



#### Утверждение для call order

Существуют следующие способы описания call order:

- с помощью метода `Call.After()` ([link](#after)):

  ```go
  callFirst := mockDoer.EXPECT().DoSomething(1)
  callA := mockDoer.EXPECT().DoSomething(2).After(callFirst)
  callB := mockDoer.EXPECT().DoSomething(3).After(callFirst)
  ```

  Утверждает, что после `callFirst` должно произойти вызов или `callA` или `callB`.

- с помощью метода `gomock.InOrder()` ([link](#func-inorder)), который указывает точный порядок вызова методов. Это менее гибко, чем использование метода `Call.After`, но более человеко-понятно, особенно для длинных последовательностей вызовов:

  ```go
  gomock.InOrder(
      mockDoer.EXPECT().DoSomething(1),
      mockDoer.EXPECT().DoSomething(2),
      mockDoer.EXPECT().DoSomething(3),
      mockDoer.EXPECT().DoSomething(4),
  )
  ```









#### Детали:

- При использовании Go 1.14+ и mockgen 1.5.0+ больше не нужно явно вызывать `ctrl.Finish()`, это будет сделано автоматически. 

- Сам процесс проверки *expectation*'s выполняется в методе `Controller.Finish()`. 

- *Mock controller* можно использовать несколько раз для создания нескольких *mock*'ов:

  ```go
  	ctrl := gomock.NewController(t) // Получение mock controller'а
  
  	mockService := service.NewMockService(ctrl) // Создание mock'а для interface Service
  	mockService1 := service.NewMockService1(ctrl) // Создание mock'а для interface Service1
  ```



#### Приемы

**Описания *expectation*, что значение, возвращаемое методом (например `err`), соответствует ожидаемому.** 

Например, можно проверить GD для случая, когда *mock* возвращает `err`. 

```go
	err := errors.New("alarm")

	// Ожидает, что метод DoSomething будет вызван один раз со значением 123, вернет из метода nil
	mockService.EXPECT().DoSomething(123).Return(err)

	// Создание экземпляра handler'а и инджекция в него mockService
	testHandler := handler.Handler{
		Service: mockService,
	}

	// Тестовый вызов handler'а
	result := testHandler.Handle()

	if result != err {
		t.Fail()
	}
```

Лучше использовать пакет `github.com/stretchr/testify/require`

**Указание того, что mock несколько раз вернет одно значение, а затем другое**

Например, 2 раза будет ошибка и 1 раз – успех.

```go
client.EXPECT().GetDocumentsByTypesV1(ctx, in).Return(nil, errors.New("error")).Times(2)
client.EXPECT().GetDocumentsByTypesV1(ctx, in).Return(out, nil).Times(1)
```





## Документация

### `func InOrder()`

```go
func InOrder(calls ...*Call)
```

Утверждает, что указанные `calls` должны выполняться в соответствующем порядке.



### `type Call`

```go
type Call struct {
	// contains filtered or unexported fields
}
```

`Call` описывает ожидаемый вызов к *mock*'у.  

Методы для типа `Call` разбиваются на группы:

- управление порядком вызовов:
  - `After()`

- количество вызовов:
  - `MaxTimes()`
  - `MinTimes()`
  - `Times()`
- Возвращаемое значение
  - `Return()`





#### `After()`

```go
func (c *Call) After(preReq *Call) *Call
```

`After()` утверждает, что вызов должен происходить только после вызова `preReq`.

#### `AnyTimes()`

```go
func (c *Call) AnyTimes() *Call
```

`AnyTimes()` предполагает, что будет сделано 0 или более вызовов.

#### `Do()`

```go
func (c *Call) Do(f interface{}) *Call
```

Функция `f`  вызывает каждый раз, когда происходит вызов метода *mock*'а. Возвращать из функции ничего не нужно, возвращаемое значение игнорируется. Чтобы задать возвращаемое значение, необходимо использовать `DoAndReturn()`.

[link](#matcher-на-несколько-аргументов)

#### `DoAndReturn()`

```go
func (c *Call) DoAndReturn(f interface{}) *Call
```

Функция `f`  вызывает каждый раз, когда происходит вызов метода *mock*'а. Возвращаемое значение функции `f()` – возвращается методом mock'а. 

Сигнатура функции `f()` должна совпадать с сигнатурой метода, для которого создается *mock*.

[link](#matcher-на-аргументы-и-возвращаемое-значение-все-сразу)

#### `MaxTimes()`

```go
func (c *Call) MaxTimes(n int) *Call
```

`MaxTimes()` требует, что было сделано не более `n` вызовов. 

#### `MinTimes()`

```go
func (c *Call) MinTimes(n int) *Call
```

`MinTimes` требует, чтобы было сделано как минимум `n` вызовов.

#### `Return()`

```go
func (c *Call) Return(rets ...interface{}) *Call
```

`Return()` указывает значение, которое будет возвращено в результате вызова функции *mock*'а.

#### `Times()`

```go
func (c *Call) Times(n int) *Call
```

`Times()` указывает точное количество ожидаемых выполнений вызова.

`Times()` можно использовать не только как *expectation*, но он также позволяет указать, сколько раз делать `Return()` какого-то значения ([link](#приемы))

### `type Matcher`

```go
type Matcher interface {
	// Возвращает, соответствует ли x или нет
	Matches(x interface{}) bool

	// Описывает, чему должен соответствовать x в соответствии с этим matcher'ом
  // Используется для генерации человеко-читаемого для неудачных тестов
	String() string
}
```

#### `Any()`

```go
func Any() Matcher
```

`Any()` возвращает `Matcher`, которое совпадает с любыми значениями любых типов.

Используется чаще всего, чтобы указать, что нам не важно значение аргумента:

```
mockDoer.EXPECT().DoSomething(gomock.Any(), "Hello GoMock")
```

#### `Eq()`

```
func Eq(x interface{}) Matcher
```

`Eq()` возвращает `Matcher`, который проверяет равенство значений (DeepEqual???).

Аргументы, которые имеют тип НЕ `Matcher`, автоматически приводятся к матчеру `Eq`. 

Два выражения ниже эквивалентны:

```go
mockDoer.EXPECT().DoSomething("Hello GoMock")
mockDoer.EXPECT().DoSomething(gomock.Eq("Hello GoMock"))
```

#### `Nil()`

```go
func Nil() Matcher
```

`Nil()` возвращает `Matcher`, который совпадает со значением *nil*.

#### Реализация собственного matcher'а

Собственный *matcher* – это `type`, который реализует `gomock.Matcher` *interface* ([link](#type-matcher)).

Пример *matcher*'а, который проверяет, что тип аргумента соответствует строке:

```go
package ofType

import (
    "reflect"
    "github.com/golang/mock/gomock"
)

type matcher struct{ t string }

func New(t string) gomock.Matcher {
    return &matcher{t}
}

func (o *matcher) Matches(x interface{}) bool {
    return reflect.TypeOf(x).String() == o.t
}

func (o *matcher) String() string {
    return "is of type " + o.t
```

Пример его использования:

```go
mockDoer.EXPECT().
    DoSomething(match.OfType("int")).
    Return(nil).
    Times(1)
```







# Testify

https://github.com/stretchr/testify

## `assert` package

`assert` *package* предоставляет множество методов, которые позволяют:

- выводит описания ошибок
- каждый *assertion* может быть дополнен анотацией (?)

Особенности прототипов функций в `assert` *package*:

- Каждая функция `assert.XXX` принимает `testing.T` *object* в качестве первого аргумента. 

  ```go
  func TestSomething(t *testing.T) {
    assert.Equal(t, 123, 123)
  }
  ```

- Каждая функция `assert.XXX` возвращает `bool`, указывающий, было ли *assertion* успешным или нет.

  Это полезно, если необходимо продолжить выполнение дальнейших *assertion*'s при определенных условиях. Например:

  ```go
  func TestSomething(t *testing.T) {
    if assert.NotNil(t, object) {
      // теперь объект точно NotNil, можно делать дальнейшие assertions без возникновения ошибок
      assert.Equal(t, "Something", object.Value)
    }
  }
  ```

  