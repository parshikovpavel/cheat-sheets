# Синтаксис

## Операторы манипуляции данными

### `SELECT`

```mysql
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
      [HIGH_PRIORITY]
      [STRAIGHT_JOIN]
      [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
      [SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr ...]
    [FROM table_references
      [PARTITION partition_list]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
    [HAVING where_condition]
    [WINDOW window_name AS (window_spec)
        [, window_name AS (window_spec)] ...]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
      | INTO DUMPFILE 'file_name'
      | INTO var_name [, var_name]]
    [FOR {UPDATE | SHARE} [OF tbl_name [, tbl_name] ...] [NOWAIT | SKIP LOCKED] 
      | LOCK IN SHARE MODE]]
```

#### Условия в `JOIN ON` и `WHERE`

Где располагать условие соединения - в `JOIN ON` или `WHERE`? Зависит от типа соединения:

* Для `INNER JOIN` – разницы нет. Оптимизатор скорее всего построит одинаковый план для любого варианта.

* Для `OUTER JOIN` – разница есть:

  * условие в `WHERE` выполняется *после* соединения. Таблицы вначале соединяются, а потом строки фильтруются по условию.

    ```mysql
    SELECT *
    FROM t1 LEFT JOIN t2 ON t1.id = t2.id
    WHERE t2.id = 1
    +--------+--------+
    | t1.id  | t2.id  |
    +--------+--------+
    | 1      | 1      |
    +--------+--------+
    ```
  
* условие в `JOIN ON` выполняется *до* соединения. При этом будут отфильтрованы только строки из правой таблицы, а строкам из левой таблицы может быть подставлен `NULL`.
  
    ```mysql
    SELECT *
    FROM t1 LEFT JOIN t2 ON t1.id = t2.id ON t2.id = 1
  +--------+--------+
  | t1.id  | t2.id  |
    +--------+--------+
    | 1      | 1      |
    | 2      | NULL   |
    +--------+--------+
  ```
  

Для удобства чтения запроса следует придерживаться правил:

* условие соединения помещается в `JOIN ON`
* условия фильтрации помещаются в `WHERE`.

#### `COUNT`

`COUNT` может применяться для подсчетов.

- Количество не `NULL` значений выражения `expr`

  ```mysql
  COUNT(expr)
  ```

  Если нет соответствующих строк, `COUNT()` возвращает 0.

  Например, найти все строки, где `id IS NOT NULL`

  ```mysql
  COUNT(id)
  ```

- Количество строк с *различными* не `NULL` значениями `expr`

  ```mysql
  COUNT(DISTINCT expr)
  ```

- Количество строк вообще.

  Лучше всего использовать специальную форму:

  ```mysql
  COUNT(*)
  ```

  которая вовсе не сводится к подстановке вместо метасимвола `*` полного списка столбцов таблицы, столбцы вообще игнорируются, а подсчитываются только строки.

Если MySQL точно знает, что выражение внутри скобок не может быть равно NULL, то просто подсчитывает строки. 

 Также можно использовать аналоги, вроде COUNT(1), т.к. в скобках записана константа и значений NULL нет.

Одна из наиболее часто встречающихся ошибок – задание имени столбца в скобках, когда требуется подсчитать строки. Если нужно узнать, сколько строк в результирующем наборе, всегда употребляйте COUNT(*). Тем самым вы избежите возможного падения производительности.

Для MyISAM COUNT(*) выполняется очень быстро, если SELECT извлекает данные из одной таблицы, никакие другие столбцы не извлекаются, и нет WHERE. Это возможно, потому что MyISAM хранит точное количество строк в таблице. MyISAM не обладает никакими магическими возможностями для подсчета строк, когда в запросе есть фраза WHERE, равно как и для подсчета значений, а не строк.

Например:

SELECT COUNT(*) FROM student;

Иногда оптимизацию COUNT(*) в MyISAM можно использовать, если требуется подсчитать все строки, кроме очень небольшого числа, т.е. заменить

SELECT COUNT(*) FROM world.city WHERE ID > 5;

на

SELECT (SELECT COUNT(*) FROM world.city)- COUNT(*)

FROM world.City 

WHERE ID <= 5; 

В этом варианте читается меньше строк, поскольку на стадии оптимизации подзапрос преобразуется в константу: 

+----+-------------+-------+...+------+------------------------------+ 
 | id | select_type | table |...| rows | Extra                        | 
 +----+-------------+-------+...+------+------------------------------+ 
 | 1  | PRIMARY     | City  |...| 6    | Using where; Using index     | 
 | 2  | SUBQUERY    | NULL  |...| NULL | Select tables optimized away | 
 +----+-------------+-------+...+------+------------------------------+

