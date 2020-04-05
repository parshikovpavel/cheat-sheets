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

#### Приоритеты загрузки

При установке пакета через `brew install` поиск *formul*'ы по имени выполняется в следующем порядке:

- *core formul*'ы из `homebrew/core` *tap*'а
- другие *tap*'ы

Если нужно установить *formul*'у с некоторым именем из конкретного *tap*'а, нужно указан полный путь к этой *formul*'е:

```
brew install vim                       # установить из homebrew/core
brew install <vendor>/<repository>/vim # установить из другого tap'а
```

### Formula (формула)

 Файл на *Ruby* (**.rb*) с определением пакета. Файл описывает программу и как ее устанавливать.

Внутри tap папки находится папка с папка с *Formulae*.

Возможно несколько *Formul* на разные версии одного приложения, например `php` и `php@7.3`.

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

### `Cellar` (погреб)

Папка `/usr/local/Cellar` – папка для установки всех программ (место хранения *Kegs*).

### Keg (бочонок)

Папка для установки программы из конкретной *Formula*

<u>Пример:</u>

- `/usr/local/Cellar/foo/0.1`
- `/usr/local/Cellar/mongodb-community/4.2.1`

### `opt`

Папка `/usr/local/opt/`, в которой автоматически создаются *symbolic link*'s на все *keg*'s. Эта папка нужна, чтобы при изменении версии приложения не изменялся путь к нему.

Например, при изменении версии приложения `4.2.1 → 4.2.2` меняется путь к *Keg*'у, но не меняется путь к `opt` папке:

```
/usr/local/opt/mongodb-community -> ../Cellar/mongodb-community/4.2.1
/usr/local/opt/mongodb-community -> ../Cellar/mongodb-community/4.2.2
```

Особенно это актуально для *keg-only пакетов*, т.к. для них не создается *symlink* в папке `/usr/local/bin` . Поэтому обращаться к ним следует через *symlink* в `opt` папке.

### Cask (бочка)

[Расширение Homebrew](https://github.com/Homebrew/homebrew-cask), которое позволяет устанавливать *native MacOS Application* (такие как *Google Chrome, Dropbox, VLC*) из командной строки. При этом не требуется скачивать `.dmg` файлы и перетаскивать их в папку «Приложения», достаточно запустить команду `install` из командной строки.

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

- если это не *keg-only Formula*, создается *symlink* в папке `/usr/local/bin` на *binary file* в *Keg*:

  ```
  /usr/local/bin/mongo -> usr/local/Cellar/mongodb-community/4.2.1/bin/mongo
  ```

- создается *symlink* на *Keg* папку в `opt` папке (`/usr/local/opt/`):

  ```
  /usr/local/opt/mongodb-community -> ../Cellar/mongodb-community/4.2.1
  ```

  

## Keg-only пакеты

*Keg-only* пакеты устанавливаются только в *Keg* и для них не создается *symlink* в папке `/usr/local/bin`.  Можно превратить *Non-keg-only* пакет в *keg-only* пакет, используя команду `brew unlink`.

Пакет помечается как *Keg-only*, если он может конфликтовать с приложениями, установленными в `/usr/local`. Например, если:

- для данного пакета приложения существует более новая версия приложения
- в системе уже установлено по умолчанию это приложение

Чтобы создать *symlink* в папке `/usr/local/bin` для таких пакетов, нужно использовать `brew link`.

Либо можно добавить вручную в `PATH` путь к `opt` папке пакета.

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

### Информация о пакетах

#### `brew list`

```bash
brew list [ formula ]
```

- если `formula` не указана – вывести список всех установленных *formula*'s.
- если `formula` указана – вывести пути, соответствующие этой установленной *formul*'е

#### `brew search`

```bash
brew search [ text |
```

Выполните поиск подстроки `text` в названиях *cask*'s и *formula*'s. Поиск выполняется в *tap*'ах: `homebrew/core` и `homebrew/cask`.

#### `brew info`

```bash
brew info [ formula ]
```

Показать краткую информацию о *formul*'е.

#### `brew cat`

```bash
brew cat [ formula ]
```

Показать содержание *formul*'ы.

#### `brew link`

Создает *symlink*'и в папке `/usr/local` на некоторые файлы *package*. Эта операция выполняется автоматически при установке *Non-keg-only* формул.

```bash
brew link [options] formula
```

Опции:

- `-n`, `--dry-run`: список файлов, которые будут *symlink* и которые будут удалены из `/usr/local ` с опцией `--overwrite`. При этом никакого *symlink* и удаление по этой команде не выполняется. 
- `--overwrite`: Разрешить удаление файлов, которые уже существуют в `/usr/local/` и должны быть удалены при создании новых *symlink*.
- `-f`, `--force`: Обязательная опция для *key-only formul*

#### `brew unlink`

Удалить *symlink*'и в папке `/usr/local` на файлы *package*.

```bash
brew unlink [options] formula
```

Опции:

- `-n`, `--dry-run`: список *symlink*'ов, которые будут удалены из `/usr/local `. При этом самого удаления по этой команде не выполняется. 

# Homebrew services

Позволяет запускать *homebrew formul*'ы через `launchctl`.

## `sudo`

- Если использовано `sudo`, то помещается в `/Library/LaunchDaemons` (как *daemon*, *start* при загрузке *at boot*).
- Если не указано `sudo`, то помещается в `~/Library/LaunchAgents` (как *agent*, *start* при входе в систему *at login*).

## `start`

*Start* `<service>` немедленно и зарегистрировать его для запуска *at login* (как *agent*) или *at boot* (как *daemon*):

```
[sudo] brew services start <service>
```

## `stop`

*Stop* `<service>` немедленно и отменить регистрацию его для запуска *at login* (как *agent*) или *at boot* (как *daemon*):

```
[sudo] brew services stop <service>
```

## `run`

*Start* `<service>`, но не регистрировать его для запуска ни *at login*, ни *at boot*:

```
brew services run <service>
```

## `restart`

*Stop* и *start* `<service>` немедленно и зарегистрировать его для запуска *at login* (как *agent*) или *at boot* (как *daemon*):

```
[sudo] brew services restart <service>
```

## `list`

Список всех *servic*'ов под управлением `brew services`:

```
brew services list
```

## `cleanup`

Удалите все неиспользуемые *servic*'ы:

```
[sudo] brew services cleanup
```

# Launch

`launchd` - это *daemon*, который управляет запуском *servic*'ов (*process*'ов). Все процессы в MacOS запускаются `launchd`. При загрузке `launchd` вызывается ядром как первый процесс с `PID = 1` и дальше вся система стартует с помощью него. Так же `launchd` следит за тем чтобы процесс был запущен. Если он вдруг упадет, `launchd` ему поможет и поднимет его.

