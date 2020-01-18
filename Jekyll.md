### Command line

`jekyll new PATH` – создать заготовку сайта с использование `Minima` *them*e'ы 

`jekyll new PATH --blank` – создать заготовку сайта без *theme*'ы.

`jekyll build` – однократно сгенерировать код сайта и вывести в каталог `_site`.

`jekyll serve` – автоматически генерировать код сайта при изменении исходных файлов, вывести его в каталог `_site`, запустить веб-сервер на `http://localhost:4000`.

`jekyll doctor` – вывести проблемы с *configuration*



### Шаблоны и liquid

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

#### Tag

##### Стандартные *Liquid tag*'и

https://shopify.github.io/liquid/tags/

###### assign

Задать переменную

```
{% assign <name> = <value> %}
{% assign variable = false %}
```

###### if

```
{% if <condition> %}
  Body
{% endif %}
```

###### for

```
{% for <variable> in <array> %}
  {{ <variable> }}
{% endfor %}
```

##### Дополнительные *Jekyll tag*'и

https://jekyllrb.com/docs/liquid/tags/

###### link

Сгенерировать постоянную ссылку на *post*, *page*, *document* или *file*. Указывается путь относительно *root directory*, расширение файла исходное (`.md`, а не `html`)

```liquid
{% link _posts/2016-07-26-name-of-post.md %}
```

###### post_url

Сгенерировать постоянную ссылку на *post*

```liquid
{% post_url 2010-07-21-name-of-post %}
```



#### Filter

##### where

Отфильтровать массив. Выбрать элементы, у которых выполняется `el.property == value`.

```
{% assign <variable> = <array> | where: "<property>", "<value>" %}
{% assign author = site.authors | where: 'short_name', page.author %}
```

##### first

Выбрать первый элемент в массиве. Часто используется с `where` фильтром, чтобы найти один элемент:

```
{% assign <variable> = <array> | first}
{% assign author = site.authors | where: 'short_name', page.author | first %}
```

### Page

Используется для контента, который не привязан к дате (как *post*' ы) и который не является группой (как *collection*'ы). 

*Page*'s копируются в целевую папку сайта `_site`. 

Page's могут помещаться в корневую папку или подпапки. Структура подпапок сохраняется в `_site`.

Возможны типы файлов:

- HTML-файлы
- `.md` файлы – при сборке конвертируются в `.html`

### Front matter