Для транзакционной InnoDB невозможно хранение точного количества строк, т.к. одновременно могут совершаться несколько транзакций, каждый из которых может влиять на количество. Если достаточно приблизительного количества строк, то можно использовать команду [SHOW TABLE STATUS](https://dev.mysql.com/doc/refman/5.7/en/show-table-status.html).

###### Оптимизация запросов с COUNT

Запросы, содержащие COUNT(), с трудом поддаются оптимизации, поскольку обычно они должны подсчитать много строк (то есть «прошерстить» большой объем данных). При таком запросе требуется поднять все данные в кеш (например, в innodb_buffer_pool), если они там не находятся.

Единственная возможность оптимизации внутри самого сервера MySQL – воспользоваться покрывающим индексом. 

Если этого недостаточно, рекомендуется внести изменения в архитектуру приложения. Например, самостоятельно реализовать счетчик количества строк и вручную этот счетчик изменять. Однако, такой счетчик может стать узким местом, если таблица активно изменяется. Счетчик может храниться:

·      в сводной таблице в самой базе данных (например, на fishki такой таблицей является fishki_user_stat).

·      в in-memory кеше (например, memcached). 

Придется выбрать 2 из трех – быстро, точно, просто.

COUNT(distinct id) – число уникальных id

 

 

 

 

##### HAVING

Идет сразу после GROUP BY, аналог WHERE для группированных полей.

##### Примеры

Допустим есть таблица user_group(user_id, group_id), которая связывает таблицы user и group. Необходимо найти количество уникальных групп, в которых состоят пользователи :user1 и :user2. При вычитании останутся только те группы, которые в списке были по 2 раза.

SELECT

​     COUNT(`*group_id*`) - COUNT(DISTINCT `group_id`)

FROM

​     `user_group`

WHERE

​     `user_id` IN (:user1,:user2)

#### INSERT

INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]

​    [INTO] *tbl_name*

​    [PARTITION (*partition_name* [, *partition_name*] ...)]

​    [(*col_name* [, *col_name*] ...)]

​    {VALUES | VALUE} (*value_list*) [, (*value_list*)] ...

​    [ON DUPLICATE KEY UPDATE *assignment_list*]

 

INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]

​    [INTO] *tbl_name*

​    [PARTITION (*partition_name* [, *partition_name*] ...)]

​    SET *assignment_list*

​    [ON DUPLICATE KEY UPDATE *assignment_list*]

 

INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]

​    [INTO] *tbl_name*

​    [PARTITION (*partition_name* [, *partition_name*] ...)]

​    [(*col_name* [, *col_name*] ...)]

​    SELECT ...

​    [ON DUPLICATE KEY UPDATE *assignment_list*]

 

*value*:

​    {*expr* | DEFAULT}

 

*value_list*:

​    *value* [, *value*] ...

 

*assignment*:

​    *col_name* = *value*

 

*assignment**_**list*:

​    *assignment* [, *assignment*] ...

IGNORE –  игнорировать ошибки, возникающие при выполнении и не прерывать выполнение оператора. Например, при дублировании UNIQUE индекса строка просто пропускается.

#### UPDATE

UPDATE [LOW_PRIORITY] [IGNORE] table_reference

​    SET assignment_list

​    [WHERE where_condition]

​    [ORDER BY ...]

​    [LIMIT row_count]

 

assignment_list:

​    assignment [, assignment] ...

 

assignment:

​    col_name = value

 

value:

​    {expr | DEFAULT}

В качестве значения (value) может быть указано ключевое слово DEFAULT, что означает использование значения по умолчанию для данного столбца. 

Если обновляются несколько столбцов и значение ранее обновленного столбца используются в последующих обновлениях, то используются уже новое значение, а не исходное. Например, здесь col2=(col1+1)+1, т.е. второе присваивание использует уже новое (обновленное) col1 значение. Это поведение отличается от стандарта SQL.

UPDATE t1 SET col1 = col1 + 1, col2 = col1;

 

### GROUP_CONCAT

Возвращает строку с конкатенированными не NULL значениями из группы, или NULL если нет не NULL значений. 

GROUP_CONCAT([DISTINCT] *expr* [,*expr* ...]

​             [ORDER BY {*unsigned_integer* | *col_name* | *expr*}

​                 [ASC | DESC] [,*col_name* ...]]

​             [SEPARATOR *str_val*])

mysql> SELECT student_name,

​         GROUP_CONCAT(test_score)

​       FROM student

​       GROUP BY student_name;

mysql> SELECT student_name,

