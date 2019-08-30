#  Состав приложения

В папке `bin` находятся следующие основные файлы:

* `mongod` – сервер
* `mongo` – клиентская консоль

* `mongod.cfg` – конфигурационный файл. Возможно его требуется создать вручную. 

  До версии 2.4 формат файла (как `ini` файл):

  ```ini
  <setting> = <value>
  ```

  Минимальная конфигурация:

  ```yaml
  dbpath = /data/db/ # Папка для хранения данных
  ```

  С версии 2.4 используется *YAML* формат.

  Минимальная конфигурация:

  ```yaml
  storage:
    dbPath: /data/db/
  ```

Запуск сервера:

```bash
mongod --config /path/to/mongod.cfg
```

# Термины

* *База данных* – самый большой контейнер
* В базе данных размещаются *коллекции* (аналог таблицы). *Коллекция* не хранит информации о  структуре и набор возможных полей (в отличии от таблицы в SQL).
* Коллекции состоят из *документов* (аналог строки). *Документ* хранит информацию о полях, которые находятся внутри него. Разные *документы* могут иметь различные наборы полей. 
* Документ состоит из *полей* (аналог полей и колонок).  
* *Индекс*
* При запросе данных в ответ возвращается *курсор*, через который мы может взаимодействовать с данными 

# Консоль

Команды в консоли записываются на *Javascript*. 

## Глобальные команды

| Команда | Описание            |
| ------- | ------------------- |
| `help`  | Справка по командам |
| `exit`  | Выход               |

## Работа с базой данных

Доступ к командам для управления текущей базой данных выполняется через объект `db`. 

* Получить список методов, доступных для объекта `db`:

    ```javascript
    db.help()
    ```

- Узнать текущую БД:

  ```javascript
  > db
  test
  ```

- Переключиться на другую БД или создать БД, если она не существует:

  ```javascript
  use <databaseName>
  ```

  

## Работа с коллекциями

Доступ к командам для управления коллекцией выполняется через объект `db.<collectionName>`. Создание коллекции выполняется через `db.collection.insert()`

- Получить список коллекций в текущей БД

  ```javascript
  db.getCollectionNames()
  ```

## CRUD операции с документами

### Create

Вставка нового документа в коллекцию. Если коллекция не существует, то она создается:

```javascript
db.collection.insert(<document>);
db.collection.insert({  /* Вставляемый документ */
    name: "Pavel",      /* Поле name */ 
    age: 34             /* Поле age */  
});
```

### Read

 Выбор всех документов:



метод find, который вернет список документов:db.unicorns.find()

 

Каждый документ должен иметь уникальное поле _id. Можете генерировать его сами

