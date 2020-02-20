- *enumerable* – перечисляемые *properties*
- *own* – собственные *properties*





### `Object.assign()`

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

### Изменение папки глобальных *package*'s

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




