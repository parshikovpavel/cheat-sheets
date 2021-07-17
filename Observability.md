Мониторинг состоит из частей:

- сбор метрик
- хранение
- визуализация
- алертинг

Graphite – использует очень простой протокол. Конектишься на порт Graphite и отправляешь строку в формате:

```
<path> <value> [timestamp]
moira.checkers.error.description 0.0
```

- `<path` – метрика, записанная в виде пути. Как правило, в формате `<имя проекта>.<имя сервиса>....`

У Graphite удобный API, который позволяет использовать:

- pattern, со звездочкой `moira.checkers.error.*`
- использовать функции `sumSeries()`

Чтобы осуществлять алертинг необходимо описать trigger'ы. Для описания trigger'а нужен:

- описание metric'и
- лимиты для состояния. Как правило используются состояния:
  - WARN – что-то идет не так
  - ERROR – что-то упало и нужно срочно действовать
  - NODATA – данные не поступают какое-то время

https://ish-ar.io/observability/

https://coderlessons.com/articles/programmirovanie/prometei-protiv-grafita-chto-vybrat-dlia-vremennykh-riadov-ili-monitoringa

# Graphite

Graphite выполняет две функции:

1. Хранение числовых *time-series* данные
2. Отображение графиков из этих данных по запросу

Graphite - это хорошо масштабируемая система реального времени для построения графиков. Приложение собирает числовые *time-series* данные и отправляет их в бэкэнд *Graphit*'а (*carbon*), который хранит данные в специализированной базе данных. Затем данные могут быть визуализированы через веб-интерфейс *Graphit*'а.

## *Placeholder*'ы для *metric*

https://graphite.readthedocs.io/en/latest/render_api.html#graphing-metrics

## Функции

https://graphite.readthedocs.io/en/latest/functions.html

### `alias()`

```
alias(seriesList, newName)
alias(Sales.widgets.largeBlue,"Large Blue Widgets")
```

Принимает одну *metric* или *wildcard seriesList* и строку в кавычках. Печатает строку вместо *metric name* в легенде.





### `aliasByNode()`

```
aliasByNode(seriesList, *nodes)
```

Принимает *seriesList* и применяет *alias*, полученный из одной или нескольких частей *“node”* целевого *name* или *tags*. Индексы node's начинаются с 0.

Каждая *node* может быть целым числом, указывающим на *node* в *series name*, или строкой, идентифицирующей *tag*.



### `aliasSub()`

```
aliasSub(seriesList, search, replace)
```

Применят для *series names* регулярное выражение из `search` и делает замену на `replace`.



### `consolidateBy()`

Полезно:

- чтобы отображать правильные *total* по метрикам в *graphana*, когда нужно сделать `sum`
- когда нужно, чтобы *graphite* не усреднял значения во временных интервалах.

```
consolidateBy(seriesList, consolidationFunc)
```

```
aliasByNode(consolidateBy(env.$env.apps.services.adv-credit-broker.sravni.*.*, 'sum'), 6, 7)
```



Принимает одну *metric*, *wildcard seriesList* и *consolidation function name*.

Допустимые имена функций: `‘sum’`, `‘average’/’avg’`, `‘min’`, `‘max’`, `‘first’` & `‘last’`.

Когда строится график, ширина которого в пикселях меньше, чем количество точек данных для построения графика, Graphite объединяет значения, чтобы предотвратить перекрытие линий. Функция `consolidateBy()`  изменяет функцию консолидации со значения по умолчанию `‘average’` на одно из `‘sum’`, `‘max’`, `‘min’`, `‘first’`, или `‘last’`.





### `exclude()`

```
exclude(seriesList, pattern)
exclude(servers*.instance*.threads.busy,"server02")
```

- `seriesList` – *metric* или *wildcard seriesList*
- `pattern` – *regular expression* в двойных кавычках. 

Исключает *metric*'s, соответствующие *regular expression*.

<u>Пример:</u>

Исключение ручки `_info`:

```
exclude(..., '_info_GET')
```







### `movingAverage()`

Отображает скользящее среднее *metric*'и (или *metric*) за фиксированное количество прошлых *point*'s или *time interval* для каждой точки на графике.

