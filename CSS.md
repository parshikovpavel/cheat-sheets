# Flex

https://tproger.ru/translations/how-css-flexbox-works/

### Flex Direction

`flex-direction` задаёт направление оси в *container*'е и тем самым определяет положение  flex'в. 

Значения:

- `row` – главная ось направлена слева направо.
- `row-reverse` – главная ось направлена справа налево.
- `column` – главная ось направлена сверху вниз.
- `column-reverse` – главная ось направлена снизу вверх.

### Flex Grow

`flex-grow` определяет, сколько пространства может занимать *flex* внутри *container*'а. В качестве значения принимаются числа, они задают пропорции каждого *flex*'а. Т.е.  `flex-grow` принимает не абсолютные значения, а относительные. Это значит, что не важно, какое значение у `flex-grow`, важно, какое оно по отношению к другим *flex*'ам. 

К примеру, если для всех элементов установлено значение `1`, то они получатся равного размера. 

Если имеется 6 *flex*'ов и `flex-grow` одного *flex*'а устанавливает в 2, то *container* делится на `7` частей (`1 + 1 + 2 + 1 + 1 + 1`). Один *flex* занимает `2/7` пространства, остальные — по `1/7`.

По умолчанию значение `flex-grow` равно 0. Это значит, что квадратам запрещено расти (занимать оставшееся место в контейнере).

| Значение по умолчанию | 0                 |
| :-------------------- | ----------------- |
| Наследуется           | Нет               |
| Применяется           | К флекс-элементам |
| Анимируется           | Да                |

Стоит помнить, что `flex-grow` работает только для главной оси.

# Свойство z-index

Если свойства z-index и позиционирование у элементов не заданы явно, то порядок наложения элементов определяется в соответствии с порядком следования элементов в HTML. Выше лежат элементы, описанные в HTML ниже.

Для того чтобы изменить порядок наложения элемента на странице нужно:

·  явно задать элементу позиционирование, кроме static (relative, absolute....)

·  задать элементу значение z-index.

Контекст наложения

Каждый контекст наложения включает:

·  некоторый корневой элемент в HTML структуре

·   и все его дочерние элементы, которые автоматически попадают в этот контекст наложения. 

При перемещении на передний или задний план корневого элемента контекста наложения, дочерние элементы, находящиеся в рамках этого контекста наложения, перемещаются вместе с ним. Если один контекст наложения (1) находится ниже другого (2) по z-index, то никакой элемент из первого контекста (1) не может оказаться выше любого элемента из второго (2) контекста. Значения свойства z-index дочерних элементов контекста наложения учитываются только внутри этого контекста наложения и не влияют на соседние контексты наложения.

Элемент HTML формирует новый контекст наложения и становится корневым элементом в этом контексте наложения, если для него выполняется одно из условий:

·  элемент – корневой элемент документа

·  для элемента явно изменен порядок наложения (задано позиционирование и значение z-index)

·  для элемента установлено opacity (прозрачность) менее 1.

·   для элемента установлено любое значение transform

·  и еще некоторые более редкие свойства CSS