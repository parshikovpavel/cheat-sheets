#### Запуск тестов

Запуск тестов выполняется с помощью утилиты phpunit. 

phpunit [options] UnitTest [UnitTest.php]

Если указано UnitTest, то PHPUnit будет искать исходный файл UnitTest.php в текущей рабочей директории и класс UnitTest с тестами в нем. 

Если указано UnitTest.php, то PHPUnit загрузит этот файл и возьмет первый класс в нем для теста. 

 

 



#### Тестовые «двойники»

Методы createMock($type) и getMockBuilder($type) возвращают объект тестового двойника для указанного типа (интерфейса или класса). 

Dummy (манекен) 

В результате вызова createMock($type) и getMockBuilder($type) по умолчанию все методы исходного класса заменяются фиктивной (dummy) реализацией, которая просто возвращает null. 

*/** *Класс* **/
\* **class** SomeClass
 {
   **public function** doSomething() {}
 }

 */** *Тест* *этого* *класса* **/
\* **class** DummyTest **extends** TestCase
 {
   **public function** testDummy()
   {
     $stub = $this->createMock(SomeClass::***class\***);
     *var_dump*($stub->doSomething()); *# NULL
\*   }
 }

Однако getMockBuilder($type) позволяет дополнительно, более тонко настроить тестовый «двойник». Ниже таблица с доступными методами для настройки.

| setMethods(**array** $methods)       | Указать явно  методы, которые будут заменены заглушками (остальные методы не заменяются) |
| ------------------------------------ | ------------------------------------------------------------ |
| setMethodsExcept(**array** $methods) | Указать явно  методы, которые не заменяются заглушками       |
| disableOriginalConstructor()         | Отключить вызов  __construct()                               |
| disableOriginalClone()               | Отключить вызов  __clone()                                   |
| disableArgumentCloning()             | Отключить клонирование  аргументов                           |
| setConstructorArgs(array $args)      | Параметры для конструктора (который по умолчанию не  заменяется заглушкой) |

Аналог createMock($type) через getMockBuilder($type). 

$stub = $this->getMockBuilder(SomeClass::***class\***)
   ->disableOriginalConstructor()
   ->disableOriginalClone()
   ->disableArgumentCloning()
   ->disallowMockingUnknownTypes()
   ->getMock();

 

Stub (заглушка) 

Для создания stub, тестовый «двойник» дополнительно настраивается, его поведение описывается с помощью текучего интерфейса (fluent interface). 

Для замены поведения методов используется will(....).Методы, объявленные как final, private и static не могут быть подменены (stubbed). 

Если не требуется заменять все методы фиктивными реализациями, нужно использовать getMockBuilder($type):

Конкретное значение. Наиболее часто требуется указать конкретное значение, которое метод будет всегда возвращать. Т.е. значения входных параметров игнорируются. Для этого используется will($this->returnValue($value)) или в более коротком виде willReturn($value)

*/\* Класс \*/
\* **class** SomeClass
 {
   **public** **function** doSomething() {}
 }

 */\* Тест этого класса \*/
\* **class** StubTest **extends** TestCase
 {
   **public** **function** testStub()
   {
     *// создать* *stub**
\*     $stub = $this->createMock(SomeClass::***class\***);

     *// указать конкретное, жестко заданное возвращаемое значение

\*     $stub->method(**'****doSomething****'**)
       ->willReturn(**'****foo****'**);

     *var**_dump*($stub->doSomething()); *#* *string**(3) "**foo**"

\*   }
 }

Один из аргументов. Вернуть аргумент в качестве значения метода will($this->returnArgument(0))

*/\* Класс \*/
\* **class** SomeClass
 {
   **public function** doSomething() {}
 }

 */\* Тест этого класса \*/
\* **class** StubTest **extends** TestCase
 {
   **public function** testStub()
   {
     *// создать stub
\*     $stub = $this->createMock(SomeClass::***class\***);

     *//* *возврат* *первого* *аргумента**

\*     $stub->method(**'doSomething'**)
       ->will($this->returnArgument(0));

     *var_dump*($stub->doSomething(5)); *# int(5)

\*   }
 }

Карты соответствия. Вернуть соответствующее значение, по заранее заданной карте соответствия will($this->returnValueMap($map)). Карта соответствия содержит: в первых стобцах – значения аргументов, в последнем столбце – возвращаемое значение.

*/\* Класс \*/
\* **class** SomeClass
 {
   **public** **function** doSomething() {}
 }

 */\* Тест этого класса \*/
\* **class** StubTest **extends** TestCase
 {
   **public** **function** testStub()
   {
     *// создать* *stub**
\*     $stub = $this->createMock(SomeClass::***class\***);

     *// Создать карту аргументов для возврата значений

\*     $map = [
       [0, **'****zero****'**],
       [1, **'****one****'**]
     ];

     *// указать конкретное, жестко заданное возвращаемое значение

\*     $stub->method(**'****doSomething****'**)
       ->will($this->returnValueMap($map));

     *var**_dump*($stub->doSomething(1)); *#* *string**(3) "**one**"

\*   }
 }

