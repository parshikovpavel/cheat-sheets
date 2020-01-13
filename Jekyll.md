### Команды

`jekyll new PATH` – создать заготовку сайта с использование `Minima` *them*e'ы 

`jekyll new PATH --blank` – создать заготовку сайта без *theme*'ы.

`jekyll build` – построить сайт и вывести его код в каталог `_site`.

`jekyll serve` – перестроить сайт (если были изменены файлы кода), вывести его код в каталог `_site`, запустить веб-сервер на `http://localhost:4000`.

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

Стандарные *Liquid tag*'и – https://shopify.github.io/liquid/tags/

Дополнительные *Jekyll tag*'и – https://jekyllrb.com/docs/liquid/tags/

##### assign

Задать переменную

```
{% assign <name> = <value> %}
{% assign variable = false %}
```



##### if

```
{% if <condition> %}
  Body
{% endif %}
```

##### for

```
{% for <variable> in <array> %}
  {{ <variable> }}
{% endfor %}
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

### Post'ы и Collection

Общие особенности:

- *post*'ы (для блога) следует рассматривать как *collection* с именем `posts`
- *post*'ы состоят из *post*'ов, а *collection* – из *document*'ов

Collection'ы (кроме *post*'ов) должны быть [определены в *configuration file*](#Настройка-collection).







#### Файл с содержанием

Общие особенности:

- *post*'ы и *document*'ы размещаются в папке `_<collection_name>` (*post*'ы соответственно в папке `_posts`)
- имеют формат *markdown* (`.md`).

Имя файла:

- *document* – любое имя

- *post* – имя имеет вид:

  ```
  <date>-<title>.<ext>
  2018-08-20-bananas.md
  ```

<u>Front matter</u>

Может содержать *front matter* вида:

```
---
layout: post   # Шаблон одиночного document'а (post'а)

author: xxx    # Пользовательские переменные, которые будут доступны в шаблонах
---
```

Либо может использоваться [front matter по умолчанию]() (часто для переменной `layout`).

#### Шаблон одиночного document'а (post'а)

Доступны переменные:

- `page.<variable>` – переменная `variable` из *front matter*.
- `page.url` – URL страницы *document*'а (*post*'а)
- `page.excerpt` – первый параграф текста
- `content` –  содержимое *document*'а (*post*'а)

Для *post*'ов предопределены переменные:

- `page.date` –  `date` из имени файла
- `page.title` – `title`  из имени файла

#### Шаблон списка document'ов (post'ов)

*Document*'ы (*post*'ы) доступны через переменную `site.<collection_name>` (`site.posts`).

Пример перебора:

```
{% for document in site.<collection_name> %}
    {{ document.xxx }}
{% endfor %}
```

Доступные свойства `document` (и `post`):

- `document.content` – содержимое *document*'а (*post*'а)
- `document.<variable>` – переменная `variable` из *front matter*.
- и другие переменные `page.xxx` из [шаблона одиночного document'а (post'а)]() доступны как `document.xxx` 



- 

### Configuration

Размещается в файле `./_config.yml`.

После изменения конфигурации требуется рестарт *jekyll*.

#### Настройки collection

Определение *collection*:

```
collections:
  <collection_name>:
    [ output: true ] 
```

<u>Опции:</u>

- `output: true` – включить шаблон одиночного *document*'а:

#### Front matter по умолчанию

Front matter по умолчанию для page, document'ов, post'в задается так:

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

### Документация

Глобальные переменные – https://jekyllrb.com/docs/variables/

