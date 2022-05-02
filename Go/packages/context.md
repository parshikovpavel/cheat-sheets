https://stackoverflow.com/questions/56224836/stop-child-goroutine-when-parent-returns

https://www.youtube.com/watch?v=U5Y_St7lESk

https://medium.com/codex/go-context-101-ebfaf655fa95

https://www.sohamkamani.com/golang/context-cancellation-and-values/

https://levelup.gitconnected.com/context-in-golang-98908f042a57

https://habr.com/ru/company/nixys/blog/461723/

https://go.dev/blog/pipelines

https://go.dev/blog/context

# `context`

Когда Go используется для обработки *request*'ов на сервере, каждый *request* обрабатывается в собственной *goroutine*. 

*Request handler* запускает дополнительные *goroutine*'s, чтобы:

- сделать запрос в *database*
- сходить по RPC в сервис.

`context` *package* позволяет передавать через *API boundaries* (границы API) в подчиненные *goroutine*'s, которые обрабатывают *request*:

- *request-scoped values*, данные из запроса, которые проходят сквозь процессы и API: *userId*, токены авторизации.
- *request's deadline* и *cancelation signal* (сигнал отмены). Когда для *request* происходит *cancel* или *timeout*, все *goroutine*'s,  работающие над этим *request*'ом, должны быстро завершиться, чтобы система могла освободить все ресурсы, которые они используют.

## Принцип использования `Context`:

- *Handler*'ы, обрабатывающие входящие *request*'ы к *server*'у, должны создавать `Context` (мое: или использовать уже существующий, созданный при старте *service*) 
- код, выполняющий исходящие *request*'ы к другим *server*'ам, должны принимать `Context`. 

Цепочка *function call*'s между ними должна *propagate* (распространять) контекст

*Context* привязывает время жизни сущностей (*goroutine*, ...) ко времени жизни *request*'а. После того как *context* был *cancel*, все связанные сущности должны быть завершены. Ошибка – создавать *goroutine*, которая привязана к *context*'у, но при этом не реагирует на его *cancel* и живет дольше, чем живет *context* (и *request*).

В Google мы требуем, чтобы программисты Go передавали `ctx Context` параметр в качестве первого аргумента каждой функции на пути вызовов между входящими и исходящими *request*'ами. Это позволяет пакетам, разработанным разными командами, хорошо взаимодействовать. Он обеспечивает простой контроль времени *timeout*'ов и cancellation (отмен). А также гарантирует, что критические *value*, такие как *security credential*'s (разрешения по безопасности), правильно передаются в различные функции (??? тут, наверное, как-то помогает передача *request-scoped values*)

Использование *context*'а как общего *interface* для *request-scoped data* и *cancellation*, упрощает для разработчиков распространение и совместное использование кода для создания масштабируемых сервисов.



### *Derived context*

*Package* `context` предоставляет функции для получения нового *derived Context*'а из существующего *Context*'а:

-  [`WithCancel()`]()
- [`WithDeadline()`]()
- [`WithTimeout()`]()

Эти функции принимают *parent Context* и возвращают:

- *derived (child) Context*
- `CancelFunc`. 

([`WithValue()`]() – эта функция на самом деле не создает *derived* `Context`, а только копирует *parent* `Context`)

Эти *context*'ы образуют дерево: когда для `Context` выполняется *cancel*, для всех *derived* `Context`'ов (*derived* от него) также выполняется *cancel*.

Функции `WithCancel()`, `WithDeadline()` и `WithTimeout()` 

Читать про [обязательный вызов `cancel()`](обязательный-вызов-cancel)









Программы, использующие `Context`, должны следовать нижеописанным правилам, чтобы поддерживать согласованность *interface*'s между *package*'s и дать возможность *static analysis tool*'зам проверить *context propagation*:

- Не храните `Context` внутри *struct type*. Вместо этого явно передавайте `Context` в *function*. `Context` должен быть первым *parameter*'ом. Этот *parameter* обычно называют `ctx`:

  ```go
  func DoSomething(ctx context.Context, arg Arg) error {
  	// использование `ctx`
  }
  ```

- Не передавайте `nil` в качестве `Context`, даже если функция выполняет проверку на `nil`. Если вы не уверены, какой `Context` использовать – передайте `context.TODO()`.

