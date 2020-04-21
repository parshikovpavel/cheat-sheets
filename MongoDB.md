

# Конфигурирование

## Установка

- Подключить *mongodb tap*:

  ```bash
  brew tap mongodb/brew
  ```

- установить:

  ```php
  brew install mongodb-community
  ```

- запустить:

  ```bash
  brew services start mongodb-community
  ```

### Возможные проблемы

Ошибка в `mongo.log`:

```
Attempted to create a lock file on a read-only directory: /usr/local/var/mongodb, terminating
```

Необходимо изменить *file owner*'а для *directory*:

```bash
sudo chown -Rv <user_name> /usr/local/var/mongodb
```



##  Состав приложения

Бинарные файлы находятся в папке `/usr/local/bin/`:

* `mongod` – сервер
* `mongo` – клиентская консоль

Другие файлы и папки, которые создаются автоматически:

- `/usr/local/etc/mongod.conf` – конфигурационный файл

* `/usr/local/var/log/mongodb` – директория для логов
* `/usr/local/var/mongodb` – директория для данных

## `mongod.conf`

Минимальная конфигурация:

```yaml
storage:
  dbPath: /usr/local/var/mongodb
```

# Термины

* *База данных* – самый большой контейнер

* В базе данных размещаются *коллекции* (аналог таблицы). *Коллекция* не хранит информации о  структуре и набор возможных полей (в отличии от таблицы в SQL).

  Специальные виды коллекций:

  - Материализованные представления (materialized views) – результаты агрегации данных в виде материализованной коллекции. Результаты последующих агрегаций могут быть добавлены к ранее посчитанным результатам.
  - Ограниченные коллекции (capped collection) – коллекции фиксированного размера, подобные циклическому буферу: по мере заполнения коллекции новые документы начинают вставляться на место старых.

* Коллекции состоят из *документов* (аналог строки). *Документ* хранит информацию о полях, которые находятся внутри него. Разные *документы* могут иметь различные наборы полей.  Невозможно автоматически изменить структуру всех документов сразу, при необходимости нужно изменять структуру каждого документа по отдельности. 

  Документы хранятся в формате BSON – это бинарное представление для формата JSON.

  ```json
  {
      field1: value1,
      ...
  }
  ```

  где valueN – значение одного из типов данных BSON.

* Документ состоит из *полей* (аналог полей и колонок).  

* *Индекс*

* При запросе данных в ответ возвращается *курсор*, через который мы может взаимодействовать с данными 

* Виды (view) – *read-only* коллекции из существующих коллекций или других видов

* 

# Типы данных BSON

BSON – бинарный формат для хранения документов.

Перечень типов:


- `double` – 64 bit

- `string` – используются формат *UTF8*

- `array`

  ```json
  city: ['Moscow', 'Saint-Petersburg']
  ```

- `object` – вложенные документы (*embedded document*)

   ```json
   name: { first: "Alan", last: "Turing" }
   ```


- `objectId`
- `date`
- `Array`
- `int` – 32 bit
- `long` – 64 bit
- `decimal` – 128 bit числа с плавающей запятой, эмулирующие отсутствие потери точности. Для денежных расчетов и т.д. 
- ...