Для того чтобы *Jekyll* обрабатывал файл (*object*'ы, *tag*'и, *filter*'ы), в начале файла должен быть размещен *front matter* (возможно пустой). Однако *post*'ы обрабатываются всегда, даже если не содержат *front matter*.

Это YAML код, размещенный между символами `---`.

```
---
  #####
---
```

В нем задаются:

- пользовательские переменные (например, `title`), которые будут доступны как свойство объекта `page` (например, `page.title`).
- Специальные переменные:
  - `layout` – *layout* для страницы. Задается без *extension*'а.
  - `permalink` – URL для страницы, если нужно изменить URL по умолчанию.
  - `published: false` – не публиковать страницу в `_site`, аналог *draft*
  - и [специальные переменные для post'а](#специальные-переменные-для-post- а)
  - [*Category* и *Tag* для *post*'ов](category-и-tag-для-post-ов)

```
---
layout: post   # Layout для одиночного page, document'а, post'а, draft

title: My blog # Пользовательские переменные, которые будут доступны в шаблонах
---
{{ page.title }}
```

Объявленные пользовательские переменные будут доступны:

- ниже в этом файле
- во всех *layout*'ах, в которые вставляет текущая страница. Например, переменная `title` будет доступна в *layout* (`post.html`).
- во всех include'ах, которые включаются в текущую страницу.



Могут быть [установлены значения front matter]() по умолчанию.





### Layout

*Layout* (макет) – удобны для общих шаблонов, внутрь которых вставляется контент более мелких шаблонов. Например, макет для общего шаблона всего сайта. Располагаются в папке `_layouts`.

В *layout* всегда определена переменная `content`. В эту переменную подставляется контент страницы, для который установлен этот *layout*. 

*Layout* устанавливается в [front matter](#front-matter).

### Include

*Include* (включение) – шаблон (блок), который может включаться в другие шаблоны. Размещается в папке `_includes`. 

Для вставки *include* в шаблон необходимо использовать `include` *tag*.

```
{% include navigation.html %}
```

### Data

*Data* (данные) – файл данных в формате *YAML*, *JSON* или *CSV*. Размещается в папке `_data`.

Данные из файла `_data/xxx.yml` будут доступны в переменной `site.data.xxx`.

<u>Пример перебора:</u>

### Asset

Asset (ресурс).  Располагаются в папке `assets`. 

В папку помещают: 

- `css`
- `js`
- картинки
- файлы (вроде `.pdf`)

```
.
├── assets
|   ├── css
|   ├── images
|   └── js
```

Папка `assets` просто копируется в папку сайта.

Пример ссылки:

```html
<link rel="stylesheet" href="/assets/css/styles.css">
```

### Post'ы и Collection

<u>Collection</u>

*Collection* предназначены для сгруппированного контента (например, пользователи).

*Collection*'ы (кроме *post*'ов) должны быть [определены в *configuration file*](#Настройка-collection).

*Document*'ы сортируются по дате создания файла (или путям?). Изменить сортировку можно в [configuration](#configuration).

<u>Post'ы</u>

*Post*'ы (для блога) следует рассматривать как *collection* с именем `posts`

Post'ы сортируются по `date` переменной

<u>Общие особенности:</u>

- 
- *post*'ы состоят из *post*'ов, а *collection* – из *document*'ов
- ресурсы (картинки, файлы) размещают в папке `assets`



#### Файл с содержанием

Общие особенности:

- *post*'ы и *document*'ы размещаются в папке `_<collection_name>` (*post*'ы соответственно в папке `_posts`)
- имеют формат *markdown* (`.md`) или `.html`.

Имя файла:

- *document* – любое имя

- *post* – имя имеет вид:

  ```
  <year>-<month>-<day>-<title>.<ext>
  2018-08-20-bananas.md
  ```

Шаблон задается через переменную `layout` в [front matter](#front-matter).

#### Шаблон одиночного document'а (post'а)

Доступны переменные:

- `page.<variable>` – переменная `variable` из *front matter*.

- `page.url` – URL страницы *document*'а (*post*'а)

- `page.excerpt` – аннотация текста. 

  По умолчанию, первый абзац текста. Разделитель для аннотации можно настроить в переменной `excerpt_separator` в *front_matter* или `_config.yml`.

- `content` –  содержимое *document*'а (*post*'а)

##### Специальные переменные для *post*'а

Для *post*'ов предопределены переменные:

- `page.date` –  `date` из имени файла. Может быть переопределена в формате `YYYY-MM-DD HH:MM:SS`
- `page.title` – `title`  из имени файла

##### Специальные переменные для collection

- `page.date` – какая-то дата *collection*?

#### Шаблон списка document'ов (post'ов)

*Document*'ы (*post*'ы) доступны через переменную `site.<collection_name>` (`site.posts`). Доступ ко всем коллекциям сразу можно получить через массив `site.collections`.

Пример перебора:

```
{% for document in site.<collection_name> %}
    {{ document.xxx }}
{% endfor %}
```

Доступные свойства `document` (и `post`):

- `document.content` – содержимое *document*'а (*post*'а) (не отрендеренное?)
- `document.output` – содержимое *document*'а (*post*'а) (отрендеренное?)
- `document.<variable>` – переменная `variable` из *front matter*.
- и другие переменные `page.xxx` из [шаблона одиночного document'а (post'а)]() доступны как `document.xxx` 

#### *Category* и *Tag* для *post*'ов

В *collection*'ях не работает.

Отличие:

-  Все *category*'ии включаются в URL *post*'а (например, `/<category1>/<category2>/<year>/<month>/<day>/<title>.html`). 
- *Tag*'и не включаются в URL *post*'a

<u>Файл с содержанием</u>

*Category*'и и *tag*'и указываются в *front matter*:

```liquid
categories: [blog, travel]
tags: [hot, summer]
```

<u>Шаблон списка</u>

Jekyll автоматически собирает из *post*'ов:

-  *category*'ии – в массив `site.categories`
- *tag*'и – в массив `site.tags` 

Структура массивов `site.categories` и `site.tags`:

```
[
	0 => <category_name> / <tag_name>
	1 => [  # массив post'ов
		<post1>,
		<post2>,
		...
	]
]
```

Пример построение списка:

```liquid
{% for category in site.categories %}
	{{ category[0] }}                     # category_name
  	{% for post in category[1] %}         # posts
  		{{ post.xxx }}					
    {% endfor %}
{% endfor %}
```