- Используйте `Context.Value` только для *request-scoped* данных, которые передаются в (???) *process*'ы и API, а не для передачи опциональных параметров в функции.

## Кейсы использования *context*'а

*Context*, связанный с входящим *request*'ом обычно *cancel*, когда *request handler* делает *return*.

Можно запустить несколько параллельных *request*'ов, создав для каждого отдельный *derived context*, а затем при необходимости сделать *cancel* для лишних *request*'ов.

*Context* предназначен для *cancel* со стороны *parent goroutine*. Если *child goroutine* в процессе обработки обнаруживает ошибку, она должна возвращать *error* (по *channel*). Когда *parent goroutine* получит *error* может:

- отменить другие задачи, сделав *cancel* для *context*'а
- как-то по другому отреагировать, например, запустить еще одну *goroutine*

*Context* может только передать сигнал об отмене (через *cancel* или по *timeout*). Он не умеет останавливать какие-либо *goroutine*'s. Сама *goroutine* отвечает за проверку `ctx.Done()`, а также за досрочное прерывание.

# Структура

## Variable

*Channel* `ctx.Done()` будет *close* в двух случаях:

1. когда истек *context timeout*
2. когда *context* был явно *cancel*, через `Cancelfunc()`

И в обоих случаях можно узнать причину отмены через `ctx.Err()`, которая будет возвращать одну из двух *error*'s:

- `Canceled`— это *error*, возвращаемая `Context.Err()`, когда для *context*'а выполняется *cancel*.

  ```go
  var Canceled = errors.New("context canceled")
  ```

  Пример:

  ```go
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    cancel()
    fmt.Println(ctx.Err()) // context canceled
  ```

  

- `DeadlineExceeded` — это *error* возвращаемая `Context.Err()`, когда истекает *deadline context*'а.

  ```go
  var DeadlineExceeded error = deadlineExceededError{}
  ```

  Пример:

  ```go
    ctx, cancel := context.WithTimeout(context.Background(), 1 * 77time.Second)
    defer cancel()
  
    time.Sleep(2 * time.Second)
    fmt.Println(ctx.Err()) // context deadline exceeded
  ```

  

## Function

### `WithCancel()`

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

`WithCancel()` возвращает копию `parent` *context*'а  с новым `Done` *channel*.

`Done` *channel* возвращенного *context*'а закрывается (*close*), когда случается (что произойдет раньше):

- когда вызывается возвращенная функция `cancel` 
- когда `Done` *channel* `parent` *context*'а закрывается (*close*)

Читать про [обязательный вызов `cancel()`](обязательный-вызов-cancel)

<u>Пример:</u>

Пример показывает, как сделать *cancel* для *context*'а без утечки *goroutine*.

```go
func main() {
  // gen() – генератор integer, запускает отдельную goroutine и
  // отправляет их в channel, который возвращается из gen().
  // Когда caller завершает получение integer's, он должен сделать cancel для context,
  // чтобы не допустить leak для внутренней goroutine, которую запускает gen()
  gen := func(ctx context.Context) <-chan int {
    dst := make(chan int)
    go func() {
      n := 1
      for {
        select {
        case <-ctx.Done():
          return  // делает return, чтобы не допустить leak для goroutine
        case dst <- n:
          n++
        }
      }
    }()
    return dst
  }

  ctx, cancel := context.WithCancel(context.Background())
  defer cancel() 

  // Здесь 1 goroutine
  
  for n := range gen(ctx) {
    // Здесь 2 goroutines
    fmt.Println(runtime.NumGoroutine())
    fmt.Println(n)
    if n == 5 {
      break
    }
  }

  // после defer будет снова 1 goroutine
}
```



### `WithDeadline()`

```go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)
```