Функция обратного вызова. Можно явно указать функцию, которая будет обрабатывать входные аргументы и считать результат will($this->returnCallback($callable);

*/\* Класс \*/
\* **class** SomeClass
 {
   **public** **function** doSomething() {}
 }

 */\* Тест этого класса \*/
\* **class** StubTest **extends** TestCase
 {
   **public** **function** testStub()
   {
     *// создать* *stub**
\*     $stub = $this->createMock(SomeClass::***class\***);

     *// указать конкретное, жестко заданное возвращаемое значение

\*     $stub->method(**'****doSomething****'**)
       ->will($this->returnCallback(**function** (int $i): int {
         **return** $i + 1;
       }));

     *var**_dump*($stub->doSomething(1)); *#* *int**(2)

\*   }
 }

- Выбрасывание исключения. Варианты:
  -  `will($this->throwException(new Exception));`. 
  - `willThrowException(new Exception())`.

*/\* Класс \*/
\* **class** SomeClass
 {
   **public function** doSomething() {}
 }

 */\* Тест этого класса \*/
\* **class** StubTest **extends** TestCase
 {
   **public function** testStub()
   {
     *// создать stub
\*     $stub = $this->createMock(SomeClass::***class\***);

     *// указать конкретное, жестко заданное возвращаемое значение

\*     $stub->method(**'doSomething'**)
       ->will($this->throwException(**new** Exception));

     *var_dump*($stub->doSomething()); *# Fatal error: Uncaught Exception

\*   }
 }

Mock («подставной объект»). 

Для настройки ожиданий используются методы expects() и with(). 

expects() позволяет указать мощность ожидания –сколько раз метод должен быть вызван.

with() позволяет указать, какие условия накладываются на переданные методу параметры. Если на параметр нужно наложить несколько условий, применяется метод withConsecutive().

В примере ниже указано, что метод SomeClass::func() должен быть вызват один раз с параметром 'param':

**class** SomeClass
 {
   **public function** func() {}
 }

 */\* Тест этого класса \*/
\* **class** StubTest **extends** TestCase
 {
   **public function** testStub()
   {
     *# Создать mock
\*     $stub = $this->createMock(SomeClass::***class\***);

     *# Настроить ожидание

\*     $stub->expects($this->once())
       ->method(**'func'**)
       ->with($this->equalTo(**'param'**));

     *# Сам тест

\*     $stub->func(**'param'**);
   }
 }

Для более сложной проверки параметров нужно использовать ограничение callback():

*/\* Класс \*/
\* **class** SomeClass
 {
   **public function** func() {}
 }

 */\* Тест этого класса \*/
\* **class** StubTest **extends** TestCase
 {
   **public function** testStub()
   {
     *# Создать mock
\*     $stub = $this->createMock(SomeClass::***class\***);

     *# Настроить ожидание

\*     $stub->expects($this->once())
       ->method(**'func'**)
       ->with($this->callback(**function**($param) {
         **return** $param **instanceof** SomeClass;
       }));

     *# Сам тест

\*     $stub->func($stub);
   }
 }

В методе expects() могут использовать такие сопоставления для мощности ожидания:

| any()               | Любое  количество                                            |
| ------------------- | ------------------------------------------------------------ |
| never()             | Ни одного раза                                               |
| atLeastOnce()       | Как минимум  один                                            |
| once()              | Один                                                         |
| exactly(int $count) | Точное  количество раз                                       |
| at(int $index)      | Отличается от  остальных и не задает количество вызовов. Позволяет указать ожидаемые  параметры для вызова под номером $index. Т.е. все вызовы различаются по  номерам. |

Некоторые возможные ограничения (https://phpunit.readthedocs.io/ru/latest/assertions.html#appendixes-assertions-assertthat-tables-constraints):

| equalTo($value)        | Равно значению           |
| ---------------------- | ------------------------ |
| fileExists()           | Файл существует          |
| contains(mixed $value) | Массив содержит  элемент |
| ...                    |                          |

#### Файловая система

vfsStream (virtual file system)— обёртка потока для имитации реальной файловой системы.

Пример метода, для которого актуально использование виртуальной файловой системы:

**class** Example
 {
   **public function** createDirectory($directory)
   {
     **if**(!*is_dir*($directory)) {
       *mkdir*($directory, 0700, **true**);
     }
   }
 }

С помощью виртуальной файловой системы мы можем протестировать метод createDirectory(), не используя реальную файловую систему. 

Пример теста:

**class** ExampleTest **extends** TestCase
 {
   **public function** setUp(): void
   {
     vfsStreamWrapper::*register*();
     vfsStreamWrapper::*setRoot*(**new** vfsStreamDirectory(**'exampleDir'**));
   }

   **public function** testDirectoryIsCreated()
   {
     $example = **new** Example();
     $this->assertFalse(vfsStreamWrapper::*getRoot*()->hasChild(**'exampleDir'**));

     $example->createDirectory(vfsStream::*url*(**'exampleDir'**));
     $this->assertTrue(vfsStreamWrapper::*getRoot*()->hasChild(**'exampleDir'**));

   }
 }

# Установка

```bash
composer require --dev phpunit/phpunit ^8
```

где `8` – последняя мажорная версия (будут браться версии до следующей мажорной, [здесь](Composer.md#теги))

```json
{
    "require-dev": {
        "phpunit/phpunit": "^8"
    }
}
```

# Написание тестов

```php
use PHPUnit\Framework\TestCase;

final class EmailTest extends TestCase
{
    public function testCanBeCreatedFromValidEmailAddress(): void
    {
        /* ... */
    }
```

# Запуск тестов

Запуск всех тестов из всех файлов `*Test.php` в папке `tests`:

```bash
./vendor/bin/phpunit tests
```

Запуск всех тестов из одного файла `tests/EmailTest.php`:

```bash
./vendor/bin/phpunit tests/EmailTest
```

В документации еще написано, что  с помощью ключа `--bootstrap` нужно подключать автозагрузчик *Composer*.  Но вроде и без этого работает.

```bash
./vendor/bin/phpunit --bootstrap vendor/autoload.php tests 
```

## TestDox

Можно сделать более наглядным вывод результатов теста следующим образом.

* Написать в CamelCase стиле в названии функции поведение, которая она проверяет

  ```php
  public function testReturnsCorrectListOfComments() { /* ... */ }
  ```

* При запуске тестов использовать ключ `--testdox`

  ```bash
  ./vendor/bin/phpunit tests --testdox
  ```

В результате будут выводится список поведений, которые проверялись, и результаты проверки:

```
Email
 ✔ Can be created from valid email address
 ✘ Returns correct list of comments
   │
   │ Failed asserting that two arrays are equal.
...
```







##### 19 августа 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [11:43](https://vk.com/im?sel=2733257&msgid=81709)

- 

  [https://phpunit.readthedocs.io/ru/latest/installation..](https://vk.com/away.php?to=https%3A%2F%2Fphpunit.readthedocs.io%2Fru%2Flatest%2Finstallation.html&cc_key=)

  [**1. Установка PHPUnit — Руководство PHPUnit..**phpunit.readthedocs.io](https://vk.com/away.php?to=https%3A%2F%2Fphpunit.readthedocs.io%2Fru%2Flatest%2Finstallation.html)

  

  

- 

  composer require —dev phpunit/phpunit ^latest

  

  

- 

  Тесты для класса Class содержатся в классе ClassTest.

  ClassTest наследуется (чаще всего) от PHPUnit\Framework\TestCase.

  Тесты — общедоступные методы с именами test*

  

  

- 

  Внутри тестовых методов для проверки того, соответствует ли фактическое значение ожидаемому используются методы-утверждения

  

  

- 

  Пример 2.1 Тестирование операций с массивами с использованием PHPUnit

  <?php use PHPUnit\Framework\TestCase; class StackTest extends TestCase { public function testPushAndPop() { $stack = []; $this->assertSame(0, count($stack)); array_push($stack, 'foo'); $this->assertSame('foo', $stack[count($stack)-1]); $this->assertSame(1, count($stack));

  

  

##### 20 августа 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:34](https://vk.com/im?sel=2733257&msgid=81755)

- 

  Мартин Фаулер (Martin Fowler):

  Всякий раз, когда возникает соблазн что-то распечатать, используя print, или написать отладочное выражение, напишите тест вместо этого.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:40](https://vk.com/im?sel=2733257&msgid=81756)

- 

  PHPUnit поддерживает объявление явных зависимостей между тестовыми методами.

  

  

- 

  о они позволяют возвращать экземпляр (данные) фикстуры теста, созданные поставщиком (producer) для передачи его зависимым потребителям (consumers).

  

  

- 

  аннотацию [@depends](https://vk.com/depends) д

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:58](https://vk.com/im?sel=2733257&msgid=81759)

- 

  первый тест, testEmpty(), создаёт новый массив и утверждает, что он пуст. Затем тест возвращает фикстуру в качестве результата. Второй тест, testPush(), зависит от testEmpty()

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [20:41](https://vk.com/im?sel=2733257&msgid=81815)

- 

  у PHPUnit пропускает выполнение теста, когда зависимый тест (тест с зависимостью) провалился (не прошёл)

  

  

- 

  Тест, содержащий более одной аннотации [@depends](https://vk.com/depends), получит фикстуру от первого поставщика в качестве первого аргумента, фикстуру от второго поставщика вторым аргументом и т.д.

  

  

##### 21 августа 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [8:33](https://vk.com/im?sel=2733257&msgid=81817)

- 

  аргументы могут быть предоставлены одним или несколькими методами провайдеров данных (data provider) (см. additionProvider() в Пример 2.5). Метод, который будет использован в качестве провайдера данных, обозначается с помощью аннотации @dataProvider.

  Метод провайдера данных должен быть объявлен как public и возвращать либо массив массивов, либо объект, реализующий интерфейс Iterator и возвращать массив при каждой итераци

  

  

- 

  я тестовый метод с элементами массива в качестве его аргументов.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [11:33](https://vk.com/im?sel=2733257&msgid=81820)

- 

  х полезно указывать для каждого из них строковый ключ, вместо использования числового ключа по умолчанию. Вывод станет более подробным, так как он будет содержать имя набора данных, не прошедший тест.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [11:39](https://vk.com/im?sel=2733257&msgid=81821)

- 

  х полезно указывать для каждого из них строковый ключ, вместо использования числового ключа по умолчанию. Вывод станет более подробным, так как он будет содержать имя набора данных, не прошедший тест.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [11:50](https://vk.com/im?sel=2733257&msgid=81822)

- 

  public function additionProvider() { return [ 'adding zeros' => [0, 0, 0], 'zero plus one' => [0, 1, 1], 'one plus zero' => [1, 0, 1], 'one plus one' => [1, 1, 3] ]; }

  

  

- 

  Пример 2.7 Использование провайдера данных, который возвращает объект Iterator

  

  

- 

  public function additionProvider() { return new CsvFileIterator('data.csv'); }

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [21:30](https://vk.com/im?sel=2733257&msgid=81827)

- 

  Тестирование исключений

  Пример 2.11 показывает, как использовать метод expectException()для проверки того, было ли выброшено исключение в тестируемом коде.

  

  

- 

  В дополнение к методу expectException() существуют методыexpectExceptionCode(),expectExceptionMessage() иexpectExceptionMessageRegExp(), чтобы установить ожидания для исключений, вызванных тестируемым кодом.

  

  

- 

  Кроме того, вы можете использовать аннотации @expectedException,@expectedExceptionCode,@expectedExceptionMessage и@expectedExceptionMessageRegExp, чтобы установить ожидания для исключений, вызванных тестируемым кодом.

  

  

##### 22 августа 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:39](https://vk.com/im?sel=2733257&msgid=81830)

- 

  Пример 2.13 Ожидание ошибки PHP в тесте, используя @expectedException

  

  

- 

  /** * @expectedException PHPUnit\Framework\Error\Error */

  

  

- 

  Классы PHPUnit\Framework\Error\NoticePHPUnit\Framework\Error\Warningпредставляют уведомления и предупреждения PHP соответственно

  

  

- 

  проверка исключения на соответствие классу Exception с помощью @expectedException или expectException() больше не разрешена.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [9:48](https://vk.com/im?sel=2733257&msgid=81834)

- 

  ь метод expectOutputString() для установки ожидаемого вывода. Если этот ожидаемый вывод не будет сгенерирован, тест будет считаться проваленным.

  

  

- 

  void expectOutputRegex(string $regularExpression)Проверить, что вывод соответствует регулярному выражению $regularExpression.void expectOutputString(string $expectedString)Проверить, что вывод соответствует строке $expectedString.bool setOutputCallback(callable $callback)Устанавливает функцию обратного вызова, используемую, например, для нормализации фактического вывода.string getActualOutput()

  

  

##### 23 августа 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [1:04](https://vk.com/im?sel=2733257&msgid=81900)

- 

  Исполнитель тестов командной строки PHPUnit можно запустить с помощью команды phpunit.

  

  

- 

  $ phpunit ArrayTest

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [1:37](https://vk.com/im?sel=2733257&msgid=81902)

- 

  phpunit UnitTest

  Запускает тесты, представленные в классе UnitTest

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [3:18](https://vk.com/im?sel=2733257&msgid=81903)

- 

  --coverage-html

  Генерирует отчёт о покрытии кода тестами в формате HTML

  

  

- 

  --testdox-html и —testdox-text

  Генерирует agile-документацию в HTML или текстовом формате для запущенных тестов

  

  

##### 25 августа 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [0:39](https://vk.com/im?sel=2733257&msgid=82018)

- 

  --configuration, -c

  Прочитать конфигурацию из XML-файла. С

  

  

- 

  Если файл phpunit.xml илиphpunit.xml.dist (в таком порядке) существует в текущей рабочей директории, а опция —configurationне используется, конфигурация будет автоматически прочитана из этого файла.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [0:40](https://vk.com/im?sel=2733257&msgid=82020)

- 

  TestDox

  Функциональность TestDox PHPUnit просматривает тестовый класс и все названия его тестовых методов, и преобразует их из имён PHP в стиле написания CamelCase (или snake_case) в предложения:testBalanceIsInitiallyZero() (или test_balance_is_initially_zero()) становится «Balance is initially zero». Если есть несколько тестовых методов, названия которых отличаются только одной или более цифрой на конце, напримерtestBalanceCannotBecomeNegative() иtestBalanceCannotBecomeNegative2(), предложение «Balance cannot become negative» появится только один раз, при условии, что все эти тесты прошли успешно.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [12:25](https://vk.com/im?sel=2733257&msgid=82023)

- 

  Одной из наиболее трудозатратных частей при написании тестов является написание кода для настройки тестового окружения в известное состояние, а затем возврат его в исходное состояние, когда тест будет завершён. Это известное состояние называетсяфикстурой теста.

  В разделе Тестирование операций с массивами с использованием PHPUnitфикстурой был простой массив, который хранится в переменной $stack. Однако, в большинстве случаев, фикстура будет более сложной, чем простой массив, и количество кода, необходимое для её настройки, будет соответственно расти. Фактическое содержание теста потеряется в шуме настройки фикстуры. Эта проблема становится хуже, когда вы пишите несколько тестов с похожими фикстурами. Б

  

  

##### 26 августа 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [20:22](https://vk.com/im?sel=2733257&msgid=82067)

- 

  PHPUnit поддерживает общий код установки. Перед выполнением тестового метода будет вызван шаблонный метод setUp(): void.setUp(): void — это место, где вы создаёте тестируемые объекты. После того, как тестовый метод выполнится, вне зависимости успешно или нет, вызывается другой шаблонный метод с названием tearDown(): void. tearDown(): void - это место, где вы очищаете протестированные объекты.

  

  

- 

  Фикс туры разделяются

  

  

- 

  Между методами

  

  

- 

  мы использовали отношения продюсер-потребитель между тестами для совместного использования фикстур.

  

  

- 

  не сама фикстура повторно использовалась, а код, который её создаёт.

  

  

- 

  Пример 4.1 Использование setUp(): void для создания фикстуры

  

  

##### 27 августа 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:44](https://vk.com/im?sel=2733257&msgid=82088)

- 

  Шаблонные методы setUp(): void и tearDown(): void вызываются по одному разу при каждом выполнении тестового метода (и для нового экземпляра) тестового класса.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:58](https://vk.com/im?sel=2733257&msgid=82089)

- 

  орошим примером фикстуры для совместного использования между тестами может быть соединение с базой данных: вы подключаетесь к базе данных только один раз и затем повторно используете это соединение к базе данных вместо создания нового подключения для каждого теста. Это позволяет сделать ваши тесты быстрее.

  Пример 4.3 использует шаблонные методы setUpBeforeClass(): void: void иtearDownAfterClass(): void: void для подключения к базе данных до выполнения первого теста в тестовом классе и закрытие соединения с базой данных после запуска последнего теста, соответственно.

  Пример 4.3 Совместное использование фикстур тестами в тестовом наборе

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [11:30](https://vk.com/im?sel=2733257&msgid=82090)

- 

  Используйте Di и не используйте глобальное состояние

  

  

- 

  Если глобальное состояние используется то

  

  

- 

  До версии 6, PHPUnit по умолчанию запускал тесты таким образом, что изменения в глобальных и суперглобальных переменных ($GLOBALS,$_ENV, $_POST, $_GET, $_COOKIE,$_SERVER, $_FILES, $_REQUEST) не влияли на другие тесты.

  Начиная с версии 6, PHPUnit больше не делает операции резервного копирования и восстановления глобальных и суперглобальных переменных по умолчанию.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [20:40](https://vk.com/im?sel=2733257&msgid=82098)

- 

  самый простой способ составить набор тестов — это держать все исходные файлы тестов в тестовом каталоге. PHPUnit может автоматически обнаруживать и запускать тесты путём рекурсивного обхода тестового каталога.

  

  

- 

  классы тестов в каталоге tests отражают структуру пакета и классов тестируемой системы в каталоге src:

  

  

- 

  Для запуска всех тестов библиотеки нам просто нужно указать исполнителю тестов командной строки PHPUnit каталог с тестами:

  

  

##### 30 августа 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [8:35](https://vk.com/im?sel=2733257&msgid=82148)

- 

  XML-файл конфигурации PHPUnit

  

  

- 

  минимальной настройкой, который добавит все классы *Test, находящиеся в файлах*Test.php, после рекурсивного обхода каталога tests.

  Пример 5.1 Составление набора тестов, используя конфигурацию XML

  

  

- 

  задать порядок выполнения тестов, используя конфигурационный XML-файл.

  

  

- 

  Если phpunit.xml или phpunit.xml.dist(в этом порядке) существует в текущем рабочем каталоге, а опция —configuration не используется, то конфигурация будет автоматически считана из этого файла.

  Порядок выполнения тестов можно сделать явным:

  Пример 5.2 Составление набора тестов, используя конфигурацию XML

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [12:38](https://vk.com/im?sel=2733257&msgid=82152)

- 

  Тесты должны явно иметь аннотацию либо [@small](https://vk.com/small), [@medium](https://vk.com/medium) или [@large](https://vk.com/large), чтобы сработало ограничение выполнения теста по времени.

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [12:48](https://vk.com/im?sel=2733257&msgid=82153)

- 

  PHPUnit\Framework\IncompleteTestError — стандартная реализация

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [12:56](https://vk.com/im?sel=2733257&msgid=82154)

- 

  PHPUnit\Framework\IncompleteTestError — стандартная реализация

  

  

- 

  неполный или в данный момент ещё не реализован.

  

  

- 

  , Incomplete: 1.

  

  

- 

  ). Вызывая удобный метод markTestIncomplete()

  

  

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [21:22](https://vk.com/im?sel=2733257&msgid=82195)

- 

  Неполный тест обозначается I

  

  

##### 1 сентября 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:12](https://vk.com/im?sel=2733257&msgid=82241)

- 

  Не все тесты могут выполняться в любом окружении

  

  

- 

  я, тесты для драйвера MySQL могут выполняться только в том случае, если доступен сервер MySQL

  

  

- 

  мы проверяем, доступно ли расширение MySQLi, и используем метод markTestSkipped() для пропуска этого теста в противном случае.

  Пример 7.2 Пропуск теста

  <?php

  

  

- 

  Пропущенный тест обозначается S

  

  

- 

  Skipped

# Тестирование базы данных

Расширение `DbUnit` значительно упрощает настройку базы данных для целей тестирования и позволяет проверять содержимое базы данных после выполнения операций.

Подключение `DbUnit`:

```bash
composer require --dev phpunit/dbunit
```

База данных по существу является глобальной переменной. Два теста в одном `TestCase` могут работать с одной и той же базой данных, и, возможно, повторно использовать эти данные несколько раз. Неудачи в одном тесте могут легко повлиять на результат последующих тестов, тем самым затрудняя процесс тестирования. 

Тестирование базы данных требует выполнения следующих этапов:

1. Очистка базы данных. PHPUnit выполняет операцию `TRUNCATE` для всех таблиц, чтобы вернуть их в пустое состояние.
2. Настройка фикстур. PHPUnit вызывает функцию `getDataSet()` и получает первоначальное содержимое базы данных. Затем это содержимое вставляется в соответствующие таблицы. 
3. Испытание системы под тестом.
4. Проверка утверждений для базы данных. Состоит из 3 шагов:
   1. Получить фактические данные, выбрать данные из таблицы в БД в виде `IDataSet` или `ITable`  
   2. Получить ожидаемые данные, загрузить в виде `IDataSet` или `ITable` 
   3. Проверить, что объекты совпадают
5. Возврат к исходному состоянию (`teardown`).





Для тестирования базы данных необходимо использовать трейт `TestCaseTrait` из `DbUnit`:

```php
namespace PHPUnit\DbUnit;

trait TestCaseTrait
{
	/* ... */
    
    /**
     * Returns the test database connection.
     *
     * @return Connection
     */
    abstract protected function getConnection();
    
    /**
     * Returns the test dataset.
     *
     * @return IDataSet
     */
    abstract protected function getDataSet();
}
```

## `getConnection()`

`getConnection()` – возвращает соединение с базой данных. 

Чаще всего в этом методе создается соединение с базой данных с использованием `PDO`. А затем это соединение передается в метод `createDefaultDBConnection`:

```php
trait TestCaseTrait
{
	/**
     * Creates a new DefaultDatabaseConnection using the given PDO connection
     * and database schema name.
     *
     * @param PDO    $connection
     * @param string $schema
     *
     * @return DefaultConnection
     */
    protected function createDefaultDBConnection(PDO $connection, $schema = '')
    {
        return new DefaultConnection($connection, $schema);
    }
}
```

Этот метод оборачивает экземпляр `PDO` и имя базы данных `$schema` в объект интерфейса `PHPUnit\DbUnit\Connection`.

Приложение использует другое соединение (и не обязательно `PDO`), это соединение используется только для настройки фикстуры.

Метод `getConnection()` лучше всего вынести в `abstract` класс, от которого потом можно будет наследоваться. Это дает преимущества:

- можно использовать один `getConnection()` в разных `TestCase`
- в `getConnection()` можно сохранить соединение в *static property* и не выполнять *reconnect* для каждого  `TestCase`.

```php
<?php
use PHPUnit\Framework\TestCase;
use PHPUnit\DbUnit\TestCaseTrait;

abstract class DatabaseTestCase extends TestCase
{
    use TestCaseTrait;

    // сохранить соединения для других Test case's
    static private $pdo = null;

    // сохранить экземпляр PHPUnit\DbUnit\Database\Connection для других Test case's
    private $conn = null;

    final public function getConnection()
    {
        if ($this->conn === null) {
            if (self::$pdo === null) {
                self::$pdo = new PDO( $GLOBALS['DB_DSN'], $GLOBALS['DB_USER'], $GLOBALS['DB_PASSWD'] );
            }
            $this->conn = $this->createDefaultDBConnection(self::$pdo, $GLOBALS['DB_DBNAME']);
        }

        return $this->conn;
    }
}
```

Переменные `$GLOBALS['DB_DSN']`, `$GLOBALS['DB_USER']`, ... необходимо задавать через  [XML-конфигурацию](). Конфигурацию необходимо разместить в файле `phpunit.xml`, конфигурация из этого файла загружается по умолчанию. Файл `phpunit.xml` необходимо разместить текущей рабочей директории или папке, из которой запускаются тесты (`/tests`).

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<phpunit>
    <php>
        <var name="DB_DSN" value="mysql:dbname=myguestbook;host=localhost" />
        <var name="DB_USER" value="user" />
        <var name="DB_PASSWD" value="passwd" />
        <var name="DB_DBNAME" value="myguestbook" />
    </php>
</phpunit>
```

## `getDataSet()`

Метод `getDataSet()` должен вернуть состояние базы данных перед выполнением (каждого? ) теста (`TestCase`?) в виде объекта `PHPUnit\DbUnit\DataSet\IDataSet`. 

Метод `getDataSet()` вызывается только один раз во время `setUp(): void` для извлечения набора данных фикстуры и вставки его в базу данных. 

## Схема базы данных

Схема база данных (таблицы, триггеры...) должна быть создана до запуска теста. 

Этого можно достичь:

1. Создать вручную схему базы данных до запуска тестов (MySQL Workbench)
2. Автоматически создавать схему базу данных в отдельном скрипте. Для этого необходимо загружать этот скрипт через опцию `bootstrap` (?командной строки) и *configuration file* (там тоже есть опция `bootstrap`).





## DataSet и DataTable

В `DbUnit` введены два ключевых интерфейса:

- `IDataSet` – абстракция вокруг набора таблиц (базы данных)
- `ITable` – абстракция вокруг одной таблицы.

Эти интерфейсы используются, чтобы представить содержимое базы данных в виде объектов в коде.  

Над этими объектами выполняются следующие операции:

- `IDataSet` используется для настройки *fixture*. `IDataSet` задает первоначальное содержимое базы данных, должен быть возвращен в методе `getDataSet()`
- `IDataSet` и `ITable` используются в тесте для извлечения содержимого таблицы (набора таблиц) из БД в виде объекта.
- `IDataSet` и `ITable` используются в тесте для получения ожидаемых данных в тестах. При проверки утверждений два объекта сравниваются.

### DataTable

Структура:

```php
interface ITable
{
    /**
     * Returns the number of rows in this table.
     *
     * @return int
     */
    public function getRowCount();
    
    /* ... */
}
```

### DataSet

Структура:

```php
interface IDataSet extends IteratorAggregate
{
     /**
     * Returns a table object for the given table.
     *
     * @param string $tableName
     *
     * @return ITable
     */
    public function getTable($tableName);
    
    /* ... */
}
```



#### Dataset на основе файлов

*Dataset* на основе файлов используется в основном в 1 и 3 случаях.

##### `FlatXmlDataSet`

Очень простой (*flat*) XML-формат. Все таблицы определяются на одном уровне. Тег внутри корневого узла `<dataset>` представляет ровно одну строку в базе данных. Имена тегов соответствуют таблице, куда будут добавляться строки (записи), а атрибуты тега представляют столбцы записи. 

Пример для таблицы

```mysql
CREATE TABLE guestbook (
	id,
    content,
    user,
    created
)
```

```xml
<?xml version="1.0" ?>
<dataset>
    <guestbook id="1" content="Hello buddy!" user="joe" created="2010-04-24 17:15:23" />
    <guestbook id="2" content="I like it!" user="nancy" created="2010-04-26 12:14:20" />
</dataset>
```

Недостаток:

- сложности с описание значения `NULL` в виде строки (подробнее в документации)

Создать `FlatXmlDataSet` по заданному файлу `$xmlFile`.

```php
$this->createFlatXMLDataSet($xmlFile)
```

##### `XmlDataSet`

Более многословный формат XML, которые не имеет проблем с описание `NULL`

Внутри корневого узла `<dataset>` можно указать теги `<table>`, `<column>`, `<row>`, `<value>` и `<null>` />. В итоге, каждая таблица – это отдельный узел `<table>`.

```xml
<?xml version="1.0" ?>
<dataset>
    <table name="guestbook">
        <column>id</column>
        <column>content</column>
        <column>user</column>
        <column>created</column>
        <row>
            <value>1</value>
            <value>Привет, дружище!</value>
            <value>joe</value>
            <value>2010-04-24 17:15:23</value>
        </row>
        <row>
            <value>2</value>
            <value>Мне нравится это!</value>
            <null />
            <value>2010-04-26 12:14:20</value>
        </row>
    </table>
</dataset>
```

Создать `XmlDataSet` по заданному файлу `$xmlFile`.

```php
$this->createXMLDataSet($xmlFile)
```

##### `MysqlXmlDataSet`

Файлы в этом формате могут быть сгенерированы с помощью утилиты `mysqldump`.

```bash
$ mysqldump --xml -t -u [username] --password=[password] [database] > /path/to/file.xml
```

Создать `MysqlXmlDataSet` по заданному файлу `$xmlFile`.

```php
$this->createMySQLXMLDataSet($xmlFile)
```

##### `YamlDataSet`

Данные в формате YAML:

```yaml
guestbook:
  -
    id: 1
    content: "Привет, дружище!"
    user: "joe"
    created: 2010-04-24 17:15:23
  -
    id: 2
    content: "Мне нравится это!"
    user:
    created: 2010-04-26 12:14:20
```

Создать `YamlDataSet` по заданному файлу `$yamlFile`.

```php
new YamlDataSet($yamlFile);
```

##### `CsvDataSet`

Каждая таблица записывается в отдельный CSV-файл.

```csv
id,content,user,created
1,"Привет, дружище!","joe","2010-04-24 17:15:23"
2,"Мне нравится это!","nancy","2010-04-26 12:14:20"
```

Создать `CsvDataSet` и добавить в него таблицу `guestbook` с данными в файле `$csvFile`.

```php
$dataSet = new CsvDataSet();
$dataSet->addTable('guestbook', $csvFile);
```

##### `ArrayDataSet`

`DataSet` на основе массива PHP.

Формат массива:

```php
$array = [
                'guestbook' => [
                    [
                        'id' => 1,
                        'content' => 'Привет, дружище!',
                        'user' => 'joe',
                        'created' => '2010-04-24 17:15:23'
                    ],
                    [
                        'id' => 2,
                        'content' => 'Мне нравится это!',
                        'user' => null,
                        'created' => '2010-04-26 12:14:20'
                    ],
                ];
```

Ключи 1-го уровня –  таблицы, ключи 2-го уровня – столбцы.

Создание `ArrayDataSet` из массива `$array`:

```php
$dataset = new ArrayDataSet($array)
```

#### DataSet на основе запросов

Используются в основном во 2 случае (для извлечения содержимого таблицы (набора таблиц) из БД в виде объекта.)

##### `QueryDataSet`

Создание `QueryDataSet` по запросу к таблице:

```php
$ds = new PHPUnit\DbUnit\DataSet\QueryDataSet($this->getConnection());
$ds->addTable('guestbook', 'SELECT id, content FROM guestbook ORDER BY created DESC');
```

Создание `QueryDataSet` с полным содержимым таблицы:

```php
$ds = new PHPUnit\DbUnit\DataSet\QueryDataSet($this->getConnection());
$ds->addTable('guestbook');
```

это аналогично следующему определению по запросу:

```php
$ds = new PHPUnit\DbUnit\DataSet\QueryDataSet($this->getConnection());
$ds->addTable('guestbook', 'SELECT * FROM guestbook');
```

##### `FilteredDataSet`

Создание `FilteredDataSet` по всем таблицам со всем их содержимым из базы данных, которая указана в `getConnection()`:

```php
$dataSet = $this->getConnection()->createDataSet();
```

Создание `FilteredDataSet` по некоторым таблицам из базы данных:

```php
$tableNames = ['guestbook'];
$dataSet = $this->getConnection()->createDataSet($tableNames);
```

## `Connection`

Интерфейс `Connection` возвращается из метода `getConnection()`:

```php
interface Connection
{
    /**
     * Создать `FilteredDataSet`. Использование описано в (1)[#filtereddataset]
     *
     * @param array $tableNames
     *
     * @return IDataSet
     */
    public function createDataSet(array $tableNames = null);

    /**
     * Создать экземпляр `class QueryTable implements ITable`. Использование
     * описано в (1)[#утверждение-состояния-таблицы], для утверждения содержания таблицы
     *
     * @param string $resultName
     * @param string $sql
     *
     * @return ITable
     */
    public function createQueryTable($resultName, $sql);

    /**
     * Следует использовать для утверждения количества строк в таблице. 
     * Использование описано в (1)[#утверждение-количества-строк-в-таблице]
     *
     * @param string $tableName
     * @param string $whereClause
     * @param int
     */
    public function getRowCount($tableName, $whereClause = null);
    
    /* ... */
}
```

## Утверждения

### Утверждение количество строк таблицы

Проверить, что таблица содержит определённое количество строк.

```php
class ObserverTest extends DatabaseTestCase
{
	public function getDataSet()
    {
        return new ArrayDataSet([
                'log' => []
        ]);
    }
	
	public function testCount()
	{
        $this->assertEquals(0, $this->getConnection()->getRowCount('log'), "Pre-Condition");
	    /* Inserting 1 row */
        $this->assertEquals(1, $this->getConnection()->getRowCount('log'), "Logging failed");
    }
}

```

### Утверждение состояния таблицы и SQL-запроса

Проверить, что содержимое таблицы или SQL-запроса после теста совпадает с ожидаемым. Необходимо получить фактические и ожидаемые данные в виде `ITable` и сравнить их.

```php
class GuestbookTest extends DatabaseTestCase
{
    public function testAddEntry()
    {
        $guestbook = new Guestbook();
        $guestbook->addEntry("suzy", "Hello world!");

        /* 1 шаг проверки утверждений, получение фактических данных */
        $queryTable = $this->getConnection()->createQueryTable(
            'guestbook', 'SELECT * FROM guestbook');
        /* 2 шаг проверки утверждений, получение ожидаемых данных */
        $expectedTable = $this->createFlatXmlDataSet("expectedBook.xml")
                              ->getTable("guestbook");
        /* 3 шаг проверки утверждений, сравнение объектов */
        $this->assertTablesEqual($expectedTable, $queryTable);
    }
}
```

### Утверждения состояния нескольких таблиц

Необходимо получить фактические и ожидаемые данные в виде `IDataSet` и сравнить объекты.

```php
class GuestbookTest extends DatabaseTestCase
{
    public function testFoo()
    {
        /* 1 шаг проверки утверждений, получение фактических данных */
        $dataSet = new QueryDataSet();
        $dataSet->addTable('guestbook', 'SELECT id, content, user FROM guestbook'); 
        
        /* 2 шаг проверки утверждений, получение ожидаемых данных */
        $expectedDataSet = $this->createFlatXmlDataSet('guestbook.xml');

        /* 3 шаг проверки утверждений, сравнение объектов */
        $this->assertDataSetsEqual($expectedDataSet, $dataSet);
    }
}
```



Xdebug либо





[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [22:54](https://vk.com/im?sel=2733257&msgid=82737)

- 

  \10. Анализ покрытия кода

  

  

##### 10 сентября 2019

[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [10:04](https://vk.com/im?sel=2733257&msgid=82740)

- 

  --coverage-clover

  Генерирует файл логов в формате XML с информацией о покрытии кода

  

  

- 

  --coverage-html

  Генерирует отчёт о покрытии кода тестами в формате HTML

  

  

- 

  --coverage-php

  Генерирует сериализованный объект класса PHP_CodeCoverage с информацией о покрытии кода тестами.

  

  




--coverage-text

Генерирует файл логов или вывод командной строки в человекочитаемом формате





[![Павел](https://sun9-11.userapi.com/c857124/v857124641/726bd/NdDYf3YgDRA.jpg?ava=1)](https://vk.com/id2733257)

[Павел](https://vk.com/id2733257) [19:37](https://vk.com/im?sel=2733257&msgid=82789)



Конфигурационный XML-файл