1 сентября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:47](https://vk.com/im?sel=2733257&msgid=57763)

 

или позволить MongoDB самой сгенерировать для

 

По умолчанию _id - индексируемое поле, вследствие чего и создается коллекция system.indexes. Давайте взглянем на system.indexes:db.system.indexes.find()Вы увидите имя индекса, базы данных и коллекции, для которой индекс был создан, а также полей, которые включены в него.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:54](https://vk.com/im?sel=2733257&msgid=57765)

 

селекторы запросов.

 

Селектор - это JSON-объект, в простейшем случае это может быть даже {}, что означает выборку всех документов (аналогичным образом работает null). Если нам нужно выбрать всех единорогов (англ. ''unicorns'') женского рода, можно воспользоваться селектором {gender:'f'}.

 

удалим всё, что до этого вставляли в коллекцию unicorns с помощью команды: db.unicorns.remove() (поскольку мы не передали селектора, произойдет удаление всех документов). Т

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [14:01](https://vk.com/im?sel=2733257&msgid=57770)

 

{поле: значение} используется для поиска всех документов, у которых поле равно значение. {поле1: значение1, поле2: значение2} работает как логическое И. Специальные операторы $lt, $lte, $gt,$gte и $ne используются для выражения операций ''меньше'', ''меньше или равно'',''больше'', ''больше или равно'', и ''не равно''. Например, чтобы получить всех самцов единорога, весящих более 700 фунтов, мы можем написать:db.unicorns.find({gender: 'm', weight: {$gt: 700}})//или (что не полностью эквивалентно, но приведено здесь в демонстрационных целях) db.unicorns.find({gender: {$ne: 'f'}, weight: {$gte: 701}})

 

Оператор $exists используется для проверки наличия или отсутствия поля, например: db.unicorns.find({vampires: {$exists: false}})

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [20:35](https://vk.com/im?sel=2733257&msgid=57774)

 

Если нужно ИЛИ вместо И, мы можем использовать оператор $or и присвоить ему массив значений, например:db.unicorns.find({gender: 'f', $or: [{loves: 'apple'}, {loves: 'orange'}, {weight: {$lt: 500}}]})Вышеуказанный запрос вернет всех самок единорогов, которые или любят яблоки, или любят апельсины, или весят менее 500 фунтов.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [20:52](https://vk.com/im?sel=2733257&msgid=57775)

 

и - поле loves это массив. MongoDB поддерживает массивы как объекты первого класса.

 

я выборка по значению массива: {loves: 'watermelon'} вернет нам все документы, у которых watermelon является одним из значений поля loves.

 

. Самый гибкий оператор - $where, позволяющий нам передавать JavaScript для его выполнения на сервере.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [22:06](https://vk.com/im?sel=2733257&msgid=57783)

 

и селекторы могут быть использованы с командой find. Они также могут быть использованы с командой remove, которую мы кратко рассмотрели, командой count, на которую мы пока не взглянули, но которую вы скорее всего изучите, и командой update,

 

Objectid, сгенерированный MongoDB для поля _id, подставляется в селектор следующим образом:db.unicorns.find({_id: ObjectId("TheObjectId")})

 

В простейшей форме, update принимает 2 аргумента: селектор (where) для выборки и то, чем обновить соответствующее поле. Чтобы Roooooodles прибавил в весе, используем следующий запрос:db.unicorns.update({name: 'Roooooodles'}, {weight: 590})

 

второй параметр используется для полной замены оригинала. Иными словами, update нашел документ по имени и заменил его целиком на новый документ (свой второй параметр). Вот в чём отличие от SQL-команды update.

 

Однако, если вам нужно всего лишь изменить пару полей, лучше всего использовать модификатор $set:

 

db.unicorns.update({name: 'Roooooodles'}, {$set: {weight: 590}})

 

модификатор $inc служит для того, чтобы изменить поле на положительную (увеличить) или отрицательную (уменьшить) величину . Например, если единорог Pilot был ошибочно награжден за убийство пары лишних вампиров, мы можем исправить эту ошибку следующим образом:db.unicorns.update({name: 'Pilot'}, {$inc: {vampires: -2}})

2 сентября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [7:46](https://vk.com/im?sel=2733257&msgid=57790)

 

Если Aurora внезапно пристрастилась к сладостям, мы можем добавить соответствующее значение к ее полю loves с помощью модификатора $push:db.unicorns.update({name: 'Aurora'}, {$push: {loves: 'sugar'}})

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [8:20](https://vk.com/im?sel=2733257&msgid=57791)

 

Обновление/вставка обновляет документ, если он найден, или создаёт новый - если не найден.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:08](https://vk.com/im?sel=2733257&msgid=57792)

 

е. Чтобы разрешить вставку при обновлении, установите третий параметр в true.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:09](https://vk.com/im?sel=2733257&msgid=57793)

 

счетчик посещений для веб-сайта

 

если разрешить вставку при обновлении, результаты будут иными:db.hits.update({page: 'unicorns'}, {$inc: {hits: 1}}, true);

 

update - это, то что он по умолчанию обновляет лишь один документ.

 

Чтобы это сработало, нужно установить четвертый параметр в true:db.unicorns.update({}, {$set: {vaccinated: true }}, false, true); db.unicorns.find({vaccinated: true});

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:25](https://vk.com/im?sel=2733257&msgid=57797)

 

find принимает второй необязательный параметр. Это - список полей, которые мы хотим получить.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:02](https://vk.com/im?sel=2733257&msgid=57798)

 

все имена единорогов следующим запросом:db.unicorns.find(null, {name: 1});

 

Поле _id по умолчанию возвращается всегда. Мы можем явным способом исключить его, указав {name:1, _id: 0}.За исключением поля _id, нельзя смешивать включения и исключения полей.

 

Синтаксис sort примерно такой же, как у выбора полей,

 

, используя 1 для сортировки по возрастанию и -1 для сортировки по убыванию. Например://сортируем по весу - от тяжёлых к лёгким единорогам db.unicorns.find().sort({weight: -1})//по имени вампира, затем по числу убитых вампиров: db.unicorns.find().sort({name: 1, vampires: -1})

 

без индекса MongoDB ограничивает размер сортируемых данных. Если вы попытаетесь отсортировать большой объем данных, не используя индекс, вы получите ошибку

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:55](https://vk.com/im?sel=2733257&msgid=57803)

 

азбиение на страницы может быть осуществлено с помощью методов limit и skip. Чтобы получить второго и третьего по весу единорога, можно выполнить:db.unicorns.find().sort({weight: -1}).limit(2).skip(1)Используя limit вместе с sort можно избежать проблем с сортировкой по неиндексированным полям.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [18:41](https://vk.com/im?sel=2733257&msgid=57807)

 

count прямо над коллекцией: db.unicorns.count({vampires: {$gt: 50}})На практике же count - это метод курсора, консоль просто обеспечивает удобное сокращение. С драйверами, не поддерживающим подобного сокращения, нужно писать что-то вроде этого (конечно, и в консоли тоже так можно):db.unicorns.find({vampires: {$gt: 50}}).count()

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [18:49](https://vk.com/im?sel=2733257&msgid=57808)

 

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