https://golang.cafe/blog/golang-httptest-example.html

https://stackoverflow.com/questions/55890124/what-is-the-difference-between-client-http-request-and-server-http-request-in-go

https://stackoverflow.com/questions/45682353/httptest-newrequest-vs-http-newrequest-which-one-to-use-in-tests-and-why

https://stackoverflow.com/questions/51256080/serve-multiple-handlers-with-httptest-to-mock-multiple-requests

https://stackoverflow.com/questions/16154999/how-to-test-http-calls-in-go-using-httptest

https://medium.com/what-i-talk-about-when-i-talk-about-technology/go-code-examples-httptest-newserver-f965fb349884

https://deliveroo.engineering/2018/09/11/testing-with-third-parties-in-go.html





# `net/httptest`

Сценарии, в которых будет полезен:

- тестирование *handler*'ов для *http server*'ов – позволяет *mock*'ать *response writer*, что передать его в *server handler*.
- тестирование *http client*'ов – позволяет *mock*'ать *HTTP server*



## Тестирование *handler*'ов для *http server*'ов

TODO!!!



## Тестирование *http client*'ов

### *mock* сервера

*Package* `net/httptest` позволяет выполнить *mock* сервера. Для этого используется [`type Server`](#type-server). 

Пример *mock* сервера, который возвращает строку:

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"net/http/httptest"
)

func main() {
	ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello, client")
	}))
	defer ts.Close()

	res, err := http.Get(ts.URL)
	if err != nil {
		log.Fatal(err)
	}
	greeting, err := io.ReadAll(res.Body)
	res.Body.Close()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%s", greeting)
}
```

### Тестирование client'а

Пример типичного *http client*'а. Он ходит в ручку `/upper?word=xxx`, чтобы получить слово в верхнем регистре `XXX`.

```go
package main

import (
    "io/ioutil"
    "net/http"

    "github.com/pkg/errors"
)

type Client struct {
    url string
}

func NewClient(url string) Client {
    return Client{url}
}

func (c Client) UpperCase(word string) (string, error) {
    res, err := http.Get(c.url + "/upper?word=" + word)
    if err != nil {
        return "", errors.Wrap(err, "unable to complete Get request")
    }
    defer res.Body.Close()
    out, err := ioutil.ReadAll(res.Body)
    if err != nil {
        return "", errors.Wrap(err, "unable to read response data")
    }

    return string(out), nil
}
```

`url` должен инджектиться в `Client`.

Пример теста для client'а:

```go
package main

import (
    "fmt"
    "net/http"
    "net/http/httptest"
    "strings"
    "testing"
)

func TestClientUpperCase(t *testing.T) {
    expected := "dummy data"
    /* Создание mock'а сервера, который на любые ручки всегда возвращает строку `expected` */
    svr := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, expected)
    }))
    defer svr.Close()
  
    // используем url адрес server'а для client'а 
    c := NewClient(svr.URL)
    res, err := c.UpperCase("anything")
    if err != nil {
        t.Errorf("expected err to be nil got %v", err)
    }
    // res: expected\r\n
    // due to the http protocol cleanup response
    res = strings.TrimSpace(res)
    if res != expected {
        t.Errorf("expected res to be %s got %s", expected, res)
    }
}
```







# Структура

## `type Server`

`type Server` – локальный HTTP сервер, который слушает (*listen*) выбранный системой *port* на локальном *loopback interface* (???интерфейс обратной связи), для которого можно описать логику handler'а для использования в *end-to-end HTTP test*'ах (??? почему e2e???).

```go
type Server struct {
    URL      string // базовый URL в форме http://ipaddr:port без trailing slash
    Listener net.Listener

    // EnableHTTP2 controls whether HTTP/2 is enabled
    // on the server. It must be set between calling
    // NewUnstartedServer and calling Server.StartTLS.
    EnableHTTP2 bool // Go 1.14

    // TLS is the optional TLS configuration, populated with a new config
    // after TLS is started. If set on an unstarted server before StartTLS
    // is called, existing fields are copied into the new config.
    TLS *tls.Config

    // Config may be changed after calling NewUnstartedServer and
    // before Start or StartTLS.
    Config *http.Server
    // contains filtered or unexported fields
}
```



### `NewServer()`

```go
func NewServer(handler http.Handler) *Server
```

`NewServer()` стартует и возвращает новый `Server`. *Caller* должен вызвать `Close()` при завершении (в `defer`), чтобы *close* его.



### `Close()`

```go
func (s *Server) Close()
```

`Close()` блокирует *server* до тех пор, пока не будут выполнены все незавершенные запросы, и останавливает его.





