## Понятие фреймворка

Фре́ймворк (иногда фреймво́рк; от framework — каркас, структура) — платформа, определяющая структуру программной системы.

Конечное приложение состоит из двух частей:

·   Постоянная часть — неизменный каркас, имеющий точки расширения для написания пользовательского кода. Эти места также называют холодными точками (frozen) фреймворка.

·   Переменная часть – пользовательский код, написанный в точках расширения фреймворка. Эти места называют горячими (hot) точками.

Отличия фреймворка от библиотеки: 

·   фреймворк диктует правила построения архитектуры приложения, определяет «каркас», который расширяется в определенных точках. 

·   фреймворк реализует инверсию управления. Т.е. фреймворк сам вызывает методы с пользовательским кодом, встроенные в точках расширения в каркас фреймворка. 

·   расширямость – добавление пользовательского кода в точки расширения

Фреймворки пишутся на основе принципов ООП, а точки расширения представляются абстрактными классами и их методами. 

## Типы архитектур

### Push-based и pull-based

Большинство фреймворков (Symphony, YII) являются примерами push-based архитектуры, также назваемой action-based. Обработка запроса выполняется в некотором действии (action), из которого данные вталкиваются (push) на уровень представления для формирования HTML. 

Альтернативой этому является фреймворки на основе pull-based архитектуры, также называемой component-based. Обработка запроса стартует с уровня представления, на котором данные вытягиваются (pull) из нескольких контроллеров по мере необходимости. В этой архитектуре одно представление может задействовать несколько контроллеров. Существуют фреймворки, имеющие черты как push- так и pull-based архитектур.

Наш фреймворк можно также считать как push-, так и pull-based архитектурой. Так из шаблонов можно обращаться к контроллерам через функцию генерации блока:

$this->block('content/action/param');

### MVC

*Model-View-Controller* (*MVC*, «Модель-Представление-Контроллер», «Модель-Вид-Контроллер») — схема разделения компонента доступа к данным, пользовательского интерфейса и управляющей логики на три отдельных компонента:

- Модель (*Model*) 

- Представление (*View*) 

- Контроллер (*Controller*)

#### Преимущества

- разделение ответственности в логике приложения (*SRP*): бизнес-логики (*model*), пользовательского интерфейса (*view*), логики обработки запросов пользователей (*controller*)

- каждый слой может изменяться независимо от другого

- каждый слой может повторно использоваться независимо от другого

- высокая связность (*high cohesion*)

- низкое зацепление (*low coupling*)

- можно строить разные комбинации слоев друг с другом: 
  - одни и те же данные *model*'и можно выводит через несколько разных видов *view*'ов: например, в мобильном и десктопном варианте. 
  - одни и те же *view*'ы можно использовать для вывода при различных действиях пользователя в *controller*'е

- каждый слой можно разрабатывать параллельно и независимо от другого. Например, поручить разработку *view* дизайнеру, разработку *model* и *controller* – программисту. 


#### Компоненты архитектуры

##### Model

Модель (*model*) – бизнес-логика системы, отражение предметной области системы, ее "содержание". Модель включает

- методы для доступа к данным, как правило из базы данных

- проверки данных на корректность

##### View


Представление (*View*) – графический пользовательский интерфейс (GUI) для отображения информации

##### Controller

Контроллер (*Controller*). Находится в центре обработки запросов. 

![MVC]( https://parshikovpavel.github.io/img/mvc.png )

Стрелки в обе стороны. Задачи:

- *View*→*Controller*. В результате действий пользователя во *view* в браузере запросы поступают в *controller*.
- *Controller*→*Model*. Обработав входящие данные, *controller* передает их в *model* для дальнейшего сохранения.
- *Model*→*Controller*. *Model* возвращает результат сохранения данных в *controller*. 
- *Controller*→*View*. *Controller* передает данные из *model* в *view* для формирования нового представления.

#### Особенности реализации

Отсутствует общепринятое представление о месте расположения бизнес-логики. Она может находиться в любом слое. 

- В *Controller*. *Model* содержит методы только для доступа к данным, а *Controller* содержит так же бизнес-логику. В этом случае стали получаться «ТТУК» («Толстые, тупые, уродливые контроллеры»; *Fat Stupid Ugly Controllers*). 
- В *Model*. Наиболее частое место размещения бизнес-логики. Контроллеры «тонкие» и выполняют выполняют исключительно функцию связующего звена (*glue layer*).

- В *View*. Например, *Javascript* может выполнять первоначальный контроль корректности входных данных.

Чаще всего бизнес-логика размазана по всем слоям.

#### Реализация на клиенте

В классической архитектуре, каждый запрос инициирует перезагрузку страницы и формирование нового *View*. Это очень медленно, неудобно и вызывает проблемы, когда сервер не доступен.

Вместо классической схемы, применяются асинхронные запросы и реализация MVC на клиенте. Веб-приложения начинают напоминать обычные настольные приложения. 

Преимущества:

- взаимодействие с сервером выполняет асинхронно и в фоне. И если сервер не доступен, это не блокирует работу пользователя. 

### MVVM

MVVM (Model – View – ViewModel — «модель – представление – модель представления»)

![mvvm]( https://parshikovpavel.github.io/img/mvvm.png?1223 )

В шаблоне *MVC* изменения в *View* в браузере не влияют непосредственно на *Model*, а предварительно идут запросом через *Controller*. 

Особенности MVVM:

- взаимодействие с сервером выполняет асинхронно и в фоне. Приложение мгновенно реагируют на действия пользователя И если сервер не доступен, это не блокирует работу пользователя. 
- бизнес -логика размещается в одном месте (*ViewModel*) и не размазывается по слоям.

Основные части:

- *View* (представление) – (как и для *MVC*) графический пользовательский интерфейс (GUI) для отображения информации. Однако, здесь отсутствует какая-либо бизнес-логика (например, контроль корректности данных на *Javascript*). *View* лишь выводит информацию. 
- *Data binding* (связывание данных) – обеспечивает *реактивность* приложения (*reactive*):
  - с одной стороны, обеспечивает, чтобы любые изменения *application state* в *ViewModel* были автоматически отражены в *View*.
  - с другой стороны, обеспечивает, чтобы при любых действиях пользователя в *View* происходили соответствующие изменения *application state* в *ViewModel*.
- *ViewModel* (модель представления) – хранит данные приложения в объекте-хранилище (*store*). В *store* хранится состояние приложения (*aplication state*). Как *Controller* в *MVC*, отвечает за сохранение данных в *Model*, однако делает это асинхронно. 
- *Model* (модель) (частично похоже MVC) – играет роль постоянного репозитория для данных приложения. Однако, часто используется исключительно как репозиторий и не содержит никакой бизнес-логики. Вся бизнес-логика размещается в *ViewModel*.






## PHP-фреймворки


Во многих языках для Web-разработки имеются глобальные фреймворки: 

- *Ruby* – *Ruby on Rails*

- *Python* – *Django* 

- *Java*  – *Spring*. 

В отличие от них, в РНР не сформировалось глобального фреймворка, а есть множество популярных: *Yii*, *Symfony*, *Zend*, *Phalcon*, *Laravel*…