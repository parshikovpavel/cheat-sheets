# `net/http/httputil`

*Package* `httputil` HTTP утилиты, дополняющие более общие функции в *package* `net / http`.



## `DumpRequest()`

```go
func DumpRequest(req *http.Request, body bool) ([]byte, error)
```

`DumpRequest()` возвращает *request* `req` в его HTTP/1.x *representation*. Его должны использовать только *server*'ы для отладки *client request*'ов (ну и *client*'ы наверно могут отлаживать свои request'ы). Возвращенное *representation* является приблизительным; некоторые детали первоначального *request*'а теряются при его разборе в `http.Request`. В частности, теряется порядок и регистр *header name*'s. Порядок *value*'s в *multi-valued header*'ах сохраняется. *HTTP/2 request*'ы дампятся в HTTP/1.x *representation*, а не в свое исходное *binary representation*.

Если `body == true`, `DumpRequest()` также возвращает *body*. Для этого он используется `req.Body`, а затем заменяет его новым `io.ReadCloser`, который возвращает те же *byte*'s. Если `DumpRequest()` возвращает *error*, состояние `req` (??? что это значит) не определено.

В документации для `http.Request.Write` подробно описано, какие поля `req` включаются в дамп.

<u>Пример:</u>

- *Client* (`http.DefaultClient`) отправляет *request* (`http.NewRequest()`) на *server* (`httptest.NewServer()`). 
- *Server* делает дамп *request*'а (`httputil.DumpRequest()`) и возвращет его в своем *response* (`fmt.Fprintf(w, "%q", dump)`). 
- *Client* вычитывает *response* (`io.ReadAll(resp.Body)`) и выводит его на экран (`fmt.Printf("%s", b)`)

```go
func main() {
	ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		dump, err := httputil.DumpRequest(r, true)
		if err != nil {
			http.Error(w, fmt.Sprint(err), http.StatusInternalServerError)
			return
		}

		fmt.Fprintf(w, "%q", dump)
	}))
	defer ts.Close()

	const body = "Go is a general-purpose language designed with systems programming in mind."
	req, err := http.NewRequest("POST", ts.URL, strings.NewReader(body))
	if err != nil {
		log.Fatal(err)
	}
	req.Host = "www.example.org"
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	b, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%s", b)
}
```

Вывод:

```
"POST / HTTP/1.1\r\nHost: www.example.org\r\nAccept-Encoding: gzip\r\nContent-Length: 75\r\nUser-Agent: Go-http-client/1.1\r\n\r\nGo is a general-purpose language designed with systems programming in mind."
```

