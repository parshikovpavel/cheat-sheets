# Установка

```bash
brew install redis
brew services start redis
```

# Общее

Особенности:

- хранит все данные в памяти
- может сохранять данные на диск для обеспечения *persistence*

Причины высокой скорости:

- данные в оперативной памяти
- специализированные *data type*'s (типы данных)

Недостатки:

- *data type*'s специализированы и не универсальны

# Persistence

Можно:

-  настроить (в *configuration*) Redis так, чтобы он сохранял *snapshot* каждые N секунд, если в наборе данных есть хотя бы M изменений 
- вручную вызывать команду `SAVE` .

По умолчанию, Redis сохраняет *snapshot*  с разными интервалами, в зависимости от количества измененных записей:

- изменилось 9 ключей – 15 минут
- ...
- изменилось 1000 или более ключей – 60 секунд.

## Append-only file (AOF)

Данные, сохраненные в память в промежутках между созданием *snapshot*'ов могут быть утеряны. Если  риск потери данных последних N секунд в случае аппаратной или программной ошибки неприемлем, следует использовать AOF.

Запись в AOF включается в дополнение к созданию *snapshot*'ов. Каждый раз, когда Redis получает команду, которая изменяет данные, он добавляет ее в AOF. Когда вы перезапустите Redis, он заново проиграет AOF, чтобы восстановить исходное состояние.

Иногда можно отключить AOF, чтобы повысить производительность в ущерб надежности.

# Профилирование

Примерная скорость выполнения операций / секунду – 10K-100K операций/секунду.

Точно можно замерить утилитой `redis-benchmark`.

# Консольная утилита

```bash
$ redis-cli
```

# Database

Аналогично реляционным БД, данные размещаются в *database*. *Database* предназначена для группировки всей информации конкретного приложения в одном месте и изоляции ее от других приложений.

Название *database* – целое число. Название *database* по умолчанию – `0`. 

## `SELECT`

Для смены текущей *database* используется команда `SELECT`:

```
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]>
```

Название текущей *database* указано в квадратных скобках (`[1]`).



# Key, value

Данные любых типов хранятся в виде пары:

```
key value
```

`key` – любая двоичная последовательность (вплоть до набора байт из содержимого файла), для поиска *value*

`value` – произвольный массив байт

# Структуры данных

Redis включает пять специализированных *data type*'s

## String

String (строка) – самый простой тип, скалярная величина, базовый тип для key-value DB. Этот тип является единственным в Memcached.

### `SET`

Установите `key` значение `value`.

```redis
SET key value
SET ab "cd"
```

### `GET`

Получить значение

```
GET key
```

### `APPEND`

Добавить `value` в конец строки с ключом `key`.

```redis
APPEND key, value
APPEND ab "end"
```

### `GETRANGE`

Найти подстроку с позиции `start` до  `end` (включительно).

```redis
GETRANGE key start end
```

### `STRLEN`

Длина строки.

```redis
STRLEN mykey
```

### `INCR`, `INCRBY`

Инкрементировать значение на 1 или на на `increment`

```
INCR mykey
INCRBY key increment
```

### `DECR`, `DECRBY`

Декрементировать значение на 1 или на на `increment`

```
DECR mykey
DECRBY key increment
```

### `GETBIT`, `SETBIT`

Прочитать и установить значение бита.

```
GETBIT key offset
SETBIT key offset value
```

`offset` должен быть в диапазоне [*0 ÷ 2<sup>32</sup>*] (размер `value`  до 512 МБ).

## Hash

Также структура вида `key value`. Е
