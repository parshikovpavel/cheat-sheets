# Native javascript

- *enumerable* – перечисляемые *properties*
- *own* – собственные *properties*

## Регулярные выражения

### Синтаксис

Существует два синтаксиса для создания регулярного выражения. В обоих случаях возвращается объект класса `RegExp`.

- С использованием объекта `RegExp`:

  ```javascript
  regexp = new RegExp('<pattern>', '<flags>');
  ```

- С использованием слешей `/.../`:

  ```javascript
  regexp = /<pattern>/; // без флагов
  regexp = /<pattern>/<flags>; // с флагами gmi (будут описаны далее)
  ```

`/.../` используются, когда мы на момент написания кода точно знаем, каким будет регулярное выражение.  `RegExp` – когда мы хотим создать регулярное выражение «на лету» из динамически сгенерированной строки.

### Функции для работы

#### `String.match`

Ищет в заданной строке `str` совпадения с регулярным выражением `regexp`.

 ```javascript
str.match(regexp): (Array | null)
 ```

<u>Параметры:</u>

- `(RegExp | String) regex` – объект регулярного выражения. 

  Если передан `String`, то он будет преобразована в объект `RegExp` через вызов конструктора `new RegExp(<regexp>)`.

<u>Возвращаемое значение:</u>

-  `Array`, содержащий результаты сопоставления.
-  `null`, если ничего не было сопоставлено (не пустой массив `[]`)

<u>Описание:</u>

Возвращаемое значение зависит от флага `g`:

1. Если у регулярного выражения есть флаг `g`, то `String.match` возвращает массив всех совпадений:

   ```javascript
   var str = 'АБВГДЕЁЖЗИЙКЛМНОПРСТУФХЦЧШЩЬЫЪЭЮЯабвгдеёжзийклмнопрстуфхцчшщьыъэюя';
   var regexp = /[А-Д]/gi;
   var matches_array = str.match(regexp);
   
   console.log(matches_array);
   // ['А', 'Б', 'В', 'Г', 'Д', 'а', 'б', 'в', 'г', 'д']
   ```

2. Если у регулярного выражения нет флага `g`, то `String.match`  возвращает то же, что и при вызове метода `RegExp.exec()` + некоторые дополнительные поля. 

   Структура возвращаемого `Array`:

   -  `0` элемент – сопоставленный текст
   - `1`, `2`, ... – текст, захваченный при сопоставлении круглыми скобками `()`. 
   - свойство `input` – оригинальная строка. 
   - свойство `index` – позиция сопоставления в исходной строке.

   ```javascript
   'foobarbaz'.match(/o(bar)b/) # 0: "obarb"
                                    # 1: "bar"
                                    # groups: undefined
                                    # index: 2
                                    # input: "foobarbaz"
                                    # length: 2
   ```

   


## `localStorage` и `sessionStorage`

`localStorage` и `sessionStorage` – это свойства `window`. И они доступны также как `window.localStorage` и `window.sessionStorage`

`localStorage` и `sessionStorage` позволяют сохранять пары *key/value* в браузере. 

Данные не имеют «времени истечения».

Особенности `localStorage`:

- `localstorage` один на все вкладки и окна в рамках *origin*'а.
- Данные не имеют срока давности, по которому истекают и удаляются. Сохраняются после перезапуска браузера и даже ОС.

Особенности `sessionStorage`:

- `sessionStorage` существует только в рамках текущей вкладки браузера. Другая вкладка с тем же *origin* будет иметь другое хранилище.

- `sessionStorage` разделяется между `iframe` на той же вкладке (при условии, что они с одним и тем же *origin*).
- данные продолжают существовать после перезагрузки страницы, но не после закрытия/открытия вкладки.

Данные в `localStorage` сохраняются после перезапуска браузера, данные в  `sessionStorage` сохраняются после обновления страницы.

*Storage* привязан к *origin*'у (домен/протокол/порт). Это значит, что разные протоколы или поддомены определяют разные объекты хранилища, и они не могут получить доступ к данным друг друга.

Просмотреть содержимое в *Google Chrome* – *DevTools* (F12), вкладка *Resourses* 

Браузеры выделяют 5мб под `localStorage`.

### Операции над  *storage*'м

Методы:

- `setItem(key, value)` – сохранить пару ключ/значение.
- `getItem(key)` – получить данные по ключу `key`.
- `removeItem(key)` – удалить данные с ключом `key`.
- `clear()` – удалить всё.
- `key(index)` – получить ключ на заданной позиции.
- `length` – количество элементов в хранилище.

Также можно получать/записывать данные, как в обычный объект (но не рекомендуется):

```javascript
// установить значение для ключа
localStorage.test = 2;

// получить значение по ключу
alert( localStorage.test ); // 2

// удалить ключ
delete localStorage.test;
```

### Тип *value*

*Key* и *value* должны быть `string`!!!

Если мы используем любой другой тип, например `int` или `object`, то он автоматически преобразуется в `string`:

```javascript
sessionStorage.user = {name: "John"};
console.log(sessionStorage.user); // [object Object]
```

Типичное решение для хранения других типов данных в *storage* – использовать `JSON`:

```javascript
sessionStorage.user = JSON.stringify({name: "John"});

let user = JSON.parse( sessionStorage.user );

console.log( user.name ); // John
```





## `Object.assign()`

(ES6) Метод копирует все *enumerable own properties* из `sources` объектов в `target` объект.  Последующие *properties* перезаписывают предыдущие. 

```javascript
Object.assign(target, ...sources)
```

Возвращаемое значение – `target` объект.

```javascript
var o1 = { a: 1 };
var o2 = { b: 2 };

var obj = Object.assign(o1, o2); // obj = o1 = { a: 1, b: 2}
```

