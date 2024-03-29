Этот *module* не входит в стандартную библиотеку и должен подключаться отдельно:

```
require (
	golang.org/x/sync v0.0.0-20201020160332-67f06af15bc9
)
```



# `errgroup`

*Package* обеспечивает синхронизацию, распространение (всплытие?) *error* и отмену *Context* для групп *goroutine*'s, работающих над подзадачами общей задачи.

[Ссылка](https://pkg.go.dev/golang.org/x/sync@v0.0.0-20210220032951-036812b2e83c/errgroup)

## `type Group`

*Group* – это набор *goroutine*'s, работающих над подзадачами, которые являются частью одной и той же общей задачи.

<u>Пример:</u>

Параллельный поход в несколько URL. Использование `Group` вместо `sync.WaitGroup`  для упрощения обработки ошибок. Этот пример был доработан из примера для `sync.WaitGroup` ([1](моя ссылка), [2](https://golang.org/pkg/sync/#example_WaitGroup)).

```go
package main

import (
	"fmt"
	"net/http"

	"golang.org/x/sync/errgroup"
)

func main() {
	g := new(errgroup.Group)
	var urls = []string{
		"http://www.golang.org/",
		"http://www.google.com/",
		"http://www.somestupidname.com/",
	}
	for _, url := range urls {
		// Launch a goroutine to fetch the URL.
		url := url // https://golang.org/doc/faq#closures_and_goroutines
		g.Go(func() error {
			// Fetch the URL.
			resp, err := http.Get(url)
			if err == nil {
				resp.Body.Close()
			}
			return err
		})
	}
	// Wait for all HTTP fetches to complete.
	if err := g.Wait(); err == nil {
		fmt.Println("Successfully fetched all URLs.")
	}
}
```

Пример с контекстом:

```go
type walletInfo struct {
	isOnlyBankTransfer bool
	balance            string
}

func getWalletInfo(ctx context.Context) error {
	eg, egCtx := errgroup.WithContext(ctx)

  info := &walletInfo{}
  
	eg.Go(func() error {
		isOnlyBankTransfer, err := h.contractor.UserUseOnlyBankTransferring(egCtx)
		if err != nil {
			return err
		}
		info.isOnlyBankTransfer = isOnlyBankTransfer
		return nil
	})

	eg.Go(func() error {
		walletInfo, err := h.billingAvitoGateway.GetWalletBalance(egCtx)
		if err != nil {
			return err
		}

		info.balance = walletInfo.Rub
		return nil
	})

	if err := eg.Wait(); err != nil {
		return nil, err
	}

	return info, nil
}
```





### Создание через `new()`

```go
g := new(errgroup.Group)
```

Без создания производного *Context* (как делает `WithContext()`).



### `WithContext()`

```go
func WithContext(ctx context.Context) (*Group, context.Context)
```

`WithContext()` возвращает новую *Group* и  *Context*, производный от `ctx`.

Производный *Context* отменяется по одному из событий, которое произойдет раньше:

- когда какая-либо функция `f`, переданная в `Go()` в первый раз возвращает *error*, отличный от нуля
- или когда функция `Wait()` возвращается в первый раз

### `Go()`

```go
func (g *Group) Go(f func() error)
```

`Go()` вызывает функцию `f()` в новой *goroutine*.

Когда происходит первый возврат *non-nil error* – вся *group* отменяется; эта *error* будет возвращена функцией `Wait()`. Отмена работы *group*'ы происходит через отмену *context*'а, как описано для функции [`WithContext()`](#withcontext).

### `Wait()`

```go
func (g *Group) Wait() error
```

`Wait()` блокирует до тех пор, пока не вернутся все вызовы функций из метода `Go()`, затем вернет первую *non-nil error* (если есть).