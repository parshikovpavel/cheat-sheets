

Пример описания схемы:

```openapi
openapi: 3.0.0
info:
  version: 1.0.0
  title: demo-service

paths:
  /demo/path:
    post:
      requestBody:
        content:
          application/json:
            schema:
              title: inputBody
              type: object
              properties:
                data:
                  type: string

      responses:
        200:
          description: 'Ok'
          content:
            application/json:
              schema:
                title: respData
                type: object
                properties:
                  result:
                    type: boolean
```





<pre>
components:               # Переиспользуемый компонент (<a href="#components">1</a>)  
  schemas:
    AsyncStatus:
      type: string
      enum: [ processing, done, not_found, error ]
paths:                   
  /web/1/broker/getItemData/{itemID}:
    operationId: get_item_data
    parameters:
      - name: itemID
        in: path
        schema:
          type: integer
    get:
      responses:
        200:
          content:
            application/json:
              schema:
                title: itemData
                type: object
                properties:
                  id:
                    type: integer
                  image:
                    type: string
                  title:
                    type: string
                  url:
                    type: string
                  price:
                    type: string
                  modification:
                    type: string
                  mileage:
                    type: string
        404:
          description: 'Not found'  
</pre>


## Определения

### Path Templating

*Path templating* – описание шаблона с parameter'ом, заключенным в фигурные скобки `{...}`. *Path parameter* обозначает часть *URL path*, которая может изменяться.

Каждый *path parameter* в URL должен соответствовать *path parameter*'у, описанному в [Path Item](#path-item-object) и/или в каждой [operation](#operation-object) внутри *path item*.







## Schema

*Schema* описывается в файле `api/schema.yaml`:

*Schema* описывается в виде *OpenAPI Object*, который является *root object* всего документа.

Перечень *field*'s:

