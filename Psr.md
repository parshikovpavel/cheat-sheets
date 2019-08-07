# PSR-5 и PSR-19. PHPDoc и теги PHPDoc

## Термины

**PHPDoc** –  это раздел документации, в котором содержится описание *Структурного элемента*. 

**Структурный элемент** – это одна из следующих конструкций:

* `class`

* `interface`

* `trait`

* `function` (включая методы в классе)

* свойство класса

* constant

* обычная переменная.

*DocBlock* должен предшествовать моменту определения *Структурного элемента*, и не должен повторяться при его использовании.

```php
/** @var int $int Это счетчик. */
$int = 0;

// здесь docblock не нужен
$int++;
```

**DocComment** - это специальный комментарий, который начинается с `/**` и заканчивается на `*/`.

*DocComment* может быть:

* однострочным:

  ```php
  /** <...> */
  ```

* многострочным (в этом случае каждая строка должна начинаться со звездочки `*`, которая должна быть выровнена со звездочкой в первой строке):

  ```php
  /**
   * <...>
   */
  ```
  

**DocBlock** – это *DocComment*, содержащий одиночный *PHPDoc*.

**Tag** – информация о *Структурном элементе* или его компоненте. Тегам посвящен отдельный *PSR-19. PHPDoc Tags*.

**FQSEN** – полностью квалифицированное имя структурного элемента (Fully Qualified Structural Element Name). *FQSEN* задано в документации в виде нотации БНФ. Наиболее часто используемые варианты *FQSEN*:

* пространство имен: `\My\Space`

* функция: `\My\Space\myFunction()`

* константа: `\My\Space\MY_CONSTANT`

* класс, интерфейс, трейт: `\My\Space\MyClass`

* метод: `\My\Space\MyClass::myMethod()`

* свойство: `\My\Space\MyClass::$my_property`

* константа класса: `\My\Space\MyClass::MY_CONSTANT`

## Правила

Описание DocBlock в виде нотации БНФ:

```
PHPDoc      = [summary] [description] [tags]
summary     = *CHAR (2*CRLF)
description = 1*(CHAR / inline-tag) 1*CRLF
tags        = *(tag 1*CRLF)
inline-tag  = "{" tag "}"
tag         = "@" tag-name [":" tag-specialization] [tag-details]
```


CHAR, насколько я понял, может включать и символ перевода строк `LF` (0x0A).

*DocBlock* уровня файлов размещается в верхней части файла исходного кода PHP: 

```php
/**
 * This is a file-level DocBlock
 */

/**
 * This is a class DocBlock
 */
class MyClass
{

}
```

Остальные типы *DocBlock* должны непосредственно предшествовать *Структурному элементу*.

### Сводка (summary)

*Summary* описывает назначение *Структурного элемента*. Объем одна, максимум две строки. 

*Summary* должна заканчиваться двумя последовательными `LF`. Если кроме *Summary* PHPDoc ничего не содержит, то `LF` не ставится.

### Описание (description)

*Summary* должно предшествовать *Description*, иначе *Description* будет считься *Summary* (сводкой). 

*Description* используется опционально, в том случае если требуется более подробно описать Summary. 

*Description* может использовать оформление текста согласно Markdown.

### Теги (Tags)

Формат отдельного тега:

```
"@" tag-name [":" tag-specialization] [tag-details]
```

Примеры ниже идентичны:

```php
/**
 * @var string This is a description.
 * @var string This is a
 *    description.
 * @var string
 *    This is a description.
 */
```

Специализация тега (tag-specialization) может использоваться для уточнения тега: 

```
@see:unit-test \Mapping\EntityTest::testGetId
```

#### Типы

Если для параметра возможно несколько разных типов, то они должны быть разделены вертикальной линией |:

```
@return int|null
```

Возможные типы:

1. `array`. Способы задания типа элементов массива:
   
   * не задавать тип элемента массива:
   
     ```
     @return array
     ```
   
   * все элементы массива имеют один тип:
   
     ```
     @return int[]
     ```
   
   * элементы массива могут иметь несколько различных типов:
   
     ```
     @return (int|string)[]
     ```
   
   * (не PSR, но поддерживается статическими анализаторами) массивы-структуры с фиксированным набором ключей, где каждый ключ имеет значение конкретного типа:
   
     ```
     @return array{scheme:string, host:string, path:string}
     ```
   
2. Класс. Способы задания:

   - *Fully Qualified Class Name* (FQCN)
   - именем класса, для классов в том же namespace

