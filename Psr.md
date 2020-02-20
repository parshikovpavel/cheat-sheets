# PSR-2 и PSR-12. Руководство по стилю кода

Перечисленные ниже правила получены путем анализа кода наиболее популярных фреймворков (*Symfony, YII, Laravel*...). 

<u>Отступы в коде</u>

Один отступ в коде – это 4 пробела. Не допускается использование символа табуляции.

<u>Файлы</u>

В качестве перевода строк должен использоваться использоваться *UNIX*-перевод строк `\n`. Не допускается использование перевода строк в стиле *Windows* `\r\n` или *Mac OS* `\n\r`.

В конце РНР-файла должна быть одна пустая строка — это позволит автоматическим системам формирования РНР-кода дописывать код с новой строки.

Закрывающий тег `?>` необходимо удалять из файлов, которые содержат только *РНР*-код. Иначе, случайно забытый пробел после `>?` приведет к началу формирования тела ответа и невозможности вывода *HTTP*-заголовков.

<u>Строки в файле</u>

- в одной строке должна быть только одна инструкция


- мягкий предел длины строки – 80 символов, его не стоит превышать без крайней необходимости. 


- жесткий предел длины – 120 символов. Более длинные строки должны разбиваться на несколько коротких строк по 80 символов.


- в конце строки не должно быть пробельных символов.


- Можно добавлять пустые строки для улучшения читаемости кода

  Ключевые слова необходимо писать в строчном регистре

  ```php
  true
  false
  null
  ```

## Структура файла

Ниже перечислены блоки, составляющие структуру файла с PHP-кодом. После каждого блока внизу должна быть вставлена пустая строка. Блоки не должны содержать пустую строку.

·      Открывающий тег <?php.

·      Docblock уровня файла.

·      Вызовы declare.

·      Объявление пространства имен namespace.

·      Создание псевдонимов пространств имен, классов, интерфейсов use 

·      Создание псевдонимов функций use function

·      Создание псевдонимов констант use const.

·      Остальная часть кода в файле.

Не допускается в одном операторе use перечислять несколько импортирований через запятую. Под каждое импортирование нужно использовать один оператор use.

**<?php**  */**  \* Docblock* *уровня* *файла*  **/*  **declare**(strict_types=1);

 **namespace** Vendor\Package;

 **use** Vendor\Package\{ClassA **as** A, ClassB, ClassC **as** C};
 **use** Vendor\Package\SomeNamespace\ClassD **as** D;
 **use** Vendor\Package\AnotherNamespace\ClassE **as** E;

 **use function** Vendor\Package\{functionA, functionB, functionC};
 **use function** Another\Vendor\functionD;

 **use const** Vendor\Package\{**CONSTANT_A**, **CONSTANT_B**, **CONSTANT_C**};
 **use const** Another\Vendor\**CONSTANT_D**;

 */**  ** *Остальная* *часть* *кода*  **/*

Максимально возможная глубина пространства имен внутри пакета – 2:

