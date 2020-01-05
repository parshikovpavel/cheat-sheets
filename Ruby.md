## RubyGems

*RubyGems* - это пакетный менеджер для языка программирования Ruby. В его состав входят:

- формат для описания пакетов (*gem*)
- инструменты для управления *gem*'ами.

### Формат для описания пакетов

*gem* – формат для описания пакета (программы или библиотеки).

*gem* с именем `gem_name` имеет структуру:

```
gem_name/
├── bin/
│   └── gem_name
├── lib/                 # Код gem
│   └── gem_name.rb
├── test/
│   └── test_gem_name.rb
├── README               # Документация
├── Rakefile             # Автоматизация тестов и генерация кода
└── gem_name.gemspec     # Спецификация gem в формате YAML
```

### Инструменты для управления gem'ами

#### Консольная утилита `gem`

Для работы с *gem* пакетами из консоли используется утилита `gem` (аналог утилиты `composer`).

##### Настройки утилиты

Получить настройки утилиты можно с помощью команды:

```bash
$ gem environment
RubyGems Environment:
  - INSTALLATION DIRECTORY: /Library/Ruby/Gems/2.6.0
  - USER INSTALLATION DIRECTORY: /Users/pavelparshikov/.gem/ruby/2.6.0
  - GEM PATHS:
     - /Library/Ruby/Gems/2.6.0
     - /Users/pavelparshikov/.gem/ruby/2.6.0
     - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0
```

Важные настройки:

- `INSTALLATION DIRECTORY` – центральная директория по умолчанию для инсталляции *gem*'ов. По умолчанию здесь указана системная глобальная директория, доступ к которой требует `sudo`. 
- `USER INSTALLATION DIRECTORY` – директория пользователя по умолчанию для инсталляции *gem*'ов.
- `GEM PATHS` – пути, по которым выполняется поиск *gem*'ов для запуска.

На эти настройки влияют следующие *environment variables*. Изначально *environment variables* не установлены, поэтому для путей используются значения по умолчанию:

- `GEM_PATH` – задает настройку `GEM PATHS`.
- `GEM_HOME` – задает настройку `INSTALLATION DIRECTORY`

Узнать путь к *library file*:

```bash
$ gem which <gem_name>
$ gem which sqlite3
/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/gems/sqlite3-1.3.13/lib/sqlite3.rb

```



##### `gem install`

Установка *gem*. По умолчанию в директорию `INSTALLATION DIRECTORY`.

```bash
gem install <gem_name>
```

<u>Опции:</u>

- `--user-install` – Установить *gem* в `USER INSTALLATION DIRECTORY`.  