Проверить тип поля в условии можно с помощью оператора [`$type`](#$type)

## MongoDB Extended JSON

Преобразование из бинарного *BSON* в текстовый *JSON* в чистом виде приведет к потере информации о типах. Поэтому придуман расширенный *JSON* – *MongoDB Extended JSON*. В  *MongoDB Extended JSON* информация о типе сохраняется вместе с самими данными. Причем возможны два способа сохранения значения и его типа вместе:

- канонический режим (Canonical Mode) – подробное описание типа 
- расслабленный режим (Relaxed Mode) – упрощенное описание типа

 *MongoDB Extended JSON* используется:

- драйверами языков программирования (в том числе *PHP*)
- некоторые утилиты *MongoDB*, например, `mongodump`.

Примеры значений различных типов в формате *MongoDB Extended JSON*:

| Тип данных | Canonical Mode                              | Relaxed Mode                           |
| ---------- | ------------------------------------------- | -------------------------------------- |
| `date`     | `{“$date”:{“$numberLong”:”1565546054692”}}` | `{“$date”:”2019-08-11T17:54:14.692Z”}` |
| `int`      | `{“$numberInt”:”10”}`                       | `10`                                   |
| `double`   | `{“$numberDouble”:”10.5”}`                  | `10.5`                                 |



# Массивы и вложенные документы

Используется точечная нотация (*dot notation*). Путь к вложенным элементам, содержащий точки `.`, должен заключаться в кавычки.

Массивы индексируются с нуля.

Доступ к элементу массива:

```
{
	fruit: [ "apple", "orange", "pineapple"]
}

"<array>.<index>"
"fruit.2"
```

Доступ к полю *embedded document*:


```
{
	name: { first: "Alan", last: "Turing" }
}

"<document>.<field>"
"name.first"
```





# Mongo shell

Подключение с параметрами по умолчанию к `localhost:27017`

```bash
mongo
```

Подключение к удаленному хосту с аутентификацией:

```bash
mongo --username <username> --password <password> --authenticationDatabase <db> --host <host> --port <port>
```
Команды в консоли записываются на *Javascript*. 

Для *Mongo shell* можно писать скрипты на javascript и есть несколько способов запуска их в консоли. 

## Глобальные команды

| Команда | Описание            |
| ------- | ------------------- |
| `help`  | Справка по командам |
| `exit`  | Выход               |

Некоторые действия можно выполнять двумя типами команд. Например, для выбора текущей БД:

- shell helpers

  ```
  use <databaseName>
  ```

- javascript equivalents

  ```javascript
  db = db.getSiblingDB(<databaseName>)
  ```

  

## Работа с базой данных

Доступ к командам для управления текущей базой данных выполняется через объект `db`. 

* Переключиться на БД или создать БД, если она не существует:

    ```javascript
    use <databaseName>
    db = db.getSiblingDB(<databaseName>)
    ```
    
    Получить список методов, доступных для объекта `db`:
    
    ```javascript
    db.help()
    ```

- Узнать текущую БД:

  ```javascript
  > db
  test
  ```

- Список всех БД на сервере:

  ```
  show dbs
  ```



## Работа с коллекциями

Доступ к командам для управления коллекцией выполняется через объект `db.<collectionName>`. 

* Создание коллекции можно выполнить специальной функцией:

  ```javascript
  db.createCollection( <name>, <options>)
  ```

  или, автоматически, вместе с первой *create* операцией, например `db.collection.insert()`

- Получить список коллекций в текущей БД

  ```javascript
  db.getCollectionNames()
  show collections
  ```
## Типы данных в Mongo Shell

*Mongo Shell* обрабатывает все числа по умолчанию, как 64-разрядный `double`.

Для создания данных конкретных типов введены специальные функции. Например:

- `date`:

  ```javascript
  Date() //текущая дата
  ISODate("2012-12-19T06:01:17.171Z")
  ```

- `long`:

  ```
  NumberLong("2090845886852")
  ```

- `int` (у меня почему-то показываются аналогично значениям, вставленным без функции, как будто они `float`)

  ```
  NumberInt("10")
  ```

- `decimal`:

  ```
  NumberDecimal(1000.55)
  ```
  
  
  
   



## CRUD операции с документами


### Create

Поведение операций вставки:

- Если в момент вставки документа коллекция не существует, то она создается. 
- Если в документе не указано поле `_id`, автоматически добавляется поле `_id` с типом `ObjectId`.

#### `insertOne()`

Вставка одиночного документа в коллекцию. 

```javascript
db.collection.insertOne(<document>);
```

```json
db.collection.insertOne( 
    { _id: 10, name: "Ivan" } 
);
```

Результат: 

```json
{ 
    "acknowledged" : true, 
    "insertedId" : 10       # _id вставленного документа
}
```

PHP:


```php
function insertOne($document, array $options = []): MongoDB\InsertOneResult
    
$insertOneResult = $collection->insertOne([
    '_id' => 10,
    'name' => 'Ivan'
]);
$insertedId = $insertOneResult->getInsertedId(); # `_id` вставленного документа
```

#### `insertMany()`

**Вставка нескольких документов**

```javascript
db.collection.insertMany(
	[ <document 1> , <document 2>, ... ]
);
```

```json
db.products.insertMany( [
      { item: "card", qty: 15 },
      { item: "envelope", qty: 20 }
   ] );
```

Результат:

```json
{
   "acknowledged" : true,
   "insertedIds" : [
      ObjectId("562a94d381cb9f1cd6eb0e1a"),
      ObjectId("562a94d381cb9f1cd6eb0e1b")
   ]
}   
```

PHP:

```php
function insertMany(array $documents, array $options = []): MongoDB\InsertManyResult
    
$insertManyResult = $db->inventory->insertMany([
    [
        'item' => 'card',
        'qty' => 15,
    ],
    [
        'item' => 'envelope',
        'qty' => 20,
    ],
]);
$insertedIds = $insertManyResult->getInsertedIds(); # `_id`s вставленных документов
```

#### `insert()`

Вставить один (как `insertOne()`) или несколько (как `insertMany()`) документов в коллекцию.

```json
db.collection.insert(
    <document or array of documents>,
    {
    	ordered: <boolean>
    }
)
```

Параметры:

- `ordered` – определяет порядок вставки документов:
  - `true` (по умолчанию) – упорядоченная вставка документов. В случае ошибки вставки одного из документов, остальные документы не вставляются.
  -  `false` – неупорядоченная вставка документов. В случае ошибки вставки одного из документов, этот документ пропускается и остальные документы обрабатываются.


Пример вставки одного документа:

```json
db.products.insert( 
    { item: "card", qty: 15 } 
)
```

Пример вставки нескольких документов:

```json
db.products.insert(
   [
     { _id: 11, item: "pencil" },
     { item: "pen", qty: 20 },
     { item: "eraser", qty: 25 }
   ]
)
```

Результат:

```json
WriteResult({ 
    "nInserted" : 1 # Число вставленных 
})
```



### Read

```javascript
db.collection.find(query, projection)
```

- `query` – [*query filter document*](#query-filter-document)
- `projection` – [projection document](#возвращаемые_поля)

<u>PHP</u>


```php
function find($filter = [], array $options = []): MongoDB\Driver\Cursor; 
```




#### Возвращаемое значение

```json
{ "_id" : ObjectId("5d739ed824be55a6feb4ce50"), "name" : "Pavel", "age" : 34 }
```

Возвращаемое значение – курсор к результатам, который нужно итерировать для получения документов. 

Если курсор в *mongo shell* не присвоен переменной через `var`, то он итерируется 20 раз. Для получения следущей страницы результатов нужно напечатать `it`. 

В любом случае получаем документы верхнего уровня коллекции, даже если фильтруем по вложенным (<u>nested</u>) массивам или документам.

#### Преобразование типов

Разные числовые типы приводятся и могут сравниваться при фильтрации. 

Но для большинства типов данных сравнение может выполняться только одних и тех же типов.

#### Возможные cursor methods

- [`сursor.pretty`](#cursor-pretty)
- [`cursor.sort`](#cursor-sort)
- [`cursor.limit`](#cursor-limit)
- [`cursor.count`](#cursor-count) и его аналог [`db.collection.count`](#db-collection-count)



#### Фильтр по вложенным документам (embedded/nested)

<u>Сопоставление по всему вложенному документу целиком</u>

```json
{ <field>: <object_value> }
{ size: { h: 14, w: 21 } }
```

```php
[ <field> => <object_value>]
[ size => [ h => 14, w => 21 ] ]
```

Будут найдены все документы, где `size` в точности равно `{ h: 14, w: 21 }`, учитывая порядок полей.

<u>Сопоставление по значению вложенного поля (*nested field*)</u>

Для фильтрации по значению вложенного поля необходимо в [query filter document](#query-filter-document) указать путь к вложенному полю через точечную нотацию `"field.nestedField"` (обязательно заключать в кавычки).

Упрощенный вариант оператора `$eq`:

```json
{ "field.nestedField": <value>}
{ "size.x": 200 }
```

```php
[ "size.x" => 200 ]
```

Общий вариант с указанием оператора:

```json
{ "field.nestedField": { <operator>: <value> } } 
{ "size.x": { $lt: 200 } }
```

```php
[ "size.x" => [ $lt: 200 ] ]
```

#### Фильтр по массивам

Возможны массивы:

- скалярных элементов:

  ```
  { tags: ["red", "blank", "plain"] }
  ```

- вложенных объектов:

  ```
  { instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] }
  ```

Простые условия:

- условие на длину массива накладывается оператором [`$size`](#size)

##### Эквивалентность всего массива целиком

```json
{ <array_field>: <array_value> }
{ tags: ["red", "blank"] }
```

```php
[ 'tags' => [ 'red', 'blank' ] ]
```

Будут найдены все документы, где `tags` в точности равно `["red", "blank"]`, учитывая порядок элементов.

##### Эквивалентность отдельных элементов массива

###### Вхождение *нескольких* элементов

Если нужно найти документы, где `tags` содержит элементы `["red", "blank"]` без учета порядка и, возможно, с наличием других элементов, то нужно использовать оператор [$all](#all):

```json
{ <array_field>: { $all: <array_value> } }
{ tags: { $all: ["red", "blank"] } }
```

```php
['tags' => ['$all' => ['red', 'blank']]]
```

######  Вхождение *одного* элемента

- <u>Массив скалярных значений.</u> Проверка, что массив содержит указанное скалярное значение. Например, `tags` содержит элемент `"red"`, возможно, с наличием других элементов:

    ```json
    { <array_field>: <scalar_value> } # упрощенный вариант оператора $eq
    { tags: "red" }
    ```
    
    ```php
    [ 'tags' => 'red' ]
    ```

- <u>Массив документов.</u> Проверка, что массив содержит указанный документ целиком. Проверяется точное совпадение, включая порядок полей. В примере, в массиве будет искаться точно такой документ.

    ```json
    { <array_field>: <object> } # упрощенный вариант оператора $eq
    { "instock": { warehouse: "A", qty: 5 } }
    ```

- <u>Массив документов.</u> Проверка, что какой-то вложенный в массив документ, содержит поле с указанным значением. Например, найти записи, где есть вложенные в массив документы с `qty==20`:

  ```json
  { "<array_field>.<nested_object_field>": <value>}
  { 'instock.qty': 20 }
  ```

##### Применение query operators к отдельным элементам

Можно использовать [query operators](#query-filter-document).

###### Одиночный query operator

- <u>Массив скалярных значений.</u> Для одиночного *query operator*, достаточно, чтобы хотя бы один элемент массива удовлетворял условию. Например, проверить, что хотя бы один элемент массива меньше 200:

    ```json
    { <array_field>: { <operator>: <value> } } # фильтр с указанием оператора
    { size: { $lt: 200 } }
    ```

- <u>Массив документов.</u> Проверка, что какой-то вложенный в массив документ, содержит поле, удовлетворяющее условию. Например, найти записи, где есть какой-то вложенный в массив документ с `qty<=20`:

  ```json
  { "<array_field>.<nested_object_field>": { <operator>: <value> } }
  { 'instock.qty': { $lte: 20 } }
  ```

###### Одиночный query operator на элемент массива с определенным индексом

- <u>Массив скалярных значений.</u> Проверить, что элемент массива с указанным индексом удовлетворяет условию.  Например, условие на второй (`id=1`) элемент массива:

  ```json
  { "dim_cm.1": { $gt: 25 } }
  ```

- <u>Массив документов.</u> Проверить, что поле вложенного (*nested*) документа внутри элемента массива с определенным индексом удовлетворяет условию. Например, что массив имеет в качестве `id=0`  документ, содержащий поле `qty<=20`:

  ```json
  { 'instock.0.qty': { $lte: 20 } }
  ```

  

###### Множественный query operator

 <u>Множественное условие, которое может удовлетворяться разными элементами</u>

- <u>Массив скалярных значений.</u> При обычном задании множественного условия, разные части этого условия могут удовлетворяться разными элементами массива. Например, проверить, что  `A[i]>15` и `A[j]<20` (возможно `i == j` или `i != j`).

    ```json
    { A: { $gt: 15, $lt: 20 } }
    ```

    ```php
    'A' => ['$gt' => 15, '$lt' => 20]
    ```

- <u>Массив документов.</u> При обычном задании множественного условия, разные части условия могут удовлетворяться разными вложенными в массив документами. 

    Например, проверить, что `A[i].qty > 10` и `A[j] <= 20` (возможно `i == j` или `i != j`).

    ```
    { "A.qty": { $gt: 10,  $lte: 20 } }
    ```

    Например, проверить, что `A[i].qty > 5` и `A[j].warehouse = 'A'` (возможно `i == j` или `i != j`).

    ```
    { "A.qty": 5, "A.warehouse": "A" }
    ```

    

<u>Множественное условие, которое должно удовлетворяться одним элементом</u>

- <u>Массив скалярных значений.</u> Проверить, что хотя бы один элемент массива удовлетворяет всем условиям. Следует использовать оператор [`$elemMatch`](#elemmatch). Например, найти массивы, где хотя бы один элемент удовлетворяет сразу обоим критериям: `>15` и `<20`.

    ```json
    { dim_cm: { $elemMatch: { $gt: 15, $lt: 20 } } }
    ```

    ```php
    'dim_cm' => ['$elemMatch' => ['$gt' => 15, '$lt' => 20] ]
    ```

- <u>Массив документов.</u> Проверить, что есть хотя бы один вложенный документ с `qty=5` и `warehouse="A"`:

    ```
{ "instock": { $elemMatch: { qty: 5, warehouse: "A" } } }
    ```
    
    Проверить, что есть хотя бы один вложенный документ с `qty>10` и `qty<=20`:
    
    ```json
    { "instock": { $elemMatch: { qty: { $gt: 10, $lte: 20 } } } }
    ```

#### Возвращаемые поля

По умолчанию, документы возвращаются полностью со всеми полями. Для того чтобы явно указать возвращаемые поля используется 2 аргумент метода `find()` – *projection document (проекционный документ)*. *Projection document* – документ, который используется, чтобы указать какие поля необходимо вернуть в результате запроса. 

##### Возврат указанных полей

- <u>Скалярные поля</u>. Требуемые поля указываются в формате :

    ``` json
    { <field1>: 1, <field2>: 1, ...}
    { item: 1, status: 1 } 
    ```
    
    Кроме указанных полей выбирается (даже если не указано) поле `_id`. 
    
    Чтобы исключить поле `_id`, необходимо указать:

    ```json
    { _id: 0, ... } 
    ```

- <u>Поля вложенных документов.</u> Используется dot notation, поле в результате остается вложенным:

    ```json
    { "size.x": 1 }
    ```

- <u>Поля документов, вложенных в массив.</u>

  ```json
  { "instock.qty" :  1 }
  ```

##### Возврат всех полей, кроме указанных

```json
{ <field1>: 0, <field2>: 0, ...}
{ status: 0, instock: 0 }
{ "size.x: 0" } # исключение из выбора поля вложенного документа
```

Смешивать включение полей `<field1>: 1` и исключение полей `<field1>: 0` нельзя (кроме случая с `_id: 0`). 

##### Возврат части массива

Доступны следующие [Projection operators](https://docs.mongodb.com/manual/reference/operator/query/)

###### $

Выбирает первый элемент из массива, удовлетворяющий условию, прописанному в самом запросе.

Пример: выбрать первый элемент массива `grades`, который `>=85`

```json
db.students.find( { semester: 1, grades: { $gte: 85 } },
                  { "grades.$": 1 } )

{ "_id" : 1, "grades" : [ 87 ] }
```

###### $elemMatch

Выбирает первый элемент из массива, удовлетворяющий условию, прописанному внутри самого `$elemMatch` оператора. Пример: выбрать первый элемент массива `students`, который `>10`.

```json
{ students: { $elemMatch: { $gt: 10} } }
```

###### $slice

Выбрать некоторое количество элементов из начала или конца массива.

```json
{ <array_field>: { $slice: <count> } }
{ <array_field>: { $slice: [ <skip>, <count> ] } }
```

Пример: выбрать 5 первых элементов массива `comments`

```json
{ comments: { $slice: 5 } }
```

Пример: выбрать 5 последних элементов массива `comments`

```json
{ comments: { $slice: -5 } }
```

Пример: выбрать элементы с 20 по 30:

```json
{ comments: { $slice: [ 20, 10 ] } }
```

#### Фильтр по `null` и отсутствующим полям

Для выбора документов, у которых `field` равен `null` или отсутствует.

```json
{ <field>: null }
```

Для выбора документов,  у которых `field` равен `null`, используется оператор [`$type`](#type) :

```json
{ <field>: { $type: “null” } }
```

Для выбора документов, у которых field отсутствует, используется оператор [$exists](#exists):

```json
{ item: { $exists: false } }
```

### Update

#### `updateOne` и `updateMany`

- `updateOne` – обновить только один, *первый* документ коллекции, соответствующий фильтру.

    ```json
    db.collection.updateOne(
        <filter>,
        <update>,
        {
        	upsert: <boolean>
        }
    )
    ```

    ```json
    db.inventory.updateOne(
       { item: "paper" },
       {
         $set: { "size.uom": "cm", status: "P" },
         $currentDate: { lastModified: true }
       }
    )
    ```

- `updateMany` – обновить все документы в коллекции, соответствующие фильтру

    ```json
    db.collection.updateMany(
        <filter>,
        <update>,
       {
           upsert: <boolean>
    }
   )
   ```
   
    ```json
    db.inventory.updateMany(
       { "qty": { $lt: 50 } },
       {
         $set: { "size.uom": "in", status: "P" },
         $currentDate: { lastModified: true }
       }
   )
    ```

Параметры:

- `filter` – это [query filter document](#query-filter-document), аналогичный операции [find](#read).

- `update` – может передаваться:

  - [update document](#update-document) – в виде документа

  - [aggregation pipeline](???) – в виде массива

    Может включать стадии:

    - `$set`
    - `$unset`
    - `$replaceWith`

    Например, составить массив `comments` из полей `misc1` и `misc2`, а затем поля `misc1` и `misc2` удалить.

    ```json
    [
        { $set: { comments: [ "$misc1", "$misc2" ] } },
        { $unset: [ "misc1", "misc2" ] }
    ]
    ```

#### `replaceOne`

Заменить один, первый документ целиком на другое документ, оставив его под тем же `_id`:

```json
db.collection.replaceOne(
    <filter>,
    <replacement_document>,
    {
        upsert: <boolean>
    }
)
```

```json
db.restaurant.replaceOne(
      { "name" : "Central Perk Cafe" },
      { "name" : "Central Pork Cafe", "Borough" : "Manhattan" }
);
```

Параметры:

- `filter` – это [query filter document](#query-filter-document), аналогичный операции [find](#read).
- `replacement_document` – заменяющий документ, может содержать только пары `key: value`, не может включать операторы [update document](#update-document). Не может содержать поле `_id`, т.к. оно остается неизменным.

#### `update`

Обновить один (как `replaceOne`) или все (как `replaceMany`) документы, или заменить целиком документ (как `replaceOne`) (старая функция). Действие зависит от переданных параметров:

```json
db.collection.update(
	  <filter>,
    <update>,
    {
        upsert: <boolean>,
		    multi: <boolean>,
        ...
    }
)
```

Параметры:

- `filter` – это [query filter document](#query-filter-document), аналогичный операции [find](#read).
- `update` может принимать:
  - [update document](#update-document) – как [выше](#updateone-и-updatemany)
  - [aggregation pipeline](???) – как [выше](#updateone-и-updatemany)
  - replacement_document – как в [replaceOne](#replaceOne). В этом случае документ заменяется целиком.
- `multi` – при `false` (по умолчанию) обновляется один документ, при `true` обновляются все документы, соответствующие фильтру.

Примеры:

- Обновить один документ:

    ```json
    db.books.update(
       { _id: 1 },
       {
         $inc: { stock: 5 },
         $set: { item: "ABC123" }
       }
    )    
    ```

- Обновить множество документов:

  ```json
  db.books.update(
     { _id: 1 },
     { $set: { item: "ABC123" } },
     { multi: true }    
  )  
  ```
  
  
  
- Заменить документ целиком:

  ```json
  db.books.update(
     { _id: 1 },
     {
       item: "XYZ123"
     }
  )
  ```

#### Опция upsert

Является аналогом `INSERT ... ON DUPLICATE KEY UPDATE` MySQL, только наоборот (обновляет, а не вставляет по умолчанию).

Если `false` (по умолчанию), то просто выполнить обновление документов.

Если `true`, то выполнить обновление/вставку (**up**date-in**sert**). Алгоритм:

- если существует документ(ы), соответствующие фильтру, то выполнить обычное обновление документа (ов);
- если нет документа (ов), соответствующих фильтру, то вставить *один* документ.

Особенности вставки документа:

- если `update` – *replacement_document*, то вставляется новый документ, с указанными парами `key: value`

- если `update`  – *update document*, то создается документ из предложений равенства в `<filter>` параметре и к нему применяются выражения из `<update>`параметра. Остальные операции из `<filter>` параметра (вроде операций сравнения) игнорируются, на новый документ никак не влияют.

  Например, в результате такой операции:

  ```javascript
  db.books.update(
     { item: "BLP921" },   // Query parameter
     {                     // Update document
        $set: { reorder: false },
        $setOnInsert: { stock: 10 }
     },
     { upsert: true }      // Options
  )
  ```

  получаем новый документ (подробнее [`$setOnInsert`](#setOnInsert)):

  ```json
  {
    "_id" : ObjectId("5da79019835b2f1c75348a0a"),
    "item" : "BLP921",
    "reorder" : false,
    "stock" : 10
  }
  ```

- если `update`  – *aggregation pipeline*, как и для предыдущего случая, создается документ из предложений равенства в `<filter>` параметре и к нему применяются выражения из `<update>`параметра.

<u>Пример</u>

Нужно вести счетчик в кеше, но неизвестно, существует уже счетчик или нет:

```javascript
db.hits.update(
  <filter>,
  {$inc: {hits: 1}}, 
  true
);
```




#### Результат

В результате выполнения возвращается:

```json
{ 
    "acknowledged" : true, 
    "matchedCount" : 1, # Число совпавших документов 
    "modifiedCount" : 1 # Число модифицированных документов
}
```



### Delete

#### `deleteMany`

Удалить все документы из коллекции, соответствующие фильтру

```json
db.collection.deleteMany(
   <filter>
)
```

```json
db.orders.deleteMany( 
    { "client" : "Crude Traders Inc." } 
);
```

#### `deleteOne()`

Удалить один, первый документ из коллекции, соответствующий фильтру

```json
db.collection.deleteOne(
   <filter>
);
```

```json
db.orders.deleteOne( 
    { "_id" : ObjectId("563237a41a4d68582c2509da") } 
);
```

#### `remove()`

Удалить один (как `deleteOne()`) или все (как `deleteMany()`) документы из коллекции.

Старый синтаксис:

```json
db.collection.remove(
   <query>,
   <justOne>
)
```

Новый синтаксис:

```
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     ...
   }
)
```

```json
db.products.remove( 
    { qty: { $gt: 20 } } 
)
```



Параметры:

- `filter` – это [query filter document](#query-filter-document), аналогичный операции [find](#read).
- `justOne` – при `false` (по умолчанию) удаляются все документы, при `true` обновляются все документы, соответствующие фильтру.

В результате выполнения возвращается:

```json
{ 
    "acknowledged" : true, 
    "deletedCount" : 8      # число удаленных документов
}
```





# Ключи и индексы

## Первичный ключ и поле `_id`

Каждый документ в коллекции должен иметь уникальное поле `_id`. Это поле рассматривается MongoDB как первичный ключ документа.

Поле `_id` может задаваться:

* явно в момент создания документа:

  ```javascript
  db.collection.insert({
      _id: 1234
  });
  ```

* иначе, MongoDB автоматически создает уникальный `_id` в форме *ObjectId* значения

  ```javascript
  > db.collection.insert({});
  { "_id" : ObjectId("5d695f12761f2b0e842ebe7b") }
  ```
  
  При использовании `_id` в форме ObjectId в фильтре требуется указывать его в виде:
  
  ```json
  db.unicorns.find(
      {_id: ObjectId("5d695f12761f2b0e842ebe7b")}
  )
  ```
  
  

*ObjectId* значение – это 12 байтовое значение в формате:

- 4 байта *timestamp* момента создания
- 8 байт случайных значений

Использование автоматически созданного `_id` позволяет:

- не хранить *timestamp* момента создания и извлекать его прямо из `_id` с помощью функции `getTimestamp()`

```bash
> db.collection.findOne()._id.getTimestamp()
ISODate("2019-08-30T16:44:57Z")
```

- сортировка по полю  `_id` примерно эквивалентна сортировке по времени создания, т.к. значения в этом поле монотонно возрастают (вместе с *timestamp*)

# JOIN



# Schema Validation

Поддерживает проверка схемы документа (а также значений полей документа) во время *insert* и *update*. Валидация задается через опцию `validator`. Опция может задаваться:

- в момент создания коллекции через `db.createCollection()`

- после создания коллекции через команду `collMod` (*collection modification*). Документы, которые уже находятся в коллекции, не валидируются автоматически, только после обновления.

  ```json
  db.runCommand( {
     collMod: "contacts",
     validator: { 
         $jsonSchema: {
             ...
         } 
     }
  } )
  ```

  

<u>Schema Validation</u>

Поддерживается через оператор `$jsonSchema`. 

Пример:

```json
db.createCollection("students", {
   validator: {
      $jsonSchema: {
         bsonType: "object",
         required: [ "name", "year", "major", "address" ],
         properties: {
            name: {
               bsonType: "string",
               description: "must be a string and is required"
            },
            year: {
               bsonType: "int",
               minimum: 2017,
               maximum: 3017,
               description: "must be an integer in [ 2017, 3017 ] and is required"
            },
            major: {
               enum: [ "Math", "English", "Computer Science", "History", null ],
               description: "can only be one of the enum values and is required"
            },
            gpa: {
               bsonType: [ "double" ],
               description: "must be a double if the field exists"
            },
            address: {
               bsonType: "object",
               required: [ "city" ],
               properties: {
                  street: {
                     bsonType: "string",
                     description: "must be a string if the field exists"
                  },
                  city: {
                     bsonType: "string",
                     "description": "must be a string and is required"
                  }
               }
            }
         }
      }
   }
})
```

<u>Валидация значений полей</u>

Для валидаций значений полей используются [query operators](#query-filter-document).

```json
db.createCollection( "contacts", {
    validator: { 
         $or: [
             { phone: { $type: "string" } },
             { email: { $regex: /@mongodb\.com$/ } },
             { status: { $in: [ "Unknown", "Incomplete" ] } }
         ]
   }
} )
```

<u>Дополнительные настройки</u>

`validationLevel` – к каким документам применять валидацию. Возможные варианты:

- `validationLevel: "strict"` (по умолчанию) – применять валидацию ко всем документам
-  `validationLevel: "moderate"` – не выполнять валидацию существующих документов, которые не соответствуют правилам
- `validationLevel: "off" ` – отключить валидацию

`validateAction` – что делать при нарушении правил валидации

- `validateAction: error` – отклоняет *insert* и *update* при нарушении правил валидации
- 

# Write Concern

Write conсern (проблема записи) – описывает уровень подтверждения, запрошенный у MongoDB для операций записи.

Задается в формате:

```json
{ 
    w: <value>, 
    j: <boolean>, 
    wtimeout: <number> 
}
```

## `w`  опция

Определяет, от каких серверов требуется получить подтверждение о записи.

Варианты:

- `<number>` – точное количество серверов, от которых требуется подтверждение записи.

  `w: 1` – значение по умолчанию, требуется подтверждение одного *standalone* или одного *primary* в *replica set*.

- `"majority"` – требует, чтобы операции записи распространилась на большинство (majority, M), несущих данные серверов. 

-  `<custom write concern name>` – требует, чтобы операции записи распространились на определенные тегированные сервера.



Другие опции:

- `j` – при `true` требует, чтобы запись была выполнена в журнал на диске (on-disk journal) (наверное, аналог журнала транзакции и *durability*)
- `wtimeout` – таймаут записи, в случае его превышения операция отваливается с ошибкой

# Резервное копирование

## `mongodump`

Сделать *backup* одной базы данных:

```bash
mongodump --db <db_name>
```



## `mongorestore`

Восстановить базу данных из папки `~/dump/database` и поместить в новую базу `<db_name>`:

```bash
mongorestore --db <db_name> ~/dump/database
```



# Query filter document

Документы фильтрации в запросе (*query filter document*) – это документы для фильтрации в любых типах запросов (C,R,U,D), аналог `where` в *SQL*.

*Query filter document* использует операторы запроса (*query operator*). 

Общий вид документа:

```json
{ <field> : {<operator>: <value>}, ... }
```

```php
[ <field1> => [ <operator> => <value1> ], ... ]
['status' => ['$in' => ['A', 'D']]]
```

## Пустой оператор

Пустой оператор используется для выбора всей коллекции целиком. Может быть задан следующим образом:

* оператор запроса вообще не указан

* передан `null`

* передан пустой документ `{}` (для PHP – пустой массив `[]`)

  ```javascript
  > db.collection.find( {} ) 
  ```

  ```php
  $cursor = $collection->find([]); 
  ```

  

## Операторы сравнения

### `$eq`

Проверка на равенство (*equal*)

```javascript
{ <field>: { $eq: <value> } }
{ "name": { $eq: "Pavel" } }
```


более простая форма оператора:

```
{ <field>: <value> }
{ "name": "Pavel" }
```

### `$gt`

Проверка на больше чем `>` (*greater than*) 

```javascript
{field: {$gt: value} }
{ qty: { $gt: 20 } }
```

### `$gte`

Проверка на больше чем или равно `>=` (*greater than or equal*) 

```javascript
{ field: { $gte: value } }
{ qty: { $gte: 20 } }
```

### `$in`

- Для скалярных значений – проверка `field` на соответствие одному из `value` в списке
- Для массивов – проверка, что массив содержит хотя бы один элемент из списка

```javascript
{ field: { $in: [<value1>, <value2>, ... <valueN> ] } }
{ qty: { $in: [ 5, 15 ] } } // qty = 5 или 15
{ tags: { $in: ["child", "summer"] } } // tags содержит child или summer
```

### `$lt`

Проверка на меньше чем `<` (*less than*)

```javascript
{field: { $lt: value } }
{ qty: { $lt: 20 } }
```

### `$lte`

Проверка на меньше чем или равно `<=` (*less than or equal*) 

```javascript
{ field: { $lte: value } }
{ qty: { $lte: 20 } }
```

### `$ne`

- проверка на неравенство (*not equal*)
- также сопоставляется с документами, в которых `field` отсутствует.

```javascript
{ <field>: { $ne: <value> } }
{ "name": { $ne: "Pavel" } }
```

### `$nin`

- Для скалярных значений – проверка, что `field` не соответствует ни одному из значений в списке (*not in*)
- Для массивов – проверка, что массив не содержит ни одного `value` из списка
- также сопоставляется с документами, в которых `field` отсутствует.

```javascript
{ field: { $nin: [<value1>, <value2>, ... <valueN> ] } }
{ qty: { $nin: [ 5, 15 ] } } // qty != 5 или 15
{ tags: { $nin: ["child", "summer"] } } // tags не содержит ни child, ни summer
```

### `$exists`

```javascript
{ field: { $exists: <boolean> } }
{ qty: { $exists: true } }
```

- `true` – проверка, что в *документе* существует поле `field` (возможно даже со значением `null`)
- `false` – проверка, что в *документе* не существует поле `field`

### `$regex`

Выполняет сравнение по Perl-совместимому регулярному выражению.

```json
{ <field>: { $regex: /pattern/<options> } }
{ <field>: /pattern/<options> }
{ sku: { $regex: /^ABC/i } }
```



### `$type`

```json
{ field: { $type: <BSON type> } }
{ "zipCode" : { $type : "string" } }
```

Выбирает документы , где `field`является экземпляром указанного [BSON типа](https://docs.mongodb.com/manual/reference/operator/query/type/#document-type-available-types) (в виде *number* или *alias*). Если `field`массив, то хотя бы один элемент массива должен быть указанного типа. 

### `$where`

Позволяет указать *Javascript* функцию, которая будет выполняться для каждого документа в коллекции. Дает гибкость написания фильтров, но при этом индексы не используются, требуется полное сканирование.

```json
db.players.find( { $where: function() {
   return (hex_md5(this.name) == "9b53e667f30cd329dca1ec9e6a83e994")
} } );
```



## Операторы над массивами

### `$all`

Проверка, что массив:

- содержит все указанные элементы
- в любом порядке
- возможно, есть другие элементы

```json
{ <field>: { $all: [ <value1> , <value2> ... ] } }
{ tags: { $all: [ "ssl" , "security" ] } }
```

### `$elemMatch`

Проверка, что массив содержит хотя бы один элемент, удовлетворяющий сразу всем критериям.

```json
{ <field>: { $elemMatch: { <query1>, <query2>, ... } } }
{ dim_cm: { $elemMatch: { $gt: 15, $lt: 20 } } }
```

### `$size`

Проверка, что массив содержит указанное число элементов.

```
{ <field>: { $size: <size> } }
{ field: { $size: 2 } }
```

Возвращает все документы, в которых `field` содержит ровно 2 элемента.

## Логические операторы

###  `$and`

Логическое И

```javascript
{ $and: [ { <expression1> }, { <expression2> } , ... , { <expressionN>} ] }
{ $and: [ { price: { $ne: 1.99 } }, { price: { $exists: true } } ] }
```

Также есть более простая форма с соединением `expressions` через `,`:

```javascript
{ { <expression1> }, { <expression2> } , ... , { <expressionN>} }
{ { price: { $ne: 1.99 } }, { price: { $exists: true } } }
```

```php
[
    'status' => 'A',
    'qty' => ['$lt' => 30],
]
```

Если `<expression1>`, `<expression2>`... применяются к одному полю, то можно использовать еще более простую форму, указав `field` только один раз:

```javascript
{ <field>: { <operator1>: <value1>, <operator2>: <value2>, ...}}
{ price: { $ne: 1.99, $exists: true } }
```

### `$or`

Логическое ИЛИ

```javascript
{ $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }
{ $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } 
```

```php
[
    '$or' => [
        ['status' => 'A'],
        ['qty' => ['$lt' => 30]],
    ],
]
```



# Update document

*Update document* – используется в [update](#update) методах, чтобы указать применяемые обновления.

Имеет форму:

```json
{
  <update_operator>: { <field1>: <value1>, ... },
  <update_operator>: { <field2>: <value2>, ... },
  ...
}
```

## Операторы обновления поля

### Field update

#### `$inc`

Инкрементирует (или декрементирует) поле на указанную величину:

```json
{ $inc: { <field>: <amount>, ... } }
```

`amount` может быть положительным и отрицательным. Если поле не существует, то оно создается с указанным значением.

```json
{ $inc: { quantity: -2, "metrics.orders": 1 } }
```

#### `$set`

Заменяет значения поля `field` на `value`. Массивы и объекты заменяются полностью новым значением. Если поле `field` не существует, то оно будет вставлено с указанным значением.

```json
{ $set: { <field>: <value>, ... } }
{ $set: { quantity: 500 } }
```

Можно устанавливать значения в массивах и вложенных документах через `.` 

```json
{ $set: { <field.nestedField>: <value>, ... } }
{ $set: { "details.make": "zzz" } }
{ $set: { "tags.1": "rain gear" } } # изменение tags[1]
```

#### `$unset`

Удалить поле `field` из документа:

```json
{ $unset: { <field>: <value>, ... } }
```

```json
{ $unset: { quantity: ""} }
```

Значение `value` может быть любое (здесь `""`).

#### `$currentDate`

Установить значение поля `field` в текущую дату.

```json
{ $currentDate: { <field>: <typeSpecification>, ... } }
```

`typeSpecification` может иметь такие значения:

- `true` – текущая дата записывается в формате `Date`

  ```json
  { $currentDate: { lastModified: true } }
  ```

- `{ $type: "timestamp" }` или `{ $type: "date" }`, для явного указания формата

  ```json
  { $currentDate: { "cancellation.date": { $type: "timestamp" } }
  ```

#### `$setOnInsert`

Если операция обновления с [`upsert: true`](#опция-upsert) приводит к вставке документа, то назначает указанные значения полям в документе. Если операция обновления не приводит к вставке, то ничего не происходит.

```json
{ $setOnInsert: { <field1>: <value1>, ... } },
```



### Array update

#### `$push`

Добавляет значение `value` в массив `field`

```json
{ $push: { <field>: <value>, ... } }
```

Если поле не существует, то оно создается с указанным значением.


```json
{ $push: { scores: 89 } }
```



# Cursor Methods

Эти методы изменяют то,  как выполняется основной запрос, к которому они присоединены.

## Порядок применения 

Порядок применения *cursor methods* не имеет значения. Следующие две строки эквивалентны:

```javascript
db.bios.find().sort( { name: 1 } ).limit( 5 )
db.bios.find().limit( 5 ).sort( { name: 1 } )
```



## `cursor.pretty`

Вывод результатов с форматированием (отступами перед полями, вложенными документами) 

```bash
db.collection.find().pretty()
{
        "_id" : ObjectId("5d739ed824be55a6feb4ce50"),
        "name" : "Pavel",
        "age" : 34
}
```

## `cursor.count`

Подсчитывает количество документов, на которые ссылается курсор. 

```javascript
db.collection.find(<query>).count(<bool applySkipLimit = false>)
```

- `applySkipLimit`  – учесть при подсчете количества наличие методов `cursor.skip` и `cursor.limit`. По умолчанию `false`, т.е. подсчитывается полное количество строк по запросу.





## `cursor.sort`

Указывает порядок сортировки.

```javascript
db.collection.find().sort( { <field>: <value>, ... } )
```

`<value>` может принимать значения:

- `<field>: 1` – сортировка по возрастанию (*ascending*)
- `<field>: -1` – сортировка по убыванию (*descending*)

Пример:

```json
db.collection.find().sort( { age : -1, posts: 1 } )
db.orders.find().sort( { "item.category": 1, "item.type": 1 } )
```

<u>Ограничение:</u>

Если отсутствует нужный индекс для получения строк в порядке индекса, то выполняется сортировка строк в памяти. Если для сортировки требуется более 32 мегабайта, запрос не будет выполнен.

## `cursor.limit`

Указывает максимальное количество документов, которое может вернуть курсор.

```javascript
db.collection.find(...).limit(<number>)
```

## `cursor.skip()`

Указывает количество документов, пропускаемых в результирующем наборе.

 ```javascript
db.collection.find(...).limit(<offset>)
 ```

Для выбора документов по смещению, вся выборка сканируется с начала. По мере увеличения смещения `cursor.skip()`станет работать медленнее.

Чтобы избежать больших смещений, иногда можно точно рассчитать `_id` нужного смещения и применить запрос без смещения:

```javascript
db.collection.find( { _id: { $gt: startValue } } )
             .sort( { _id: 1 } )
             .limit( nPerPage )
```



# Collection methods

Операции над коллекциями.

## `db.collection.count`

```javascript
db.collection.count(<query>, <options>)
```

является сокращением для цепочки `db.collection.find.count()`. Смотреть [`cursor.count`](#cursor-count)

Примеры:

```javascript
db.orders.find().count();  # полное количество документов
db.orders.find( { number: { $gt: 1 } } ).count(); # количество документов, соответствующих условию
db.orders.find( { number: { $gt: 1 } } ).limit(10).count(true); # считать количество с учетом указанного лимита
```





Первое и самое фундаментальное различие, с которым вам надо свыкнуться, это отсутствие у MongoDB аналога конструкции JOIN. Не

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [19:05](https://vk.com/im?sel=2733257&msgid=57809)

 

придется выполнять JOIN-ы на клиенте

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [23:34](https://vk.com/im?sel=2733257&msgid=57810)

 

db.employees.insert({_id: ObjectId("4d85c7039ab0fd70a117d730"), name: 'Leto'})

3 сентября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [8:08](https://vk.com/im?sel=2733257&msgid=57814)

 

Теперь добавим пару сотрудников и сделаем Leto их менеджером:db.employees.insert({_id: ObjectId("4d85c7039ab0fd70a117d731"), name: 'Duncan', m

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [8:16](https://vk.com/im?sel=2733257&msgid=57815)

 

_id может быть любым уникальным значением. Поскольку в жизни вы скорее всего станете использовать Objectid, мы также здесь используем его.)

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [8:53](https://vk.com/im?sel=2733257&msgid=57817)

 

В худших случаях отсутствие JOIN-ов чаще всего потребует дополнительного запроса (как правило индексированного).

 

она чертовски удобна, когда требуется смоделировать отношения ''один-ко-многим''или ''многие-ко-многим''. Например, если у сотрудника есть несколько менеджеров, мы просто можем сохранить их в виде массива:db.employees.insert({_id: ObjectId("4d85c7039ab0fd70a117d733"), name: 'Siona', manager: [ObjectId("4iА самое интересное, что в одних документах mana

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:45](https://vk.com/im?sel=2733257&msgid=57820)

 

в одних документах manager можно сделать скалярным значением, а в других - массивом. А наш предыдущий запрос find сработает в обоих случаях:db.employees.find({manager: ObjectId("4d85c7039ab0fd70a117d730")})

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:51](https://vk.com/im?sel=2733257&msgid=57821)

 

Кроме массивов MongoDB также поддерживает вложенные документы. Попробуйте вставить документ со вложенным документом, например:db.employees.insert({_id: ObjectId("4d85c7039ab0fd70a117d734"), name: 'Ghanima', family: {mother: 'ChВложенные документы можно запрашивать с помощью точечной нотации: db.employees.find({'family.mother': 'Chani'})

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:56](https://vk.com/im?sel=2733257&msgid=57822)

 

MongoDB поддерживает понятие под названием DBRef, которое является соглашением, принятым во многих драйверах. Когда драйвер видит DBRef, он может автоматически получить связанный документ. DBRef включает в себя коллекцию и _id документа, на который он ссылается. Э

 

Еще одна альтернатива использованию JOIN-ов - денормализация

 

ие - хранить имя пользователя (name) вместе с userid для каждого поста.Можно также вставлять небольшой встроенный документ, например, user: {id: ObjectId('Something'), name: 'Leto'}. Да, если позволить пользователям изменять своё имя, нам придётсяобновлять каждый документ (пост) - это один лишний запрос.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [11:58](https://vk.com/im?sel=2733257&msgid=57830)

 

Также полезной стратегией в случаях отношения ''один-ко-многим'' или ''многие-ко- многим'' является массив идентификаторов. Бытует мнение, что DBRef используется не так часто,

 

одиночный документ ограничен в размере до 4 мегабайт.

 

ольшинство разработчиков склоняются к использованию заданных вручную ссылок. Вложенные документы используются часто, но для небольших объёмов данных, если их желательно всегда извлекать вместе с родительским документом.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [12:11](https://vk.com/im?sel=2733257&msgid=57833)

 

MongoDB позволяет запрашивать и индексировать поля вложенных документов.

 

Учитывая то, что коллекции не привязывают нас к конкретной схеме, вполне возможно обойтись одной коллекцией, имеющей документы разной структуры

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [15:06](https://vk.com/im?sel=2733257&msgid=57851)

 

большинство разработчиков предпочитают разделять сущности. Так понятнее и яснее.

4 сентября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [14:54](https://vk.com/im?sel=2733257&msgid=57943)

 

м преимуществом документ-ориентированных баз данных является то, что они бесструктурны

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [19:56](https://vk.com/im?sel=2733257&msgid=57980)

 

Бесструктурность заманчива, однако большая часть данныхдолжна быть хорошо структурированной.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [20:12](https://vk.com/im?sel=2733257&msgid=57981)

 

преимущество бесструктурной архитектуры - это отсутствие установки и сведённые к минимуму расхождения с ООП.

 

Динамизм Ruby и популярная реализация ActiveRecord уже ощутимо сокращают расхождение объектной и реляционной моделей (object-relational

 

Вам надо сохранить объект? Сериализируйте его в JSON (на самом деле в BSON, но это почти одно и то же) и отправьте в MongoDB. Нет никакого маппинга свойств или типов. Эта простота о

 

Область, для которой MongoDB особенно подходит, - это логгирование

 

можно отправить команду записи и продолжить работу, не ожидая её возврата и действительной свершившейся записи.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [22:28](https://vk.com/im?sel=2733257&msgid=57989)

 

можем создать ограниченную коллекцию с помощью команды db.createCollection, включив флаг capped://ограничиваем размер коллекции до 1 мегабайта db.createCollection('logs', {capped: true, size: 1048576})

 

Когда наша ограниченная коллекция достигнет размера в 1 мегабайт, старые документы начнут автоматически удаляться. Можно также задать не размер коллекции, а максимальное количество документов, с помощью опции max.

 

что если нужно выяснить, вызвала ли ваша запись какие-либо ошибки (как, например, в уже упомянутом случае, когда мы не дожидаемся её завершения), можно просто выполнить следующую команду: db.getLastError(). Б

 

Одной из самых важных функций, добавленных в MongoDB 1.8, стало журналирование. Чтобы включить его, добавьте journal=true в файл mongodb.config, с

 

при отключении журналирования, возможен определенный риск

 

MongoDB не поддерживает транзакций. Есть две альтернативы: одна - замечательная, но ограниченная в использовании, а другая - громоздкая, но гибкая.Первая альтернатива - это множество атомарных операций. Они прекрасны до тех пор, пока решают вашу проблему.

 

, $inc и $set. Также существуют команды вроде findAndModify которые могут обновлять или удалять документ и автоматически его возвращать.

 

Вторая альтернатива - когда атомарных операций не хватает - это двухфазный коммит

 

Общая идея состоит в том, что вы храните состояние транзакции внутри обновляющегося документа и проходите шаги init-pending-commit/rollback вручную.

 

ля большинства задач обработки данных MongoDB использует MapReduce.

 

сть, конечно, некоторые базовые агрегирующие функции, но для чего-либо серьёзного вам понадобится MapReduc

 

Одно из преимуществ MapReduce в том, что для работы с большими объёмами данных он может выполняться параллельно.

 

Особенно мощной функцией MongoDB является её поддержка геопространственных индексов.Это позволяет сохранять x- и y-координаты у документов и затем находить документы вблизи ($near) определённых координат, или внутри ($within) прямоугольника либо окружности.

5 сентября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [8:36](https://vk.com/im?sel=2733257&msgid=58019)

 

отсутствие поддержки десятичных чисел с плавающей запятой, очевидно, будет проблемой (хотя и не обязательно непреодолимой) для систем, имеющих дело с деньгами

 

ь. Теоретически MapReduce может быть распараллелен, что позволяет обрабатывать огромные массивы данных на множестве ядер/процессоров/машин.

 

упоминалось, это пока не является преимуществом MongoDB. Вторым преимуществом MapReduce является возможность описывать обработку данных нормальным кодом. По

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [16:40](https://vk.com/im?sel=2733257&msgid=58050)

 

MapReduce - процесс двухступенчатый. Сначала делается map (отображение), затем - reduce (свёртка). На этапе отображения входные документы трансформируются (map) и порождают (emit) пары ключ = >значение (как ключ, так и значение могут быть составными). При свёртке (reduce) на входе получается ключ и массив значений, порождённых для этого ключа, а на выходе получается финальный результат. П

 

ем генерировать отчёт по дневному количеству хитов для какого- либо ресурса (например, веб-страницы). Это hello world для MapReduce. Для наших задач мы воспользуемся коллекцией hits с двумя полями: resource и date. Желаемый результат - это отчёт в разрезе ресурса, года, месяца, дня и количества.

 

Пусть в hits лежат следующие данные:resourcedateindexJan2020104:30indexJan2020105:30

 

На выходе мы хотим следующий результат:resourceyearmonthdaycountindex20101203about20101201

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [22:23](https://vk.com/im?sel=2733257&msgid=58093)

 

Для каждого документа мы должны породить ключ, состоящий из ресурса, года, месяца и дня, и примитивное значение - единицу:function() { var key = {resource: this.resource, year: this.date.getFullYear(), month: this.date.getMonth(), day: this.date.getDate()};emit(key, {count: 1});}this ссылается на текущ

 

this ссылается на текущий рассматриваемый документ. Надеюсь, результирующие данные прояснят для вас картину происходящего. При использовании наших тестовых данных, в результате получим:{resource: 'index', year: 2010, month: 0, day: 20} => [{count: 1}, {count: 1}, {count:1}]

 

Reduce-функция берёт каждое из этих промежуточных значений и выдаёт конечный результат.Вот так будет выглядеть наша функция

 

Технически в MongoDB результат выглядит так:_id: {resource: 'home', year: 2010, month: 0, day: 20}, value: {count: 3}

 

На деле reduce не всегда вызывается с полным и совершенным набором промежуточных данных.

6 сентября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [8:00](https://vk.com/im?sel=2733257&msgid=58099)

 

Коммутативность

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [8:23](https://vk.com/im?sel=2733257&msgid=58100)

 

С MongoDB мы вызываем у коллекции команду mapReduce. mapReduce принимает функцию map, функцию reduce и директивы для результата

 

Теперь можно создать map и reduce функции

 

Мы выполним команду mapReduce над коллекцией hits следующим образом:db.hits.mapReduce(map, reduce, {out: {inline:1}})

 

Установив out в inline мы указываем, что mapReduce должен непосредственно вернуть результат в консоль

 

и результат был бы сохранён в коллекцию hit stats:

 

db.hits.mapReduce(map, reduce, {out: 'hit_stats'});

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [15:04](https://vk.com/im?sel=2733257&msgid=58111)

 

В таком случае все существовавшие данные из коллекции hit_stats были бы вначале удалены. Если бы мы написали {out: {merge: 'hit_stats'}}, существующие значения по соответствующим ключам были бы заменены на новые, а другие были бы вставлены. И наконец, можно в out использовать reduce функцию - для более сложных случаев.Третий параметр принимает дополнительные значения - например, можно сортировать, фильтровать или ограничивать анализируемые данные. Мы также можем передать метод finalize, который применится к результату возвращённому этапом reduce.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [21:19](https://vk.com/im?sel=2733257&msgid=58255)

 

ели коллекцию system.indexes, которая содержит информацию о всех индексах в нашей базе данных

 

Индексы создаются с помощью ensureIndex:db.unicorns.ensureIndex({name: 1});И уничтожаются с помощью dropIndex: db.unicorns.dropIndex({name: 1});

 

Можно создавать индексы над вложенными полями (опять же, используя точечную нотацию), либо над массивами. Также можно создавать составные индексы:db.unicorns.ensureIndex({name: 1, vampires: -1});Порядок вашего индекса (1 для восходящего и —1 для нисходящего) не играет роли в случае с простым индексом, однако он может быть существенен при сортировке или лимитировании с применением составных индексов.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [22:16](https://vk.com/im?sel=2733257&msgid=58260)

 

ExplainЧтобы увидеть, используются ли индексы в ваших запросах, вызывайте у курсора метод explain:db.unicorns.findQ.explainQ

 

В результате мы видим информацию, что использовался BasicCursor (то есть не индексированный), сканирование происходило по 12 объектам, как много это времени заняло, применялся ли индекс, и если да, то какой, а также прочие полезные сведения.Если мы изменим запрос так, чтобы он использовал индекс, мы увидим, что использовался курсор BtreeCursor, а также увидим индекс, использованный при выборке:db.unicorns.find({name: 'Pilot'}).explain()

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [22:22](https://vk.com/im?sel=2733257&msgid=58262)

 

запись данных в MongoDB происходит без подтверждения. Э

 

что когда обновление или вставка нарушают условие уникальности индекса, ошибки не происходит. Чтобы узнать о возникновении ошибки, после последней записи нужно вызывать db.getLastError(). Многие драйверы обходят это и позволяют писать безопасно- часто для этого имеется специальный параметр.К сожалению, консоль не умеет этого делать,

 

MongoDB поддерживает авто-шардинг

 

Репликация

 

Типичным подходом является сочетание репликации и шардинга. Например, каждый шард может состоять из ведущего и ведомого серверов.

 

СтатистикаСтатистику базы данных можно получить с помощью вызова db.stats(). В основном информация касается размера вашей базы данных. Также можно получить статистику коллекции, например unicorns, с помощью вызова db.unicorns.statsQ. Боьшая часть получаемой информации, опять же, касается размеров коллекции

7 сентября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:49](https://vk.com/im?sel=2733257&msgid=58269)

 

Когда mongod запускается, в консоли появляется, среди прочих, строчка со ссылкой на административный веб-интерфейс. Вы можете получить к нему доступ, зайдя в браузере на http://localhost:280i7/. Чтобы получить от него максимальную отдачу, можете добавить rest=true в конфигурационный файл и перезапустить процесс mongod. Веб-интерфейс даёт много интересной информации о текущем состоянии сервера.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [15:02](https://vk.com/im?sel=2733257&msgid=58297)

 

Профайлер MongoDB можно включить с помощью следующего вызова:db.setProfilingLevel(2);Со включённым профайлером можно запустить команду: db.unicorns.find({weight: {$gt: 600}});И обратиться к профайлеру:db.system.profile.find()В результате мы увидим, что и когда запускалось, как много документов сканировалось, как много данных было возвращено.Можно выключить профайлер, повторно вызвав setProfileLevel, только передав 0 в качестве аргумента. Можно также передать 1для профилирования запросов, выполняющихся дольше 100 миллисекунд. Также, можно вторым параметром передать время в миллисекундах://профилировать всё, что занимает более 1 секунды