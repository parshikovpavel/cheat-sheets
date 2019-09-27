# Специальные файлы

## `.gitkeep`

В *Git* невозможно добавить пустую папку (директорию), поэтому среди разработчиков существует правило добавлять в пустые папки пустой файл с расширением `.gitkeep`.

Эта методика не является возможностью системы хранения (как, например, `.gitignore`), а лишь простой хак. 

# Теги

Можно создать теги двух видов:

- аннотированные тег (*annotated tag*) – содержит сообщение, и имя и почту автора тега, как при *commit*

  ```bash
  git tag -a <tagname> -m <message>
  git tag -a 0.0.1 -m "initial version 0.0.1"
  ```

- легковесный тег (*lightweight tags*) – тег без дополнительной информации.

  ```bash
  git tag <tagname>
  git tag 0.0.1
  ```

Просмотр уже созданных тегов:

```bash
$ git tag
0.0.1
```

Просмотр информации о теге:

```bash
$ git show <tagname>
$ git show 0.0.1
tag 0.0.1
Tagger: pparshikov <paw-p@yandex.ru>
...
```

По умолчанию, `git push` не отправляет метки в удаленный репозиторий. Необходимо явно  сделать *push* меток. Возможны два варианта:

- *push* конкретного тега в удаленный репозиторий:

  ```bash
  git push origin <tagname>
  git push origin 0.0.1
  ```

- push всех тегов в удаленный репозиторий:

  ```bash
  git push origin --tags
  ```

# `checkout`

https://ru.stackoverflow.com/questions/431520/%D0%9A%D0%B0%D0%BA-%D0%B2%D0%B5%D1%80%D0%BD%D1%83%D1%82%D1%8C%D1%81%D1%8F-%D0%BE%D1%82%D0%BA%D0%B0%D1%82%D0%B8%D1%82%D1%8C%D1%81%D1%8F-%D0%BA-%D0%B1%D0%BE%D0%BB%D0%B5%D0%B5-%D1%80%D0%B0%D0%BD%D0%BD%D0%B5%D0%BC%D1%83-%D0%BA%D0%BE%D0%BC%D0%BC%D0%B8%D1%82%D1%83

Обозначим начальную ситуацию на следующей схеме:

```
               (i) (wt)
A - B - C - D - ? - ?
            ↑
          master
          (HEAD)
```

`A`, `B`, `C`, `D` — *commits* в ветке `master`.
`(HEAD)` — местоположение указателя HEAD.
`(i)` — состояние индекса Git. Если совпадает c `(HEAD)` - пуст. Если нет - содержит изменения, подготовленные к следующему *commit*.
`(wt)` — состояние рабочей области проекта (*working tree*). Если совпадает с `(i)` — нет неиндексированных изменений, если не совпадает — есть изменения.
`↑` обозначает *commit*, на который указывает определенная ветка или указатель.

## Временно откатиться на некоторое состояние

Чтобы просмотреть файлы в состоянии, на которое указывает некоторый *tag* или *commit*, то можно временно откатиться к некоторому *tag* или *commit*, с помощью команды:

```
git checkout <tagName>
git checkout 0.0.1
git checkout <commitHash>

 (wt)
 (i)
  A - B - C - D
  ↑           ↑
(HEAD)    master
```

После этой команды репозиторий находится в состоянии *«detached HEAD»*. Чтобы переключиться обратно, используйте имя ветки (например, `master`):

```
git checkout master
```

## Создать новую ветку в некотором состоянии

Чтобы создать новую ветку в состоянии, на которое указывает некоторый *tag* или *commit*, используем команду:

```
git checkout -b <branchName> <tagName>
git checkout -b testBranch 0.0.1
git checkout -b <branchName> <commitHash>

  (wt)
  (i)
   A - B - C - D
   ↑           ↑
newBranch   master
 (HEAD)
```

 

Еще есть по ссылке





## Создание локальной ветки и привязка ее к удаленной

git checkout -b pparshikov_6374 remotes/origin/pparshikov_6374

 

https://habr.com/ru/post/157175/



#  Просмотр Тегов в логах

Вы также можете посмотреть теги в логе.

#### ВЫПОЛНИТЕ:



```
git hist master --all
```

#### РЕЗУЛЬТАТ:



```
$ git hist master --all
* fa3c141 2011-03-09 | Added HTML header (v1, master) [Alexander Shvets]
* 8c32287 2011-03-09 | Added standard HTML page tags (HEAD, v1-beta) [Alexander Shvets]
* 43628f7 2011-03-09 | Added h1 tag [Alexander Shvets]
* 911e8c9 2011-03-09 | First Commit [Alexander Shvets]
```

Вы можете видеть теги (`v1` и `v1-beta`) в логе вместе с именем ветки (`master`). Кроме того `HEAD` показывает коммит, на который вы переключились (на данный момент это `v1-beta`).





 

Котеров

пример команд для инициализации git-репозитория компонента ISPager:

$ git add .

$ git oosssit -am "Initialize ISPager"

$ git ranote add origin gitegithub.ooe:igorsimdyanov/pager .git $ git push -u origin master

 

 

Теперь любой желающий может клонировать проект при помощи команды $ git clone https://github.ocn/igorsiaclyanov/pager.git

Версия пакета назначается автоматически, путем выставления метки или тега в системе контроля версий Git (см. главу 54). Например, назначить версию 1.0.0 можно при помощи следующего набора команд:

$ git tag -a vl.0.0 -v 'Version 1.0.0'

$ git push origin vl.0.0

 

 

Разобраться как:

перенести коммит из одной ветки в другую?