3. `bool`

4. `int`

5. `float`

6. `string`

7. `object`

8. `iterable`

9. `resource`

10. `mixed` – любой тип

11. `void` – используется для функций, которые ничего не возвращают

12. `null`

13. `callable`

14. `self` – элемент того класса, в котором записан *DocBlock* 

15. `static` – элемент того класса, в котором используется *DocBlock* с учетом наследования (аналог позднего статического связывания, LSB)

16. `$this` – текущий экземпляр класса. Чаще всего используется, как возвращаемое значения для методов, реализующих [паттерн Fluent Interface](TODO):

```php
/**
 * @return $this
 */
public function setName($name) : self
{
    $this->name = $name;
    return $this;
}
```

#### Наследование и `@inheritdoc`

Если следующие части *PHPDoc* не указаны у потомков, то потомки наследуют следующие элементы *PHPDoc* родителя :

* Сводка (*summary*)

* Описание (*description*)

* Некоторые теги (*tags*):
  * `@author`
  * `@copyright`
  * `@version`
  
  Для классов:
  
  - `@package`
  
  Для методов:
  
  - `@param`
  - `@return`
  - `@throws`
  
  Для переменных:
  
  - `@var`

Для более понятного документирования и чтобы избежать ситуации, когда случайно ненаписанный *PHPDoc* приведет к нежелательному наследованию из родительского *PHPDoc*, рекомендуется для наследования явно использовать тег `@inheritDoc`.

`@inheritDoc` – указывает, что *Структурный элемент* наследует *PHPDoc* от родительского класса.

```php
/**
 * This is a summary.
 */
class ParentClass
{
}

/**
 * @inheritDoc
 */
class ChildClass extends ParentClass
{
}
```

Можно отнаследовать только *Description* из элемента родительского класса, и дополнить его в дочернем классе.

`{@inheritDoc}` –  inline-тег, указывает, что в этом месте следует вставить *Description* родительского класса.

```php
/**
 * Child Summary
 *
 * {@inheritDoc}
 *
 * Additional Description
 */
```

#### `@api`

Тег указывается для *Структурных элементов*, которые являются частью публичного API. Эти *Структурные элементы* могут автоматически выбираться в документацию к приложению.

```php
/**
 * This method is public-API.
 *
 * @api
 */
public function getUser()
{
    <...>
}
```

#### `@author`

Тег указывает автора *Структурного элемента*.

```php
/**
 * @author [name] [<email address>]
 * 
 * @author My Name
 * @author My Name <my.name@example.com>
 */
```

#### `@copyright`

Тег используется для указания информации об авторских правах.

```php
/**
 * @copyright <description>
 * @copyright 1997-2005 The PHP Group
 */
```

#### `@deprecated`

Тег указывает, что *Структурный элемент* использовать не рекомендуется и он будет удален в будущей версии. Можно указать версию, с которой элемент считается deprecated.

```php
/**
 * @deprecated [<"Semantic Version">] [<description>]
 */
```

#### `@internal`

Тег указывает, что *Cтруктурный элемент* предназначен только для использования внутри приложения, не является частью публичного API.  Этот элемент не следует включать в документацию.

```php
/**
 * @internal [description]
 */
```

#### `@link`

Тег показывает связь между *Структурным элементом* и внешним ресурсом по ссылке URI.

```php
/**
 * @link [URI] [description]
 */
```

Тег может быть встроенным и пояснять какой-то момент в *Description*:

```php
/**
 * Summary
 *
 * Description
 * Foo ({@link http://example.com/foo}) is used to ...
 */
```

#### @method

Тег указывает «магические» методы, явно не указанные в классе и реализованные через `__call()`. Используется только со *Структурными элементами* типа `class` или `interface`.

```php
/**
 * @method [return type] [name]([type] [parameter], [...]) [description]
 * @method void setString(int $integer)
 */
class Child extends Parent {}

```

#### @package

Тег используется для логического разделения *Структурных элементов*. Тогда как `namespace` осуществляют функциональное разделение *Структурных элементов*. Поэтому значение тега `@package` может отличаться от значения `namespace`. Если значение тега `@package` совпадает со значением `namespace`, то его НЕ РЕКОМЕНДУЕТСЯ использовать, чтобы не тратить время на поддержку актуального значения тега `@package`.

```php
/**
 * @package PSR\Documentation\API
 */
```

