https://stackoverflow.com/questions/56224836/stop-child-goroutine-when-parent-returns

# `context`

Когда Go используется для обработки *request*'ов на сервере, каждый *request* обрабатывается в собственной goroutine (для каждого создается своя новая goroutine? скорее всего да). 

*Request handler* запускает дополнительные *goroutine*'s, чтобы:

- сделать запрос в базу данных
- сходить в сервис.

`context` *package* позволяет передавать в подчиненные *goroutine*'s:

- *request-scoped values*, данные из запроса, которые проходят сквозь процессы и API: userId, токены авторизации.
- *request's deadline* и *cancelation signal* (сигнал отмены). Когда для *request* происходит *cancel* или *timeout*, все *goroutine*'s,  работающие над этим *request*'ом, должны быстро завершиться, чтобы система могла освободить все ресурсы, которые они используют.





## `type Context`

`Context` позволяет передавать *request-scoped values*, *request's deadline* и *cancelation signal* между несколькими горутинами (между *API boundaries*???). 

Методы `Context` безопасны для одновременного вызыва несколькими *goroutine*'s. Родительская *goroutine* может передать один `Context` любому количеству *goroutine*'s. А затем *cancel* (отменить) этот `Context`, чтобы передать им всем *cancelation signal*.

```go
type Context interface {
    Done() <-chan struct{}
    
    Err() error

    Deadline() (deadline time.Time, ok bool)
    
    Value(key interface{}) interface{}
}
```

У `type Context` *НЕТ* метода `Cancel() `, по той же причине что и  `Done` *channel*  – *receive-only*, Т.к. отправляет *cancelation signal* – одна функция, а получает – другая. 

Когда родительская *operation* запускает *goroutine*'s  для выполнения *sub-operation*'s, эти *sub-operation*'s не должны иметь возможности отменить родительскую *operation*. Функция `WithCancel()` предоставляет способ отменить созданный `Context`.

### `Done()`

```go
Done() <-chan struct{}
```

 `Done()` возвращает *channel*, для которого выполняется *close*, когда для этого *context* вызыван *cancel* (вручную или из-за *timeout*). Этот *channel* действует как *cancelation signal* для функций, работающих на основе этого *Context*: когда для *channel* произошел *close*, функции должны отказаться от своей работы и сделать *return*.

### `Err()`

```go
Err() error
```

`Err()` возвращает *error*, которая описывает, почему для этого *context* был выполнен *cancel*. Ее нужно вызывать после того как у `Done` *channel* произошел *close*.

### `Deadline()`

```go
Deadline() (deadline time.Time, ok bool)
```

`Deadline()` возвращает время, когда этот *context* будет *cancel* (отменен) (??? из-за *timeout*), если имеется такой *deadline* (??? *context* создан через `WithTimeout()`).

`Deadline` позволяет функциям определять, должны ли они вообще начинать работу или нет; если осталось слишком мало времени, это может не иметь смысла. Также можно использовать *deadline* для установки *timeout*'ов для I/O операций.

`Deadline()` возвращает `ok == false`, если *deadline* не установлен (через `WithTimeout()`). 

### `Value()`

```go
Value(key interface{}) interface{}
```

`Value()` возвращает *value*, которое соответствует указанному `key` или `nil`, если его нет.

`Value()` передавать данные из запроса (*request-scoped values*).. Эти данные безопасны для одновременного (*simultaneous*) использования несколькими *goroutine*'s.

## Функции для создания подчиненного *context*

Эти функции создают новый *context* из уже существующого. В итоге, *context*'s  образуют дерево: когда *context* отменяется, все производные *context*'s также отменяются.

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

### `WithValue()`

```go
func WithValue(parent Context, key, val interface{}) Context
```

`WithValue()` возвращает копию `parent`, в котором `key` соответствует значение `val`. 

Необходимо использовать только для включения *request-scoped value*'s ([1](#context)) в *context*. Нельзя использовать для передачи параметров в функции.

Заданный `key` должен быть *comparable* и не должен быть *string type* или любого другого *built-in type*, чтобы избежать коллизий между *package*'s, использующими *context*. Т.к. иначе разные *package*'s могли бы случайно определить несколько *value*'s с одним и тем же *key*. Поэтому необходимо определить свой *type* для *key*. 

(??? абзац непонятен) Чтобы избежать *allocating*  при *assigning* к `interface{}`, *context key*'s часто имеют конкретный тип `struct{}`. Альтернативно, в качестве статического типа *exported context key* используется *pointer* или *interface*.

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

### `WithCancel()`

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

`WithCancel()` возвращает копию родительского *context* с новым *Done channel*. *Done channel* возвращенного *context*'а закрывается (*close*), когда вызывается возвращенная функция `cancel` или когда *Done channel* родительского *context*'а закрывается (*close*) – что произойдет раньше.

При *Cancel* происходит освобождение ресурсов *context*'а, поэтому код должен вызывать `cancel` (обязательно???), как только операции, выполняемые в этом *context*'е, завершатся.

<u>Пример:</u>

Пример показывает, как сделать *cancel* для *context*'а без утечки *goroutine*.

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// gen генерирует integers в отдельной goroutine и
	// отправляет их в возвращаемый channel.
	// Когда вызывающий код завершает получение integers, он (или они) должен cancel (отменить) context,
	// чтобы не произошло утечки goroutine
	gen := func(ctx context.Context) <-chan int {
		dst := make(chan int)
		n := 1
		go func() {
			for {
				fmt.Println("a", n)
				select {
				case <-ctx.Done():
					fmt.Println("b", n)
          return // делает return, чтобы не допустить leak (утечки) goroutine
				case dst <- n:
					n++
					fmt.Println("c", n)
				}
			}
		}()
		return dst
	}

	ctx, cancel := context.WithCancel(context.Background())


	for n := range gen(ctx) {
		fmt.Println("d", n)
		if n == 3 {
			break
		}
		time.Sleep(time.Second)
	}

	cancel() // делает cancel, когда мы закончили извлекать числа
	time.Sleep(time.Second)
}
```

Вывод:

```go
a 1
c 2
a 2
d 1
d 2
c 3
a 3
d 3
c 4
a 4
b 4
```

Расшифровка: так работает потому что, *goroutine* вначале кладет в канал `1` , а потом пытается положить `2` и блокируется. В это время начинает потребитель разгребать значения и выбирает `1` и `2`. После этого просыпается *goroutine* и кладет в канал `3`, снова его выбирает потребитель и т.д.











## Порядок использования