`WithDeadline()` возвращает копию `parent` *context*'а с установленным *deadline* не позднее `d`. Если `parent`'s *deadline* уже раньше, чем `d`, то возвращенный *context* будет семантически эквивалентен `parent` (т.е. это два разных *context*'а, с одним и тем же *deadline*). 

`Done` *channel* возвращенного *context*'а закрывается (*close*), когда случается (что произойдет раньше):

- *deadline* истекает
- когда вызывается возвращенная функция `CancelFunc` 
- когда `Done` *channel* `parent` *context*'а закрывается (*close*)

Читать про [обязательный вызов `cancel()`](обязательный-вызов-cancel)

Пример:

```go
func main() {
  d := time.Now().Add(1 * time.Millisecond)
  ctx, cancel := context.WithDeadline(context.Background(), d)

  // Хотя deadline истечет через 1 миллисекунду, хорошая практика - всегда вызывать
  // функцию `cancel()`. Если не следовать этой практике, то context и
  // его parent context (??? почему parent, может child) могут прожить дольше, чем необходимо.
  defer cancel()

  select {
  // эта case branch не будет выполнена
  case <-time.After(1 * time.Second):
    fmt.Println("overslept")
  // раньше будет закрыт `ctx.Done()` channel  
  case <-ctx.Done():
    fmt.Println(ctx.Err())
  }
}
```



### `WithTimeout()`

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

`WithTimeout()` возвращает то же самое, что и `WithDeadline(parent, time.Now().Add(timeout))`.

Т.е. аналогично как метод `WithDeadline()`, возвращает копию `parent` *context*'а с установленным *deadline* не позднее `now+timeout`. Если `parent`'s *deadline* уже раньше, чем `now+timeout`, то возвращенный *context* будет семантически эквивалентен `parent` (т.е. это два разных *context*'а, с одним и тем же *deadline*). 

Функция `CancelFunc` освобождает ресурсы, если *timer* еще запущен.

Читать про [обязательный вызов `cancel()`](обязательный-вызов-cancel)

Пример:

```go
  // Создает context с timeout'ом, чтобы прекратить работу по истечении timeout'а

  // Pass a context with a timeout to tell a blocking function that it
	// should abandon its work after the timeout elapses.
	ctx, cancel := context.WithTimeout(context.Background(), 1 * time.Millisecond)
	defer cancel()

	select {
	case <-time.After(1 * time.Second):
		fmt.Println("overslept")
	case <-ctx.Done():
		fmt.Println(ctx.Err()) // выводит "context deadline exceeded"
	}
```







### `Background()`

```go
func Background() Context
```

`Background()` возвращает *non-nil, empty Context*. Он не может быть никогда *cancel* (отменен), не имеет *value*'s и *deadline*. 

Такой *context* используется в качестве корня для дерева *context*'ов.

Обычно он используется:

- в функции `main()`
- в функции `init()`
- в тестах
- в качестве *context*'а верхнего уровня для входящих *request*'ов.



### `TODO()`

```go
func TODO() Context
```

`TODO()` возвращает *non-nil, empty Context*. 

Код должен использовать `context.TODO()`, когда неясно, какой *context* использовать (для передачи в другую функцию), или он еще недоступен (поскольку окружающая функция еще не принимает `ctx` в параметре).



### `WithValue()`

```go
func WithValue(parent Context, key, val interface{}) Context
```

`WithValue()` возвращает копию `parent`, в котором `key` соответствует значение `val`. Другими словами, у возвращенного *context*'а метод `Value(key)` будет возвращать `val`. 

Позволяет связывать *request-scoped value*'s с *context*'ом.

[Подробнее про особенности определения `key`](#value)

Пример:

```go
package main

import (
	"context"
	"fmt"
)

func main() {
  /* Создание кастомного типа string */
	type favContextKey string

  /* Функция, принимающая context*/
	f := func(ctx context.Context, k favContextKey) {
		if v := ctx.Value(k); v != nil {
			fmt.Println("found value:", v)
			return
		}
		fmt.Println("key not found:", k)
	}

	k := favContextKey("language")
	ctx := context.WithValue(context.Background(), k, "Go")

  /* Вызов с существующим key */
	f(ctx, k)
  
  /* Вызов с отсутствующим key */
	f(ctx, favContextKey("color"))

}
```







## Type

### `type CancelFunc`

```go
type CancelFunc func()
```

`CancelFunc` сигнализирует  о необходимости прекратить работу. `CancelFunc` не ждет пока работа остановится (а как она может этого ждать, если она просто делает *close* для *channel*). Вызов `cancel()` является идемпотентным, вы можете вызывать его несколько раз конкурентно из нескольких *goroutine*'s. После первого вызова последующие вызовы `CancelFunc` ничего не делают.

Если по *context*'у произошел *timeout* или был вызван `cancel()`, его можно вызывать повторно без всяких проблем.



### `type Context`

Методы `Context` безопасны для конкурентного вызыва несколькими *goroutine*'s. *Parent goroutine* может передать один *context* любому количеству *goroutine*'s. А затем *cancel* (отменить) этот *context*, чтобы передать им всем *cancelation signal*.

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
  
    Done() <-chan struct{}
    
    Err() error
    
    Value(key interface{}) interface{}
}
```

*Cancelation signal* обычно отправляет отправляет одна *goroutine* (*parent goroutine*), а получает – другая (*child goroutine*). Поэтому *child goroutine* не может сделать *cancel* для *context*'а, поэтому у `type Context` *НЕТ* метода `cancel() `  и  `Done` *channel*  – *receive-only*. 

#### `Deadline()`

```go
Deadline() (deadline time.Time, ok bool)
```

`Deadline()` возвращает время, когда этот *context* будет *cancel*, если имеется такой *deadline*. *Deadline* может быть установлен через `WithDeadline()` или `WithTimeout()`. `Deadline()` возвращает `ok == false`, если *deadline* не установлен. 

Можно использовать *deadline* для установки *timeout*'ов для I/O операций. Также `Deadline` позволяет функциям определять, должны ли они вообще начинать работу или нет; если осталось слишком мало времени, это может не иметь смысла.

Последовательные вызовы `Deadline()` возвращают один и тот же результат.



#### `Done()`

```go
Done() <-chan struct{}
```

`Done()` возвращает *channel*, для которого выполняется *close*, когда для этого *context* сделан *cancel* или истек *timeout*. Этот *channel* действует как *cancelation signal* для функций, работающих на основе этого *Context*: когда для *channel* произошел *close*, функции должны отказаться от своей работы и сделать *return*.

`Done()` возвращает `nil`, если этот *context* никогда не может быть *cancel* (например, для `context.Background()`). Последовательные вызовы `Done()` возвращают одно и то же значение. *Close* для `Done()` *channel* может произойти асинхронно, уже после завершения функции `CancelFunc`. 

- `WithCancel()` организует *close* для `Done()` при вызове функции `CancelFunc`
- `WithDeadline()` организовывает *close* для `Done()` по истечении *deadline*
- `WithTimeout()` организовывает *close* для `Done()` по истечении *timeout*'а.

`Done()`, как правило, используется в `select` *statement*'s.



#### `Err()`

```go
Err() error
```

Возвращаемое значение:

- если `Done()` еще не *close*, `Err()` возвращает `nil`.

  ```go
  func main() {
    a := context.Background()
    fmt.Println(a.Err())       // <nil>
  }
  ```

-  если `Done()` уже *close*, `Err()` возвращает *non-nil error* (`!=nil`), которая описывает, почему это произошло.

  - [`Canceled`](#variable) – если для *context*'а был выполнен *cancel*. 
  - [`DeadlineExceeded`](#variable) –  если истек *deadline context*'а.

  Поэтому `Err()` имеет смысл вызывать после того как у `Done` *channel* произошел *close*.

  После того, как `Err()` вернул *non-nil error*, последующие вызовы `Err()` вернут ту же самую *error*.





#### `Value()`

```go
Value(key interface{}) interface{}
```

`Value()` возвращает:

- *value*, которое соответствует указанному `key` 
- `nil` – если такого *value* нет.

Эти данные должны (!!!) быть безопасны для одновременного (*simultaneous*) использования несколькими *goroutine*'s.

Последовательные вызовы `Value()` с одним и тем же `key` возвращают один и тот же результат. 

В качестве *value* необходимо использовать *request-scoped data* (данные из запроса), которые передаются через границы *proccess*'ов и API (*userId*, токены авторизации). Не нужно их использовать для передачи опциональных параметров в функции.

`key` идентифицирует конкретное *value* в *context*'е. Функции, которые хотят сохранить *value* в *context*, обычно объявляют *key* в глобальной *variable*, а затем используют этот *key* в качестве аргумента для `context.WithValue()` и `Context.Value()`. 

*Key* должен быть *comparable type* (т.е. который поддерживает *equality*, равенство). Лучше, чтобы `key` не был одного из *built-in type* (например, *string*), чтобы избежать коллизий между *package*'s, использующими *context*. Т.к. иначе разные *package*'s могли бы при передаче *context*'а по цепочке – случайно определить несколько *value*'s с одним и тем же *key*. Поэтому *package*'s должны определить `key` как *unexported type*, чтобы избежать таких коллизий. Лучше всего если реализован *package* (как [`userip` в примере](#userip)), который скрывает детали сохранения и извлечения *value* из *context*'а и предоставляет некоторый интерфейс (как в примере, функции `NewContext()` и `FromRequest()`) для доступа к *value*.

Чтобы избежать выделения (*allocation*) дополнительной памяти при присвоении (*assigning*) к `interface{}` для `key` необходимо использовать тип `struct{}`:

```go
type key struct{}
```

 Альтернативно, в качестве типа для *exported key variable* (??? для переменных в контексте) необходимо использовать *pointer* или *interface* (наверно, потому что под них не надо много памяти).

*Package*'s, которые определяют `key` для *context*'а, должны предоставлять функции, инкапсулирующие внутри себя работу с этим *unexported type* и предоставляющие доступ к *value*'s, сохраненных с использованием этого *key*: 

##### Примеры

Пример с одним видом *value*, которые можно сохранять в *context* (у которого ключ `userKey`):

```go
package user

import "context"

// `User` - это type для value, сохраненного в Context
type User struct {...}

// `key` — это unexported type для key's, определенных в этом package. 
// Это предотвращает коллизии с key's, определенными в других package's.
// Т.е. это просто variable со значением - zero value (0)
type key int

// `userKey` – это key для value типа user.User в context'е. Эта variable – unexported
// клиенты должны использовать user.NewContext() и user.FromContext() для доступа к value
// и не использовать этот key напрямую
var userKey key

// NewContext() возвращает новый Context, который хранит value `u`.
func NewContext(ctx context.Context, u *User) context.Context {
	return context.WithValue(ctx, userKey, u)
}

// FromContext() возвращает value c типом `User`, которое сохранено в `ctx`, если оно там есть
func FromContext(ctx context.Context) (*User, bool) {
	u, ok := ctx.Value(userKey).(*User)
	return u, ok
}
```



Аналогичный пример с `logger`:

```go
package logger

import (
	"context"
)

type key int

var loggerKey key

var stubLogger *Logger

func init() {
	stubLogger = <тут как-то он инициализируется>
}

// NewContext возвращает новый context, добавляя в него logger.Logger
func NewContext(ctx context.Context, log *Logger) context.Context {
	return context.WithValue(ctx, loggerKey, log)
}

// FromContext достаёт из контекста logger.Logger
func FromContext(ctx context.Context) *Logger {
	if logger, ok := ctx.Value(loggerKey).(*Logger); ok {
		return logger
	}

	return stubLogger
}
```



Пример с несколькими ключами в одном *package*, тут используются *constant*

```go
type contextKey int

// HubContextKey is a context key used to store Hub on any context.Context type
const HubContextKey = contextKey(1)

// RequestContextKey is a context key used to store http.Request on the context passed to RecoverWithContext
const RequestContextKey = contextKey(2)
```

 Можно вместо последовательных чисел `iota` писать

```go
type txKeyType int

const txKey txKeyType = iota
```





## Обязательный вызов `cancel()`

Вызов `CancelFunc()`:

- выполняет *cancel* для *child* `Context`'а, его *children* `Context`'ов
- удаляет *parent's reference* на *child* `Context`
- останавливает все связанные *timer*'ы. 

Если не была вызвана `CancelFunc` (или произошла ошибка???), то происходит *leak* для *child* `Context`'а и его *children* `Context`'ов до тех пор, пока *parent* `Context` не будет *cancel* или не сработает *timer*. Инструмент `go vet` проверяет, используются ли функции `CancelFunc` на всех путях потока управления.

Если отбросить `CancelFunc` через [black identifier](../Lang.md#blank-identifier):

```go
c, _ := context.WithTimeout(ctx, time.Second)
```

то `go vet` выдает ошибку:

```
Cancel function, возвращаемая `context.WithTimeout()`, должна вызываться, а не отбрасываться, чтобы избежать context leak.
```

При вызове функции `cancel` происходит освобождение ресурсов, ассоциированных с *context*'ом.

Иначе произойдет *leak* ресурсов, которые выделены для данного *context*'а:

- самой библиотекой:

  - *child context*'oв

  - *timer*'а (аналогично как в примере из [timer leak](#timer-leak))
  - возможно, каких-то системных *goroutine*, которые создаются функциями `WithCancel()`, `WithTimeout()`, ...

- логикой приложения:

  - запущенных *goroutine*'s
  - ...

Поэтому код должен вызывать `cancel`, как только операции, выполняемые в `ctx` *context*'е, завершатся.

Хотя иногда вызов `cancel()` – избыточен,  хорошая практика – всегда вызывать функцию `cancel()`. Если не следовать этой практике, то *context* и его *parent context* (??? почему *parent*, может *child*) могут прожить дольше, чем необходимо.

Например, здесь:

```go
c, cancel := context.WithTimeout(ctx, time.Second)
err = http.RequestWithContext(c, "some", someRequest, &response)
cancel()
```

Запрос `http.RequestWithContext()`может исполниться до истечения 1-секундного *timeout*'а и ресурсы, используемые *context*'ом, будут  освобождены только во время вызова `cancel()`. Если явно не вызвать `cancel()`, то все ресурсы *context*'а останутся в памяти до тех пор, пока не истечет *timeout*. При больших значениях *timeout* и высокой нагрузке – излишне выделенные объемы памяти могут быть существенными.

[Подробнее про `CancelFunc`](#type-cancelfunc)

Гарантировать, что `cancel()` вызывается, проще всего через `defer`:

```go
c, cancel := context.WithTimeout(ctx, time.Second)
defer cancel()
```

Можно использовать `cancel()` прямо в *goroutine*, в которую передается *context*:

- прямо вызвав ее `cancel()`
- или в `defer cancel()`.

```go
ctx, cancel := context.WithTimeout(
    context.Background(),
    time.Duration(3*time.Second))

go func(ctx context.Context) {
  defer cancel()
  // какая-то обработка
    
}(ctx)
```





## Порядок использования

### Пример 1

Функция `Stream()` рассчитывает *value* `v` с помощью функции `DoSomething()` и делает его *send* в `out` *channel*.

Выход из функции `Stream()` происходит при:

- возврате *error* из `DoSomething(ctx)`
- когда произойдет завершение *context*'а и *close* для `ctx.Done()` *channel*.

```go
func Stream(ctx context.Context, out chan<- Value) error {
	for {
		v, err := DoSomething(ctx)
		if err != nil {
			return err
		}
	
    select {
			case <-ctx.Done():
				return ctx.Err()
			case out <- v:
		}
	}
}
```



### Пример использования *context*'а

Рассмотрим пример — *HTTP-server*, который обрабатывает GET-*request*'ы на порту `8080` по URL вида `/search?q=golang&timeout=1s`. Т.е. он принимает два параметра:

- `q` – поисковый запрос (*query*). Этот параметр будет перенаправлен в [Google Web Search API](https://developers.google.com/web-search/docs/) для получения результатов поиска. 
- `timeout` – *timeout* для *request*'а в формате [`time.Duration`](). 

Код разделен на три *package*'s:

- `server`
- `userip` 
- `google` 

#### `server`

`server` содержит функцию `main()` и *handler* для `/search`.

```go
package main

import (
"context"
"html/template"
"log"
"net/http"
"time"

"example/userip"
"example/google"
)

func main() {
	// Регистрируем handler function для pattern'а `/search` в `DefaultServeMux`.
	http.HandleFunc("/search", handleSearch)

	// Запускаем прослушивание порта 8080 и прием входящих HTTP connection'ов с использованием `DefaultServeMux`
	log.Fatal(http.ListenAndServe(":8080", nil))
}

// Handler
func handleSearch(w http.ResponseWriter, req *http.Request) {
	var (
		ctx    context.Context
		cancel context.CancelFunc
	)

	// Получаем query parameter и парсим его в duration
	timeout, err := time.ParseDuration(req.FormValue("timeout"))

	if err == nil {
		// Если удалось распарсить timeout из query parameter'а,
		// то создаем context, который будет cancel через указанный timeout
		ctx, cancel = context.WithTimeout(context.Background(), timeout)
	} else {
		// Если нет, то просто создаем context, который cancel по возвращенной cancel function
		ctx, cancel = context.WithCancel(context.Background())
	}
	// делаем cancel при завершении handler'а
	defer cancel()

	// Получаем query parameter с поисковым запросом и проверяем, что он не пуст (и вообще существует в request'е)
	query := req.FormValue("q")
	if query == "" {
		http.Error(w, "no query", http.StatusBadRequest)
		return
	}

	// Для работы с IP используем отдельный НАШ package `userip`
	// Извлекаем IP из request'а
	userIP, err := userip.FromRequest(req)
	if err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	// Сохраняем IP в context
	ctx = userip.NewContext(ctx, userIP)

	// Используем НАШ package `google`
	// Отправляем query в google, передаем `ctx` context
	results, err := google.Search(ctx, query)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}


	s := struct {
		Results          google.Results
		Timeout, Elapsed time.Duration
	}{
		Results: results,
		Timeout: timeout,
	}

	// Заполняем `s` struct в `resultTemplate` и пишем получившийся результат в `w`
	err = resultsTemplate.Execute(w, s);

	if err != nil {
		log.Print(err)
		return
	}
}

var resultsTemplate = template.Must(template.New("results").Parse(`
<html>
<head/>
<body>
  <ol>
  {{range .Results}}
    <li>{{.Title}} - <a href="{{.URL}}">{{.URL}}</a></li>
  {{end}}
  </ol>
  <p>{{len .Results}} results ; timeout {{.Timeout}}</p>
</body>
</html>
`))

```

#### `userip`

*Package* `userip` содержит функции для извлечения IP-адреса из *request*'а и связывания его с *context*'ом.

`userip` скрывает детали сохранения и извлечения *value* из *context*'а и предоставляет интерфейс (функции `NewContext()` и `FromRequest()`) для доступа к *value*.

```go
package userip

import (
"context"
"fmt"
"net"
"net/http"
)

// Получаем IP-адрес из request'а, если он есть.
func FromRequest(req *http.Request) (net.IP, error) {

	// Разбивает `req.RemoteAddr` на `host` и `port`
	ip, _, err := net.SplitHostPort(req.RemoteAddr)
	if err != nil {
		return nil, fmt.Errorf("userip: %q is not IP:port", req.RemoteAddr)
	}

	userIP := net.ParseIP(ip)
	if userIP == nil {
		return nil, fmt.Errorf("userip: %q is not IP:port", req.RemoteAddr)
	}
	return userIP, nil
}

// Key имеет unexported type, чтобы предотвратить коллизии с context key's, которые были сохранены в
// других package's. Значение этого type используется для context key
type key int

// userIPkey — это context key для IP address. Его `value = 0`, но это произвольно.
// Если бы этот package определял другие context key's, им необходимо было присвоить
// отличающиеся integer value's
const userIPKey key = 0

// NewContext возвращает новый Context, который содержит userIP.
func NewContext(ctx context.Context, userIP net.IP) context.Context {
	return context.WithValue(ctx, userIPKey, userIP)
}

// FromContext извлекает IP address из `ctx`, если он там есть
func FromContext(ctx context.Context) (net.IP, bool) {
	// `ctx.Value()` возвращает nil, если `ctx` не содержит указанный `userIPKey`
	// Type assertion для типа `net.IP` вернет `ok = false` для значения nil.
	userIP, ok := ctx.Value(userIPKey).(net.IP)
	return userIP, ok
}
```



#### `google`

*Package* `google` содержит функцию `Search()` для отправки *request*'а в  Google Web Search API и парсит JSON результат.

Функция `httpDo()` принимает параметр `ctx` и затем:

- привязывает *request* к этому *context*'у
- ожидает закрытия `ctx.Done()` или завершения *request*'а.

```go
package google

import (
	"context"
	"encoding/json"
	"net/http"

	"example/userip"
)

// Results – slice из результатов поиска
type Results []Result

// Result содержит title и URL результата поиска
type Result struct {
	Title, URL string
}

// Search() отправляет query в API поиска Google и возвращает результат
func Search(ctx context.Context, query string) (Results, error) {
	// Создаем request в Google Search API и настраиваем его
	req, err := http.NewRequest("GET", "https://ajax.googleapis.com/ajax/services/search/web?v=1.0", nil)
	if err != nil {
		return nil, err
	}
	q := req.URL.Query()
	q.Set("q", query)

	// Если ctx хранит IP address, то включаем его в request
	// Google API использует IP address, чтобы отличать серверные и пользовательские request'ы
	if userIP, ok := userip.FromContext(ctx); ok {
		q.Set("userip", userIP.String())
	}
	req.URL.RawQuery = q.Encode()

	// Вызываем функцию httpDo(), которая отправит request `req` и получит response `resp`.
	// Передаем 3-м аргументом функцию, которая будет вычитывать `resp` в переменную `result`
	var results Results
	err = httpDo(ctx, req, func(resp *http.Response, err error) error {
		if err != nil {
			return err
		}
		defer resp.Body.Close()

		// Парсим результат поиска в формате JSON
		var data struct {
			ResponseData struct {
				Results []struct {
					TitleNoFormatting string
					URL               string
				}
			}
		}
		if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {
			return err
		}

		// Переносим результаты в slice `results`
		for _, res := range data.ResponseData.Results {
			results = append(results, Result{Title: res.TitleNoFormatting, URL: res.URL})
		}
		return nil
	})

	// Т.к. httpDo() в обоих case branch's ожидает на read из channel `c` (т.е. завершения функции `f()`) –
	// поэтому здесь безопасно читать значение переменной `results`
	return results, err
}

// httpDo в отдельной goroutine отправляет request `req` и response передает в качестве аргумента в функцию `f()`.
// Если channel `ctx.Done()` становится close, `httpDo()` ожидает завершения request'а (он будет cancel, т.к. он привязан к context)
// и возвращает `ctx.Err()`. Иначе, `httpDo()` возвращает результат вызова функции `f()` (чаще всего `nil`).
func httpDo(ctx context.Context, req *http.Request, f func(*http.Response, error) error) error {
	c := make(chan error, 1)
	req = req.WithContext(ctx)

	// Запускает request в отдельной goroutine и передает response в f
	// Как будто `Client.Do()` не умеет нативно работать с context (а он работает)
	go func() {
		c <- f(http.DefaultClient.Do(req))
	}()

	select {
	case <-ctx.Done():
		<-c // Ожидаем завершения request'а и вызова `f()`
		return ctx.Err()
	case err := <-c:
		return err
	}
}
```

Хотя при отмене *context*'а при срабатывании `<-ctx.Done()` мы все равно ждем возврата `f()` в операции `<-c`. Но здесь используется *request*, к которому привязан *context* (привязывается в строке `req = req.WithContext(ctx)`). Если *context* будет отменен, то и  `Client.Do()` будет *cancel* и завершится раньше с некоторой *error* (это внутри этого метода реализовано). Можно быть уверенным, что `Client.Do()` поддерживают быстрый *cancel*.

Но жесткой необходимости ждать завершения `f()` – нет. Можно убрать ожидание `<-c`.

Также тут ждем возврата `f()`, чтобы гарантировать, что запущенная *goroutine* была завершена и  `Client.Do()` был *cancel*. При этом мы не оставляем "грязь" после себя и гарантируем, что функция `f()` завершилась и читать переменную `results` – безопасно.

И, на самом деле, здесь можно упростить функцию до:

```go
func httpDo(ctx context.Context, req *http.Request, f func(*http.Response, error) error) error {
    req = req.WithContext(ctx)
    return f(http.DefaultClient.Do(req))
}
```

Т.к. *request* уже умеет работать с *context*'ом и мы в любом случае ожидаем завершения *request*'а. Прием с отдельной *goroutine* имеет смысл при вызове тех функций, которые не умеют нативно работать с *context*'ом.