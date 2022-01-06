https://cf.avito.ru/pages/viewpage.action?pageId=74139827





# `net/http`

*Package* `net/http` предоставляет реализации *HTTP client* и *HTTP server*.

## HTTP client

### Закрытие *response body*

В документации сказано:

```go
type Response struct {
  // ...
  // Ответственность за выполнение `Close()` для Body лежит на caller'е. 
  // Default HTTP client's Transport не может переиспользовать HTTP/1.x "keep-alive" TCP connection's, 
  // если Body (1) не Read до конца И (!!!) (2) close. 
	Body io.ReadCloser
}
```

Т.е. обязательное условие переиспользования *connection*'ов:

1. вычитать `Body` до конца, до EOF
2. сделать по завершению `Body.Close()`

```go
resp, err := http.Get("http://example.com/")
if err != nil {
	// handle error
}
defer resp.Body.Close()							// (2) Закрыть response body
body, err := io.ReadAll(resp.Body)  // (1) Read до конца
// ...
```

Если не сделать (1) и (2), то произойдет *resource leak*, `Transport` не сможет переиспользовать постоянное *TCP connection* к серверу для последующего *"keep-alive" request*, *connection* будет оставаться открытым, *file descriptor* не будет освобожден.

Если не сделать (1), это приведет к открытию большого количества одновременных *connection*'s в состоянии `TIME_WAIT`. `TIME_WAIT` означает, что соединение *close*, но остались не вычитанные пакеты и соединение не может быть переиспользовано, чтобы на новые запросы не выдать старые ответы.

Соединение не будет использоваться повторно и может оставаться открытым, и в этом случае файловый дескриптор не будет освобожден.

Если не уверен в том, что ответ вычитан до конца – необходимо его вычитать через `io.Copy(ioutil.Discard, resp.Body)`:

```go
	resp, err := client.Do(r)
	if err != nil {
		return nil, err
	}
	defer func() {
		_ = resp.Body.Close()
	}()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		_, _ = io.Copy(ioutil.Discard, resp.Body)
    return nil, err
	}
```

`defer resp.Body.Close()` необходимо ставить после проверки ошибки `err != nil`, т.к. в этом случае чаще всего `resp.Body == nil`. Но даже если `resp.Body != nil` при ошибках оно будет автоматически закрыто нижележащим `Transport`. В случае *error* любой ответ можно игнорировать. `Response != nil && err != nil` возникает только при *fail* в `CheckRedirect()`, но в этом случае `Response.Body` уже закрыт.

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

Поэтому эти методы нельзя использовать в *production* коде, т.к. у них не установлен *timeout*.

Можно делать вызов метода `http.DefaultClient.Do()` для настройки *request*'а:

```go
  req, err := http.NewRequest("POST", ts.URL, strings.NewReader(body))
	if err != nil {
		log.Fatal(err)
	}
	resp, err := http.DefaultClient.Do(req)
```



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



## Constant

*HTTP method*'ы:

```go
const (
	MethodGet     = "GET"
	MethodHead    = "HEAD"
	MethodPost    = "POST"
	MethodPut     = "PUT"
	MethodPatch   = "PATCH" // RFC 5789
	MethodDelete  = "DELETE"
	MethodConnect = "CONNECT"
	MethodOptions = "OPTIONS"
	MethodTrace   = "TRACE"
)
```

*HTTP status cod*'ы:

```go
const (
 	StatusContinue = 100 // RFC 7231 , 6.2.1
 	StatusSwitchingProtocols = 101 // RFC 7231 , 6.2.2
 	StatusProcessing = 102 // RFC 2518 , 10.1
 	StatusEarlyHints = 103 // RFC 8297

 	StatusOK = 200 // RFC 7231 , 6.3. 1
 	StatusCreated = 201 // RFC 7231 , 6.3.2
 	StatusAccepted = 202 // RFC 7231 , 6.3.3
 	StatusNonAuthoritativeInfo = 203 // RFC 7231 , 6.3.4
	StatusNoContent = 204 // RFC 7231 , 6.3.5
 	StatusResetContent = 205 // RFC 7231 , 6.3.6
 	StatusPartialContent = 206 // RFC 7233 , 4.1
 	StatusMultiStatus = 207 // RFC 4918 , 11.1
 	StatusAlreadyReported = 208 // RFC 5842 , 7.1
 	StatusIMUsed = 226 // RFC 3229 , 10.4.1

 	StatusMultipleChoices = 300 // RFC 7231 , 6.4.1
 	StatusMovedPermanently = 301 // RFC 7231 , 6.4.2
	StatusFound = 302 // RFC 7231 , 6.4.3
 	StatusSeeOther = 303 // RFC 7231 , 6.4.4
 	StatusNotModified = 304 // RFC 7232 , 4.1
 	StatusUseProxy = 305 // RFC 7231 , 6.4.5

 	StatusTemporaryRedirect = 307 // RFC 7231 , 6.4.7
 	StatusPermanentRedirect = 308 // RFC 7538 , 3

 	StatusBadRequest = 400 // RFC 7231 , 6.5.1
 	StatusUnauthorized = 401 // RFC 7235 , 3.1
	StatusPaymentRequired = 402 // RFC 7231 , 6.5.2
 	StatusForbidden = 403 // RFC 7231 , 6.5.3
 	StatusNotFound = 404 // RFC 7231 , 6.5.4
 	StatusMethodNotAllowed = 405 // RFC 7231 ,
 	6.5.5 StatusNotAcceptable = 406 // RFC 7231 , 6.5.6
 	StatusProxyAuthRequired = 407 // RFC 7235 , 3.2
 	StatusRequestTimeout = 408 // RFC 7231 , 6.5.7
 	StatusConflict = 409 //RFC 7231 , 6.5.8
 	StatusGone = 410 // RFC 7231 , 6.5.9
 	StatusLengthRequired = 411 // RFC 7231 , 6.5.10
 	StatusPreconditionFailed = 412 // RFC 7232 , 4.2
 	StatusRequestEntityTooLarge = 413 // RFC 7231 , 6.5.11 StatusRequestURITooLong
 	= 414 // RFC 7231 , 6.5.12
 	StatusUnsupportedMediaType = 415 // RFC 7231 , 6.5.13 StatusRequestedRangeNotSatisfiable
 	= 416 // RFC 7233 , 4.4
	StatusExpectationFailed = 417 // RFC 7231 , 6.5.14
 	StatusTeapot = 418 // RFC 7168 , 2.3.3
 	StatusMisdirectedRequest = 421 // RFC 7540 , 9.1.2
 	StatusUnprocessableEntity = 422 // RFC 4918 , 11.2
 	StatusLocked = 423 // RFC 4918 , 11.3
 	StatusFailedDependency = 424 // RFC 4918 , 11.4
 	StatusTooEarly = 425 // RFC 8470 , 5.2. 
	StatusUpgradeRequired = 426 //RFC 7231 , 6.5.15
 	StatusPreconditionRequired = 428 // RFC 6585 , 3
 	StatusTooManyRequests = 429 // RFC 6585 , 4
 	StatusRequestHeaderFieldsTooLarge = 431 // RFC 6585 , 5
 	StatusUnavailableForLegalReasons = 451 // RFC 7725 , 3

 	StatusInternalServerError = 500 // RFC 7231 , 6.6.1
 	StatusNotImplemented = 501 // RFC 7231 , 6.6.2
 	StatusBadGateway = 502 // RFC 7231 , 6.6.3
	StatusServiceUnavailable = 503 // RFC 7231 , 6.6.4
 	StatusGatewayTimeout = 504 // RFC 7231 , 6.6.5
 	StatusHTTPVersionNotSupported = 505 // RFC 7231 , 6.6.6
 	StatusVariantAlsoNegotiates = 506 // RFC 2295 , 8.1
 	StatusInsufficientStorage = 507 // RFC 4918 , 11.5
 	StatusLoopDetected = 508 // RFC 5842 , 7.2
 	StatusNotExtended = 510 // RFC 2774 , 7
 	StatusNetworkAuthenticationRequired = 511 //RFC 6585 , 6
 )
```