5.9. [@param](https://vk.com/param)

 Тег [@param](https://vk.com/param) используется для документирования одного параметра функции или метода.

 

Тег [@param](https://vk.com/param) МОЖЕТ иметь многострочное описание и не нуждается в явном разграничении.

26 ноября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:44](https://vk.com/im?sel=2733257&msgid=63990)

 

@propertyТег используется для объявления , которые «волшебные» свойства поддерживаются.

 

__get() и / или __set()«магические» метод

 

@property-readИ @property-writeварианты могут быть использованы для обозначения «магические» свойства , которые могут быть прочитаны только или письменными

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [12:33](https://vk.com/im?sel=2733257&msgid=64032)

 

Тег [@return](https://vk.com/return) используется для документирования возвращаемого значения функций или методов

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [13:31](https://vk.com/im?sel=2733257&msgid=64033)

 

Тег @see указывает ссылку из связанных «Структурных элементов» на веб-сайт или другие «Структурные элементы

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [19:12](https://vk.com/im?sel=2733257&msgid=64095)

 

При определении ссылки на другие «Структурные элементы» вы можете обратиться к определенному элементу, добавив двойной двоеточие и указав имя этого элемента (также называемое «FQSEN»).

 

Тег @see ДОЛЖЕН иметь описание, чтобы предоставить дополнительную информацию о связи между элементом и его целью. К

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [19:31](https://vk.com/im?sel=2733257&msgid=64099)

 

Тег [@since](https://vk.com/since) используется для обозначения того, когда элемент был введен или изменен, используя некоторое описание «управления версиями» для этого элемента.

27 ноября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [11:35](https://vk.com/im?sel=2733257&msgid=64125)

 

5,14. [@throws](https://vk.com/throws)

 Тег [@throws](https://vk.com/throws) используется для указания того, выбрали ли «Структурные элементы» определенный тип Throwable (исключение или ошибка).

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [20:49](https://vk.com/im?sel=2733257&msgid=64266)

 

Также РЕКОМЕНДОВАНО, что этот тег встречается для каждого случая исключения, даже если он имеет тот же тип.

 

todo

28 ноября

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:33](https://vk.com/im?sel=2733257&msgid=64287)

 

Тег @todo используется для указания того, что действие, связанное с связанными «Структурными элементами», должно все еще происходить. Каж

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:43](https://vk.com/im?sel=2733257&msgid=64289)

 

Чтобы указать отношения с внешними элементами, можно использовать тег @see.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [12:00](https://vk.com/im?sel=2733257&msgid=64290)

 

@used-byтег элемента назначения. Это можно использовать для обеспечения двунаправленного опыта

[     ](https://vk.com/id2733257)



#### `@var `

Тег может использоваться для документирования следующих *Структурных элементов*:

* константы
* свойства класса
* переменные в коде

```php
/**
 * @var ["Type"] [element_name] [<description>]
 * @var int $int This is a counter
 */
```

Этот тег ДОЛЖЕН использоваться перед *Структурными блоками*, тип которых непонятен из кода.  Многие IDE используют эту информацию для автодополнения.

```php
/** @var \Sqlite3 $sqlite */
foreach ($connections as $sqlite) {}
```

Тег ДОЛЖЕН содержать имя элемента, который он документирует. Имя элемента можно не указывать только для PHPDoc перед свойством класса. 

```php
class Foo
{
  /** 
   * @var string|null Should contain a description 
   */
  protected $description = null;
}
```

Можно документировать в одном PHPDoc сразу несколько *Структурных элементов*:

```php
/**
 * @var $this yii\web\View 
 * @var $searchModel app\models\WordSearch 
 */
/* @var $dataProvider yii\data\ActiveDataProvider */
```



[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [13:47](https://vk.com/im?sel=2733257&msgid=64300)

 

т текущую «версию» любого элемента.

[     ](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [14:21](https://vk.com/im?sel=2733257&msgid=64301)

 

### Примеры

Пример с Summary, Description, Tags:

/**
 * This is a Summary.
 *
 * This is a Description. 
 * It may span multiple lines
 *
 * @param int        $parameter1 A parameter description.
 * @param \Exception $e          Another parameter description.
 *
 * @return string
 */

Пример с Summary и Tags:

/**
 * This is a Summary.
 *
 * @param int        $parameter1 A parameter description.
 * @param \Exception $e          Another parameter description.
 *
 * @return string
 */

Пример с Summary:

/**
  \* This is a Summary.
  */

Однострочный DocBlock:

/** **@var** \ArrayObject $array */
 **public** $array = **null**;