```
movingAverage(seriesList, windowSize, xFilesFactor=None)
```

- `seriesList` – одна *metric*'а или *wildcard seriesList* (подстановочный? наверное паттерн)
- `windowSize` – число N *datapoint*'s или строка с временным периодом, например, `'1hour'` или `'5min'`  ([формат строки с интервалом](https://graphite.readthedocs.io/en/latest/render_api.html#from-until)). Отображает среднее значение предыдущих точек данных для каждой точки на графике.`from / until`
- `xFilesFactor` – сколько точек в *window* должно быть *non-null* , чтобы вывод рассматривался как *valid*.

```
&target=movingAverage(Server.instance01.threads.busy,10)
&target=movingAverage(Server.instance*.threads.idle,'5min')
```







### `movingSum()`

Отображает скользящей суммы *metric*'и (или *metric*) за фиксированное количество прошлых *point*'s или *time interval* для каждой точки на графике.

```
movingSum(seriesList, windowSize, xFilesFactor=None)
```

- `seriesList` – одна *metric*'а или *wildcard seriesList* (подстановочный? наверное паттерн)
- `windowSize` – число N *datapoint*'s или строка с временным периодом, например, `'1hour'` или `'5min'`  ([формат строки с интервалом](https://graphite.readthedocs.io/en/latest/render_api.html#from-until)). Отображает среднее значение предыдущих точек данных для каждой точки на графике.
- `xFilesFactor` – сколько точек в *window* должно быть *non-null* , чтобы вывод рассматривался как *valid*.

<u>Пример:</u>

Используется, чтобы убрать разрывы в графике

```
&target=movingSum(Server.instance01.threads.busy,10)
&target=movingSum(Server.instance*.threads.idle,'5min')
```









### `sortByMaxima(seriesList)`

```
sortByMaxima(seriesList)
sortByMaxima(server*.instance*.memory.free)
```

- `seriesList` – *metric* или *wildcard seriesList*.

Сортирует список *metric*'s в порядке убывания *maximum value* за выбранный период времени. 





### `sum()` и `sumSeries()`

Сложить *metric*'s вместе и вернуть сумму в каждой точке данных.

```
sum(*seriesLists)
sumSeries(*seriesLists)
```







### `transformNull()`

```
transformNull(seriesList, default=0, referenceSeries=None)
```

Принимает *metric* или *wildcard seriesList* и заменяет *null* значения –*default* значением (по умолчанию `0`) . 

- `referenceSeries` – *metric* или *wildcard series list*, который определяет, какие *time intervals nulls* должны быть заменены. Если указано, nulls заменяются только в тех интервалах, где non-null найдены для того же *interval* в любой из `referenceSeries`. 



# Graphana

Данные складываюся в Graphite аггрегированные, с интервалом в 30 секунд. Т.е. один блок данных - это метрика за 30 секунд (????).

## Проблема с average

Если есть проблема с расчетом *average* – это может быть связано с тем, что в расчет попадают значения `null`, которые рассматриваются как `0` (*zero*).

Можно поиграться с настройкой `Null value` (важно – результат изменения показывается в легенде только после перезагрузки страницы).

Возможные варианты:

- `null as zero` – значение `null` заменяется на `0` (`zero`)

- `null` – ничего не делать со значением `null`. При этом на большинстве графиков остается разрыв между предыдущими и последующими
  точками данных .

- `connected` – предыдущее и последующее ненулевые значения соединяются прямой линией, позволяя избежать разрывов в графике. 





# Метрики

## Пример организации метрик для web-сервиса:

 ```
apps
    services
        email-sender
            service-email-sender01
                api
                    send_POST
                        request_time
                            200
                                count
                                max
                                mean
                                ...
                            400
                            404
                            500
                        custom_timer1
                        custom_timer2
                        custom_counter1
                        custom_counter2
                received_emails
                received_fake_emails
            sender
                service-email-sender01
                    sent_mails
            fake_emails_extractor
                service-email-sender01
                    fetched_email_addresses
 ```

где