#### Draft

Draft (черновик). Располагаются в папке `_drafts`. В названии дата не указывается: `<title>.md`. Дата *draft*'а – дата изменения его файла.

При обычном запуске `jekyll build` и `jekyll serve` – *draft'ы* не копируются в папку сайта `_site` и вообще нигде не фигурируют на сайте.

При запуске с ключом `--drafts` *draft*'ы рассматриваются как обычные *post*'ы: помещаются в массив `site.posts` и копируются в те же подпапки в папке сайте `_site`. 

### Configuration

Размещается в файле `./_config.yml`.

После изменения конфигурации требуется рестарт *jekyll*.

Настройки:

- `collections_dir: <dir>` – папка для хранения всех *collection's*, *post*'ов, *draft's*. Пути будут выглядеть `<dir>/_<collection_name>`.
- 

#### Определение collection

```
collections:
  <collection_name>:
    [ output: true ] 
    [ sort_by: <variable_name> ]
    [ order: 
    	- document1.md
    	- document2.md
    ]
```

<u>Опции:</u>

- `output: true` – генерировать шаблон одиночного *document*'а
- `sort_by` – сортировать *document*'ы  по значению указанной *variable*.
- `order` – задает явный порядок *document*'ов 

#### Front matter по умолчанию

Значения по умолчанию для переменных во *Front matter* для *page*, *document*'ов, *post*'в задается в `defaults` свойстве:

```
defaults:
  # front matter для document'ов
  - scope:
      path: ""
      type: "<collection_name>"
    values:
      layout: "author"
  
  # front matter для post'ов
  - scope:
      path: ""
      type: "posts"
    values:
      layout: "post"
  
 
  
  # front matter для остальных
  - scope:
      path: ""
    values:
      layout: "default"
```

Параметры:

- `scope` – область 

  Возможные опции:

  - `path` (обязательный) – путь к файлам.

    ```
     # front matter для page's в папке `projects/`
     scope:
       path: "projects"
       type: "pages"
     values:
        # ...
    ```

    Значение `path: ""` применяется ко всем файлам. `path` может содержать подстановочный символ `*`.

  - `type` (необязательный) – тип файлов

    Могут применяться следующие типы: `pages`, `posts`, `drafts` и *collection name*.

- `values` – значения по умолчанию

### Plugin

<u>Подключение</u>

Добавить *plugin* в группу `jekyll_plugins` в `Gemfile`:

```
group :jekyll_plugins do
  gem 'jekyll-sitemap'
  # ...
end
```

Добавить строку в *configuration*:

```yaml
plugins:
  - jekyll-sitemap
```

#### `jekyll-sitemap`

Автоматически генерирует файл `sitemap.xml` – карта сайта для роботов поисковых систем. 

Настройка не требуется

#### `jekyll-feed` 

Генерирует RSS-ленту `./feed.xml` из *post*'ов.

Необходимо в основной *layout* в блок `<head>` поместить `{% feed_meta %}` тег. В этом месте будет вставлен метает со ссылкой на RSS-ленту.

```html
<head>
		...
    {% feed_meta %}
</head>
```

#### `jekyll-seo-tag`

Добавляет SEO метатеги для поисковых роботов (вроде `og:title`).

Необходимо в основной *layout* в блок `<head>` поместить `{% seo %}` тег. В этом месте будет вставлены SEO метатеги.

### Подсветка кода

Подсветка кода реализована через библиотеку [Rouge](http://rouge.jneen.net/).

Для обработки `.md` файла необходимо наличие *front matter*.

Код должен быть окружен стандартными для *Markdown* символами ` ``` `  или специальными *tag*'ами:

```liquid
{% highlight php  %}
	  # ...
{% endhighlight %}
```

[Список поддерживаемых языков](https://github.com/jayferd/rouge/wiki/List-of-supported-languages-and-lexers)

Чтобы отключить обработку специальных символов *Jekyll* в документе, необходимо указать в front matter:

```liquid
render_with_liquid: false
```

Отключаются и теги `{% highlight %}`, поэтому можно использовать только ` ``` `.

Стили подсветки необходимо взять от библиотеки [Pygments](http://jwarby.github.io/jekyll-pygments-themes/languages/javascript.html). И загруженный файл `.css` необходимо подключить в `main.css`:

```html
@import "tango.css";
```



### Документация

Глобальные переменные – https://jekyllrb.com/docs/variables/

