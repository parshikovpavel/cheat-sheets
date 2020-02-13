# Homebrew

**Homebrew** — *free* и *open-source* система управления пакетами (*package management system*), которая упрощает установку программ на *Apple's macOS*. *Homebrew* основан на *Git* и *Ruby*. 

Устанавливается путем запуска в терминале:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Принцип работы

Всего есть два официальных репозитория *Homebrew*:

- `https://github.com/Homebrew/brew`  – сам *Homebrew package manager*
- `https://github.com/Homebrew/homebrew-core` – *Homebrew formulae (формулы, пакеты)*. Это *default tap*, установленный по умолчанию.

## Папки и термины

### Основная папка *Homebrew*

Папка `/usr/local/Homebrew/` – основная папка *Homebrew*, является локальным клоном *Git* репозитория `https://github.com/Homebrew/brew`.

### Tap (кран)

Сторонний (*third-party*) *Git* репозиторий, содержащий *Formulae*. Создается локальный клон *git* репозитория. Доступны для установки только приложения, *Formula* которых описана в одном из  подключенных *Tap*. Репозиторий должен иметь имя `homebrew-XXX`, чтобы использовать упрощенную форму `brew tap`

#### `brew tap`

Возможности:

- Получить текущие подключенные (*tapped*) репозитории. Вызов без аргументов

  ```bash
  $ brew tap
  homebrew/core
  mongodb/brew
  ```

- Подключить *tap*. Сделать локальный клон репозитория `https://github.com/<vendor_name>/homebrew-<repository_subname>` (например, `https://github.com/mongodb/homebrew-brew`), содержащего *formulae*. После этого *formulae* из репозитория могут инсталлироваться.

  ```
  brew tap <vendor_name>/<repository_subname>
  brew tap mongodb/brew
  ```

#### `brew untap`

Удалить *tap* и удалить его репозиторий

```bash
brew untap <vendor_name>/<repository_name>
```

#### Папки для *tap*

- `/usr/local/Homebrew/Library/Taps/<vendor_name>/homebrew-<repository_subname>` – локальный клон *git* репозитория по адресу `https://github.com/<vendor_name>/homebrew-<repository_subname>`. 
- `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core` – локальный клон *default git* репозитория c *Formulae* по адресу `https://github.com/Homebrew/homebrew-core`
- `/usr/local/Homebrew/Library/Taps/mongodb/homebrew-brew` – локальный клон *Git* репозитория с *Formulae* по адресу `https://github.com/mongodb/homebrew-brew`

### Formula (формула)

 Файл на *Ruby* (**.rb*) с определением пакета. Файл описывает программу и как ее устанавливать.

Внутри tap папки находится папка с папка с *Formulae*.

<u>Пример папки с *Formulae*:</u>

- `/usr/local/Homebrew/Library/Taps/<vendor_name>/<repository_name>/Formula` 

<u>Пример файла *Formula*:</u>

- `/usr/local/Homebrew/Library/Taps/<vendor_name>/<repository_name>/Formula/foo.rb` 
- `/usr/local/Homebrew/Library/Taps/mongodb/homebrew-brew/Formula/mongodb-community@3.2.rb`

<u>Формат Formula</u>

```ruby
class MongodbCommunityAT32 < Formula
  # ... 
  
  url "https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-3.2.22.tgz"
  
  def install
  	# ...
  end
    
  
  /* ... */
end
```

### Cellar (погреб)

Папка `/usr/local/Cellar` – папка для установки всех программ (место хранения *Kegs*).

### Keg (бочонок)

Папка для установки программы из конкретной *Formula*

<u>Пример:</u>

- `/usr/local/Cellar/foo/0.1`
- `/usr/local/Cellar/mongodb-community/4.2.1`



## Порядок загрузки пакета

- выдать команду на установку (при этом автоматически выполняется обновление [`brew update`](#brew-update))

  ```bash
  $ brew install <formula>
  $ brew install mongodb-community@3.6
  ```

- загружается файл из параметра `url` указанной *Formula*:

  ```
  https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-3.2.22.tgz
  ```

- запускается `install` метод

- программа инсталлируется в конкретную *Keg*, которая находится внутри *Cellar*

  ```
  /usr/local/Cellar/mongodb-community/4.2.1
  ```

- создает *symlink* в папке `/usr/local/bin` на *binary file* в *Keg*:

  ```
  /usr/local/bin/mongo -> usr/local/Cellar/mongodb-community/4.2.1/bin/mongo
  ```

  

## Команды

### Обновление пакетов

Алгоритм:

- [`brew update`](#brew-update).

-   Узнать, какие пакеты устарели

  ```bash
  brew outdated
  ```

- Обновить все пакеты или конкретную *formula*:

  ```bash
  brew upgrade [<formula>]
  ```

### `brew update`

Обновить сам *Homebrew* и все *formula*'e в *taps*, получая их из *GitHub* с помощью `git pull`.

```bash
brew update
```

### `brew doctor`

Проверьте *Homebrew* на возможные проблемы:

```
brew doctor
```



### `brew uninstall`

Деинстраллировать *Formula*. Удаляет созданные ранее *symlinks* и стирает *Keg* каталог.



