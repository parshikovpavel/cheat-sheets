### Команды

`jekyll build` – построить сайт и вывести его код в каталог `_site`.

`jekyll serve` – перестроить сайт (если были изменены файлы кода), вывести его код в каталог `_site`, запустить веб-сервер на `http://localhost:4000`.

### Шаблоны

Язык шаблонов – *Liquid*.

Использует 3 основных типа блоков:

- *object* (объект) – выводит значение выражения. Используются символы `{{`и `}}`.

  ```html
  {{ <object> }}
  {{ page.title }}
  ```

- *tag* (тег) – для управления потоком выполнения. Используются символы `{%` и `%}` Например, условный оператор:

  ```
  {% if page.show_sidebar %}
    ...
  {% endif %}
  ```

- *filter* (фильтр) – для изменения значения *object*'а. *Object* и *filter* отделяются символом `|`.

  ```
  {{ <object> | <filter> }}
  {{ "hi" | capitalize }}
  ```

  Стандарные *Liquid filter*'ы – https://shopify.github.io/liquid/filters/ или https://jekyllrb.com/docs/liquid/filters/#standard-liquid-filters

  Дополнительные *Jekyll tag*'и – https://jekyllrb.com/docs/liquid/tags/

Чтобы *jekyll* обрабатывал блоки на языке *Liquid*, в начале шаблона необходимо разместить *front matter*.

### Tag

Стандарные *Liquid tag*'и – https://shopify.github.io/liquid/tags/

Дополнительные *Jekyll tag*'и – https://jekyllrb.com/docs/liquid/tags/

#### if

```
{% if <condition> %}
  Body
{% endif %}
```

#### for

```
{% for <variable> in <array> %}
  {{ <variable> }}
{% endfor %}
```



### Front matter

Это YAML код, размещенный между символами `---`.

```
---
  #####
---
```

Может использоваться для задания переменных (например, `title`), которые будут доступны как свойство объекта `page` (например, `page.title`).

```
---
title: My blog
---
{{ page.title }}
```

### Layout

*Layout* (макет) – удобны для общих шаблонов, внутрь которых вставляется контент более мелких шаблонов. Например, макет для общего шаблона всего сайта. Располагаются в папке `_layouts`.

В *layout* всегда определена переменная `content`:

```html
# layout.html
{{ page.title }}
{{ content }}
```

В эту переменную подставляется контент страницы, для который установлен этот *layout*. Чтобы установить `layout` для страницы, необходимо указать его в переменной `layout` в *front matter*.

```
# index.html
---
layout: default
title: Home
---
```

Переменная (`title`), определенная в *Front matter* `index.html`, будут доступна в *layout* (`layout.html`).

### Include

*Include* (включение) – шаблон (блок), который может включаться в другие шаблоны. Размещается в папке `_includes`. 

Для вставки *include* в шаблон необходимо использовать `include` *tag*.

```
{% include navigation.html %}
```

### Data

*Data* (данные) – файл данных в формате *YAML*, *JSON* или *CSV*. Размещается в папке `_data`.

Данные из файла `_data/xxx.yml` будут доступны в переменной `site.data.xxx`.

### Asset

Asset (ресурс). Располагаются в папке `assets`, в которой размещаются:

```
.
├── assets
|   ├── css
|   ├── images
|   └── js
```

Папка `assets` просто копируется в папку сайта.

Ссылка на *asset* имеет вид:

```html
<link rel="stylesheet" href="/assets/css/styles.css">
```

### Post

Post (пост для блога). Размещаются в папке `_posts`. Имя файла для *post* имеет вид:

```
<дата_публикации>-<название>.<расширение>
2018-08-20-bananas.md
```

<u>Файл post'а</u>

Должен содержать *front matter*:

```
---
layout: post
# some user variables
author: xxx
---
```

Дата *post*'а из имени файла доступна в переменной `page.date`.

<u>Список post'ов</u>

*Post*'ы доступны через переменную `site.posts`.

Пример перебора:

```
{% for post in site.posts %}
    {{ post.xxx }}
{% endfor %}
```

Доступны специальные свойства `post`:

- `post.url`
- `post.title` – заголовок из имени файла
- `post.excerpt` – первый параграф текста

Эти переменные могут быть предопределены в *front matter*.

### Configuration

Размещается в файле `./_config.yml`.

После изменения конфигурации требуется рестарт *jekyll*.

### Collection

Collection – похожа на массив *post*'ов, но для нее можно установить имя и не нужно указывать дату.

Определяются в *configuration file* в формате:

```
collections:
  <collection_name>:
```

*Document* – элемент *collection*. Также как и post'ы, *document*'ы – это файлы. *Document*'ы размещаются в папке `_<collection_name>`

### Документация

Глобальные переменные – https://jekyllrb.com/docs/variables/

