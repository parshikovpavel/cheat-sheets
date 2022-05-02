*Package* `text/template` реализует управляемые данными *template*'s для генерации текстового *output*.

Чтобы сгенерировать *HTML output*, смотрите *package* `html/template`, который имеет тот же *interface*, что и этот *package*, но автоматически защищает *HTML output* от определенных атак.

*Template*'s применяется к *data structure* (может быть тип *struct*, *map*, ...). *Annotation*'s (что это???) в *template* относятся к элементам в *data structure* (обычно это *field* в *struct* или *key* в *map*) для управления выполнением и получения *value*'s для вывода (т.е. с помощью конструкций шаблонизатора можно выводить элементы из *data structure* и использовать их для управления выполнением ). При обработке *template*, происходит проход по *structure* (может быть тип *struct*, *map*, ...) и устанавливается *cursor*, который обозначается точкой `.` и называется *dot*, на *value* в текущем местоположении в *structure*. *Cursor* двигается по мере выполнения.

*Input text*  для *template* – это текст в кодировке UTF-8. *Action* (структура для вычисления данных или управляющие структуры) — заключаются в символы `{{` и `}}`; весь текст вне *action* копируется в *output* без изменений. За исключением *raw string*, *action* не могут включать *newline*, но могут включать *comment*.

После операции *parse* (одной из функций `Parse...()`),  *template* может безопасно *execute* в параллель, хотя, если параллельное *execute* использует общий `Writer`, *output* может чередоваться.

<u>Пример</u>

```go
type Inventory struct {
	Material string
	Count    uint
}
sweaters := Inventory{"wool", 17}
tmpl, err := template.New("test").Parse("{{.Count}} items are made of {{.Material}}")
if err != nil { 
  // ... 
}
err = tmpl.Execute(os.Stdout, sweaters)
if err != nil {
  // ...
}
```

Выводит:

```
17 items are made of wool
```

TODO!!!



# Overview

## Action

Ниже перечислен список *action*'s. [*Argument*](#argument) и [*pipeline*](#pipeline) — вычисляются на основе данных, они подробно описаны ниже в соответствующих разделах.

```
{{/* a comment */}}
{{- /* a comment with white space trimmed from preceding and following text */ -}}
	A comment; discarded. May contain newlines.
	Comments do not nest and must start and end at the
	delimiters, as shown here.

{{pipeline}}
  Default текстовое представление значения pipeline (такое же, как было бы
	напечатано командой `fmt.Print()`) копируется
	на output.

{{if pipeline}} T1 {{end}}
	If the value of the pipeline is empty, no output is generated;
	otherwise, T1 is executed. The empty values are false, 0, any
	nil pointer or interface value, and any array, slice, map, or
	string of length zero.
	Dot is unaffected.

{{if pipeline}} T1 {{else}} T0 {{end}}
	If the value of the pipeline is empty, T0 is executed;
	otherwise, T1 is executed. Dot is unaffected.

{{if pipeline}} T1 {{else if pipeline}} T0 {{end}}
	To simplify the appearance of if-else chains, the else action
	of an if may include another if directly; the effect is exactly
	the same as writing
		{{if pipeline}} T1 {{else}}{{if pipeline}} T0 {{end}}{{end}}

{{range pipeline}} T1 {{end}}
	Значение pipeline должно быть array, slice, map, или channel.
	Если значение pipeline имеет `length = 0`, ничего не выводится на output;
	иначе dot устанавливается на последовательные элементы array,
	slice или map и выполняется T1. Если value – это map и
	key's имеют basic type с определенным порядком, элементы будут
	перебираться в порядке сортировки key's.

{{range pipeline}} T1 {{else}} T0 {{end}}
	The value of the pipeline must be an array, slice, map, or channel.
	If the value of the pipeline has length zero, dot is unaffected and
	T0 is executed; otherwise, dot is set to the successive elements
	of the array, slice, or map and T1 is executed.

{{break}}
	The innermost {{range pipeline}} loop is ended early, stopping the
	current iteration and bypassing all remaining iterations.

{{continue}}
	The current iteration of the innermost {{range pipeline}} loop is
	stopped, and the loop starts the next iteration.

{{template "name"}}
	The template with the specified name is executed with nil data.

{{template "name" pipeline}}
	The template with the specified name is executed with dot set
	to the value of the pipeline.

{{block "name" pipeline}} T1 {{end}}
	A block is shorthand for defining a template
		{{define "name"}} T1 {{end}}
	and then executing it in place
		{{template "name" pipeline}}
	The typical use is to define a set of root templates that are
	then customized by redefining the block templates within.

{{with pipeline}} T1 {{end}}
	If the value of the pipeline is empty, no output is generated;
	otherwise, dot is set to the value of the pipeline and T1 is
	executed.

{{with pipeline}} T1 {{else}} T0 {{end}}
	If the value of the pipeline is empty, dot is unaffected and T0
	is executed; otherwise, dot is set to the value of the pipeline
	and T1 is executed.

```

## Argument