Смотреть [Избежать установки gem'ов глобально](#Избежать-установки-gem-ов-глобально).

##### `gem other`

Деинсталляция:

```bash
gem uninstall <gem_name>
```

Список установленных локальных (*local*) *gems*:

```bash
gem list
```

Список доступных удаленных (*remote*) *gems*:

```bash
gem list --r [ | grep <string>]
```

или

```bash
gem search <string>
```



#### Bundler

##### Терминология

*Bundler* - это менеджер для управления зависимостями *gem*'ов в ruby приложениях. 

*Bundle* – пакет

*Bundler* является  *gem*'ом, поэтому устанавливается через консольную утилиту `gem`:

```bash
gem install bundler
```

##### Gemfile

*Bundler* при установке зависимостей использует конфигурации из файла *Gemfile*. 

*Gemfile* – файл с конфигом для *Bundler*, который описывает зависимости (*gem*'ы) и их версии для некоторого *bundle* (аналог `composer.json`)

Пример *Gemfile*:

```ruby
source "https://rubygems.org" # ресурс, откуда будут устанавливаться gems

gem "jekyll"                  # список gem'ов, которые нужны для работы приложения
gem "rack", "1.0.1"           # gem с указанием версии

# указание групп

group :jekyll_plugins do      # gem можно объединять в группы и устанавливать потом
  gem "jekyll-feed"           # только определенные группы
  gem "jekyll-seo-tag"
end

gem 'rspec', :group => 'development'  # другая формат для указания группы

gem "rails", ">=2.3.2"        # по умолчанию gem'ы включаются в группу `default`
```

##### gemfile.lock

Файл `gemfile.lock` – описывает все установленные *gem*'ы, а также их точные версии (аналог `composer.lock`). Он позволяет зафиксировать комбинацию версий всех используемых *gem*'ов. 

##### `bundle install`

Устанановить все зависимости (*gem*'ы), указанные в *gemfile*:

```bash
bundle install # аналог `composer install`
```

Также эта команда используется для установки *gem*'ов после изменения *Gemfile*. 

<u>Папка для установки</u>

По умолчанию, *gem*'ы устанавливаются в папку *rubygems*'а, которая указана в его настройке `INSTALLATION DIRECTORY`. Поэтому *gem*'ы, установленные через *bundler* без опции `--path`, будут выводиться командой `gem list`.

Смотреть [Избежать установки gem'ов глобально](#Избежать-установки-gem-ов-глобально).

Флаги не запоминаются между вызовами команд. И если их нужно запомнить, то следует использовать `bundle config` (например, `bundle config path <path>`).

<u>Флаги</u>

- `--without` – исключить из установки некоторую группу:

    ```bash
    bundle install --without=<group>
    bundle install --without=test
    ```

- `--path` – указать путь для установки *gem*'ов. Рекомендуется указывать этот флаг и устанавливать *gem*'ы для каждого *bundle*'а в изолированный каталог. В этом случае *gem*'ы некоторого *bundle*'а не будут конфликтовать с глобальными *gem*'ами. 

  ```bash
  bundle install --path <path>
  bundle install --path vendor/bundle
  ```

  

##### `bundle exec`

 Исполнить команду в контексте текущего *bundle*. В контексте текущего *bundle* означает – с теми *gem*'ами и их версиями, которые описаны в *Gemfile* текущего *bundle*.

```bash
bundle exec <command>
```

Можно попробовать запустить команду напрямую `<command>`. В этом случае используются глобальные *gem*'ы, установленные в системе. Такая команда может работать корректно в одной системе и не работать в другой, с *gem*'ами других версий.

##### `bundle config`

Установить параметры конфигурации *bundler*'а.

```bash
bundle config [<name> [<value>]]
```

Параметры конфигурации загружаются в следующем порядке:

1. Локальный конфиг ( `./.bundle/config`)
2. Глобальный конфиг ( `~/.bundle/config`)

Вывести все параметры конфигурации и где они были установлены:

```bash
bundle config
```

Вывести значение параметра конфигурации `<name>` и где оно установлено:

```bash
bundle config <name>
```

Установить значение `<value>` параметра конфигурации `<name>` в глобальном конфиге (`~/.bundle/config`) для текущего пользователя:

```bash
bundle config --global <name> <value>
```

Установить значение `<value>` параметра конфигурации `<name>` в локальном конфиге (`./.bundle/config`):

```bash
bundle config --local <name> <value>
```

<u>Возможные флаги:</u>

- `path` – путь для установки *gem*'ов:

```bash
bundle config path <path>
bundle config path vendor/bundle
```

##### `bundle env`

Настройки *bundler*

```bash
$ bundle env
## Environment

​```
Bundler       1.17.2
  Platforms   ruby, universal-darwin-19
  ...
  Gem Home    /Library/Ruby/Gems/2.6.0
  Gem Path    /Users/pavelparshikov/.gem/ruby/2.6.0:/Library/Ruby/Gems/2.6.0:/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0
  User Path   /Users/pavelparshikov/.gem/ruby/2.6.0
  ...
```

Выводятся настройки *Rubygems*:

- `Gem Home` = `INSTALLATION DIRECTORY`
- `Gem Path` =  `GEM PATHS`
- `User Path` = `USER INSTALLATION DIRECTORY`

##### `bundle init`

Создает *Gemfile* в текущем каталоге. 

#### Избежать установки gem'ов глобально

По умолчанию gem'ы устанавливаются в системную папку глобально и поэтому требуют `sudo`. Избежать этого можно следующими способами

<u>Установка gem'ов в user's home directory</u>

Все *gem*'ы хранятся в одном экземпляре для каждого пользователя.

Необходимо изменить в файле `~/.bash_profile` следующие *environment variables*:

- `GEM_HOME` – задает настройку `INSTALLATION DIRECTORY` для *Rubygems*. А также принимается *bundler*'ом по умолчанию в качестве *installation directory*. Необходимо изменить на значение настройки `USER INSTALLATION DIRECTORY`.
- `PATH` – необходимо добавить буть к `<USER INSTALLATION DIRECTORY>/bin`чтобы иметь возможность запускать *binaries* из *installation directory* напрямую.

```bash
export GEM_HOME=$(ruby -e 'puts Gem.user_dir')
export PATH=$GEM_HOME/bin:$PATH
```

После этого консольная утилита *gem* и *bundler* будут устанавливать *gem*'ы в *user's home directory*.

Можно для консольной утилиты *gem* использовать опцию `--user-install` (а можно и не использовать, т.к.`INSTALLATION DIRECTORY`= `USER INSTALLATION DIRECTORY`):

```bash
gem install <gem_name> --user-install
```

<u>Установка gem'ов в папку проекта</u>

Все *gem*'ы устанавливаются отдельно для каждого проекта в его папку (аналогично поведению по умолчанию для `composer`). Например, в папку `./vendor/bundle`.

