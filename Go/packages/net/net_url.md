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



#### `Query()`

```go
func (u *URL) Query() Values
```

`Query()` парсит `URL.RawQuery` и возвращает соответствующие *value*'s. Он молча отбрасывает неправильные *value pair*'s. Для проверки ошибок используйте `ParseQuery()`.

```go
func main() {
	u, err := url.Parse("https://example.org/?a=1&a=2&b=&=3&&&&")
	if err != nil {
		log.Fatal(err)
	}
	q := u.Query()
	fmt.Println(q["a"])   		// [1 2]
	fmt.Println(q.Get("b"))		// <пусто>
	fmt.Println(q.Get(""))		// 3
}
```





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



### `type Values`

```go
type Values map[string][]string
```

`Values` мапит *string key* на *slice* из *value*'s. Обычно `Value` используется для *query parameter* и *form value*. В отличие от `http.Header` *map*'ы, *key*'s в `Values` *map*'е чувствительны к регистру (*case-sensitive*).

Примеры:

```go
func main() {
	v := url.Values{}
	v.Set("name", "Ava")
	v.Add("friend", "Jess")
	v.Add("friend", "Sarah")
	v.Add("friend", "Zoe")
  fmt.Println(v.Encode()) 			// name=Ava&friend=Jess&friend=Sarah&friend=Zoe
	fmt.Println(v.Get("name"))		// Ava
	fmt.Println(v.Get("friend"))	// Jess
	fmt.Println(v["friend"])			// [Jess Sarah Zoe]
}
```

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



#### `Add()`

```go
func (v Values) Add(key, value string)
```

`Add()` добавляет `value` к `key`. `value` присоединяется (append) к уже существующим *value*'s, ассоциированным с `key`.





#### `Encode()`

```go
func (v Values) Encode() string
```

`Encode()` кодирует *value*'s в форму «*URL encoded*» (`bar=baz&foo=quux`), отсортированные по *key*.



#### `Set()`

```go
func (v Values) Set(key, value string)
```

`Set()` устанавливает `key` в `value`. Он заменяет все существующие *value*'s (может быть привязано к `key` несколько *value*'s).