​         GROUP_CONCAT(DISTINCT test_score

​                      ORDER BY test_score DESC SEPARATOR ' ')

​       FROM student

​       GROUP BY student_name;

### Операторы управления потоком

#### IF

[`IF``(``expr``1,``expr``2,``expr``3)`](https://dev.mysql.com/doc/refman/5.7/en/control-flow-functions.html#function_if)

Если expr1 имеет значение TRUE (expr1 <> 0 и expr1 <> NULL), IF () возвращает expr2. В противном случае он возвращает expr3.

#### CASE WHEN THEN

Первая форма:

CASE value

​     WHEN compare_value_1 THEN result_1

​     WHEN compare_value_2 THEN result_2

​     …

​     ELSE result 

END

Если value равно compare_value, то возвращает соответствующий результат result_1.  Если value не соответствует ни одному compare_value, возвращает результат, указанный в ELSE предложении.

Вторая форма:

CASE

​     WHEN condition_1 THEN result_1

​     WHEN condition_2 THEN result_2

​     …

​     ELSE result 

END

Возвращает результат, соответствующий истинному condition. Если все условия ложны, то возвращается result из ELSE.

##### Примеры

\1.    Подсчет количества значений за один проход 

\2. Выполнить сортировку по полю state. Если значение поля state IS NULL использовать при сортировке значение поля country:

SELECT *

FROM customers

ORDER BY (

​     CASE

​          WHEN state IS NULL THEN country

​          ELSE state

​     END);

\3. Увеличить значение поля salary на 500, 1000 или 1500 в зависимости от значения в поле department. Классически это делает в три запроса, но можно свести в один запрос:

UPDATE worker SET salary = 

​     CASE dept

​          WHEN 'Sales'     THEN salary+1000

​          WHEN 'IT'        THEN salary+500

​          WHEN 'Marketing' THEN salary+500

​          ELSE salary 

​     END;

### Подзапросы с ANY, SOME, IN, ALL

operand comparison_operator (ANY | SOME) (subquery)

comparison_operator это один из следующих операторов

= > < >= <= <> !=

**SOME** (некоторая) и **ANY** (любая) являются синонимами, SOME более понятно при использовании с <>. Результатом подзапроса является один столбец величин (как для IN). Если хотя бы для одного значения из подзапроса сравнение дает **TRUE**, то результат **TRUE**. Можно реализовать через **EXISTS**.

Пример: найти комментарии, которые написаны позже любой из рассылок:

SELECT * FROM comment WHERE time > ANY (SELECT time FROM mailing)

IN это псевдоним для  = ANY. NOT IN это не псевдоним для <>ANY, а псевдоним для <>ALL.

### Работа с NULL

Для проверки на NULL необходимо использовать операторы IS NULL и IS NOT NULL.

mysql> SELECT 1 IS NULL, 1 IS NOT NULL;

+-----------+---------------+

| 1 IS NULL | 1 IS NOT NULL |

+-----------+---------------+

|         0 |             1 |

+-----------+---------------+

Для проверки на NULL нельзя использовать операторы сравнения (типа = или <>), т.к. в результат проверки всегда тоже NULL.

mysql> SELECT 1 = NULL, 1 <> NULL, 1 < NULL, 1 > NULL;

+----------+-----------+----------+----------+

| 1 = NULL | 1 <> NULL | 1 < NULL | 1 > NULL |

+----------+-----------+----------+----------+

|     NULL |      NULL |     NULL |     NULL |

+----------+-----------+----------+----------+

При GROUP BY два NULLзначения считаются равными.

**select** a, *sum*(b) **FROM** (
     **select NULL** a, 1 b
     **UNION ALL      select NULL**, 2
 ) t
 **group by** a 

+------+--------+

| a    | sum(b) |

+------+--------+

| NULL |      3 |

+------+--------+

При выполнении ORDER BY ... ASC NULLзначения будут идти первыми, при выполнении ORDER BY ... DESC – последними.

## Типичные задачи

### Подсчет количества значений за один проход

Если требуется с помощью одного запроса подсчитать, сколько раз встречаются несколько разных значений в одном столбце.

SELECT 

​     SUM(IF(color = ‘blue’, 1, 0)) AS blue, 

​     SUM(IF(color = ‘red’, 1, 0)) AS red 

FROM items;

Или, учитывает что COUNT не считает значения NULL

SELECT 

​     COUNT(color = ‘blue’ OR NULL) AS blue, 

​     COUNT(color = ‘red’ OR NULL) AS red 

FROM items;

можно также короткий вариант и группировка:

select month, sum(amount>100000) from sales group by month;

SUM возвращается для каждой строки TRUE или FALSE. Булево значение приводится к числу 1 и 0, т.к. SUM оперирует числами. 

Аналог через WHEN THEN

SELECT 

​     SUM(CASE

​          WHEN color = 'blue' THEN 1

​          ELSE 0

​     END) AS 'blue',

​     SUM(CASE

​          WHEN color = 'red' THEN 1

​          ELSE 0

​     END) AS 'red'

FROM

​     items;

Посчитать лайки отмодерированных и нет:

SELECT 

​     SUM(CASE WHEN moder = 1 THEN likes END) AS moder_likes, 

​     SUM(CASE WHEN moder = 0 THEN likes END) AS unmodern_likes 

FROM posts

;

### Условная логика при проходе по строкам

Необходимо использовать when-then.

#### Нахождение корня, внутренних узлов и листьев

Дана таблица с двумя столбцами: id – номер узла, p_id – номер родительского узла. Варианты решения от худших к лучшим

**1 вариант.** Использование подзапроса с IN

SELECT id,

case 

when p_id is null then 'root'

when id in (select p_id from tree) then 'inner'

else 'leaf'

end as str

from tree

**2 вариант.** Использование подзапроса с NOT EXISTS

SELECT id,

case 

when p_id is null then 'root'

when not exists (select * from tree r WHERE r.p_id=f.id) then 'leaf'

else 'inner'

end as str

from tree f

**3 вариант**. Использование left join с колонкой p_id

SELECT f.id,

case 

when f.p_id is null then 'root'

when r.p_id is null then 'leaf'

else 'inner'

end as str

from tree f left join (select distinct p_id from tree) r on r.p_id=f.id

Можно вывести имя родителя через подзапрос:

SELECT

​     CASE

​          WHEN tree.p_id = '0' THEN tree.p_id

​          ELSE (SELECT parent.name FROM tree parent WHERE parent.id = tree.p_id)

​     END AS parentName

FROM tree

### Подсчитать количество строк в InnoDB

 

 

IN может использоваться для сравнения конструкторов строк:

mysql> SELECT (3,4) IN ((1,2), (3,4));

​        -> 1

Удалить все дубликаты из таблицы за исключением одной строки

1) Если вы хотите сохранить строку с наименьшим id:

