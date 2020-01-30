# Github pages

Существует два типа сайтов *GitHub Pages*: 

- *user site* (сайт, связанный с учетной записью пользователя). 

  Сайт размещается в репозитории с именем `<user>.github.io`.  Создание сайта заключается в создании этого репозитория.

  Источником может быть только `master` ветвь. 

  Сайт доступен по адресу `http(s)://<user>.github.io`. 

  Возможен только один такой сайт.

  

- *project site* (сайт, связанный с репозиторием).

  Сайт доступен по адресу `http(s)://<user>.github.io/<repository>`. 

  По умолчанию источником является `gh-pages` ветвь. Также источником может быть `master` ветвь или `/docs` папка. Это настраивается на *Settings* вкладке.
  
  Возможно несколько таких сайтов.
  

## Jekyll

По умолчанию из опубликованного кода генерируется сайт с помощью *Jekyll*. Если автоматическая генерация не требуется, то в корне необходимо разместить `.nojekyll` файл. 

Т.е., например, из файла `/1.md` автоматически генерируется файл `/1.html`. При этом на сайте доступны оба файла – `/1.md` и `/1.html`.

## 404 страница

404 страницу необходимо разместить в файле `404.html` или `404.md`. Для `404.md` необходимо обязательно добавить следующий *front matter*:

```liquid
---
permalink: /404.html
---
```



https://help.github.com/en/github/working-with-github-pages/about-github-pages-and-jekyll

https://stackoverflow.com/questions/41228787/what-are-github-project-pages-and-how-to-use-them

https://www.stephaniehicks.com/githubPages_tutorial/pages/orphan-ghpages.html

https://www.stephaniehicks.com/githubPages_tutorial/pages/github-intro.html