## Variable

### `DefaultClient`

```go
var DefaultClient = &Client{}
```

`DefaultClient` – `http.Client` по умолчанию, который используется функциями `Get()`, `Head()` и `Post()`.





## `type Client`

```go
type Client struct {
	// Transport определяет механизм, с помощью которого выполняются отдельные
  // HTTP requests. 
  //
  // Если nil, то используется DefaultTransport.
	Transport RoundTripper

  // CheckRedirect указывает политику для обработки redirect'ов. 
	// Если CheckRedirect != nil, клиент вызывает его перед тем, 
  // как выполнить HTTP redirect. Аргументы req и via - это 
	// предстоящий request и уже сделанные request'ы, самые старые перечислены вначале. 
  // Если CheckRedirect возвращает error, Client's Get() метод 
  // возвращает предыдущий Response (с закрытым Body)
	// и CheckRedirect's error (завернутую в url.Error)
  // вместо отправки Request req. 
	// В качестве особого случая, если CheckRedirect возвращает ErrUseLastResponse, 
	// возвращается самый последний Response с незакрытым Body 
	// и nil error
  //
	// Если CheckRedirect = nil, Client использует свою политику по умолчанию,
	// по который он останавливается после 10 последовательных request'ов. 
	CheckRedirect func(req *Request, via []*Request) error

  // Jar задает cookie jar (подробней в type CookieJar) 
	// 
	// Jar используется для вставки соответствующих cookie's в каждый 
	// исходящий Request и обновляется значениями cookie 
	// каждого входящего Response. Jar используется для каждого 
	// redirect'а, за которым следует Client. 
	// 
	// Если Jar == nil, cookie's отправляются, только если они явно 
	// установлены в Request'е. 
	Jar CookieJar

  // Timeout указывает time limit для request'ов, сделанных этим
	// Client'ом. Timeout включает connection time, любые 
	// redirect'ы и чтение response body. Timer продолжает 
  // работать после возврата из Get(), Head(), Post() или Do() 
  // и будет прерывать чтение Response.Body (т.е. timeout включает также чтение Response.Body)
	// 
	// Нулевое значение Timeout'а означает отсутствие timeout'а 
	// 
	// Client отменяет request'ы к нижележащему Transport'у 
	// как если бы Request's Context завершился. 
	// 
  // Для совместимости, Client также будет использовать deprecated method `CancelRequest()` для Transport
  // если он будет найден. Новые реализации RoundTripper должны использовать Request's Context
  // для cancel вместо реализации CancelRequest(). 
	Timeout time.Duration
}
```