DELETE n1 FROM names n1, names n2 WHERE n1.id > n2.id AND n1.name = n2.name

2) Если вы хотите сохранить строку с наибольшим id:

DELETE n1 FROM names n1, names n2 WHERE n1.id < n2.id AND n1.name = n2.name

Гораздо быстрее через временную таблицу

INSERT INTO tempTableName(cellId,attributeId,entityRowId,value)

​    SELECT DISTINCT cellId,attributeId,entityRowId,value

​    FROM tableName;

Функция CASE

1 вариант

SELECT CASE 1 WHEN 1 THEN 'one' 

WHEN 2 THEN 'two' 

ELSE 'more' END; 

-> 'one' 

2 вариант

SELECT CASE WHEN 1>0 THEN 'true' 

ELSE 'false' END; 

-> 'true' 

mysql> SELECT CASE BINARY 'B' 

WHEN 'a' THEN 1 

WHEN 'b' THEN 2 END; 

-> NULL

Оператор IF

mysql> SELECT IF(1>2,2,3); 

-> 3 

mysql> SELECT IF(1<2,'yes','no'); 

-> 'yes' 

mysql> SELECT IF(STRCMP('test','test1'),'no','yes');

 -> 'no'

Оператор IFNULL

Если expr1 не NULL, IFNULL() возвращает expr1; в противном случае expr2.

mysql> SELECT IFNULL(1,0); 

-> 1 

mysql> SELECT IFNULL(NULL,10); 

-> 10 

mysql> SELECT IFNULL(1/0,10); 

-> 10 

BINARY *str* сокращение для CAST(*str* AS BINARY).

Оператор BINARY приводит строку после него к двоичной строке. Это простой способ вынудить сравнение быть выполненным байт в байт, а не символ в символ. BINARY также заставляет конечные пробелы быть значительными.

mysql> SELECT 'a' = 'A';

– > 1

mysql> SELECT BINARY 'a' = 'A';

– > 0

mysql> SELECT 'a' = 'a ';

– > 1

mysql> SELECT BINARY 'a' = 'a ';

– > 0

Запрос, получить количество значений в таблице больше или меньше текущего значения

Можно через коррелированный подзапрос, также можно его раскрыть через JOIN

SELECT t1.num_marks, 

​    (SELECT count(t2.couple_id) 

​    FROM table_name t2 

​    WHERE t2.num_marks >= t1.num_marks ) AS num_couples 