- apps - общий префикс для приложений
- email-sender - проект
- receiver - приложение
- service-email-sender01 - hostname контейнера
- realty_predict - название контроллера (хэндлера)
- request_time - стандартное поддерево для всех сервисов - хранит время обработки запросов с разбивкой по http статусам ответов
- 200, 400, ... - время обработки запросов с соответствующим кодом ответа
- count,max,mean - стандартные агрегации brubeck для таймеров
- custom_metric1,... - другие метрики



## Таблица

| Метрика                                                      | Описание                 |
| ------------------------------------------------------------ | ------------------------ |
| apps.services.rec-mordor.service.api.*.request_time.500.count | Кол-во ошибочных ответов |
| apps.services.rec-mordor.service.api.api1mainitems_GET.request_time.200.count | Кол-во успешных ответов  |
| movingAverage(apps.services.rec-mordor.service.api.api1mainitems_GET.request_time.200.percentile.98,  '1m') | Время успешного ответа   |



| Метрика                |  Тип   | Описание                                                     |
| :--------------------- | :----: | :----------------------------------------------------------- |
| api.*_*.request_time.* | timing | Общее время (мс) обработки всех endpoint'ов с их кодами ответов |

service api request_time



# Moira

У trigger'ов есть tag'и. Когда trigger срабатывает, далее переходит проверка к блоку alerting и срабатывают те alert'ы, у которых есть все tag'и из trigger'а. 

Причем у tag'а `ERROR` особое поведение. `ERROR` системный и добавляется к триггеру "налету", когда метрика в триггере переходит в состояние ERROR

Пример:

в trigger'е указано:

```
service-geo-resolver
MONAD
```

будут срабатывать *alert*'ы, у которых есть *trigger*'ы:

```
service-geo-resolver
MONAD
ERROR (обязательно ли?? может нет??)
```

MONAD = MONitoring All Day

https://www.youtube.com/watch?v=_MJdtHbiXPE&feature=emb_logo

https://moira.readthedocs.io/en/latest/gsoc.html

## `alert.yaml`