`Client` – это HTTP-клиент. Его *zero value* ([`DefaultClient`](#defaultclient)) – это готовый *Client*, который использует `DefaultTransport`.

`Client`'s `Transport` обычно имеет внутреннее состояние (закэшированные *TCP connection*'s), поэтому `Client` следует использовать повторно, а не создавать по мере необходимости. `Client` безопасен для конкурентного использования несколькими *goroutine*'s.

`Client` находится на более высоком уровне, чем `RoundTripper` (и его реализация, `Transport`), и дополнительно обрабатывает детали HTTP, такие как *cookie* и *redirect*'ы.

TODO!!!

### `Do()`

```go
func (c *Client) Do(req *Request) (*Response, error)
```

`Do()` отправляет *HTTP reques*'t и возвращает *HTTP response*, следуя политике (например, *redirect*'ы, *cookie*'s, *auth*), настроенной для `Client`.

`error` может быть вызвана политикой `Client` (например, `CheckRedirect`) или проблемами при взаимодействии по HTTP (например, проблемой сетевого подключения). *non-2xx status code* не приводит к `error`.

Если `error == nil`, то *response* будет содержать *non-nil* `Body`. *Caller* должен сделать для него `Close()`. 

[Подробнее про закрытие *response body*](#закрытие-response-body).

Обычно вместо `Do()` используются `Get()`, `Post()` или `PostForm()`.

Если сервер отвечает *redirect*'ом, `Client` сначала использует функцию `CheckRedirect()`, чтобы определить, следует ли выполнять *redirect*. Если *redirect* разрешен, то *redirect* с кодами `301`, `302` или `303` приводит к последующему *request*'у с *method*'ом `GET` (или `HEAD`, если исходный *request* был `HEAD`) без *body*. `307` или `308` *redirect* сохраняет исходный *method* и *body*, если определена функция `Request.GetBody()`. Функция `NewRequest()` автоматически устанавливает `GetBody()` для стандартных типов *body* библиотеки.

Возвращенный `error` будет иметь тип `*url.Error`. `error.Timeout() == true`, если случился *timeout*.



### `Get()`

```go
func (c *Client) Get(url string) (resp *Response, err error)
```

`Get()` делает `GET` *request* на указанный URL. Если *response* – один из следующих *redirect code*, `Get()` выполняет *redirect* после вызова функции `Client.CheckRedirect()`:

```
301 (Moved Permanently)
302 (Found)
303 (See Other)
307 (Temporary Redirect)
308 (Permanent Redirect)
```

`error` возвращается, если функция `Client.CheckRedirect()` возвращает `error`, или при проблемах во взаимодействии по HTTP (например, проблемой сетевого подключения). *non-2xx status code* не приводит к `error`.

Возвращенный `error` будет иметь тип `*url.Error`. `error.Timeout() == true`, если случился *timeout*.

Если `error == nil`, то *response* будет содержать *non-nil* `Body`. *Caller* должен сделать для него `Close()`. 

[Подробнее про закрытие *response body*](#закрытие-response-body).

Чтобы сделать *request* с *custom header*'s, используйте `NewRequest()` и `Client.Do()`.

Чтобы сделать *request* с указанным `context.Context`, используйте `NewRequestWithContext()` и `Client.Do()`.





### `Head()`

```go
func (c *Client) Head(url string) (resp *Response, err error)
```

`Head()` делает `HEAD` *request* на указанный URL. Если *response* – один из следующих *redirect code*, `Head()` выполняет *redirect* после вызова функции `Client.CheckRedirect()`:

```
301 (Moved Permanently)
302 (Found)
303 (See Other)
307 (Temporary Redirect)
308 (Permanent Redirect)
```

Чтобы сделать *request* с указанным `context.Context`, используйте `NewRequestWithContext()` и `Client.Do()`.



### `Post()`

```go
func (c *Client) Post(url, contentType string, body io.Reader) (resp *Response, err error)
```

`Post()` делает `POST` *request* на указанный URL.

*Caller* должен сделать `resp.Body.Close()` после завершения чтения.

Если `body` является `io.Closer`, оно *closed* после *request*'а (??? само или его нужно *close*).

Чтобы сделать *request* с *custom header*'s, используйте `NewRequest()` и `Client.Do()`.

Чтобы сделать *request* с указанным `context.Context`, используйте `NewRequestWithContext()` и `Client.Do()`.

См. документацию по методу `Client.Do()` для получения подробной информации о том, как обрабатываются *redirect*'ы.



### `PostForm()`

```go
func (c *Client) PostForm(url string, data url.Values) (resp *Response, err error)
```

`PostForm()` отправляет `POST` *request* на указанный URL с *key*'s и *value*'s из `data`, закодированными в URL-encoded как request body.

Устанавливается *header* `Content-Type: application/x-www-form-urlencoded`. Чтобы установить другие *header*'ы, используйте `NewRequest()` и `Client.Do()`.

Если `error == nil`, то *response* будет содержать *non-nil* `Body`. *Caller* должен сделать для него `Close()`. 

См. документацию по методу `Client.Do()` для получения подробной информации о том, как обрабатываются *redirect*'ы.

Чтобы сделать *request* с указанным `context.Context`, используйте `NewRequestWithContext()` и `Client.Do()`.



## `type Header`

```go
type Header map[string][]string
```

`Header` представляет *key-value* пары в *HTTP header*'е.

*Key*'s должны иметь стандартизованную форму, в соответствии с тем как их возвращает `CanonicalHeaderKey`.



### `Add()`

```go
func (h Header) Add(key, value string)
```

`Add()` добавляет *key-value* пару в `Header`. Т.к. `Header` – это *map* из *slic*'ов (!!!). К каждому *key* может привязываться несколько *value*'s. И каждое очередное *value* добавляется в этот *slice*. 

*Key* – *case insensitive* (нечувствителен к регистру), он стандартизован через функцию `CanonicalHeaderKey()`.

Пример *request*'а с двумя одинаковыми *header*'ами:

```go
  h.Header.Add("X-Request", "1")
  h.Header.Add("X-Request", "2")

  res, _ := httputil.DumpRequest(h, true)
  fmt.Print(string(res))

  // GET / HTTP/1.1
  // Host: www.xxx.ru
  // X-Request: 1
  // X-Request: 2
```





## `type Request`

```go
type Request struct {
	// Method определяет HTTP method (GET, POST, PUT и т. д.). 
  // Для клиентских request'ов пустая string означает GET. 
	Method string

  
  // Для request'ов, которые приходят к server'у (server request) – URL указывает запрашиваемый URI
  // Для request'ов, которые исходят от client'а (client request) – это URL, к которому необходимо обратиться
  //
	// Для server request'ов, URL парсится из URI 
	// указанного в Request-Line, который сохранен в RequestURI. Для 
	// большинства request'ов все поля, кроме Path и RawQuery, будут пустыми. 
	// 
	// Для client request'ов, URL's Host указывает server, 
	// к которому нужно подключиться, а Request's Host (опционально) 
	// указывает значение Host header, который будет отправлен в HTTP request'е
	URL *url.URL

	// The protocol version for incoming server requests.
	//
	// For client requests, these fields are ignored. The HTTP
	// client code always uses either HTTP/1.1 or HTTP/2.
	// See the docs on Transport for details.
	Proto      string // "HTTP/1.0"
	ProtoMajor int    // 1
	ProtoMinor int    // 0

  
  // Header содержит request header's, которые получены server'ом или будут отправлены клиентом. 
  // Если сервер получил request с header line's:
  // 
	//	Host: example.com
	//	accept-encoding: gzip, deflate
	//	Accept-Language: en-us
	//	fOO: Bar
	//	foo: two
  // 
  // тогда:
  //
  // Header = map[string][]string{
	//		"Accept-Encoding": {"gzip, deflate"},
	//		"Accept-Language": {"en-us"},
	//		"Foo": {"Bar", "two"},
	//	}
	// 
  // При создании request'а через `NewRequest()` поле `Header` инициализируется пустой map.
  //
	// Для входящих request'ов – header `Host` перемещается 
	// в поле `Request.Host` и удаляется из map'ы `Header`.
  // 
  // В стандарте HTTP указано, что header name – case-insensitive (нечувствителен к регистру). 
  // При парсинге header name приводится с помощью функции CanonicalHeaderKey() –  
	// делает первый символ и символы, следующие за дефисом – верхним регистром,
  // а остальные символы – нижним регистром. 
	// 
	// Для client request'ов, некоторые header'ы, такие как Content-Length
  // и Connection автоматически записываются при необходимости, и
  // значения в `Header` могут быть проигнорированы.
	Header Header

	// Body is the request's body.
	//
	// For client requests, a nil body means the request has no
	// body, such as a GET request. The HTTP Client's Transport
	// is responsible for calling the Close method.
	//
	// For server requests, the Request Body is always non-nil
	// but will return EOF immediately when no body is present.
	// The Server will close the request body. The ServeHTTP
	// Handler does not need to.
	//
	// Body must allow Read to be called concurrently with Close.
	// In particular, calling Close should unblock a Read waiting
	// for input.
	Body io.ReadCloser

	// GetBody defines an optional func to return a new copy of
	// Body. It is used for client requests when a redirect requires
	// reading the body more than once. Use of GetBody still
	// requires setting Body.
	//
	// For server requests, it is unused.
	GetBody func() (io.ReadCloser, error)

	// ContentLength records the length of the associated content.
	// The value -1 indicates that the length is unknown.
	// Values >= 0 indicate that the given number of bytes may
	// be read from Body.
	//
	// For client requests, a value of 0 with a non-nil Body is
	// also treated as unknown.
	ContentLength int64

	// TransferEncoding lists the transfer encodings from outermost to
	// innermost. An empty list denotes the "identity" encoding.
	// TransferEncoding can usually be ignored; chunked encoding is
	// automatically added and removed as necessary when sending and
	// receiving requests.
	TransferEncoding []string

	// Close indicates whether to close the connection after
	// replying to this request (for servers) or after sending this
	// request and reading its response (for clients).
	//
	// For server requests, the HTTP server handles this automatically
	// and this field is not needed by Handlers.
	//
	// For client requests, setting this field prevents re-use of
	// TCP connections between requests to the same hosts, as if
	// Transport.DisableKeepAlives were set.
	Close bool

	// For server requests, Host specifies the host on which the
	// URL is sought. For HTTP/1 (per RFC 7230, section 5.4), this
	// is either the value of the "Host" header or the host name
	// given in the URL itself. For HTTP/2, it is the value of the
	// ":authority" pseudo-header field.
	// It may be of the form "host:port". For international domain
	// names, Host may be in Punycode or Unicode form. Use
	// golang.org/x/net/idna to convert it to either format if
	// needed.
	// To prevent DNS rebinding attacks, server Handlers should
	// validate that the Host header has a value for which the
	// Handler considers itself authoritative. The included
	// ServeMux supports patterns registered to particular host
	// names and thus protects its registered Handlers.
	//
	// For client requests, Host optionally overrides the Host
	// header to send. If empty, the Request.Write method uses
	// the value of URL.Host. Host may contain an international
	// domain name.
	Host string

	// Form contains the parsed form data, including both the URL
	// field's query parameters and the PATCH, POST, or PUT form data.
	// This field is only available after ParseForm is called.
	// The HTTP client ignores Form and uses Body instead.
	Form url.Values

	// PostForm contains the parsed form data from PATCH, POST
	// or PUT body parameters.
	//
	// This field is only available after ParseForm is called.
	// The HTTP client ignores PostForm and uses Body instead.
	PostForm url.Values

	// MultipartForm is the parsed multipart form, including file uploads.
	// This field is only available after ParseMultipartForm is called.
	// The HTTP client ignores MultipartForm and uses Body instead.
	MultipartForm *multipart.Form

	// Trailer specifies additional headers that are sent after the request
	// body.
	//
	// For server requests, the Trailer map initially contains only the
	// trailer keys, with nil values. (The client declares which trailers it
	// will later send.)  While the handler is reading from Body, it must
	// not reference Trailer. After reading from Body returns EOF, Trailer
	// can be read again and will contain non-nil values, if they were sent
	// by the client.
	//
	// For client requests, Trailer must be initialized to a map containing
	// the trailer keys to later send. The values may be nil or their final
	// values. The ContentLength must be 0 or -1, to send a chunked request.
	// After the HTTP request is sent the map values can be updated while
	// the request body is read. Once the body returns EOF, the caller must
	// not mutate Trailer.
	//
	// Few HTTP clients, servers, or proxies support HTTP trailers.
	Trailer Header

	// RemoteAddr allows HTTP servers and other software to record
	// the network address that sent the request, usually for
	// logging. This field is not filled in by ReadRequest and
	// has no defined format. The HTTP server in this package
	// sets RemoteAddr to an "IP:port" address before invoking a
	// handler.
	// This field is ignored by the HTTP client.
	RemoteAddr string

	// RequestURI is the unmodified request-target of the
	// Request-Line (RFC 7230, Section 3.1.1) as sent by the client
	// to a server. Usually the URL field should be used instead.
	// It is an error to set this field in an HTTP client request.
	RequestURI string

	// TLS allows HTTP servers and other software to record
	// information about the TLS connection on which the request
	// was received. This field is not filled in by ReadRequest.
	// The HTTP server in this package sets the field for
	// TLS-enabled connections before invoking a handler;
	// otherwise it leaves the field nil.
	// This field is ignored by the HTTP client.
	TLS *tls.ConnectionState

	// Cancel is an optional channel whose closure indicates that the client
	// request should be regarded as canceled. Not all implementations of
	// RoundTripper may support Cancel.
	//
	// For server requests, this field is not applicable.
	//
	// Deprecated: Set the Request's context with NewRequestWithContext
	// instead. If a Request's Cancel field and context are both
	// set, it is undefined whether Cancel is respected.
	Cancel <-chan struct{}

	// Response is the redirect response which caused this request
	// to be created. This field is only populated during client
	// redirects.
	Response *Response
	// contains filtered or unexported fields
}
```

`Request` описывает *HTTP request*, полученный server'ом или отправленный *client*'ом.

Смысл полей немного отличается при использовании *client*'а и *server*'а. 

### `NewRequest()`

```go
func NewRequest(method, url string, body io.Reader) (*Request, error)
```

`NewRequest()` – обертка для `NewRequestWithContext()` с `ctx = context.Background()`.



### `NewRequestWithContext()`

```go
func NewRequestWithContext(ctx context.Context, method, url string, body io.Reader) (*Request, error)
```

`NewRequestWithContext()` возвращает новый `Request` с указанным методом (`method`), URL-адресом (`url`) и опциональным *body* (`body`).

Если `body` также является `io.Closer`, возвращенный `Request.Body` устанавливается для *body* (??? *body request*'а ???) и будет *close* (??? автоматически)  методами `Client.Do()`, `Client.Post()` и `Client.PostForm()` и `Transport.RoundTrip()`.

`NewRequestWithContext()` возвращает `Request()`, подходящий для использования с методами `Client.Do()` или `Transport.RoundTrip()`. 

Чтобы создать *request* для использования с тестированием *Server Handler* необходимо использовать:

- функцию `NewRequest()` из `net/http/httptest` *package*
- использовать `ReadRequest`
- или вручную обновить поля `Request`. 

Для исходящего *client request*, *context* контролирует все время существования *request*'а и его *response*: установку *connection*, отправку *request*'а и чтение *response*.

Если *body* имеет тип `*bytes.Buffer`, `bytes.Reader` или `strings.Reader`, `ContentLength` возвращенного *request*'а устанавливается в его точное значение (вместо `-1`), `GetBody()` заполняется (поэтому 307 и 308 *redirect*'ы могут воспроизводить *body*). `Body` устанавливается в `NoBody`, если `ContentLength` равно 0.



## `type Response`

```go
type Response struct {
	Status     string // e.g. "200 OK"
	StatusCode int    // e.g. 200
	Proto      string // e.g. "HTTP/1.0"
	ProtoMajor int    // e.g. 1
	ProtoMinor int    // e.g. 0

	// Header maps header keys to values. If the response had multiple
	// headers with the same key, they may be concatenated, with comma
	// delimiters.  (RFC 7230, section 3.2.2 requires that multiple headers
	// be semantically equivalent to a comma-delimited sequence.) When
	// Header values are duplicated by other fields in this struct (e.g.,
	// ContentLength, TransferEncoding, Trailer), the field values are
	// authoritative.
	//
	// Keys in the map are canonicalized (see CanonicalHeaderKey).
	Header Header

	// Body представляет собой response body. 
	// 
	// Response body вычитывается по запросу при чтении	поля Body
  // Если network connection fail'ится или server завершает передачу response, 
  // вызов `Body.Read()` возвращает error. 
	// 
	// `Client` и `Transport` гарантируют, что `Body` – всегда non-nil,
  // даже для response без body или response с body нулевой длины. 
	// 
  // Подробнее про [закрытие response body](#закрытие-response-body)
	// 
	// `Body` автоматически dechunk'ается, если server
	// ответил "chunked" Transfer-Encoding.
	// 
	// Начиная с Go 1.12, `Body` также будет реализовывать `io.Writer` 
	// при успешном response «101 Switching Protocols», 
	// который используется WebSocket'ами и режимом «h2c» HTTP/2. 
	Body io.ReadCloser

	// ContentLength records the length of the associated content. The
	// value -1 indicates that the length is unknown. Unless Request.Method
	// is "HEAD", values >= 0 indicate that the given number of bytes may
	// be read from Body.
	ContentLength int64

	// Contains transfer encodings from outer-most to inner-most. Value is
	// nil, means that "identity" encoding is used.
	TransferEncoding []string

	// Close records whether the header directed that the connection be
	// closed after reading Body. The value is advice for clients: neither
	// ReadResponse nor Response.Write ever closes a connection.
	Close bool

	// Uncompressed reports whether the response was sent compressed but
	// was decompressed by the http package. When true, reading from
	// Body yields the uncompressed content instead of the compressed
	// content actually set from the server, ContentLength is set to -1,
	// and the "Content-Length" and "Content-Encoding" fields are deleted
	// from the responseHeader. To get the original response from
	// the server, set Transport.DisableCompression to true.
	Uncompressed bool

	// Trailer maps trailer keys to values in the same
	// format as Header.
	//
	// The Trailer initially contains only nil values, one for
	// each key specified in the server's "Trailer" header
	// value. Those values are not added to Header.
	//
	// Trailer must not be accessed concurrently with Read calls
	// on the Body.
	//
	// After Body.Read has returned io.EOF, Trailer will contain
	// any trailer values sent by the server.
	Trailer Header

	// Request is the request that was sent to obtain this Response.
	// Request's Body is nil (having already been consumed).
	// This is only populated for Client requests.
	Request *Request

	// TLS contains information about the TLS connection on which the
	// response was received. It is nil for unencrypted responses.
	// The pointer is shared between responses and should not be
	// modified.
	TLS *tls.ConnectionState
}
```

`Response` представляет собой *response* на *HTTP request*.

`Client` и `Transport` возвращают `Response` от *server*'а сразу после получения *response header*'ов. *Response body* вычитывается по запросу во время чтении поля `Body`.





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
В зависимости от версии протокола HTTP и клиента, вызов `Write()` или `WriteHeader()` может предотвратить чтение `Request.Body` в будущем. Для *HTTP/1.x request*'ов, *handler*'ы должны читать любые необходимы данные *request body* перед записью *response* . После того как *header*'s были *flush*'d (из-за явного вызова `Flusher.Flush()` или записи достаточного количества данных для запуска *flush*'а) *request body* может быть недоступно. 
	
	
	
	
	
	
	
	
	
	

## `type RoundTripper`

TODO!!!

```go
var DefaultTransport RoundTripper = &Transport{
	Proxy: ProxyFromEnvironment,
	DialContext: (&net.Dialer{
		Timeout:   30 * time.Second,
		KeepAlive: 30 * time.Second,
	}).DialContext,
	ForceAttemptHTTP2:     true,
	MaxIdleConns:          100,
	IdleConnTimeout:       90 * time.Second,
	TLSHandshakeTimeout:   10 * time.Second,
	ExpectContinueTimeout: 1 * time.Second,
}
```

`DefaultTransport` – это реализация `Transport` *by default*, используемая `DefaultClient`. Он устанавливает сетевые соединения по мере необходимости и кэширует их для повторного использования при последующих вызовах. Он использует *HTTP proxy*'s в соответствии с указаниями *environment variable*'s `$HTTP_PROXY` и `$NO_PROXY`.









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