Клонирование объекта:

```javascript
var obj = { a: 1 };
var copy = Object.assign({}, obj); // copy = { a: 1 }
```

## `Promise`

!!!

# Function

Объявление метода:

```javascript
{
    name: function () {}
}
```

Стандарт ES6 (другое название ES2015) поддерживает сокращенный синтаксис объявления метода

```javascript
{
    name() {}
}
```

# Node.js

Установка *node.js* и *npm*. *Npm* включен в установщик *Node.js*.:

```bash
brew install node
```

Проверить установку

```
node -v
npm -v
```

## npm

*npm* (*node package manager*) – *package manager* для *JavaScript*. Это также *package manager* по умолчанию для *Node.js*. 

В состав входят:

- клиент командной строки `npm`
- *npm registry* – онлайн база данных бесплатных публичных *package*'s и платных приватных *package*'s. 

### `package.json`

*Package* – это директория, в которой создан файл `package.json`.

`package.json` имеет *JSON* формат.

Для генерации `package.json` можно использовать команду:

```bash
npm init
```



Структура `package.json` (аналог `composer.json`) зависит от способа использования *package*'а:

- *package* не будет публиковаться. `package.json`  только описывает *dependency*'s 
- *package* будет публиковаться. `package.json`  описывает сам *package* (имя, версия, ...) и его *dependency*'s.

Общая структура:

```json
{
  "name": "<name>",
  "version": "<version>",
  "dependencies": {
    "<dependency_name>": "<version>",
    ...
  },
  "devDependencies" : {
    "<dependency_name>": "<version>",
    ...
  }    
}
```

Свойства:

- `dependencies` – *package*'s, которые требуются во всех окружениях.
- `devDependencies` – *package*'s, которые требуются только в *dev*-окружении 

### Посмотреть список установленных *package*'s

- список локальных *package*'s

  ```bash
  npm ls
  ```

- список глобальных *package*'s (`-g`), с глубиной зависимостей 0 (`depth=0`). Также выводится папка, где они установлены.

  ```bash
  npm ls -g --depth=0
  ```

### Установка *package*

*Package* может устанавливаться:

- локально (по умолчанию).

  *Package* устанавливается в папку`./node_modules/<package_name>` относительно текущего проекта, *package* доступен только этому проекту.

  Исполняемые файлы устанавливаются в папку`./node_modules/.bin`

- глобально – требует использования `-g` флага. 

  *Package* устанавливается в `<prefix>/lib/node_modules`.Посмотреть текущее значение конфигурационного параметра `prefix`:

  ```bash
  npm config get prefix
  /usr/local
  ```

  Текущий путь к папке с глобальными *package*'s:

  ```bash
  npm root -g
  /usr/local/lib/node_modules
  ```

  *Package* доступен для использования из любого проектах в любой папке. 

  Исполняемые файлы устанавливаются в папку `<prefix>/bin`.
  
  После установки пакета его копия сохраняется в кэше в папке `~/.npm`, чтобы при следующей  установке не требовалось использовать сеть.

#### Изменение папки глобальных *package*'s

- Создать папку `~/.node_modules_global`

    ```bash
    cd ~
    mkdir .node_modules_global
    ```

- Указать эту папку в качестве `<prefix>`:

    ```bash
    npm config set prefix=$HOME/.node_modules_global
    ```

- Добавить папку исполняемых файлов `~/.node_modules_global/bin` в переменную окружения `$PATH`:

  ```bash
  export PATH="$HOME/.node_modules_global/bin:$PATH"
  ```

  


#### Ручное редактирование `package.json`

- Надо вручную включить зависимость в `package.json` в блок `dependencies` или `devDependencies`.

- Выполнить:

  ```bash
  npm install
  ```

#### Автоматически из командной строки

Эти команды позволяют автоматически добавить зависимость в `package.json` и сразу скачать.

- Добавить *package* в качестве зависимости в `dependencies` блок:

  ```bash
  npm install <package_name>
  ```

- Добавить *package* в качестве зависимости в `devDependencies` блок:

  ```bash
  npm install <package_name> --save-dev
  ```

По умолчанию, *package* устанавливается локально. Для глобальной установки необходимо использовать `-g` флаг.

```bash
npm install <package_name> -g
```

### Удаление *package*

- Удаление локально установленного *package*:

  ```bash
  npm uninstall <package_name>
  ```

- Удаление глобально установленного *package*:

  ```bash
  npm uninstall <package_name> -g
  ```

  

### `package-lock.json`

После установки версии *dependency*'s фиксируются в файле `package-lock.json` (аналог `composer.lock`)

### CLI commands

#### `npm view`

 Показать информацию о *package*

```bash
npm view <package_name>
```



## Node.js

Особенности *Node.js*:

- написан на C++
- это интерпретатор языка *JavaScript*, который получает на входе *JavaScript*-код и выполняет его.

Запуск скрипта на обработку из консоли:

```bash
node test.js
```

# Yarn

*Yarn* – *package manager*, основанный на npm, разработанный *Facebook*. 

Особенности:

- полностью совместим с *npm*:
  - скачивает *package*'s из npm-реестра
  - работает с тем же файлом `package.json`
  - устанавливает *package*'s в директорию `node_modules`

Преимущества:

- быстрее на 30% — 50%, чем *npm*

Установка:

```bash
brew install yarn
```

Установка *dependency*'s из `package.json` после загрузки проекта:

```bash
yarn install
```

# Import

```javascript
import 'imgAreaSelect'
import 'imgAreaSelect/imgAreaSelect.css' # CSS!!!
import { NodeSelection } from 'prosemirror-state'
```