[1](http://stash.msk.avito.ru/projects/PYTHON/repos/alert-autoconf/browse/README.md) [2](https://cf.avito.ru/display/BD/Moira)

Файл конфигурации состоит из двух блоков:

- Настроика триггеров (triggers)
- Настройка контактов для оповещения (alerting)

В файле конфигурации обязятельно должен присутствовать блок контактов.

Блок триггеров состоит из следующих полей:

```
---
version: "1"                  # Версия пакета "1" или "1.1"
prefix: "any-string-"         # Аналог namespace для имён триггеров и тегов в moira,
                              # дополняет triggers[].name, triggers[].tags[] и alerting[].tags
                              # (только для version: "1.1")

# Блок триггеров
triggers:
  - name: Trigger_1           # Имя триггера
    targets:                  # Список метрик
      - stats.timers.service.rec-collab.service.avito-dev.api.rec_rec_POST.request_time.404.mean
      - stats.timers.service.rec-collab.service.avito-dev.api.rec_rec_POST.request_time.404.mean
    desc: Test description 1  # Описание триггера
    dashboard: "http://mntr.avito.ru/grafana/d/000000596/graphite-clickhouse?panelId=17&fullscreen"
                              # Ссылка на конкретную панель дашборда в Grafana (http://mntr.avito.ru/grafana)
    is_pull_type: true        # Ходить за данными в Graphite, не используя внутреннее хранилище Moira. По умолчанию Flase.
    pending_interval:0        # Сколько времени должен триггер находится в новом состоянии, перед тем как Мойра отправит по нему нотификацию
    warn_value: 15            # Выполнить рассылку со статусом 'Warning', если значение графика будет меньше указанного
    error_value: 10           # Выполнить рассылку со статусом 'Error', если значение графика будет меньше указанного

                              # (Если значение поля error_value больше или равно warn_value, то алгоритм оповещения
                              #  будет следующий:
                              #  Выполнить 'Warning' или 'Error' - если значение метрики больше указанного
                              # )

    ttl: 600                  # Количество секунд, через которе нужно сделать рассылку со статусом из поля 'ttl_state'
    tags:                     # Группа для оповещения
      - service 1
      - "cluster-{cluster}"   # Динамическая подстановка имени кластера
                              # (строки, в которых используется {cluster}, рекомендуется оборачивать в кавычки, чтобы парсер yaml не ругался)
    ttl_state: OK             # OK | WARN | ERROR | NODATA | DEL
                              # все состояния здесь - http://moira.readthedocs.io/en/latest/user_guide/efficient.html

    parents:                  # Список родительских триггеров (необязательно)
      - name: "DC shutdown"   # Ссылка на родителя — это его имя и набор тэгов
        tags:
        - datacenters
        - global
      - name: "autoconf maintenance"
        tags:
        - autoconf

    expression: "(t1 > 10) ? ERROR : ((t2 > 4) ? WARN : OK)"
                              # Ввод выражения.
                              # Поддерживает синтаксис govaluate и graphite. (t1, t2, tN - в порядке, указанном в targets)
                              # Подробнее - http://moira.readthedocs.io/en/latest/user_guide/advanced.html

                              # (По приоритету выше, чем warn_value && error_value )
    time_start: "12:30"       # Время начала применения данного триггера
    time_end: "23:59"         # Время окончания применения данного триггера
    day_disable:              # Дни, в которых данная настройка игнорируется (list)
      - Tue                   # Возможные значения:
                              #   Mon, Tue, Wed, Thu, Fri, Sat, Sun

    saturation:                        # Настройка насыщений
     - type: "cmdb-maintainer-tag"     # Тип насыщения, полный список в http://stash.msk.avito.ru/projects/DO/repos/service-fan/browse/saturators/const.go
       fallback: "fallback-tag"        # Фоллбек (необязательно, зависит от типа насыщения)
     - type: "sample-saturation"
       parameters:                     # некоторые насыщения принимают параметры
         animal: cow                   # список параметров и их значение зависят от конкретного типа насыщения
         sound: moo

  - name: Trigger_2
    targets:
      - movingAverage(apps.services.{cluster}.monitoring.modules.worker.*.handleMessage.time.count, 2)
    dashboard: mntr.avito.ru/grafana/d/000000596/graphite-clickhouse?panelId=17&fullscreen
    is_pull_type: true
    ttl: 60
    ttl_state: ERROR
    expression: "t1 <= 0.3 || t1 > 5 ? ERROR : OK"
    tags:
      - sms
      - mail
      - slack


# Блок оповещения
alerting:
  - tags:                            # Список тегов, задействованных в описании триггеров
      - service 1                    # Достаточно совпадения одного тега с тегом триггер
    time_start: "12:30"              # Время начала применения
    time_end: "23:50"                # Время окончания применения
    day_disable:                     # Дни, в которых данная настройка игнорируется
      - Tue
    contacts:                        # Список доступных контактов
      - type: mail                   # тип оповещения. Возможно указать следующие значения:
                                     # mail, send-sms, slack, twilio sms, twilio voice
        value: qwe@qwe.qwe           # значение контакта. Текст в оле нужно заключтиь в кавычики,
                                     # если используются спец-символы #,@,+ и т.д.

  - tags:
      - service 1
      - service 2
    contacts:
      - type: mail
        value: qwe@qwe.qwe
      - type: mail
        value: qwe@qwe.qwe

  - tags:
      - service 1
      - service 2
      - service 3
    contacts:                    # Список допустимых типов контакта
      - type: mail
        value: qwe@qwe.qwe
      - type: send-sms
        value: '+71234567891'
      - type: mail
        value: qwe@qwe.qwe
      - type: pushover
        value: test
      - type: slack
        value: '#chat-name'
      - type: twilio sms
        value: tw_sms
      - type: twilio voice
        value: tw_vioce
```

### Свойства триггеров по умолчанию

Триггерам можно задавать свойства по умолчанию в зависимости от тэгов. Например:

```
defaults:
  - condition:      # условие
      tags:         # сейчас поддерживается только `tags`: список тэгов
        - "MONAD"
        - hardware  # все триггеры с тэгами MONAD и hardware (одновременно)
    values:         # значения по умолчанию
      parents:      # сейчас поддерживается только `parents`
        - tags:     # всем триггерам, подходящим под условие, добавляется этот родитель (если его ещё нет)
          - global
          name: 'global parent'
```