# `net/url`

*Package* `url` парсит URL'ы и реализует *query escaping*.



## Type

### `type URL`

```go
type URL struct {
	Scheme      string
  Opaque      string    // закодированные opaque data (???)
	User        *Userinfo // username и password information
	Host        string    // host или host:port
	Path        string    // path (относительный path может опускать ведущий slash)
  RawPath     string    // закодированный path hint (смотри EscapedPath() method)
	ForceQuery  bool      // присоединить query ('?') даже если RawQuery пуст
	RawQuery    string    // закодированные query values, без '?' 
  Fragment    string    // fragment for references, без '#' (это часть типа `#anchor`)
  RawFragment string    // закодированный fragment hint (смотри EscapedFragment() method)
}
```

`URL` представляет собой распарсенный URL (технически, *URI reference*).

Формат URL:

```
[scheme:][//[userinfo@]host][/]path[?query][#fragment]
```

URL-адрес, которые не начинается с *slash*'а после `scheme`, интерпретируются как:

```
scheme:opaque[?query][#fragment]
```

Поле `Path` хранится в декодированной форме – например, там будет записано `/Go/`, а не `/%47%6f%2f`. Поэтому невозможно определить, какие *slash*'и в `Path` были *slash*'ами в raw URL, а какие были `%2f`. Это различие редко бывает важным, но когда это так, код должен использовать `RawPath`, опциональное поле, которое устанавливается только в том случае, если *default encoding* отличается от `Path` (????).

Метод `URL.String()` использует метод `EscapedPath()` для получения *path*. Подробнее смотрите метод `EscapedPath()`.

<u>Пример:</u>

```go
func main() {
	u, err := url.Parse("http://bing.com/search?q=dotnet")
	if err != nil {
		log.Fatal(err)
	}
	u.Scheme = "https"
	u.Host = "google.com"
	q := u.Query()
	q.Set("q", "golang")
	u.RawQuery = q.Encode()
	fmt.Println(u)				// https://google.com/search?q=golang
}
```



#### `Parse()`

```go
func Parse(rawURL string) (*URL, error)
```

`Parse()` парсит *raw url* в `type URL`.

URL может быть относительным (`path`, без `host`) или абсолютным (начиная со `scheme`). 

Попытка парсить `host` и `path` без `scheme` невалидна, но не обязательно возвращает *error* из-за неоднозначности парсинга.



#### `ParseRequestURI()`

```go
func ParseRequestURI(rawURL string) (*URL, error)
```

`ParseRequestURI()` парсит *raw url* в `type URL`.

Предполагается, что *URL* был получен в *HTTP request*'е, поэтому URL интерпретируется только как *absolute URI* или *absolute path* (что это???).

Предполагается, что URL не имеет суффикса `#fragment`. (Веб-браузеры удаляют `#fragment` перед отправкой URL на *web server*.)





#### `String()`

```go
func (u *URL) String() string
```

`String()` собирает `URL` в валидную *URL string*. Общий вид результата - один из следующих:

```
scheme:opaque?query#fragment
scheme://userinfo@host/path?query#fragment
```

Если `u.Opaque` не пусто, `String()` использует первую форму; в противном случае используется вторая форма. Любые *non-ASCII characters* в `host` экранируются. Чтобы получить `path`, `String()` использует `u.EscapedPath()`.

Во второй форме действуют следующие правила:

```
- если `u.Scheme` пуст, `scheme:` – опускается.
- если `u.User == nil`, `userinfo@` – опускается.
- если `u.Host` пуст, `host/` – опускается.
- если `u.Scheme` и `u.Host` пусты, а `u.User == nil`,
   полностью `scheme://userinfo@host/` – опускается.
- если `u.Host` не пуст и `u.Path` начинается с `/`,
   в блоке `host/path` не добавляется еще один `/`.
- если `u.RawQuery` пуст, `?query` – опускается.
- если `u.Fragment` пуст, `#fragment` – опускается.
```

Пример:

```go
u := &url.URL{
		Scheme:   "https",
		User:     url.UserPassword("me", "pass"),
		Host:     "example.com",
		Path:     "foo/bar",
		RawQuery: "x=1&y=2",
		Fragment: "anchor",
	}
	fmt.Println(u.String()) // https://me:pass@example.com/foo/bar?x=1&y=2#anchor
	u.Opaque = "opaque"
	fmt.Println(u.String()) // https:opaque?x=1&y=2#anchor
```

