# Homebrew

**Homebrew** — *free* и *open-source* система управления пакетами (*package management system*), которая упрощает установку программ на *Apple's macOS*. *Homebrew* основан на *Git* и *Ruby*. 

Устанавливается путем запуска в терминале:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Принцип работы

<u>Термины:</u>

| Term              | Description                                                  | Example                                                      |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Formula (формула) | Файл на *Ruby* (**.rb*) с определением пакета. Файл описывает программу и как ее устанавливать | `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/foo.rb` |
| Tap (кран)        | Git репозиторий, содержащий *Formulae* и/или *commands*      | `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core`    |
|                   |                                                              |                                                              |

Всего есть два официальных репозитория *Homebrew*:

- `https://github.com/Homebrew/brew`  – сам *Homebrew package manager*
- `https://github.com/Homebrew/homebrew-core` – *Homebrew formulae (формулы, пакеты)*. Это *default tap*, установленный по умолчанию.

<u>Структура папок:</u>

- `/usr/local/Homebrew/` – основная папка *Homebrew*, является локальным клоном *Git* репозитория `https://github.com/Homebrew/brew`.

- `/usr/local/Homebrew/Library/Taps/<vendor_name>/<repository_name>` – локальный клон *tap* по адресу `https://github.com/<vendor_name>/<repository_name>`. 

  Например:

  - `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core` – локальный клон *default Git* репозитория c *Formulae* по адресу `https://github.com/Homebrew/homebrew-core`
  - `/usr/local/Homebrew/Library/Taps/mongodb/homebrew-brew` – локальный клон *Git* репозитория с *Formulae* по адресу `https://github.com/mongodb/homebrew-brew`

- `/usr/local/Homebrew/Library/Taps/<vendor_name>/<repository_name>/Formula` – папка с *Formulae*, находящихся в этом *Tap*

  Например:

  - `/usr/local/Homebrew/Library/Taps/mongodb/homebrew-brew/Formula/mongodb-community@3.2.rb`

<u>Формат Formula (файл *.rb с описанием пакета)</u>

```ruby
class MongodbCommunityAT32 < Formula
  /* ... */
  
  url "https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-3.2.22.tgz"
  
  /* ... */
end
```

<u>Порядок загрузки пакета</u>

- выдать команду на установку

  ```bash
  $ brew install <formula>
  $ brew install mongodb-community@3.6
  ```

- загружается файл из параметра `url` указанной *Formula*:

  ```
  https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-3.2.22.tgz
  ```

- 





## `brew update`

Выполняет `git pull` ~~какого-то~~ репозитория.







Когда вы устанавливаете его в первый раз, он все настраивает (это по умолчанию; вы можете установить его в другом месте, но мы официально не поддерживаем его) в качестве Git-репозитория, который является точным клоном репозитория [Homebrew / homebrew](https://github.com/Homebrew/homebrew) GitHub. Все основные файлы, а также формулы (пакеты) находятся в этом репозитории *. Эти формулы представляют собой файлы Ruby, которые описывают устанавливаемое ими программное обеспечение и содержат инструкции по его установке и тестированию.`/usr/local`

Каждый раз, когда вы запускаете, `brew update`он делает `git pull`вывод, что изменилось из различий и дает вам хороший обзор.

## `brew tap`

Сделать локальный клон репозитория *tap*, содержащий *formulae*.

```
brew tap mongodb/brew
```