FROM table_name t1 

GROUP BY t1.num_marks 

ORDER BY t1.num_marks DESC;

### DELETE JOIN

Примеры

Удаление строк из двух таблиц t1и t2:

DELETE t1, t2

FROM t1

​     INNER JOIN t2 ON t1.key = t2.key

WHERE condition;

Удаление строк с id=1 в таблице t1 и строки с ref=1 в таблице t2:

DELETE t1 , t2 

FROM t1

​     INNER JOIN t2 ON t2.ref = t1.id

WHERE t1.id = 1;

Удаление строки из таблицы t1, не имеющих соответствующих строк в таблице t2:

DELETE t1

FROM t1

​     LEFT JOIN t2 ON t1.key = t2.key 

WHERE t2.key IS NULL;

### UPDATE JOIN

Синтаксис UPDATE с несколькими таблицами:

UPDATE [LOW_PRIORITY] [IGNORE] table_references

​    SET assignment_list

​    [WHERE where_condition]

В table_references указываются таблицы, участвующие в соединении. Могут использовать любые типы соединений, разрешенные в операторе SELECT, например LEFT JOIN. Обновляются только те таблицы и те таблицы, которые указаны в блоке SET.

Обновление двух таблиц:

UPDATE t1, t2

SET  t1.c2 = t2.c2,

​     T2.c3 = expr

WHERE T1.c1 = T2.c1 AND condition

Можно явно написать JOIN:

UPDATE T1,T2

​     INNER JOIN T2 ON T1.C1 = T2.C1

SET  T1.C2 = T2.C2,

​     T2.C3 = expr

WHERE condition

Обновление записей, которые не имеют соответствующих в другой таблице

UPDATE  t1 

​     LEFT JOIN t2 ON t1.id = t2.id

SET t1.col1 = newvalue

WHERE t2.id IS NULL

## Команды SHOW

### Show table status

Информация о таблице 

mysql> show table status like 'fishki_pg_data' \G;

*************************** 1. row ***************************

​           Name: fishki_pg_data

​         Engine: InnoDB  //подсистема

​        Version: 10

​     Row_format: Compact //формат строки, возможно dynamic, compressed

​           Rows: 40067 //число строк

 Avg_row_length: 170  //средняя длина строки в байтах

​    Data_length: 6832128 //объем данных в байтах

Max_data_length: 0 //макс объем данных

   Index_length: 77725696 //размер индексов

​      Data_free: 18304991232

 Auto_increment: 40001

​    Create_time: 2015-11-07 18:08:55

​    Update_time: NULL

​     Check_time: NULL

​      Collation: utf8_general_ci

​       Checksum: NULL

 Create_options:

​        Comment:

1 row in set (5.92 sec)

### Show engines

Доступные движки

​                                                  

### Show databases

Cписок названий всех доступных текущему пользователю баз данных. 

SHOW DATABASES [LIKE 'pattern']

mysql> SHOW DATABASES;

+--------------------+

| Database           |

+--------------------+

| information_schema |

| fishki_utf8        |

| newfishki          |

| test               |

+--------------------+

### Show tables

Список таблиц конкретной базы данных db_name.

SHOW TABLES [{FROM | IN} db_name] [LIKE 'pattern']

mysql> SHOW TABLES;

+----------------------------------+

| Tables_in_newfishki              |

+----------------------------------+

| fishki_account                   |

| fishki_admin_message             |

+----------------------------------+

### Show columns

Список полей таблицы

SHOW [FULL] COLUMNS {FROM | IN} tbl_name [{FROM | IN} db_name] [LIKE 'pattern']

или полные алиасы этой команды (без FULL)

SHOW FIELDS FROM tbl_name

 

DESCRIBE tbl_name

 

DESC tbl_name

 

mysql> SHOW COLUMNS FROM fishki_tag;

+--------------+------------------+------+-----+---------+----------------+

| Field        | Type             | Null | Key | Default | Extra          |

+--------------+------------------+------+-----+---------+----------------+

| id           | int(11)          | NO   | PRI | NULL    | auto_increment |

| post_id      | int(11)          | NO   | MUL | 0       |                |

| gallery_id   | int(11)          | NO   |     | NULL    |                |

| category_id  | int(11)          | NO   | MUL | 0       |                |

| tag_old      | varchar(100)     | YES  | MUL | NULL    |                |

| tag_id       | int(11) unsigned | NO   | MUL | NULL    |                |

| community_id | int(11)          | NO   | MUL | 0       |                |

+--------------+------------------+------+-----+---------+----------------+

 