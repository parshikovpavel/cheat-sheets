После изучения всех теоретиских тем посмотреть пошаговое руководство https://jekyllrb.com/docs/step-by-step/01-setup/

Остановился https://jekyllrb.com/docs/collections/ (Permalinks)

# Установка bundler и jekyll

Установить свежую версию ruby через homebrew

```bash
brew install ruby
```

Добавить пути в PATH в файле `~/.bash_profile`:

```
export PATH=/usr/local/opt/ruby/bin:$PATH
export GEM_HOME=$(ruby -e 'puts Gem.user_dir')
export PATH=$GEM_HOME/bin:$PATH
```

Проверить пути через :

```bash
gem env
```

Установить в `USER INSTALLATION DIRECTORY`.  

```bash
gem install --user-install bundler jekyll
```

# Общее

*Jekyll* –  это *gem*. 

Список *gem*'ов, используемых на сайте, описывается в файле `Gemfile` (подробнее [1](Ruby.md#gemfile)). 



# Command line

Рекомендуется использовать перед всеми командами `bundle exec`, иначе Bundler и Gemfile использоваться не будут (точно??? возможно нужно только использования gem'ов из vendor):

```bash
bundle exec jekyll serve
```

После установки gem'а jekyll появляется возможность запускать на исполнение команду `jekyll`.

Структура команд:

```bash
jekyll command [argument] [option] [argument для option]
```

Полный список *options* ([1](https://jekyllrb.com/docs/configuration/options/#build-command-options)) 

## `jekyll new`

Cоздать заготовку сайта с использование `Minima` *them*e'ы 

```bash
bundle exec jekyll new PATH
```

Cоздать заготовку сайта без *theme*'ы.

```bash
bundle exec jekyll new PATH --blank
```



## `jekyll build`

Однократно сгенерировать код сайта и вывести в каталог `_site`.

```bash
bundle exec jekyll build
```

Используется для генерации сайта для продакшена.



## `jekyll serve`

Автоматически генерировать код сайта при изменении исходных файлов, вывести его в каталог `_site`, запустить веб-сервер на `http://localhost:4000`.

```bash
bundle exec jekyll serve
```

Используется для локальной разработки.

Автоматически обновлять страницу при каждом изменении, которое вы вносите в исходные файлы:

```bash
bundle exec jekyll serve --livereload
```





Если возникает ошибка с `webrick`, то нужно в папке с сайтом:

```bash
bundle add webrick
```



## `jekyll doctor`

Вывести проблемы с *configuration*

```bash
bundle exec jekyll doctor
```





# Создание темы

Можно устанавливать jekyll и его зависимости gem'ы:

- системную *gem installation directory* (по умолчанию, проще)
- прямо в project directory (как composer по умолчанию, в папке `./vendor/bundle/`, сложнее, описано в [1](https://jekyllrb.com/tutorials/using-jekyll-with-bundler/)) 





# Шаблоны и liquid

Язык шаблонов – *Liquid*.

## Типы данных

- `array`

  К элементу массива можно получить доступ по ключу `<array>[<key>]`.

## Типы блоков

### Object

*object* (объект) – выводит значение выражения. Используются символы `{{`и `}}`.

  ```html
  {{ <object> }}
  {{ page.title }}
  ```

Чтобы *jekyll* обрабатывал блоки на языке *Liquid*, в начале шаблона необходимо разместить *front matter*.

### Tag

*tag* (тег) – для управления потоком выполнения. Используются символы `{%` и `%}` Например, условный оператор

 ```
  {% if page.show_sidebar %}
    ...
  {% endif %}
 ```

#### Стандартные *Liquid tag*'и

https://shopify.github.io/liquid/tags/

##### `assign`

Задать переменную

```liquid
{% assign <name> = <value> %}
{% assign variable = false %}
```

##### `if`

```
{% if <condition> %}
  Body
{% endif %}
```

##### `for`

```
{% for <variable> in <array> %}
  {{ <variable> }}
{% endfor %}
```

##### `capture`

Перенаправить вывод в переменную.

```
{% capture <variable> %}
<Some output>
{% endcapture %}
```

#### Дополнительные *Jekyll tag*'и

https://jekyllrb.com/docs/liquid/tags/

##### link

Сгенерировать постоянную ссылку на *post*, *page*, *document* или *file*. Указывается путь относительно *root directory*, расширение файла исходное (`.md`, а не `html`)

```liquid
{% link _posts/2016-07-26-name-of-post.md %}
```

##### post_url

Сгенерировать постоянную ссылку на *post*

```liquid
{% post_url 2010-07-21-name-of-post %}
```



### Filter

*filter* (фильтр) – для изменения значения *object*'а. *Object* и *filter* отделяются символом `|`.

```
{{ <object> | <filter> }}
{{ "hi" | capitalize }}
```

Jekyll фильтры – https://jekyllrb.com/docs/liquid/filters/. 

Стандарные *Liquid filter*'ы – https://shopify.github.io/liquid/filters/ или https://jekyllrb.com/docs/liquid/filters/#standard-liquid-filters

#### where

Отфильтровать массив. Выбрать элементы, у которых выполняется `el.property == value`.

```
{% assign <variable> = <array> | where: "<property>", "<value>" %}
{% assign author = site.authors | where: 'short_name', page.author %}
```

#### first

Выбрать первый элемент в массиве. Часто используется с `where` фильтром, чтобы найти один элемент:

```
{% assign <variable> = <array> | first}
{% assign author = site.authors | where: 'short_name', page.author | first %}
```

#### `default`

Возвращает default значение, если переменная не определена (аналог `??`): 

```liquid
{{ <variable> | default: <default_value> }}
```



# Variable

| Глобальные объекты | Описание                                                     |
| ------------------ | ------------------------------------------------------------ |
| `site`             | Данные всего сайта                                           |
| `page`             | Данные текущей страницы + *front matter* переменные, установленные на *page*'s |
| `layout`           | Данные текущего *layout* + *front matter* переменные, установленные в *layout*'ах |
| `content`          | Доступна в *layout*, контент страницы, для который установлен этот *layout*. |
| `paginator`        | Данные пейджинатора                                          |

Глобальные переменные – https://jekyllrb.com/docs/variables/



# Page

Используется для контента, который не привязан к дате (как *post*' ы) и который не является группой (как *collection*'ы). 

*Page*'s копируются в целевую папку сайта `_site`. 

Page's могут помещаться в корневую папку или подпапки. Структура подпапок сохраняется в `_site`.

Возможны типы файлов:

- HTML-файлы
- `.md` файлы – при сборке конвертируются в `.html`

Пример:

```
─ about.md          # => http://example.com/about.html
├── documentation     # folder containing pages
│   └── doc1.md       # => http://example.com/documentation/doc1.html
├── design            # folder containing pages
│   └── draft.md      # => http://example.com/design/draft.html
```

Если во *front matter* разместить *permalink* ([1](#permalink)), то в *built sit*e этот файл будет размещен в том месте, куда указывает *permalink* (а не в исходном).

# Front matter

*Front matter* предназначен для указания конфигурации *page*'s, *post*'s, .... Например, *layout, title, date/time*. 



Для того чтобы *Jekyll* обрабатывал файл (*object*'ы, *tag*'и, *filter*'ы), в начале файла должен быть размещен *front matter* (даже пустой). Если front matter не указан, то файл рассматривается как *static file*. Однако *post*'ы обрабатываются всегда, даже если не содержат *front matter*.

*Front matter* записывается в YAML формате, и размещается между символами `---`.

В нем задаются:

- *custom variable*'s (пользовательские переменные). Переменные, объявленные на страницах, будут доступны как свойство объекта `page` (например, `page.title`). Переменные, объявленные в *layout*, будут доступны как свойство объекта `layout` (например, `layout.title`).
- *Predefined variable*'s (специальные переменные):
  
  - `layout` – *layout* для страницы. Задается без *file extension*'а (например, без `.html`) 
  
    Особые значения
  
    - `null` – создание файла без использования *layout file*. Настройка перезаписывается, если файл – *post/document* и имеет *layout* заданный в *front matter defaults*.
    -  `none` – для *post/document* - создание файла без использования *layout file*, не зависимо от *front matter defaults*. Для *page* - вызовет использование *layout* с именем `none`.
  
  - `permalink` – URL для страницы, если нужно изменить URL по умолчанию.
  
  - `published: false` – не публиковать страницу при генерации сайта в `_site`, аналог *draft*
  
  - и [специальные переменные для post'а](#специальные-переменные-для-post- а)
  
  - [*Category* и *Tag* для *post*'ов](category-и-tag-для-post-ов)

<u>Пример:</u>

```
---
layout: post   # Layout для одиночного page, document'а, post'а, draft

title: My blog # Пользовательские переменные, которые будут доступны в шаблонах
---
{{ page.title }}
```

<u>Пример пустого *front matter*:</u>

```
---
---
```



Объявленные пользовательские переменные (на страницах и *layout*'ах) будут доступны:

- ниже в этом файле
- во всех *layout*'ах, в которые вставляет текущая страница. Например, переменная `title` будет доступна в *layout* (`post.html`).
- во всех include'ах, которые включаются в текущую страницу.



Могут быть [установлены значения front matter by defaut](#front-matter-by-default).





# Layout

*Layout* (макет) – удобны для общих шаблонов, внутрь которых вставляется контент более мелких шаблонов. Например, макет для общего шаблона всего сайта. Располагаются в папке `_layouts`. Рекомендуется базовый *layout* назвать `default.html`.

В *layout* всегда определена переменная `content`. В эту переменную подставляется контент страницы, для который установлен этот *layout*. 

*Layout* устанавливается в [front matter](#front-matter).

Пользовательские переменные, объявленные в *front matter layout*'а будут доступны как свойство объекта `layout` (например, `layout.title`).

# Include

*Include* (включение) – шаблон (блок), который может включаться в другие шаблоны. 

<u>Включение из папки `_includes`</u>

Как правило, *include's* размещаются в папке `_includes`. Для вставки используется `include` *tag*.

```
{% include navigation.html %}
```

<u>Включение из любой папки, относительно текущей</u>

Можно включить *include* из любой папки, указав относительный путь относительно текущего файла. Для вставки используется `include_relative` тег.

```liquid
{% include_relative somedir/footer.html %}
```

Можно имя *include* передать в переменной:

```
{% include {{ <variable> }} %}
```

<u>Передача параметров</u>

Параметры в *include* передаются так:

```liquid
{% include <include_filename> <param1>="<value1>" <param2>="<value2>" %} # значение-строка
{% include <include_filename> <param>=<variable> %} # значение-переменная
```

Параметры доступны внутри include'а через объект `include`:

```liquid
{{ include.<param> }}
```



# Data

*Data* (данные) – файл данных в формате *YAML*, *JSON* или *CSV*. Размещается в папке `_data`.

Данные из файла `_data/<name>.yml` будут доступны в переменной `site.data.<name>`.

Пример перебора *data* файла, в котором хранится *list*'а:

```liquid
{% for <element> in site.data.<name> %}
  {{ <element>.<property> }}
{% endfor %}
```

Получение доступа к значению элемента по `key` ключу, если в *data* файле хранится *map*:

```liquid
{% assign <name> = site.data.<name>[<key>] %}
```



<u>Подпапки</u>

Можно сгруппировать несколько *data* файлов в подкаталоге `_data/<subfolder>/<name>.yml`.  

Пример перебора *data* файлов в подкаталоге, в каждом файле хранится *map*:

```liquid
{% for <hash> in site.data.<subfolder> %}
{% assign <data> = <hash>[1] %}
   {{ <data>.<key> }}
{% endfor %}
```



# Asset

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

Пример картинки:

```markdown
![My helpful screenshot](/assets/screenshot.jpg)
```

Пример ссылки на файо в тексте:

```markdown
... you can [get the PDF](/assets/mydoc.pdf) directly.
```



# Post'ы и Collection

<u>Collection</u>

*Collection* предназначены для сгруппированного контента.

Примеры:

- пользователи
- организации
- ...

*Collection*'ы (кроме *post*'ов) должны быть [определены в *configuration file*](#Настройка-collection).

*Document*'ы сортируются по дате создания файла (или путям?). Изменить сортировку можно в [configuration](#configuration).

<u>Post'ы</u>

- *Post*'ы (для блога) следует рассматривать как *collection* с именем `posts`
- *Post*'ы сортируются по `date` переменной
- *Post*'ы должны начинаться с *front matter* (а может и не обязательно???). Можно использовать пустой *front matter*.

<u>Общие особенности:</u>

- *post*'ы состоят из *post*'ов, а *collection* – из *document*'ов
- ресурсы (картинки, файлы) размещают в папке `assets`



## Файл с содержанием

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

## Шаблон одиночного document'а (post'а)

Доступны переменные:

- `page.<variable>` – переменная `variable` из *front matter*.

- `page.url` – URL страницы *document*'а (*post*'а) (для *document* страница по указанному URL, будет доступна если стоит `output: true` в *Configuration file*)

- `page.excerpt` – *excerpt* (антотация) для *post* (только для *post*???) ([1](#excerpt))

- `content` –  содержимое *document*'а (*post*'а). Сюда помещается все, что находится в файле после *front matter*.

### Специальные переменные для *post*'а

Для *post*'ов предопределены переменные:

- `page.date` –  `date` из имени файла. Может быть переопределена в *front matter* в формате `YYYY-MM-DD HH:MM:SS`
- `page.title` – `title`  из имени файла
- `hidden: true` – не выводить *post* при пагинации.
- `category`, `categories` – [link](#category-и-tag-для-post-ов)
- `tag`, `tags` – [link](#category-и-tag-для-post-ов)

### Специальные переменные для collection

- `page.date` – какая-то дата *collection*?

## Шаблон списка document'ов (post'ов)

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

## *Category* и *Tag* для *post*'ов

В *collection*'ях не работает (??? сейчас)

Отличие:

-  Все *category*'ии (написано, могут быть включены??) включаются в URL *post*'а (generated URL???) (например, `/<category1>/<category2>/<year>/<month>/<day>/<title>.html`). 
- *Tag*'и не включаются в URL *post*'a

<u>Файл с содержанием</u>

*Category*'и и *tag*'и указываются в *front matter*. Можно указывать как массив `[]` или через пробел ` `.

```yaml
tag: hot                      // Одиночный тег
tags: [hot, summer]           // Множественные теги 
tag: classic hollywood        // аналогично - tag: "classic hollywood"
tags: classic hollywood       // аналогично - tags: ["classic", "hollywood"]
```

```yaml
category: classic hollywood   // аналогично - tag: "classic hollywood"
categories: classic hollywood // аналогично - categories: ["classic", "hollywood"]
```

*Category* также можно определять по пути к файлу *post*'а. Любой каталог выше (!!!) `_post` будет рассматриваться как категория. 

Например, если *post* находится по пути `movies/horror/_posts/2019-05-21-bride-of-chucky.markdown`, то `movies` и `horror` автоматически учитываются как *category* для этого *post*'а. Если у *post*'а есть *category*'s в *front matter*, то они просто мерджатся. Поэтому *post* с указанными выше *categories* во *front matter* будет иметь URL `movies/horror/classic/hollywood/2019/05/21/bride-of-chucky.html`.

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

## Excerpt

Excerpt – аннотация текста, доступна как `page.excerpt`.

По умолчанию, первый абзац текста. Разделитель для аннотации можно настроить в переменной `excerpt_separator` в *front_matter* или `_config.yml`.

```html
---
excerpt_separator: <!--more-->
---

Excerpt with multiple paragraphs

Here's another paragraph in the excerpt.
<!--more-->
Out-of-excerpt
```



## Draft

Draft (черновик). Располагаются в папке `_drafts`. В названии дата не указывается: `<title>.md`. Дата *draft*'а – дата изменения его файла.

При обычном запуске `jekyll build` и `jekyll serve` – *draft'ы* не копируются в папку сайта `_site` и вообще нигде не фигурируют на сайте.

При запуске с ключом `--drafts` *draft*'ы рассматриваются как обычные *post*'ы: помещаются в массив `site.posts` и копируются в те же подпапки в папке сайте `_site`. 

В качестве `date` – используется дата изменения файла.

# Permalink

*Permalink* (постоянная ссылка)

Способы задания форматы *permalink*:

- *Permalink* в *configuration*

  Задает глобальный *permalink* в ключе `permalink`, в нем следует использовать *placeholder*'ы.

  ```yaml
  permalink: /<placeholder1>/<placeholder2>/<placeholder3>:output_ext
  permalink: /:categories/:year/:month/:day/:title:output_ext
  ```

  *Placeholder*'ы, которые не имеют смысла для текущей страницы, пропускаются.

  Можно использовать [встроенные форматы](https://jekyllrb.com/docs/permalinks/#built-in-formats).

- *Permalink* в *front matter*

  Задает *permalink* для текущей страницы, переопределяет глобальный *permalink*. Как правило не использует *placeholder*'ы.

  ```yaml
  permalink: /about/
  ```

- *Permalink* для конкретной коллекции. Следует использовать *placeholder*'ы.

  Задается при [определении collection](#определение-collection).



[Перечень доступных *placeholder*'ов](https://jekyllrb.com/docs/permalinks/#placeholders).

# Configuration

Все параметры – https://jekyllrb.com/docs/configuration/

Размещается корневом каталоге в файле `/_config.yml`.

После изменения *configuration* требуется рестарт *jekyll*, т.к. значения из *configuration file* считываются один раз.

Настройки:

- `collections_dir: <dir>` – папка для хранения всех *collection's*, *post*'ов, *draft's*. Пути будут выглядеть `<dir>/_<collection_name>`.
- `permalink` – глобальный формат [permalink](#permalink).



## Пользовательские настройки

Настройка с любым именем `<name>` будет доступна через глобальную переменную `site`:

```liquid
{{ site.<name> }}
```



## Определение collection

В формате *list* (без параметров):

```yaml
collections:
  - <collection_name>
  - <collection_name>
```

В формате *map* (с параметрами):

```
collections:
  <collection_name>:
    [ output: true ] 
    [ sort_by: <variable_name> ]
    [ order: 
    	- document1.md
    	- document2.md
    ]
    [ permalink: <format> ]
```

<u>Опции:</u>

- `output: true` – генерировать (*render*) страницу (в папке `_site`???) для каждого *document* в *collection*. По умолчанию, `output: false`.
- `sort_by` – сортировать *document*'ы  по значению указанной *variable*.
- `order` – задает явный порядок *document*'ов 
- `permalink` – *permalink* конкретной коллекции.

## Front matter by default

Предназначен для задания значений по умолчанию для переменных во *Front matter* для *page*, *document*'ов, *post*'ов.

Могут задаваться *predefined variable*'s:

- `layout`
- `category`
- ...

И также custom variable's: 

- `author`,
-  ...

Использует `defaults` свойство:

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
  
	# front matter для collection's
	- scope:
      path: ""
      type: "my_collection" 
    values:
      layout: "default"
  
  # front matter для остальных
  - scope:
      path: ""
    values:
      layout: "default"
```

Параметры:

- `scope` – область, где будет применяться значение

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

    Значение `path: ""` применяется ко всем файлам. 

    `path` может содержать подстановочный символ `*`. Например, `path: "section/*/special-page.html"`.

  - `type` (необязательный) – тип файлов

    Могут применяться следующие типы:
    
    -  `pages`
    - `posts`
    - `drafts`
    - *collection name*.

- `values` – значения по умолчанию для этого scope

### Приоритеты

Порядок приоритетов следующий (от высокого к низкому):

1. Front matter вверху post, page,...
2. Front matter by default. 
   - Более высокий приоритет имеют те `values`, для которых указан более конкретный `path`. Например, `path: "projects"` более конкретный, чем `path: ""`.
   - (насчет более приоритетных `type`???, наверное с `type` более приоритетно, чем без `type`)

# Plugin

[Список *plugin*'ов](https://pages.github.com/versions/), поддерживаемых *Github Pages*. Если нужен plugin, который не поддерживается *Github pages*, то нужно самостоятельно генерировать сайт и выкладывать уже сгенерированный код на *GitHub*.

## Подключение plugin'а

Добавить *plugin* в группу `jekyll_plugins` в `Gemfile`, в этом случае их можно не указывать в ключе `plugins` *configuration*'ии:

```
group :jekyll_plugins do
  gem 'jekyll-sitemap'
  # ...
end
```

Установить через *bundler*:

```bash
bundle install
```



## `jekyll-sitemap`

Автоматически генерирует файл `sitemap.xml` – карта сайта для роботов поисковых систем. 

Настройка не требуется

## `jekyll-feed` 

Генерирует RSS-ленту `./feed.xml` из *post*'ов.

Необходимо в основной *layout* в блок `<head>` поместить `{% feed_meta %}` тег. В этом месте будет вставлен метает со ссылкой на RSS-ленту.

```html
<head>
		...
    {% feed_meta %}
</head>
```

## `jekyll-seo-tag`

Добавляет SEO метатеги для поисковых роботов (вроде `og:title`).

Необходимо в основной *layout* в блок `<head>` поместить `{% seo %}` тег. В этом месте будет вставлены SEO метатеги.

## `jekyll-paginate`

Предназначен для пагинации.

Чтобы включить пагинацию, необходимо добавьте в *configuration* ключ `paginate` – количество элементов на странице:

```
paginate: <int>
```

Пагинация может выполняться только в одном файле с именем `index.html`. Этот файл может располагаться в корне или каком-то подкаталоге. В параметре `paginate_path` *configuration*'ии необходимо указать путь к этому файлу и шаблон страницы пагинации. В пути можно использовать плейсхолдер `:num`. Например, для файла `./<path>/index.html` настройка может иметь вид:

```yaml
paginate_path: "/<path>/page:num/"
```

В этом случае ссылки на страницы будут выглядеть:

- 1 страница – `blog/index.html`
- `<N>` страница – `blog/page<N>/index.html`

НЕ поддерживается пагинация для *category*'ий, *tag*'ов, *collection*'ий. Т.е. plugin просто итерирует по списку *post*' ов (за исключением, тех у которых `hidden: true`). Для расширенных возможностей следует использовать *plugin* [ jekyll-paginate-v2](https://github.com/sverrirs/jekyll-paginate-v2), но он не поддерживается *Github pages*.

В шаблонах данные пагинации доступны через объект `paginator`. [Доступные свойства `paginator`](https://jekyllrb.com/docs/pagination/#liquid-attributes-available)

<u>Вывод списка постов:</u>

```liquid
<!-- Post list -->
{% for post in paginator.posts %}
	{{ post.<property> }}
{% endfor %}
```

<u>Ссылка на предыдущую страницу:</u>

```liquid
{% if paginator.previous_page %}
    <a href="{{ paginator.previous_page_path }}" class="previous">
      Previous
    </a>
{% else %}
    <span class="previous">Previous</span>
{% endif %}
```

<u>Ссылка на следующую страницу:</u>

```liquid
{% if paginator.next_page %}
    <a href="{{ paginator.next_page_path }}" class="next">Next</a>
{% else %}
    <span class="next ">Next</span>
{% endif %}
```

<u>Ссылка на текущую страницу:</u>

```liquid
<span class="page_number ">
    Page: {{ paginator.page }} of {{ paginator.total_pages }}
</span>
```

<u>Список ссылок на все страницы:</u>

```liquid
{% for page in (1..paginator.total_pages) %}
    {% if page == paginator.page %}
      <em>{{ page }}</em>
    {% elsif page == 1 %}
      <a href="{{ paginator.previous_page_path | relative_url }}">{{ page }}</a>
    {% else %}
      <a href="{{ site.paginate_path | relative_url | replace: ':num', page }}">{{ page }}</a>
    {% endif %}
  {% endfor %}
```





# Подсветка кода

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

# Static file

*Static file* – все файлы, которые не имеют *front matter*.

`site.static_files` – массив данных *static file*'ов. Можно перебрать массив и получить данные о *static file*'ах.

# Общая структура папок

```
.
├── _config.yml       # Configuration
├── _data
|   └── <name>.yml
├── _drafts
|   └── <name>.md
├── _includes
|   └── <name>.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   └── <name>.md
├── _sass
|   └── _<name>.scss
├── _site             # Generated site
└── index.html (index.md)
```

Все остальные папки и файлы, кроме перечисленных выше, будут просто скопированы в папку `_site` (например, `css`, `images`).

# `.gitignore`

```
/_site/
/.jekyll-cache/
/.idea/
```



# Паттерны

Навигация – https://jekyllrb.com/tutorials/navigation/

# Theme

*Theme* – это *gem*, в котором хранятся следующие папки сайта: `assets`, `_layouts`, `_includes`, `_sass`

## Подключение *theme*'ы

1. Добавить в `Gemfile` ссылку на *gem* с *theme*'ой:

  ```
  gem "<theme name>"
  ```

2. Установить *theme*'у:

  ```bash
  bundle install
  ```

3. Указать *theme*'у в *configuration*'и:

  ```yaml
  theme: jekyll-theme-mini
  ```

## Создание новой *theme*'ы

<u>Заполнение содержимого</u>

1. Создать новую *theme*'у через *command line*:

   ```bash
   jekyll new-theme <theme_name>
   jekyll new-theme jekyll-theme-awesome
   ```

2. Добавить файлы в папки `assets`, `_layouts`, `_includes`, `_sass`.

   Стили размещаются в файле `_sass/<theme>.scss`. Затем подключаются к остальным стилям на сайте через команду `@import "{{ site.theme }}";`.

3. Заполнить файлы `.gemspec` и `README`

4. Включить примеры контента (например, `index.md` или `page.md`), чтобы продемонстрировать работу *theme*'ы.

<u>Публикация на  RubyGems.org</u>

При каждом изменении *theme*'ы необходимо выполнять этот алгоритм.

1. Поставить новую версию темы в `<theme_name>.gemspec`

2. Закоммитить

3. Запаковать тему в *gem*:

   ```bash
   gem build <theme_name>.gemspec
   ```

4. Запушить тему в Rubygems:

   ```
   gem push <thema_name>-*.gem
   ```

   

Чтобы переопределять файл из *theme*'ы, необходимо скопировать этот файл из папки *gem*'а в папку `_site`.

Чтобы полностью скопировать *theme*'у, необходимо:

1. скопировать все файлы в папку своего сайта
2. посмотреть на github в файле `<theme>.gemspec`, какие *plugin*' ы используются. Подключить эти *plugin*'ы по [инструкции](#подключение-plugin-а). 
3. удалить из `Gemfile` и *configuration* ссылку на *theme*'у.

# Github pages

https://help.github.com/en/github/working-with-github-pages

*Github* поддерживает только некоторые темы, которые оформлены как *gem*'ы.

Можно разместить произвольную тему на GitHub и подключить ее с помощью ключа *configuration*'и `remote_theme`.

# Jekyll темы

- [jamstackthemes.dev](https://jamstackthemes.dev/ssg/jekyll/)
- [jekyllthemes.org](http://jekyllthemes.org/)
- [jekyllthemes.io](https://jekyllthemes.io/)
- [jekyll-themes.com](https://jekyll-themes.com/)

[Minima](https://github.com/jekyll/minima) .

https://rubygems.org/search?utf8=%E2%9C%93&query=jekyll-theme

# Deployment

## Автоматический на Github Pages

https://jekyllrb.com/docs/github-pages/

http://jmcglone.com/guides/github-pages/

## Travis CI

https://jekyllrb.com/docs/deployment/automated/

https://jekyllrb.com/docs/continuous-integration/travis-ci/

https://ayastreb.me/deploy-jekyll-to-github-pages-with-travis-ci/

https://medium.com/@mcred/supercharge-github-pages-with-jekyll-and-travis-ci-699bc0bde075

## Ручное копирование папки `_site`

# Математические формулы

MathJax

