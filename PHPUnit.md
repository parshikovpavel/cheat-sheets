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