| Field Name     | Type                                                         | Description                                                  |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `openapi`      | TODO                                                         |                                                              |
| `info`         | [Info Object](#info-object)                                  | **REQUIRED**. Предоставляет *metadata* об API.               |
| `servers`      | [[Server Object](#server-object)]                            | *Array* из *Server object*'s, каждый описывает информацию о адресе сервере. Если `servers` отсутствует, то значение *by default* – *Server object* с `url: /`.<br/>(swagger-ui, появляется выпадающий список *server*'ов) |
| `paths`        | [Paths Object](#paths-object)                                | **REQUIRED**. Содержит *relative path*'s к *endpoint*'s и их *handler*'s. |
| `components`   | [Components Object](#components-object)                      | Предназначен для описания переиспользуемых *object*'s.       |
| `security`     | [[Security Requirement Object](#security-requirement-object)] | TODO                                                         |
| `tags`         | [[Tag Object](#tag-object)]                                  | Список *tag*'s с метаданными. Используется для группировки *operation*'s. Порядок *tag*'s учитывается (???) при отображении групп в swagger. Tag's, которые указаны в [Operation object](#operation-object) не обязательно должны быть объявлены здесь. Необъявленные теги в swagger перечисляются в случайном порядке (????). |
| `externalDocs` | [External Documentation Object](#external-documentation-object) | Дополнительная *external documentation*. Должен быть *object*, а не *array*!!! Т.е. возможен только один *External documentation object*. (swagger, выводится вверху страницы). |

Основные блоки:

### `info`

**REQUIRED**. Предоставляет *metadata* об API.

Указывается:

- имя сервиса. 









### Server Object

Информация о сервере.

(swagger-ui, появляется выпадающий список server'ов)

Доступные *field*'s:

| Field name    | Type                                                         | Description                                                  |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `url`         | `string`                                                     | **REQUIRED**. URL host'а. URL поддерживает *server variable*'s и может быть *relative*. Подстановка *server variable*'s будет выполнена в тех местах, где они обозначены в фигурных скобках `{variable}`. |
| `description` | `string`                                                     | Описание host'а (поддерживается *CommonMark*).               |
| `variables`   | Map[string, [Server Variable Object](#server-variable-object)] | *Array of server variable*'s для подстановки в `url`         |

<u>Пример:</u>

```
servers:
- url: https://development.gigantic-server.com/v1
  description: Development server
- url: https://staging.gigantic-server.com/v1
  description: Staging server
```



### Server Variable Object

Описание *server variable* для постановки в *server URL*.



Доступные *field*'s:

| Field name    | Type       | Description                                                  |
| ------------- | ---------- | ------------------------------------------------------------ |
| `enum`        | [`string`] | *Array of string*, который могут использоваться для подстановки в `url`. Может быть не определен и тогда в качестве *server variable* можно использовать любое значение. |
| `default`     | `string`   | **REQUIRED** (а если `enam` не пустой???). *Default value* для подстановки в `url`. Если `enum` определен, *default value* должно входить в этот `enum`.  Должно быть указано, если `enum` не определен (а `enum` определен,  то не должно???) |
| `description` | `string`   | Описание для *server variable* (можно в *CommonMark*)        |

Итак, возможные варианты:

- пользователь может указать любое значение. Для такого кейса: `enum` – не указывается, `default` – указано
- пользователь может выбрать одно из значений. Для такого кейса: `enum` – указан, `default` – указан (а может и не обязательно???)

<u>Пример:</u>

```yaml
servers:
- url: https://{username}.gigantic-server.com:{port}
  description: The production API server
  variables:
    username:
      # можно указать любое значение
      default: demo
      description: this value is assigned by the service provider, in this example `gigantic-server.com`
    port:
      # можно выбрать одно значение из списка
      enum:
        - '8443'
        - '443'
      default: '8443'
```



### Components Object

(swagger, выводятся внизу страницы по группам).

Предназначен для описания переиспользуемых *object*'s. *Object*, описанный в *Component object*, никак не влияет на API, если на него нет где-то явного *reference*.

Доступные *field*'s:

| Field name        | Type                                                         | Description                                                  |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `schemas`         | Map[`string`, [Schema Object](#schema-object) \| [Reference Object](#reference-object)] | Описание переиспользуемых [Schema Object](#schema-object).   |
| `responses`       | Map[`string`, [Response Object](#response-object) \| [Reference Object](#reference-object)] | Описание переиспользуемых [Response Object](#response-object). |
| `parameters`      | Map[`string`, [Parameter Object](#parameter-object) \| [Reference Object](#reference-object)] | Описание переиспользуемых [Parameter Object](#parameter-object). |
| `examples`        | Map[`string`, [Example Object](#example-object) \| [Reference Object](#reference-object)] | Описание переиспользуемых [Example Object](#example-object). |
| `requestBodies`   | Map[`string`, [Request Body Object](#request-body-object) \| [Reference Object](#reference-object)] | Описание переиспользуемых [Request Body Object](#request-body-object). |
| `headers`         | Map[`string`, [Header Object](#header-object) \| [Reference Object](#reference-object)] | Описание переиспользуемых [Header Object](#header-object).   |
| `securitySchemes` | Map[`string`, [Security Scheme Object](#security-scheme-object) \| [Reference Object](#reference-object)] | Описание переиспользуемых [Security Scheme Object](#security-scheme-object). |
| `links`           | Map[`string`, [Link Object](#link-object) \| [Reference Object](#reference-object)] | Описание переиспользуемых [Link Object](#link-object).       |
| `callbacks`       | Map[`string`, [Callback Object](#callback-object) \| [Reference Object](#reference-object)] | Описание переиспользуемых [Callback Object](#callback-object) |

*Key* для *object* должны использовать регулярному выражению `^[a-zA-Z0-9\.\-_]+$`, например:

```
User
User_1
User_Name
user-name
my.org.User
```

<u>Пример:</u>

```yaml
components:
  schemas:
    GeneralError:
      type: object
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
  parameters:
    skipParam:
      name: skip
      in: query
      description: number of items to skip
      required: true
      schema:
        type: integer
        format: int32
  responses:
    NotFound:
      description: Entity not found.
    IllegalInput:
      description: Illegal input for operation.
    GeneralError:
      description: General Error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/GeneralError'
```







### Paths Object

**REQUIRED**. Содержит *relative path*'s к *endpoint*'s и их *handler*'s.



| Ключ      | Тип значения                          | Описание                                                     |
| --------- | ------------------------------------- | ------------------------------------------------------------ |
| `/{path}` | [Path Item Object](#path-item-object) | *relative path* к *endpoint*'s и его *handler*'. Должен начинаться с `/`. *Path* добавляется к `url` из  [`Server Object`](https://swagger.io/specification/#server-object)'s  чтобы построить полный URL. Разрешены [Path templating](#path-templating). |



<u>Пример:</u>

<pre>
{
  "/pets": {
    <<a href="#path-item-object">Path Item object</a>>
  }
}  
</pre>

### Path Item Object

Описывает *operation*'s, доступные на одном *path*. 

Доступные *field*'s:

| Field Name    | Type                                                         | Description                                                  |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `$ref`        | `string`                                                     | Позволяет *external definition* для этого *path item*.       |
| `summary`     | `string`                                                     | (в swagger не выводится, даже если в *operation* не переопределено). Короткое описание, которое будет применено ко всем *operation*'s по данному *path*. |
| `description` | `string`                                                     | (в swagger не выводится, даже если в *operation* не переопределено) Длинное описание, которое будет применено ко всем *operation*'s по данному *path*. |
| `get`         | [Operation Object](#operation-object)                        | Определение для GET *operation* по этому *path*.             |
| `post`        | [Operation Object](#operation-object)                        | Определение для POST *operation* по этому *path*.            |
| `put`         | [Operation Object](#operation-object)                        | Определение для PUT *operation* по этому *path*.             |
| `delete`      | [Operation Object](#operation-object)                        | Определение для DELETE *operation* по этому *path*.          |
| `parameters`  | [[Parameter Object](#parameter-object) \| [Reference Object](#reference-object)] | Список *parameter*'s, которые применяются для всех *operation*'s, описанных для этого *path*. Эти *parameter*'s могут быть перезаписаны на уровне *operation*, но не могут быть удалены там. Ключом для *parameter* (по которому проверяется что они уникальны и что их надо перезаписывать) является комбинация `name` + `in`. |
|               |                                                              |                                                              |
|               |                                                              |                                                              |



<u>Пример:</u>

<pre>
    parameters: 
      <<a href="#parameter-object">Parameter object</a>>
    post:
      <<a href="#operation-object">Operation object</a>>  
</pre>


### Operation Object

Описывает одну *API operation* по некоторому *path*.

Доступные *field*'s:

| Field name     | Type                                                         | Description                                                  |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `tags`         | [`string`]                                                   | (swagger, выводится на закладке группы, tag может быть не описан или описан в виде [Tag object](#tag-object)) Список *tag*'s для группировки *operation*'s. Скорее всего может содержать любые символы, включая русские и пробелы. |
| `summary`      | `string`                                                     | (в swagger выводится на закладке) Короткое описание для *operation* |
| `description`  | `string`                                                     | (в swagger выводится внутри закладки, как описание *operation*) Длинное описание для *operation*. Можно использовать *CommonMark*. |
| `externalDocs` | [External Documentation Object](#external-documentation-object) | (выводится в swagger, под надписью Find more details) Дополнительная внешняя документация |
| `operationId`  | `string`                                                     | [ниже](#operationId)                                         |
| `parameters`   | [[Parameter Object](#parameter-object) \| [Reference Object](#reference-object)] [] | Список *parameter*'s, которые применяются для этой *operation*. Если *parameter* уже определен на уровне [Path Item Object](#path-item-object), то определение здесь будет перезаписывать его, но не можеть быть удалено здесь. Ключом для *parameter* (по которому проверяется что они уникальны и что их надо перезаписывать) является комбинация `name` + `in`. |
| `requestBody`  | [Request Body Object](#request-body-object) \| [Reference Object](#reference-object) | Тело запроса, которое применяется для этой *operation*. Имеет смысл только для тех *HTTP method*'s, которые поддерживют *request body*.<br/>Является опциональным и может быть не задан. |
| `responses`    | [Responses Object](#responses-object)                        | **REQUIRED**. Список возможных *response*'s, которые возвращаются при выполнении этой *operation*. |
| `deprecated`   | `boolean`                                                    | *Operation* – *deprecated*. По умолчанию, `false`.           |
|                |                                                              |                                                              |

Пример:

```
tags:
- pet
summary: Updates a pet in the store with form data
operationId: updatePetWithForm
parameters:
- <Parameter Object>
requestBody:
  <RequestBody>
responses:
  <Responses Object>
```







#### `operationId`

Уникальная строка, используемая для идентификации *operation*. `operationId` должен быть уникальным среди всех *operation*'s, описанных в API. 

Av: `operationId` можно использоваться, чтобы задать пакету handler'а название.

<u>Пример:</u>

Для *Shema*:

```
/demo/path/{id}:
  get:
    operationId: fetch_for_id
```

генерируется *package name* – `fetch_for_id`:

### Parameter Object

Описывает одиночный *parameter* для *operation*.

Ключом для *parameter* (по которому проверяется что они уникальны и что их надо перезаписывать) является комбинация `name` + `location`.

Возможны 4 значения для поля `in` (т.н. *location*):

- `path` – используется вместе с  [Path Templating](#path-templating), где в качестве *parameter value* подставляется часть URL. Например, в `/items/{itemId}`, *path parameter*  – `itemId`. (Av: Поддерживаются типы URL-path переменных: `string`, `integer`, `float`, `boolean`)
- `query` – *parameter*, который добавляется к URL. Например, в `/items?id=123`, *query parameter* – `id`.
- `header` – *header*, который приходит в запросе.
- `cookie` – *cookie*, который приходит в запросе.

Доступные *field*'s:

| Field name          | Type                                                         | Description                                                  |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `name`              | `string`                                                     | **REQUIRED**. Имя параметра. Name – *case sensitive*.        |
| `in`                | `string`                                                     | **REQUIRED**. Местоположени parameter'а. Возможные значения – `"query"`, `"header"`, `"path"` или `"cookie"`. |
| `description`       | `string`                                                     | (swagger, выводится как описание параметра, под заголовком Description)<br/>Описание *parameter*'а. Можно писать примеры использования. Можно использовать синтаксис CommonMark |
| `required`          | `boolean`                                                    | Является ли этот *parameter* обязательным. Если `in: "path"`, то обязательно должно быть указано  `required: true`. Значение по умолчанию  `false`. |
| `deprecated`        | `boolean`                                                    | *Parameter* – *deprecated* и его не следует использовать. Значение по умолчанию `false`. |
| `style` и `explode` | `string`                                                     | описывают, как *parameter* будет сериализован в строке. В документации есть таблица с примерами. |
| `schema`            | [Schema Object](#schema-object) \| [Reference Object](#reference-object) | *Schema*, определяющая тип *parameter*'а.                    |
| `example`           | Any                                                          | (swagger, выводится прямо в edit со значением *parameter*'а, как будто это значение *by default*)<br/>Пример значения параметра. Нужно использовать или `example` или `examples`, но не оба. Этот `example` имеет более высокий приоритет, чем `example` из `schema` |
| `examples`          | Map[`string`, [Example Object](#example-object) \| [Reference Object](#reference-object)] | (swagger, появляется dropdown со списком examples, `examples.summary` – текст в dropdown, `examples.description` – описание появляется ниже поля ввода параметра)<br/>Примеры значений параметра. Нужно использовать или `example` или `examples`, но не оба.  `examples` имеет более высокий приоритет, чем `example` из `schema` |

Пример:

```yaml
    parameters:
      - name: u
        in: cookie
        schema:
          type: string
      - name: itemID
        in: path
        schema:
          type: integer
```



Пример для `examples`:

```yaml
        name: itemID
        required: true
        in: query
        schema:
          type: integer
          maximum: 50
        examples:       # Multiple examples
          zero:         # Distinct name
            value: 20    # Example value
            summary: A sample limit value  # Optional description
          max: # Distinct name
            value: 50   # Example value
            summary: A sample limit value  # Optional description
```

### Request Body Object

Описывает *body* одного *request*.

Доступные *field*'s:

| Field name    | Type                                                   | Description                                                  |
| ------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| `description` | `string`                                               | (swagger, выводится вверху блока *Request body*). Описание тела запроса. Может включать примеры использования. Можно использовать CommonMark. |
| `content`     | Map[`string`, [Media Type Object](#media-type-object)] | [ниже](#content-request)                                     |
| `required`    | `boolean`                                              | Обязательно ли *request body* в запросе. По умолчанию `false`. |

#### `content` (request)

**REQUIRED**. Содержимое тела запроса. Ключ – *media type*, значение – описание запроса. 

Av: поддерживаются следующие *media type*'s:

- `application/json`
- `application/x-www-form-urlencoded`
- `multipart/form-data`

##### AV: Правила описания для разных типов

##### `application/json`

...

##### `application/x-www-form-urlencoded`

...

##### `multipart/form-data`



### Media Type Object

Описывает форматы запроса или ответа (*shema*, *example*'s) для некоторого *media type*.

Доступные *field*'s:

| Field name | Type                                                         | Description                                                  |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `schema`   | [Schema Object](#schema-object) \| [Reference Object](#reference-object) | Схема, описывающая формат запроса и ответа                   |
| `example`  | Any                                                          | (swagger, точно в таком же виде выводится в поле ввода на закладке *Example value*). Пример для запроса или ответа с этим *media type*. Нужно использовать или `example` или `examples`, но не оба. Этот `example` имеет более высокий приоритет, чем `example` из `schema` |
| `examples` | Map[`string`, [Example Object](#example-object) \| [Reference Object](#reference-object)] | (swagger, рядом с выпадающим списком *Media type* появляется выпадающий список *Examples* и при выборе нужного *Example* он появляется в поле ввода *Example value*).<br/>Примеры для запроса или ответа с этим *media type*. Нужно использовать или `example` или `examples`, но не оба.  `examples` имеет более высокий приоритет, чем `example` из `schema` |

Пример:

```yaml
application/json: 
  schema:
    $ref: "#/components/schemas/Pet"
  examples:
    cat:
      summary: An example of a cat
      value:
        name: Fluffy
        petType: Cat
        color: White
        gender: male
        breed: Persian
    frog:
      $ref: "#/components/examples/frog-example"
```



### Responses Object

Объект, в который вкладываются `Resonse object`'s, описывающие отдельные ответы. Этот объект сопоставляет *HTTP response code* (`"200"`, `"301"`, ...) к формату ожидаемого ответа в виде [Response Object](#response-object).

Должен содержать описание формата хотя бы для одного успешного (`"200"`) *HTTP Status Code*.

Доступные *field*'s:

| Field name                                        | Type                                                         | Description                                                  |
| ------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| HTTP Status Code (`"200"`, `"400"`, `"500"`, ...) | [Response Object](#response-object) \| [Reference Object](#reference-object) | Каждый HTTP Code может указываться только один раз. *HTTP Code* нужно заключать в кавычки `"200"`для совместимости с JSON и YAML. Допускается указывать следующие диапазоны: `1XX`, `2XX`, `3XX`, `4XX`, и `5XX`. Явно указанный код имеет приоритет над диапазоном. |
| `default`                                         | [Response Object](#response-object) \| [Reference Object](#reference-object) | Описывает формат ответа *by default* для *HTTP Code*'s, которые не объявлены. |
|                                                   |                                                              |                                                              |

Пример:

```yaml
'200':
  description: a pet to be returned
  content: 
    application/json:
      schema:
        $ref: '#/components/schemas/Pet'
default:
  description: Unexpected error
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/ErrorModel'
```

### Response Object

Описывает одиночный *response* для *Operation Object*.

Доступные *field*'s:

| Field name    | Type                                                         | Description                                                  |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `description` | `string`                                                     | (swagger, выводится напротив соответствующего *HTTP Status Code* в блоке *Responses*). <br/>**REQUIRED**. Описание ответа. Можно использовать CommonMark. |
| `headers`     | Map[`string`, [Header Object](#header-object) \|[Reference Object](#reference-object)] | Сопоставление *header name* – его формат в виде *Header Object*. |
| `content`     | Map[`string`, [Media Type Object](#media-type-object)]       | [ниже](#content-response)                                    |
| `links`       | Map[`string`, [Link Object](#link-object) \| [Reference Object](#reference-object)] | (swagger, скорее всего??? отображается в колонке Links)<br/>Сопоставляет короткое *link name* (по аналогии с именами для [Component Objects](#components-object)) –  ссылкам (??? на *operations*), по которым можно перейти из *response*. |

#### `content` (response)

**REQUIRED**. Содержимое тела *response*. Ключ – *media type*, значение – описание *response*. 

Av: поддерживается только *media type* – `application/json`.

<u>Пример:</u>

```yaml
description: A complex object array response
content: 
  application/json:
    schema: 
      type: array
      items:
        $ref: '#/components/schemas/VeryComplexType'
```





### Example Object

Доступные *field*'s:

| Field name      | Type     | Description                                                  |
| --------------- | -------- | ------------------------------------------------------------ |
| `summary`       | `string` | (swagger, появляется dropdown со списоком examples, `examples.summary` – текст в dropdown) Короткое описание |
| `description`   | `string` | (swagger, появляется dropdown со списоком examples, `examples.description` – описание появляется ниже поля ввода параметра). Длинное описание. Можно использовать CommonMark. |
| `value`         | Any      | Значение с примером. Можно использовать `value` или `externalValue`, но не оба. |
| `externalValue` | `string` | URL, который указывает на пример. Следуеь использовать для сложных примеров. Можно использовать `value` или `externalValue`, но не оба. |



<u>Примеры:</u>

```
summary: A foo example
value: {"foo": "bar"}
```

```
summary: This is an example in XML
externalValue: 'http://example.org/examples/address-example.xml'
```

Вместо *Example object* можно использовать *Reference Object*:


```
zip-example: 
  $ref: '#/components/examples/zip-example'
```



### Tag Object

Описывает метаданные для *tag*. Используется для группировки *operation*'s. *Tag*'s, которые указаны в [Operation object](#operation-object) не обязательно должны быть объявлены здесь (их можно не объявлять).

Доступные *field*'s:

| Field name     | Type                                                         | Description                                                  |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `name`         | `string`                                                     | **REQUIRED**. Имя тега. Скорее всего может содержать любые символы, включая русские и пробелы (Выводится на закладке жирными буквами) |
| `description`  | `string`                                                     | Описание тега. Можно использовать CommonMark. (выводится рядом с названием, нежирными буквами) |
| `externalDocs` | [External Documentation Object](#external-documentation-object) | Дополнительная *external documentation* для *tag*.           |

 Пример:

```yaml
name: pet
description: Pets operations
```



### Reference Object

Объект, позволяющий ссылаться на другие объекты внутри текущего документа, а также на внешние документы.

Описание Reference Object взято из документа [JSON Reference](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03).

Доступные *field*'s:

| Field name | Type     | Description                      |
| ---------- | -------- | -------------------------------- |
| `$ref`     | `string` | **REQUIRED**. Строка со ссылкой. |

Пример внутренней ссылки:

```yaml
$ref: '#/components/schemas/Pet'
```

Пример внешней ссылки:

```yaml
$ref: definitions.yaml#/Pet
```

При этом выполняются следующие преобразования:

- последовательность `~1` преобразуется в `/`
- последовательность `~0` преобразуется в `~`.



### Schema Object

Используется для определения ограничениям на входные и выходные данные.

*Schema* проверяет, что предоставленный *JSON document* (он называется *instance*) и его элементы (??? они тоже называются *instance*) удовлетворяют некоторым ограничениям.

Структура *Schema object* основана на [JSON Schema Specification](https://json-schema.org/). Конкретно, нужно смотреть следующие спецификации:

- [JSON Schema Core](https://tools.ietf.org/html/draft-wright-json-schema-00): описание *type*'s.
- [JSON Schema Validation](https://tools.ietf.org/html/draft-wright-json-schema-validation-00): описание правил проверки значений.

#### Типы данных и форматы

Возможные *type*'s и *format*'s:

- `type: integer`

  ```yaml
  intField1:
    type: integer
  ```

  ```go
  IntField1 int64 `json:"intField1"`
  ```

- `format: float`

  ```yaml
  floatField1:
    type: integer
    format: float
  ```

  ```go
  FloatField1 float64 `json:"floatField1"`
  ```

- `type: string`

  ```yaml
  stringField1:
    type: string
  ```

  ```go
  StringField1 string `json:"stringField1"`
  ```

  - `enum`, `type: string`:

    ```yaml
    enumStringField1:
    	type: string
    	enum: [ processing, done, not_found, error ]
    ```

    

- `type: boolean`

  ```yaml
  boolField1:
    type: boolean
  ```

  ```go
  BoolField1 bool `json:"boolField1"`
  ```

- `type: object`

  ```yaml
  objectField:
    type: object
    properties:
      field:
        type: string
  ```

  ```go
  ObjectField ObjectField `json:"objectField"`
  type ObjectField struct {
     Field string `json:"field"`
  }
  ```

- `type: array`

  ```yaml
  arrayField:
    type: array
    items:
      type: string
  ```

  ```go
  ArrayField []ArrayFieldItem `json:"arrayField"`
  type ArrayFieldItem string
  ```

  

Доступные *field*'s:

| Field name             | Type                                  | Description                                                  |
| ---------------------- | ------------------------------------- | ------------------------------------------------------------ |
| `type`                 | `string`                              | Тип ([типы данных](#типы-данных))                            |
| `nullable`             | `boolean`                             | При значении `true` – для типа можно использовать также значение `null`. Значение по умолчанию `false` |
| `title`                | `string`                              | (swagger, не выводится) написано, что предназначен для описания *instance* в UI, по факту не выводится в swagger. <br/>AV: используется для названия *type* для *request* и *response* при генерации кода |
| `description` | `string` | (swagger, выводится на закладке Model) Описание |
| `multipleOf`           |                                       |                                                              |
| `maximum`              |                                       | (AV: поддерживается)                                         |
| `exclusiveMaximum`     |                                       |                                                              |
| `minimum`              |                                       | (AV: поддерживается)                                         |
| `exclusiveMinimum`     |                                       |                                                              |
| `maxLength`            |                                       | (AV: поддерживается)                                         |
| `minLength`            |                                       | (AV: поддерживается)                                         |
| `pattern`              |                                       | (AV: подерживается) регулярное выражение                     |
| `maxItems`             | `integer`                             | (AV: поддерживается) только для `type: array`. Значение  должно быть `>=0`. Валидация *array instance* успешна, если array size `<=maxItems`. |
| `minItems`             | `integer`                             | (AV: поддерживается) только для `type: array`. Значение  должно быть `>=0`. Валидация *array instance* успешна, если array size `>=minItems`. |
| `uniqueItems`          |                                       |                                                              |
| `maxProperties`        |                                       |                                                              |
| `minProperties`        |                                       |                                                              |
| `required`             | `array`                               | (AV: поддерживается с особенностями, [link](#required)) только для `type: object`. Элементы array-значения должны быть `string`. *Object instance* валидный, если содержит property's со всеми указанными названиями. |
| `enum`                 | `array`                               | (AV: поддерживатся) Элементы в `array` могут быть любого типа, включая `null`.<br/>Успешно, если `instance` равен одному из элементов в массиве `enum`.<br/><u>Пример:</u><br/>`enum: [ processing, done, not_found, error ]`<br/>еще пример выше. |
| `allOf`                |                                       |                                                              |
| `oneOf`                |                                       |                                                              |
| `anyOf`                |                                       |                                                              |
| `not`                  |                                       |                                                              |
| `items`                | *Schema object* \| *Reference object* | [ниже](#items)                                               |
| `properties`           | `object`                              | [ниже](#properties)                                          |
| `additionalProperties` | `bool` | `object`                     |
| `description`          |                                       |                                                              |
| `format`               |                                       | вроде можно проверять email                                  |
| `default`              |                                       | наверно можно использовать вместе с `required`, когда поле не указано в `required` |
| `discriminator`        |                                       |                                                              |
| `readOnly`             |                                       |                                                              |
| `writeOnly`            |                                       |                                                              |
| `xml`                  |                                       |                                                              |
| `externalDocs`         |                                       |                                                              |
| `example`              |                                       |                                                              |
| `deprecated`           |                                       |                                                              |

#### `items`

Только для `type: array` (*array instance*).

Значение `items` – *Schema object* | *Reference object*.

Все элементы *array instance* должны удовлетворять этому Schema object (??? хотя [в последней версии документации](https://tools.ietf.org/html/draft-wright-json-schema-validation-00) написано, что проверка всегда будет успешной).

```
items:
  <Schema object>
```

Пример:

```
items:
	type: object
	properties:
		id:
			type: string
		payment:
			type: string
```





#### `properties`

Только для `type: object` (*object instance*).

Значение `properties` – `object`. Каждый элемент этого `object` – это описание отдельного `property` в виде *Schema object* | *Reference object*. (??? Если `properties` не указан, то считается что это пустой объект, т.е. нет `property`).

```
properties: 
  propertyName: 
    <Schema object> | <Reference object>
```

<u>Пример:</u>

```
properties:
  amount:
  	type: integer
  	nullable: true
  term:
  	type: integer
  	nullable: true
```





#### `required`

Обязательность присутствия какого-либо значения в parameters или dto.
Задается параметром `required = true` для `parameters`:

```yaml
parameters:
  - name: salt
    in: path
    required: true
```

`required = ['field1', 'field2']` для объектов:

```yaml
requestBody:
     content:
       application/json:
         schema:
           required: [ 'val2']
           type: object
           properties:
             val1:
               type: array
               items:
                 required: [ 'depVal1' ]
                 type: object
                 properties:
                   depVal1:
                     type: string
                     nullable: true
 
             val2:
               type: string
               nullable: false
```



#### `nullable`

Пример:

```yaml
depVal1:
    type: string
    nullable: true
```