## Типы *servic*'ов

Servic'ы делятся на:

- *daemon*'ы – запускаются после запуска системы до входа в систему, от имени `root`. 
- *agent*'ы – запускаются после входа в систему, от имени вошедшего в систему пользователя. 

## Job definition

*Job definition* – `.plist` файл, который определяет порядок запуска *servic*'а. `.plist` имеет стандартизованный формат в виде *XML*. Файлы  `.plist` размещаются в каталогах:

| Type           | Location                        | Run on behalf of                                             |
| -------------- | ------------------------------- | ------------------------------------------------------------ |
| User Agents    | `~/Library/LaunchAgents`        | *agent*'ы конкретного пользователя. Исполняются при авторизации конкретного пользователя |
| Global Agents  | `/Library/LaunchAgents`         | *agent*'ы всех пользователей. Исполняются при авторизации любого пользователя. |
| Global Daemons | `/Library/LaunchDaemons`        | *daemon*'ы всех пользователей. Исполняются после запуска системы. |
| System Agents  | `/System/Library/LaunchAgents`  | Создаются операционной системой, пользователь не должен их изменять |
| System Daemons | `/System/Library/LaunchDaemons` | Создаются операционной системой, пользователь не должен их изменять |

После запуска системы `lanchd` запускает *daemon*'ы из каталогов `/System/Library/LaunchDaemons` и `/Library/LaunchDaemons`. Когда пользователь входит в систему, `lanchd` запускает *agent*'ы из каталогов `/System/Library/LaunchAgents`, `/Library/LaunchAgents` и `~/Library/LaunchAgents`.

## `launchctl`

Приложение командной строки, через которое можно управлять работой `launchd`.

Наиболее частые случаи использования:

- *Load* и *start job*'ы

  ```bash
  launchctl load -w
  ```

- *Stop* и *unload job*'ы

  ```bash
  launchctl unload -w
  ```

  

### `list`

Посмотреть список загруженных *job*'s:

```bash
$ launchctl list
PID	Status	Label
-	0	    com.apple.SafariHistoryServiceAgent
-	-9	    com.apple.progressd
845	0	    abnerworks.Typora.6092
```

`PID` – PID *job*'ы, если она *is running*.

### `load` и `unload`

*Load* и *start job*'у до тех пор пока *job*'а не будет помечена как `disabled`:

```bash
launchctl load
```

*Stop* и *unload job*'у. *Job*'а будет заново запущена при следующем *login/reboot*.

```bash
launchctl unload
```

------

*Load* и *start job*'у, а также пометить ее как `not disabled`. *Job*'а будет заново запущена при следующем *login/reboot*.

```bash
launchctl load -w
```

Stop и unload job'у, а также помечает ее как `disabled`. *Job*'а НЕ будет заново запущена при следующем *login/reboot*.

```bash
launchctl unload -w
```

### `start` и `stop`

Используются только для ручного start и stop job'ы при тестировании и отладке.

*Start job*'ы:

```bash
launchctl start
```

*Stop job*'ы:

```bash
launchctl stop
```

# `hosts`

Команда:

```bash
sudo nano /private/etc/hosts
```