**use** Vendor\Package\SomeNamespace\{
     ClassA,                                  *# Допустимо - глубина 1*     SubnamespaceOne\ClassB,                  *# Допустимо - глубина 2*     SubnamespaceOne\AnotherNamespace\ClassC  *# Не допустимо - глубина 3* };

 

## Классы

Термин «класс» относится к структурам `class`, `interface`, `trait`. 

При создании объекта класса обязательно необходимо ставить круглые скобки, даже если у конструктора отсутствуют аргументы:

```php
new Foo();
```





### Заголовок класса

Открывающая фигурная скобка размещается на следующей строке после названия класса. Перед и после открывающей скобки недопустимы пустые строки.

Закрывающая фигурная скобка размещается на следующей строке после тела класса. Перед закрывающей фигурной скобкой недопустимо пустая строка.

Ключевые слова extends и implements должны быть объявлены на одной строке с именем  класса.

**class** ClassName **extends** ParentClass **implements** \ArrayAccess, \Countable
 {
     *// constants, properties, methods* }

Если класс расширяет множество интерфейсов, то каждый интерфейс можно разместить на отдельной строке, и каждая строка предваряется отступом: 

**class** ClassName **extends** ParentClass **implements**     \ArrayAccess,
     \Countable,
     \Serializable
 {
     *// constants, properties, methods* }

### Объявление трейта

Особенности:

·      трейты начинают объявляться на следующей строке после открывающейся фигурной скобки

·      каждый трейт объявляется отдельным оператором use на отдельной строке.

·      после объявления трейтов ставится пустая строка и далее идет объявление членов класса.

**class** ClassName
 {
     **use** FirstTrait;
     **use** SecondTrait;

     **private** **$property**;
 }

Если используются операторы insteadof и as, то ставятся следующие отступы:

**class** Talker
 {
     **use** A, B, C {
         B::*smallTalk* **insteadof** A;
         A::*bigTalk* **insteadof** C;
         C::*mediumTalk* **as** FooBar;
     }
 }

 

 

 

 

### Члены класса

К членам класса относятся константы, свойства и методы.

Все члены класса должны предваряться модификатором доступа (private, public, protected). 

Не следует предварять имена членов класса символом подчеркивания _ для обозначения закрытой (private) или защищенной (protected) областей видимости.

### Объявление метода (функции)

Так же как и в случае класса, открывающая фигурная скобка размещается на следующей строке после заголовка метода (функции); закрывающая фигурная скобка размещается на следующей строке после тела метода (функции)

Между названием метода (функции) и открывающейся круглой скобкой не должно быть пробела.

В списке аргументов один пробел ставится только после каждой запятой. Пробел вначале и в конце списка аргументов не допускается.

Пример объявления метода:

```php
public function fooBarBaz($arg1, &$arg2, $arg3 = [])
{
    // method body 
}
```

Если аргументов много, то каждый аргумент можно разместить на отдельной строке, и каждая строка предваряется отступом. В этом случае закрывающая скобка и открывающая скобка должны быть размещены на одной строке и между ними ставится один пробел.

```php
public function aVeryLongMethodName(
    ClassTypeHint $arg1,
    &$arg2,
    array $arg3 = []
) {
    // method body 
}
```

 Если указывается тип возвращаемого методом значения, то

- тип возвращаемого значения должен быть на той же строке, где закрывающаяся круглая скобка

- перед двоеточием пробел не ставится

- после двоеточия ставится один пробел

```php
public function functionName(int $arg1, $arg2): string
{
    return 'foo';
}

public function anotherFunction(
    string $foo,
    string $bar,
    int $baz
): string {
    return 'foo';
}
```

Для обнуляемых (*nullable*) типов аргументов и возвращаемых значений знак вопроса `?` ставится вплотную к типу:

```php
public function functionName(?string $arg1, ?int $arg2): ?string
{
    return 'foo';
}
```

### Вызов метода (функции)

Правила вызова аналогичны объявлению метода:

- между названием метода (функции) и открывающейся круглой скобкой не должно быть пробела.

- в списке аргументов один пробел ставится только после каждой запятой. Пробел вначале и в конце списка аргументов не допускается.


Пример:

```php
Foo::bar($arg2, $arg3);
```

Также если аргументов много, то каждый аргумент можно разместить на отдельной строке:

```php
$foo->bar(
    $longArgument,
    $longerArgument,
    $muchLongerArgument
);
```

Если аргумент метода записывается в несколько строк (например, анонимная функция или массив), то остальные аргументы записываются в обычном порядке, т.е. не переносятся на отдельные строки:

```php
somefunction($foo, $bar, [
    // ... 
], $baz);

$app->get('/hello/{name}', function ($name) use ($app) {
    return 'Hello ' . $app->escape($name);
});
```

А если каждый аргумент размещен на отдельной строке, то заголовок и закрывающая скобка записываются на отдельных строках, тело записывается со смещением:

```php
$foo->bar(
    $arg1,
    function ($arg2) use ($var1) {
        // body     
    },
    $arg3
);
```

### Ключевые слова abstract, static, final

Ключевые слова abstract и final должны предшествовать модификатору доступа. 

Ключевое слово static, наоборот, должно располагаться за модификатором доступа.

**abstract class** ClassName
 {
     **protected static** $foo;

     **abstract protected function** zim();
     
     **final public static function** bar()
     {
         *// method body*     }
 }

## Управляющие структуры

Общие правила для управляющих структур:

- Должен быть один пробел после ключевого слова управляющей структуры


- Не должно быть пробела после открывающей и перед закрывающей круглыми скобками


- Открывающая фигурная скобка должна располагаться на той же строке, где и ключевое слово. При этом должен быть один пробел между закрывающей круглой скобкой и открывающей фигурной скобкой


- Тело управляющей структуры должно быть заключено в фигурные скобки. Это позволяет исключить ошибки при добавлении новых строк в тело.


- Закрывающая фигурная скобка должна находиться на следующей строке после тела


Условие в круглых скобках для всех операторов (`if`, `switch`, `while`...) можно разбивать на несколько строк по следующим правилам:

- каждая часть условия на отдельной строке с отступом (как минимум один, или больше)


- закрывающаяся круглая скобка и открывающаяся фигурная скобка размещаются на отдельной строке, между ними ставится пробел. 


- логические операторы между условиями ставятся либо в начале, либо в конце строки, но не в перемешку

```php
if (
    $expr1
    && $expr2
) {
    // if body
}

```

### if, elseif, else

Особенности написания условных операторов:

·      else и elseif находятся на той же строке, что и предшествующая закрывающая фигурная скобка

·      `elseif` следует использовать вместо `else if`, чтобы все ключевые слова выглядели как одно слово.

**if** ($expr1) {
     // if body
 } **elseif** ($expr2) {
     // elseif body } **else** {
     // else body; }

### switch, case

Особенности:

`·       `case` смещается на один отступ от уровня `switch``

·      break находится на одном уровне с телом блока case

`·       `если в непустом блоке case преднамеренно не указан break, то необходимо указать комментарий //no break`:```

**switch** ($expr) {
     **case** 0:
         **echo** **'First case, with a break'**;
         **break**;
     **case** 1:
         **echo** **'Second case, which falls through'**;
     *// no break*     **case** 2:
     **case** 3:
     **case** 4:
         **echo** **'Third case, return instead of break'**;
         **return**;
     **default**:
         **echo** **'Default case'**;
         **break**;
 }

### `while`

```php
while ($expr) {
    // structure body
}
```

###  `do-while`

```php
do {
    // structure body;
} while ($expr);
```

### `for`

```php
for ($i = 0; $i < 10; $i++) {
    // for body
}
```

Выражения в круглых скобках можно разбить на несколько строк, аналогично общим правилам для управляющих структур:

```php
for (
    $i = 0;
    $i < 10;
    $i++
) {
    // for body
}
```

### foreach

Синтаксис foreach:

**foreach** ($iterable **as** $key => $value) {
     *// foreach body* }

### try, catch, finally

Синтаксис:

**try** {
     *// try body* } **catch** (FirstThrowableType $e) {
     *// catch body* } **catch** (OtherThrowableType $e) {
     *// catch body* } **finally** {
     *// finally body* }

## Операторы

Слева и справа всех бинарных и тернарных (НО не унарных) операторов должен стоять один пробел. Для удобства чтения можно использовать несколько пробелов. К таким операторам относятся: 

·      арифметические

·      логические

·      поразрядные

·      операторы сравнения

·      присваивание

·      конкатенация строк

·       также бинарные операторы: insteadof, as.

**if** ($a === $b) {
     $foo = $bar ?? $a ?? $b;
 } **elseif** ($a > $b) {
     $variable = $foo ? **'foo'** : **'bar'**;
 }

## Анонимные функции

Особенности:

- после ключевого слова `function`, до и после ключевого слова `use` ставится пробел.


- как в методах класса, в списке аргументов и списке переменных один пробел ставится только после каждой запятой. Пробел в начале и в конце списка аргументов не допускается.

```php
$closureWithArgs = function ($arg1, $arg2) {
    // body
};

$closureWithArgsAndVars = function ($arg1, $arg2) use ($var1, $var2) {
    // body
};

$closureWithArgsVarsAndReturn = function ($arg1, $arg2) use ($var1, $var2): bool {
    // body
};

```

Как в методах класса, списки аргументов и списки переменных могут разбиваться на несколько строк, так что:

- каждый аргумент или каждая переменная на отдельной строке с однократным отступом


- закрывающая круглая скобка списка (аргументов или переменных) и открывающая фигурная скобка блока должны быть размещены на одной строке с одним пробелом между ними.


Самый длинный пример – с длинным списком аргументов и переменных:

```php
$longArgs_longVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
    // body
};

```

## Анонимные классы

Подобно анонимным функциям открывающаяся фигурная скобка ставится на одной строке вместе с заголовком.

```php
$instance = new class extends \Foo implements \HandleableInterface {
     // Class content 
};
```

Однако, если интерфейсы выписываются в отдельные строки, то открывающая скобка ставится на отдельной строке:

```php
$instance = new class extends \Foo implements
    \ArrayAccess,
    \Countable,
    \Serializable
{
     // Class content
};
```

 

 

 

 

## Автоматическая проверка стиля кода

Из всех PSR, автоматически можно проверить только соблюдение стиля кода (*PSR-1*, *PSR-2*, *PSR-12*). Остальные стандарты определяют интерфейсы и не могут быть автоматически проверены.  

Наиболее популярная утилита для автоматической проверки стиля кода стандартам *PSR-1* и *PSR-2* – [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer). 

Глобальная установка через composer:

```bash
composer global require squizlabs/php_codesniffer
```

**Проверка**

Проверка выполняется утилитой `vendor/bin/phpcs`.

<u>Указать проверяемые файлы и папки.</u>

```bash
phpcs index.php # проверка файла
phpcs src tests # проверка каталогов
```

<u>Указать проверяемый стандарт.</u> 

Узнать доступные для проверки стандарты:

```bash
$ phpcs -i
The installed coding standards are MySource, PEAR, PSR1, PSR12, PSR2, Squiz and Zend
```

Узнать проверки, входящие в стандарт:

```bash
$ phpcs --standard=PSR1 -e
The PSR1 standard contains 8 sniffs

Generic (4 sniffs)
------------------
  Generic.Files.ByteOrderMark
....
```

При запуске нужно обязательно указывать проверяемый стандарт. Возможные варианты:

- при прогоне с помощью опции `--standard`

  ```bash
  $ phpcs --standard=PSR12 index.php # Один стандарт
  $ phpcs --standard=PSR1,PSR2 index.php # Несколько стандартов
  ```

- поменять значение по умолчанию

  ```bash
  $ phpcs --config-set default_standard PSR12
  ```

<u>Указать расширения файлов для проверки</u>

```bash
$ phpcs --extensions=php,js,css,inc /path/to/code
```

<u>Исключить из проверки папки</u>

```bash
 $ phpcs --ignore=*/tests/*,*/vendor/* /path/to/code
```

Формат отчета:

```
FILE: index.php
----------------------------------------------------------------------
FOUND 7 ERRORS AND 1 WARNING AFFECTING 6 LINES
----------------------------------------------------------------------
  1 | ERROR   | [x] End of line character is invalid; expected "\n"
    |         |     but found "\r\n"
  7 | ERROR   | [ ] Content of the @author tag must be in the form
    |         |     "Display Name <username@example.com>"

```

Помеченные X ошибки могут быть автоматически исправлены, специальной утилитой:

```bash
$ phpcbf /path/to/code
Processing init.php [PHP => 7875 tokens in 960 lines]... DONE in 274ms (12 fixable violations)
​    => Fixing file: 0/12 violations remaining [made 3 passes]... DONE in 412ms
Patched 1 files
```

 

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
tag-details        = *SP (SP tag-description / tag-signature )
tag-description    = 1*(CHAR / CRLF)
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
 * @deprecated 1.0.0 No longer used by internal code and not recommended.
 */
```

#### `@internal`

Тег указывает, что *Cтруктурный элемент* предназначен только для использования внутри приложения, не является частью публичного API.  Этот элемент не следует включать в документацию.

```php
/**
 * @internal [description]
 */
```

Тег может быть встроенным, в этом случае в нем размещается примечание, которое будут отображаться только для разработчиков и не будет включаться в документацию:

```php
/**
 * {@internal [description]}
 * {@internal Developers should note that ... }
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

#### `@method`

Тег указывает «магические» методы, явно не указанные в классе и реализованные через `__call()`. Используется только со *Структурными элементами* типа `class` или `interface`.

```php
/**
 * @method [return type] [name]([type] [parameter], [...]) [description]
 * @method void setString(int $integer)
 */
class Child extends Parent {}

```

#### `@package`

Тег используется для логического разделения *Структурных элементов*. Тогда как `namespace` осуществляют функциональное разделение *Структурных элементов*. Поэтому значение тега `@package` может отличаться от значения `namespace`. Если значение тега `@package` совпадает со значением `namespace`, то его НЕ РЕКОМЕНДУЕТСЯ использовать, чтобы не тратить время на поддержку актуального значения тега `@package`.

```php
/**
 * @package [level 1]\[level 2]\[etc.]
 * @package PSR\Documentation\API
 */
```

#### `@param`

 Тег используется для документирования одного параметра функции или метода.

```php
/**
 * @param ["Type"] [name] [<description>]
 * @param mixed[] $items Array structure to count the elements of.
 */
```

Тег МОЖЕТ иметь многострочное описание и не нуждается в явных границах.

#### `@property[-x]`

Теги `@property`, `@property-read`, `@property-write` используются только для *Структурных элементов* `class` и `trait`.

Тег `@property` используется, чтобы объявить какие "магические" свойства доступны через "магические" методы `__get()` и/или `__set()`.

Тег `@property-read` МОЖЕТ быть использован вместо `@property` для "магических" свойств, которые только читаются.

Тег `@property-write` МОЖЕТ быть использован вместо `@property`  для "магических" свойств, которые только пишутся.

```php
/**
 * @property[<-read|-write>] ["Type"] [name] [<description>]
 * @property-read string $full_name
 */
```

#### @return

Тег используется только со *Структурными элементами* типа функция и метод. Для документирования возвращаемого значения.

```php
/**
 * @return <"Type"> [description]
 * @return string|null The label's text or null if none provided.
 */
```

Тег может иметь многострочное описание и не нуждается в явных границах.

 Тег может быть опущен для методов без возвращаемого значения, это аналогично `@return void`.

#### @see

Тег позволяет задать ссылку на веб-сайт или другой *Структурный элемент*.

```php
/**
 * @see [URI | "FQSEN"] [<description>]
 * @see MyClass::$items           For the property whose items are counted.
 * @see MyClass::setItems()       To set the items for this collection.
 * @see http://example.com/my/bar Documentation of Foo.
 * @see:unit-test \Mapping\EntityTest::testGetId Description
 */
```

Тег ДОЛЖЕН иметь описание, чтобы указать, как связаны Структурный элемент и ссылка. 

Тег МОЖЕТ быть дополнен специализацией в виде `@see:unit-test`.

#### @since

Тег позволяет указать, когда элемент был добавлен или изменен. Для тега РЕКОМЕНДУЕТСЯ указывать семантическую версию и МОЖЕТ быть добавлено описание.

```php
/**
 * @since 2.0.0 introduced
 */
class Foo {}

/**
 * @since 2.1.5 bar($arg1 = '', $arg2 = null) 
 *        Introduced the optional $arg2
 * @since 2.1.0 bar($arg1 = '')
 *        Introduced the optional $arg1
 * @since 2.0.0 bar()
 *        Introduced new method bar()
 */
function bar() {}
```

#### @throws

Тег позволяет указать, что *Структурный элемент* выбрасывает объект, унаследованный от `Throwable` (*exception* или *error*).

```php
/*
 * @throws ["Type"] [<description>]
 * @throws InvalidArgumentException if the provided argument is not of type `array`
 */
```

Рекомендуется писать тег для каждого исключения в теле Структурного элемента, даже если несколько исключений имеют один и тот же тип. 

####  @todo

Тег позволяет указать, что *Структурный элемент* требует доработки.

```php
/**
 * @todo [description]
 * @todo add an array parameter to count
 */
```

#### @uses, @used-by

Тег позволяет указать, что ассоциированный *Структурный элемент* использует файл или другой *Структурный элемент* в текущем проекте. Тег может указывать только на внутренние элементы (путь к *file* или *FQSEN*)

Например, можно указать, что контроллер использует файл шаблона.

```php
/**
 * @uses [file | "FQSEN"] [<description>]
 * @uses MyView.php
 */
```

Рекомендуется в *PHPDoc Структурного элемента*, который использован, добавить тег `@used-by` (этот тег не описан в *PSR*). Тем самым отношение будет документировано с обоих сторон тегами `@uses` — `@used-by`.

Для того чтобы указать связь с внешними элементами через *URL*, нужно использовать тег `@see`.

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

#### @version

Тег позволяет документировать текущую «версию» *Структурного элемента*.

```php
/**
 * @version ["Semantic Version"] [<description>]
 * @version 2.1.7 MyApp (номер версии приложения)
 * @version $Id$ (ключевое слово, которое Git заменяет на номер версии)
 */
```

### Примеры

*PHPDoc* c: *Summary*, *Description*, *Tags*:

```php
/**
 * Это Summary.
 *
 * Это Description. Он может быть в несколько строк
 * и содержать `Markdown` разметку
 *
 * @see Markdown
 *
 * @param int        $parameter1 A parameter description.
 * @param \Exception $e          Another parameter description.
 *
 * @throws InvalidArgumentException if the provided argument is not of type 'int'.
 *
 * @return string
 */
function test($parameter1, $e): string
{
   
}
```

*PHPDoc* c: *Summary* и *Tags*:

```php
/**
 * Это Summary.
 *
 * @see Markdown
 *
 * @param int        $parameter1 A parameter description.
 * @param \Exception $e          Another parameter description.
 *
 * @throws InvalidArgumentException if the provided argument is not of type 'int'.
 *
 * @return string
 */
function test($parameter1, $e): string
{
   
}
```

*PHPDoc* только с *Summary* (не приветствуется, т.к. информация о параметрах и возвращаемом значении обязательна):

```php
/**
 * Это Summary.
 */
function test($parameter1, $e): string
{
   
}
```




# PSR-6. Кэширование с пулом

Описывает общий интерфейс для кеширующих систем. 

Концепции:

- *Item* (элемент) – ключ-значение в кеше
- *Pool* (пул) – коллекция из всех закешированных *items*. 

Общий принцип:

* *Item* извлекаются из *Pool* , обрабатываются и возвращаются в него. *Item* не создаются самостоятельно *Calling Library*, а всегда извлекаются из *Pool*

 Основные термины:

* *Calling Library* (вызывающая библиотека) – код, обращающийся к кеширующей библиотеке
*  *Implementing Library* (реализующая библиотека). Библиотека ДОЛЖНА реализовывать `CacheItemInterface` и `CacheItemPoolInterface`.
* *TTL* ([Time To Live](Dictionary.md#ttl)) – время жизни *item* в кеше. Задается в виде интервала времени `\DateInterval` или в секундах `int`.
* *Expiration* – момент истечения срока жизни *item*. Задается в виде `\DateTimeInterface`. *Expiration* и *TTL* связаны друг с другом и при изменении одного, изменяется другое. *Item* МОЖЕТ "протухнуть" раньше *Expiration*, но ДОЛЖЕН "протухнуть" гарантированно после *Expiration*. Если *Expiration* и *TTL* не указаны, то элемент хранится вечно.
* *Key*. Должны поддерживаться (как минимум) ключи из символов `A-Z`, `a-z`, `0-9`, `_`, и `.` и длиной до 64 символов.
* *Hit* (попадание)  – *item* существует и не "протух". 
* *Miss* (промах) – *item* не существует или "протух". *Miss* =! *Hit*.
* *Deferred* (отложенный) – *item* может сохраняться в персистентное хранилище с задержкой. Задержку определяет сама система хранения. Система хранения может оптимизировать сохранение в персистентное хранилище, используя групповые операции. Однако при вызове `commit()` все *deffered items* должны быть гарантировано сохранены в персистентное хранилище. *Deffered items* должны возвращаться по запросу в кеш, даже если они еще не сохранены в персистентное хранилище.

*Implementing libraries* должны поддерживать кеширования всех типов данных (включая `array`, `object`, `null`). Данные должны возвращаться назад в том же виде, как они были переданы для сохранения, включая тип данных. Поэтому вместе с данными должен быть сохранен и их тип. Проще всего использовать функции `serialize()`/`unserialize()`. 

## `CacheItemInterface`

`CacheItemInterface` – интерфейс для класса закешированного *Item*.

*Calling Libraries* НЕ ДОЛЖНЫ самостоятельно инстанцировать `CacheItemInterface` объекты. Они должны создаваться только `CacheItemPoolInterface` объектом. Получить `CacheItemInterface` объект можно только через `CacheItemPoolInterface::getItem()` метод.

```php
interface CacheItemInterface
{
    /**
     * Получить ключ для текущего item
     * @return string
     */
    public function getKey();

    /**
     * Получить значение для текущего item
     * Если `isHit()` возвращает false, этот метод должен возвращать null. 
     * @return mixed
     */
    public function get();

    /**
     * Произошел Hit
     * @return bool
     */
    public function isHit();

    /**
     * Устеновить value. 
     * Тип данных $value - любой, сериализацию выполняет сама Implementing Library
     * @param mixed $value
     * @return static Для fluent interface
     */
    public function set($value);

    /**
     * Устанавливает expiration time
     * @param \DateTimeInterface|null $expiration
     * @return static Для fluent interface
     */
    public function expiresAt($expiration);

    /**
     * Устанавливает TTL (интервал времени или в секундах)
     * @param int|\DateInterval|null $time
     * @return static Для fluent interface
     */
    public function expiresAfter($time);

}
```

## `CacheItemPoolInterface`

`CacheItemPoolInterface` – интерфейс для *Pool* класса.

Основная задача – генерировать `CacheItemInterface` объекты, пустые или с закешированным значением.

```php
interface CacheItemPoolInterface
{
    /**
     * Возвращает Item по ключу $key
     * Всегда возращает CacheItemInterface объект, даже в случае промаха
     * @param string $key
     * @return CacheItemInterface
     */
    public function getItem($key);

    /**
     * Возвращает массив CacheItemInterface объектов, соответствующих массиву ключей. 
     * Для каждого ключа должен быть возвращен CacheItemInterface объект.
     * @param string[] $keys
     * @return CacheItemInterface[]
     */
    public function getItems(array $keys = array());

    /**
     * В Pool имеется Item с указанным ключом. Не использовать совместно с getItem()
     * из-за возможного race condition. Вместо этого использовать isHit()+get().
     *
     * @param string $key
     * @return bool
     */
    public function hasItem($key);

    /**
     * Очистить Pool
     *
     * @return bool Успешно или нет
     */
    public function clear();

    /**
     * Удалить Item из Pool
     *
     * @param string $key
     * @return bool Успешно или нет
     */
    public function deleteItem($key);

    /**
     * Удалить набор Items из Pool
     *
     * @param string[] $keys
     * @return bool Успешно или нет
     */
    public function deleteItems(array $keys);

    /**
     * Сохранить персистентно CacheItemInterface немедленно
     *
     * @param CacheItemInterface $item
     * @return bool Успешно или нет
     */
    public function save(CacheItemInterface $item);

    /**
     * Сохранить CacheItemInterface как Deffered Item
     *
     * @param CacheItemInterface $item
     * @return bool Успешно или нет
     */
    public function saveDeferred(CacheItemInterface $item);

    /**
     * Сохранить все Deferred Items в персистентное хранилище
     *
     * @return bool Успешно или нет
     */
    public function commit();
}
```

## Паттерны применения

```php
/**
 * Получение данных из кеша. 
 * Если данные отсутствуют, то они вычисляются и кладутся в кеш на час
 */
    $item = $pool->getItem($key);
    if (!$item->isHit()) {
        $value = compute();
        $item->set($value);
        $item->expiresAfter(3600);
        $pool->save($item);
    }
    return $item->get();
}
```

```php
/**
 * Массовое мультисохранение записей в кеш
 */
$values = [];
$items = $pool->getItems($keys);
foreach ($items as $key => $item) {
    if ($item->isHit()) {
        $value = $item->get();
    } else {
        $value = load($key);
        $item->set($value);
        $item->expiresAfter(3600);
        $pool->saveDeferred($item);
    }
    $values[$key] = $value;
}
$pool->commit(); 
```

## Преимущества подхода

* По сравнению с самостоятельным инстанцированием `CacheItemInterface` в *Calling Library* – этот подход более гибкий. Не требует включения `CacheItemInterface` конструктора в интерфейсы, что лучше унифицирует *Implementing Library*.
* По сравнению с простым подходом, когда `CacheItemInterface` отсутствует, данные сохраняются без "обертки", в "голом" виде – было бы невозможно определить разницу между *cache miss* и извлечением данного, обозначающего *cache miss* (часто `false` или `null`)

# PSR-16. Простое кеширование

Решает ту же проблему, что и *PSR-6*. Недостатки *PSR-6*:

* слишком много классов для простых случаев

В описании используются термины из PSR-6 с поправками:

* *Implementing Library* (реализующая библиотека) ДОЛЖНА реализовывать `CacheInterface`.

Основное отличие от *PSR-6* – в случае *cache miss* возвращается `null`. Т.е. сохранить значение `null` возможно, но отличить сохраненное значение `null` от `cache miss` – невозможно.

## `CacheInterface`

Экземпляр `CacheInterface` соответствует некоторому хранилищу с коллекцией *Item*. Он эквивалентен *Pool* в PSR-6. 

```php
nterface CacheInterface
{
    /**
     * Получить значение из кеша
     *
     * @param string $key     
     * @param mixed  $default Возвращается, если ключ не существует
     *
     * @return mixed Значение Item или $default в случае cache miss.
     */
    public function get($key, $default = null);

    /**
     * Сохранить данные в кеш
     *
     * @param string $key
     * @param mixed $value 
     * @param null|int|\DateInterval $ttl   Если неустановлено - хранить вечно
     * @return bool Успешно или нет
     */
    public function set($key, $value, $ttl = null);

    /**
     * Удалить Item из кеша
     *
     * @param string $key 
     * @return bool Успешно или нет
     */
    public function delete($key);

    /**
     * Очистить кэш
     *
     * @return bool Успешно или нет
     */
    public function clear();

    /**
     * Получить множество Items
     *
     * @param iterable $keys   
     * @param mixed    $default Возвращается для всех несуществующих ключей
     *
     * @return iterable Массив key => value. Для несуществующих ключей value=$default
      */
    public function getMultiple($keys, $default = null);

    /**
     * Сохранить множество Items в виде key => value 
     *
     * @param iterable $values 
     * @param null|int|\DateInterval $ttl  Если неустановлено - хранить вечно
     * @return bool Успешно или нет
     */
    public function setMultiple($values, $ttl = null);

    /**
     * Удалить множество Items
     *
     * @param iterable $keys 
     *
     * @return bool True Успешно или нет
     */
    public function deleteMultiple($keys);

    /**
     * Определяет наличие Item
     *
     * Не рекомендуется использовать совместно с get(), возможен race condition
     *
     * @param string $key 
     *
     * @return bool
     */
    public function has($key);
}
```