*Argument* — это простое *value*, описываемое одним из следующих способов. *Argument* является частью [pipeline](#pipeline)

```
- A boolean, string, character, integer, floating-point, imaginary
  or complex constant in Go syntax. These behave like Go's untyped
  constants. Note that, as in Go, whether a large integer constant
  overflows when assigned or passed to a function can depend on whether
  the host machine's ints are 32 or 64 bits.
- The keyword nil, representing an untyped Go nil.
- Символ '.' (точка):
	.
  Результат – это значение dot.
- A variable name, which is a (possibly empty) alphanumeric string
  preceded by a dollar sign, such as
	$piOver2
  or
	$
  The result is the value of the variable.
  Variables are described below.
- Имя field of struct, перед которым стоит точка `.`, например:
	.Field
  Результатом является значение field. Имена field's мог быть собраны в цепочку:
    .Field1.Field2
  Field также может быть получено для variable, также можно собирать цепочку:
    $x.Field1.Field2
- The name of a key of the data, which must be a map, preceded
  by a period, such as
	.Key
  The result is the map element value indexed by the key.
  Key invocations may be chained and combined with fields to any
  depth:
    .Field1.Key1.Field2.Key2
  Although the key must be an alphanumeric identifier, unlike with
  field names they do not need to start with an upper case letter.
  Keys can also be evaluated on variables, including chaining:
    $x.key1.key2
- The name of a niladic method of the data, preceded by a period,
  such as
	.Method
  The result is the value of invoking the method with dot as the
  receiver, dot.Method(). Such a method must have one return value (of
  any type) or two return values, the second of which is an error.
  If it has two and the returned error is non-nil, execution terminates
  and an error is returned to the caller as the value of Execute.
  Method invocations may be chained and combined with fields and keys
  to any depth:
    .Field1.Key1.Method1.Field2.Key2.Method2
  Methods can also be evaluated on variables, including chaining:
    $x.Method1.Field
- The name of a niladic function, such as
	fun
  The result is the value of invoking the function, fun(). The return
  types and values behave as in methods. Functions and function
  names are described below.
- A parenthesized instance of one the above, for grouping. The result
  may be accessed by a field or map key invocation.
	print (.F1 arg1) (.F2 arg2)
	(.StructValuedMethod "arg").Field
```

*Argument* может иметь любой *type*; если он является *pointer*, реализация автоматически выводит *value* (в доке написано *base type*), когда это необходимо. Если вычисление дает *function value*, например *field of struct* c *function value*, *function* не вызывается автоматически, но ее можно использовать в качестве значения `true` для `if` *action* и т.п. Чтобы вызвать его, используйте `call` *function*, определенную ниже.

## Pipeline

*Pipeline* — это последовательность *command*'s. *Pipeline*'s можно связывать в цепочку. 

*Command* – это:

- простое значение (*argument*)
- вызов *function* или *method*'а, возможно, с несколькими *argument*'ами:

```
Argument
	Результат – значение вычисления argument'а.
.Method [Argument...]
	Method может быть единственным или последним элементом цепочки (а средним???), но
	в отличие от method'ов в середине цепочки, он может принимать argument'ы.
	Результатом является значение вызова method'а с argument'ами:
		dot.Method(Argument1, etc.)
functionName [Argument...]
  Результат – значение вызова указанной function:
		function(Argument1, etc.)
	Доступные function's описаны ниже.
```

*Pipeline*'s можно связывать в цепочку путем разделения последовательности *command* символом `|`. В цепочке *pipeline*'s результат каждой *command*'ы передается как последний *argument* следующей *command*'ы. Вывод последней *command*'ы в *pipeline* — это значение *pipeline*.

Выводом *command*'ы будет либо одно значение, либо два значения, второе из которых имеет тип `error`. Если это второе значение присутствует и является *non-nil*, выполнение прекращается, и *error* возвращается в качестве результата `Execute()`.



TODO!!!

## Examples

Вот несколько примеров однострочных *template*'s, демонстрирующих *pipeline* и *variable*. Все они выводят `"output"` (в кавычках):

```
{{"\"output\""}}
	String constant.
{{`"output"`}}
	Raw string constant.
{{printf "%q" "output"}}
	Вызов function.
{{"output" | printf "%q"}}
	A function call whose final argument comes from the previous
	command.
{{printf "%q" (print "out" "put")}}
	A parenthesized argument.
{{"put" | printf "%s%s" "out" | printf "%q"}}
	A more elaborate call.
{{"output" | printf "%s" | printf "%q"}}
	A longer chain.
{{with "output"}}{{printf "%q" .}}{{end}}
	A with action using dot.
{{with $x := "output" | printf "%q"}}{{$x}}{{end}}
	A with action that creates and uses a variable.
{{with $x := "output"}}{{printf "%q" $x}}{{end}}
	A with action that uses the variable in another action.
{{with $x := "output"}}{{$x | printf "%q"}}{{end}}
	The same, but pipelined.
```

## Function

Во время выполнения, *function*'s находятся в двух *function map*'s: 

- в *template*
- в глобальной *function map*. 

По умолчанию, в *template* не определены никакие *function*'s, но для их добавления можно использовать метод `Funcs()`.

Ниже – список предопределенных глобальных *function*:

```
and
	Returns the boolean AND of its arguments by returning the
	first empty argument or the last argument. That is,
	"and x y" behaves as "if x then y else x."
	Evaluation proceeds through the arguments left to right
	and returns when the result is determined.
call
	Returns the result of calling the first argument, which
	must be a function, with the remaining arguments as parameters.
	Thus "call .X.Y 1 2" is, in Go notation, dot.X.Y(1, 2) where
	Y is a func-valued field, map entry, or the like.
	The first argument must be the result of an evaluation
	that yields a value of function type (as distinct from
	a predefined function such as print). The function must
	return either one or two result values, the second of which
	is of type error. If the arguments don't match the function
	or the returned error value is non-nil, execution stops.
html
	Returns the escaped HTML equivalent of the textual
	representation of its arguments. This function is unavailable
	in html/template, with a few exceptions.
index
	Returns the result of indexing its first argument by the
	following arguments. Thus "index x 1 2 3" is, in Go syntax,
	x[1][2][3]. Each indexed item must be a map, slice, or array.
slice
	slice returns the result of slicing its first argument by the
	remaining arguments. Thus "slice x 1 2" is, in Go syntax, x[1:2],
	while "slice x" is x[:], "slice x 1" is x[1:], and "slice x 1 2 3"
	is x[1:2:3]. The first argument must be a string, slice, or array.
js
	Returns the escaped JavaScript equivalent of the textual
	representation of its arguments.
len
	Возвращает integer length для своего argument.
not
	Returns the boolean negation of its single argument.
or
	Returns the boolean OR of its arguments by returning the
	first non-empty argument or the last argument, that is,
	"or x y" behaves as "if x then x else y".
	Evaluation proceeds through the arguments left to right
	and returns when the result is determined.
print
	An alias for fmt.Sprint
printf
	An alias for fmt.Sprintf
println
	An alias for fmt.Sprintln
urlquery
	Returns the escaped value of the textual representation of
	its arguments in a form suitable for embedding in a URL query.
	This function is unavailable in html/template, with a few
	exceptions.
```



TODO!!!



# Type

## `type Template`

```go
type Template struct {
	*parse.Tree
	// contains filtered or unexported fields
}
```

`Template` — это представление *parsed template* (распарсенного). Поле `*parse.Tree` является *exported* и предназначено только для использования `html/template` и должно рассматриваться как *non-exported* всеми другими клиентами.

<u>Пример:</u>

```go
func main() {
	// Define a template.
	const letter = `
Dear {{.Name}},
{{if .Attended}}
It was a pleasure to see you at the wedding.
{{- else}}
It is a shame you couldn't make it to the wedding.
{{- end}}
{{with .Gift -}}
Thank you for the lovely {{.}}.
{{end}}
Best wishes,
Josie
`

	// Prepare some data to insert into the template.
	type Recipient struct {
		Name, Gift string
		Attended   bool
	}
	var recipients = []Recipient{
		{"Aunt Mildred", "bone china tea set", true},
		{"Uncle John", "moleskin pants", false},
		{"Cousin Rodney", "", false},
	}

	// Create a new template and parse the letter into it.
	t := template.Must(template.New("letter").Parse(letter))

	// Execute the template for each recipient.
	for _, r := range recipients {
		err := t.Execute(os.Stdout, r)
		if err != nil {
			log.Println("executing template:", err)
		}
	}

}
```



### `Must()`

```go
func Must(t *Template, err error) *Template
```

`Must()` — это *helper*, которая оборачивает вызов функции, возвращающей `(*Template, error)`, и бросает *panic*, если *error* – *non-nil*. Он предназначен для использования в инициализации *variable*, например:

```go
var t = template.Must(template.New("name").Parse("text"))
```



### `New()`

```go
func New(name string) *Template
```

`New()` создает новый *undefined template* с указанным `name`.



### `Execute()`

```go
func (t *Template) Execute(wr io.Writer, data any) error
```

`Execute()` заполняет *parsed template* указанным `data` *object* и *write* результат в `wr`. Если при *executing the template* или *writing* его *output* возникает *error*, *execution* останавливается, но частичные результаты могут быть уже записаны в *output writer*. *Template* может безопасно *execute* в параллель, хотя, если параллельное *execution* использует общий `Writer`, *output* может чередоваться.

Если `data` – это `reflect.Value`, *template* применяется к конкретному *value*, которое содержит `Reflect.Value`, как в `fmt.Print()`.



### `Parse()`

```go
func (t *Template) Parse(text string) (*Template, error)
```

`Parse()` парсит `text` как *template body* для `t`. *Named template definition* (`{{define ...}}` или `{{block ...}}` *statement*) в `text` определяет дополнительный *template*, асоциированный с `t`, и удаляется из *definition* самого `t`.

*Template* можно переопределить при последовательных вызовах `Parse()`. *Template definition* с *body*, содержащим только *space* и *comment*, считается пустым и не заменяет *body* существующего *template*. Это позволяет использовать `Parse()` для добавления новых *named template definition* без перезаписи *main template body*.

#### 



