## `encoding/json`

*Package* `json` реализует *encoding* и *decoding* для JSON, как определено в [RFC 7159](https://rfc-editor.org/rfc/rfc7159.html) .  *Mapping* JSON и Go описано в документации для функций `Marshal()` и `Unmarshal()`.





# Структура

## `Marshal()`

```go
func Marshal(v interface{}) ([]byte, error)
```

`Marshal()` возвращает *JSON encoding* для `v`.

`Marshal()` рекурсивно обходит *value* `v`. Если обнаруженное *value* (??? возможно вложенное) реализует `Marshaler` *interface* и не является *nil pointer*, `Marshal()` вызывает его `MarshalJSON()` *method* для построения JSON. Если `MarshalJSON()` *method* отсутствует, но вместо этого *value* реализует `encoding.TextMarshaler`, `Marshal()` вызывает его `MarshalText()` *method* и кодирует результат как *JSON string*. 

Иначе `Marshal()` реализует следующие *default encoding*, зависящие от *type*:



- *Pointer value*. При кодировании *pointer value* выполняется кодирование самого *value*, на которое указывает *pointer*. *Nil pointer* кодируется в *null JSON value*.







<u>Пример:</u>

```go
type Message struct {
    Name string
    Body string
    Time int64
}

func main() {
	m := Message{"Alice", "Hello", 1294706395881547000}
	b, err := json.Marshal(m)
}
```

В случае успеха:

```go
err == nil
b == []byte(`{"Name":"Alice","Body":"Hello","Time":1294706395881547000}`)
```

Особенности использования `Marshal` с разными *type*'s:

- `struct` type – выполняется *encode* только (!!!) для *exported field*'s ([1](#exported-identifier), которые начинаются с *Unicode upper case* )
- `map` type – поддерживаются только `string` в качестве ключей; поэтому `map` должен быть типа `map[string]T` (где `T` – любой тип).

Каждое *exported struct field* становится *JSON field*, при этом в качестве имени для *JSON field* используется имя *struct field* (за исключением случая использования tag's).



#### Decoding

Для выполнения *decoding* данных из JSON необходимо использовать функцию `Unmarshal()`:

```go
func Unmarshal(data []byte, v interface{}) error
```

Сначала нужно создать место, где будут храниться декодированные данные

```go
var m Message
```

и вызвать `json.Unmarshal()`, передав ему `[]byte` с JSON данными и *pointer* на `m`

```go
err := json.Unmarshal(b, &m)
```

Если `b` содержит допустимый JSON, который соответствует `m`, после вызова будет `err == nil` и данные из `b` будут сохранены в структуре `m`, так что:

```go
m = Message{
    Name: "Alice",
    Body: "Hello",
    Time: 1294706395881547000,
}
```

Чтобы найти, куда декодировать данные из поля `Foo` в структуру, `Unmarshal`  просматривает поля структуры назначения, чтобы найти *field* (в порядке предпочтения):

- *Exported field* (!!!) с тегом `"Foo"` 
- *Exported field* (!!!) с именем `"Foo"`
- *Exported field* (!!!) с именем `"FOO"` или `"FoO"` или другое нечувствительное к регистру совпадение `"Foo"`

Если JSON  данные не совсем соответствует типу Go, например:

```go
b := []byte(`{"Name":"Bob","Food":"Pickle"}`)
var m Message
err := json.Unmarshal(b, &m)
```

`Unmarshal()` будет декодировать только те *field*'s, которые он может найти в типе назначения. В этом случае будет заполнено только поле `Name` в `m`, а поле `Food` будет игнорироваться. Это можно использовать, чтобы декодировать только несколько определенных полей из большого JSON-объекта. Это также означает, что `Unmarshal()` не затронет любые *non-exported field*'s в структуре назначения.

<u>Пример 1:</u>

Декодирование JSON вида:

```json
{"fruits":["apple","banana","cherry","date"]}
```

```go
json := []byte(`{"fruits":["apple","banana","cherry","date"]}`)
var m map[string][]string
err := json.Unmarshal(json, &m)
```

<u>Пример 2:</u>

Декодирование JSON, который имеет нерегулярную структуру (разные структуры в ключах):

```json
{
    "sendMsg":{"user":"ANisus","msg":"Trying to send a message"},
    "say":"Hello"
}
```

Для этого декодирование выполняется по частям. На каждом шаге вложенная структура декодируется в `json.RawMessage`. 

```go
var objmap map[string]json.RawMessage
err := json.Unmarshal(data, &objmap)
```

Затем вложенную структуру можно декодировать дальше:

```go
var s sendMsg
err = json.Unmarshal(objmap["sendMsg"], &s)
```

#### Потоковые *encoder* и *decoder*

Для потокового чтения и записи JSON данных используются `Decoder` и `Encoder` *type*'s. 

##### `json.Decoder`

*Type*, предназначен для чтения и декодирования JSON данные из входного потока.

```go
type Decoder struct {
    // contains filtered or unexported fields
}
```

###### `json.NewDecoder()`

```go
func NewDecoder(r io.Reader) *Decoder
```

Функция, возвращает новый экземпляр типа `Decoder`, который читает из `r`.

###### `Decoder.Decode()`

```go
func (dec *Decoder) Decode(v interface{}) error
```

*Method*, читает JSON значение со своего входа, декодирует и сохраняет его в `v`.

(???) Преобразование JSON в Go value выполняется аналогично `Unmarshal()` ([link](#decoding)).

##### `json.Encoder`

*Type*, предназначен для кодирования и записи JSON данных в выходной поток.

```go
type Encoder struct {
    // contains filtered or unexported fields
}
```

###### `json.NewEncoder()`

```go
func NewEncoder(w io.Writer) *Encoder
```

Функция, возвращает новый экземпляр типа `Encoder`, который пишет в `w`.

###### `Encoder.Encode()`

```go
func (enc *Encoder) Encode(v interface{}) error
```

*Method*, который кодирует значение из `v` в JSON и пишет его выходной поток + добавляет символ *newline*.

(???) Преобразование Go value в JSON выполняется аналогично `Marshal()` ([link](#encoding)).

##### *Tag*'s

Порядок кодирования (декодирования) может быть настроен с помощью *format string*, описанного в ключе `json` в *struct field's tag*.

*Format string* указывает имя поля, за которым, возможно, следует список параметров, разделенных запятыми `,`. Это имя поля необходимо указывать, чтобы декодировать (кодировать) поля из (в) JSON, которые отличаются от названия в полей в *struct type*. В том числе для декодирования (кодирования) полей из (в) JSON, которые не начинаются с заглавных букв. Имя может быть пустым (можно пропустить имя поля), чтобы указать параметры без переопределения исходного имени поля (3 пример ниже).

Возможные параметры:

- параметр `omitempty` указывает, что поле должно быть исключено из JSON-объекта, если поле – *empty* (имеет пустое значение): `false`, `0`, `nil`, пустой `array`, `slice`, `map` или `string`.
- если *tag* указан как  `json:"-"` , то поле всегда опускается. Чтобы указать поле с именем `-` нужно использовать *tag* – `-,`.

Примеры *tag*'s и их значения:

```go
// В JSON вместо имени `Field` будет использовано имя `myName`
Field int `json:"myName"`

// Поле Field в JSON преобразуется в поле с именем "myName" и
// поле будет опущено, его значение - empty.
Field int `json:"myName,omitempty"`

// Поле Field в JSON преобразуется в поле с именем "Field" (по умолчанию) и
// поле будет опущено, его значение - empty.
// Note the leading comma.
Field int `json:",omitempty"`
```







Пример структуры для декодирования (кодирования) полей из (в) JSON, которые не начинаются с заглавных букв:

```go
type Sample struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}
```



