https://cf.avito.ru/pages/viewpage.action?pageId=74139827





# `net/http`

*Package* `net/http` предоставляет реализации *HTTP client* и *HTTP server*.

## HTTP client

*Client* должен закрыть *response body* при завершении:

```go
resp, err := http.Get("http://example.com/")
if err != nil {
	// handle error
}
defer resp.Body.Close()
body, err := io.ReadAll(resp.Body)
// ...
```

### Запросы через `http.DefaultClient`

Для отправки *HTTP(S) request* могут использоваться функции (НЕ методы):

- `Get()`
- `Head()`
- `Post()`
- `PostForm()`

```go
resp, err := http.Get("http://example.com/")
...
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})
```

Фактически, это функции являются псевдонимом для методов переменной `DefaultClient` ([link](#defaultclient))

Поэтому эти методы нельзя использовать в production коде, т.к. у них не установлен timeout.



### Запросы через кастомный `http.Client`

Для более глубокой настройки (header, redirect policy, timeout и др.) необходимо использовать `type Client`:

#### Использование `Client.Get()`, `Client.Post()`...

Позволяет сконфигурировать `http.Client` (например, `Timeout`) и сделать запрос с методом GET, POST, ...

```go
client := &http.Client{
	Timeout: 10 * time.Second,
}

resp, err := client.Get("http://example.com")
```

#### Сконфигурировать Request

Можно сконфигурировать `http.Request` и сделать запрос через метод `Client.Do()`:

```go
// создание http.Cliebt
client := &http.Client{
	Timeout: 10 * time.Second,
}

// создание http.Request
req, err := http.NewRequest("GET", "http://example.com", nil)
// ...
req.Header.Add("If-None-Match", `W/"wyzzy"`)

// запрос, параметр – http.Request
resp, err := client.Do(req)
// ...
```

#### Настройка HTTP транспорта

Для управления HTTP транспортом (proxy, TLS configuration, keep-alive, compression, ...)  `http.Client`  имеет поле `Client.Transport`:

```go
tr := &http.Transport{
	MaxIdleConns:       10,
	IdleConnTimeout:    30 * time.Second,
	DisableCompression: true,
}
client := &http.Client{Transport: tr}
resp, err := client.Get("https://example.com")
```

`http.Client` и `http.Transport` безопасны для одновременного использования несколькими *goroutine*'s, и для повышения эффективности их следует создавать только один раз и повторно использовать.

### Стандартная структура клиента

Пакет с клиентом к микросервису обычно выглядит так:

```go
// Структура клиента сервиса
type Client struct {
	url        string
	httpClient *http.Client
}

// Конструктор клиента
func NewClient(url string, timeout time.Duration) *Client {
	httpClient := &http.Client{
	Timeout: timeout,
}
	return &Client{url: url, httpClient: httpClient}
}
```



дальше [link](../../Design.md#functional-options)

# Структура



## Variable

### `DefaultClient`

```go
var DefaultClient = &Client{}
```

`DefaultClient` – `http.Client` по умолчанию, который используется функциями `Get()`, `Head()` и `Post()`.



## `type ResponseWriter`

*Interface* `ResponseWriter` используется *HTTP handler*'ом для построения *HTTP response*.

`ResponseWriter` нельзя использовать после того как метод `Handler.ServeHTTP()` сделал `return`.

```go
type ResponseWriter interface {
	Header() Header

	Write([]byte) (int, error)

	WriteHeader(statusCode int)
}
```

### `Header()`

Пример добавления *header*'а в *response*:

```go
package main

import (
	"io"
	"net/http"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/sendstrailers", func(w http.ResponseWriter, req *http.Request) {
		w.Header().Set("Content-Type", "text/plain; charset=utf-8") // normal header
		w.WriteHeader(http.StatusOK)

		io.WriteString(w, "This HTTP response has both headers before this text and trailers at the end.\n")
	})
}

```





### `Write()`

```go
Write([]byte) (int, error)
```

`Write()` записывает данные в *connection* как часть *HTTP reply*. Если `WriteHeader()` еще не был вызван, `Write()` вызывает `WriteHeader(http.StatusOK)` перед записью данных. Если заголовок `Header` не содержит строку `Content-Type`, `Write()` добавляет `Content-Type` к результату передачи начальных 512 байтов записанных данных в `DetectContentType()`. Кроме того, если общий размер всех записываемых данных меньше нескольких КБ и нет вызовов `Flush()` – заголовок `Content-Length` добавляется автоматически. 
В зависимости от версии протокола HTTP и клиента, вызов `Write()` или `WriteHeader()` может предотвратить чтение `Request.Body` в будущем. Для *HTTP/1.x request*'ов, *handler*'ы должны читать любые необходимы данные *request body* перед записью *response* . После того как *header*'s были *flush*'d
(из-за явного вызова `Flusher.Flush()` или записи достаточного количества данных для запуска *flush*'а) *request body* может быть недоступно. 
	
	
	
	
	
	
	
	
	
	









## `type ServeMux`

`ServeMux` – это *multiplexer* для *HTTP request*'ов. Он матчит URL каждого входящего *request*'а со списком зарегистрированных *pattern*'ов и вызывает *handler* для *pattern*'а, который наиболее точно матчится с URL-адресом.

*Pattern*'ами называются:

- фиксированные пути, начинающиеся с `/`. Например, `/favicon.ico`
- *subtree*, начинающиеся с `/` и заканчивающиеся на `/`. Например, `/images/`. 

Более длинные *pattern*'ы имеют приоритет над более короткими. 

Например *handler*'ы:

- для `/images/thumbnail/` – будет вызыван для путей, начинающихся с `/images/thumbnail/`
- для `/images/` – будет вызван для всех остальных путей в *subtree* `/images/`.

Паттерн `/` – матчится на всех пути, которые не сматчились на другие зарегистрированные *pattern*'ы.

Если *subtree* зарегистрировано и получен *request* с указанием *subtree* без завершающего `/`, `ServeMux` перенаправляет (редиректит?) этот *request* в *subtree* (добавляя конечный `/`). Это поведение можно изменить отдельной регистрацией пути без `/` в конце. Например, регистрация `/images/` заставляет `ServeMux` перенаправить запрос `/images` на `/images/`, если `/images` не был зарегистрирован отдельно.

*Pattern*'ы могут опционально начинаться с имени хоста, ограничивая совпадения URL-адресами только на этом хосте. *Pattern*'ы для хоста имеют приоритет над общими *pattern*'ами.

`ServeMux` также выполняет *sanitize* для *URL path* и заголовка `Host`, удаляя номер *port*'а и перенаправляя любой *request*, содержащий `.` или `..`  или повторяющиеся `//` на эквивалентный чистый URL.

```go
type ServeMux struct {
    // contains filtered or unexported fields
}
```



### `NewServeMux()`

```go
func NewServeMux() *ServeMux
```

`NewServeMux()` делает *allocate* и возвращает новый `ServeMux`.



### `HandleFunc()`

```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```

`HandleFunc()` регистрирует функцию-*handler* для данного *pattern*'а.



