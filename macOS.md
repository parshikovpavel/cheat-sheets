# Homebrew

**Homebrew** — *free* и *open-source* система управления пакетами (*package management system*), которая упрощает установку программ на *Apple's macOS*. Написан на *Ruby*.

Устанавливается путем запуска в терминале:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Принцип работы

<u>Термины:</u>

| Term              | Description                                                  | Example                                                      |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Formula (формула) | Файл на *Ruby* с определением пакета. Файл описывает программу и как ее устанавливать | `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/foo.rb` |
| Tap (кран)        | Git репозиторий, содержащий *Formulae* и/или *commands*      | `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core`    |
|                   |                                                              |                                                              |

<u>Структура папок:</u>

| Directory                          | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| `/usr/local/Homebrew/`             | Основная папка *Homebrew*, является локальным клоном *Git* репозитория `https://github.com/Homebrew/brew`. |
| `/usr/local/Homebrew/Library/Taps` | Содержит набор папок. Каждая папка – ~~локальный клон~~ некоторого *tap*, содержащего *formulae*. |
|                                    |                                                              |

*Homebrew* основан на *Git* и *Ruby*. Папка размещается в `/usr/local/Homebrew` и эта папка является локальным клоном *Git* репозитория `https://github.com/Homebrew/brew`. 

Всего есть два репозитория *Homebrew*:

- `https://github.com/Homebrew/brew`  – сам *Homebrew package manager*
- `https://github.com/Homebrew/homebrew-core` – *Homebrew formulae (формулы, пакеты)*. Это *default tap*, установленный по умолчанию.

## `brew update`

Выполняет `git pull` ~~какого-то~~ репозитория.

Когда вы устанавливаете его в первый раз, он все настраивает (это по умолчанию; вы можете установить его в другом месте, но мы официально не поддерживаем его) в качестве Git-репозитория, который является точным клоном репозитория [Homebrew / homebrew](https://github.com/Homebrew/homebrew) GitHub. Все основные файлы, а также формулы (пакеты) находятся в этом репозитории *. Эти формулы представляют собой файлы Ruby, которые описывают устанавливаемое ими программное обеспечение и содержат инструкции по его установке и тестированию.`/usr/local`

Каждый раз, когда вы запускаете, `brew update`он делает `git pull`вывод, что изменилось из различий и дает вам хороший обзор.

## `brew tap`

`brew tap` добавляет репозиторий в список *formulae*.