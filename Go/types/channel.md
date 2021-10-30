Одна из основных проблем *concurrent programming* – корректный доступ к *shared variable*.  При разработке на Go рекомендуется передавать *shared variable* по *channel*.

*Rob Pike* сказал: не коммуницируйте через разделяемую память, разделяйте память посредством коммуникации  (don't communicate by sharing memory, share memory by communicating).

Т.е. есть два способа взаимодействия в *concurrent programming*:

- *communicate by sharing memory* – это традиционные техники *concurrency synchronization*, такие как *mutex* для защиты *shared memory* от *data races*. 
- *share memory by communicating* – через *channel*. В итоге, доступ к конкретной ячейке памяти имеет только одна *goroutine*. *Race condition* не может произойти *by design*.

*Channel* предоставляет механизм для взаимодействия конкурентных *goroutine*'s с помощью отправки (через [send statement](../Lang.md#send-statement)) и получения (через [receive operator](../Lang.md#receive-operator)) значений указанного *element type*. Значение *uninitialized channel*  равно `nil`.

<pre>
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) <a href="#array-type">ElementType</a> .  
</pre>

Опциональный оператор `<-`  указывает *channel direction*. При объявлении указывается `ElementType`, по *channel* могут пересылаться значения только этого *type*.

Обозначим *element type* – `T`. Есть *bi-directional* и *single-directional channel types*.

- *bidirectional channel type* для *send* и *receive* значений

  ```go
  chan T
  ```

- *send-only channel type*. Компилятор не позволяет делать *receive* значений из такого *channel*.

  ```go
  chan<- T
  ```

- *receive-only channel type*. Компилятор не позволяет делать *send* значений в такой канал.

  ```go
  <-chan T 
  ```

Значение *bidirectional channel type* `chan T` может быть преобразовано в *send-only type* `chan<- T` и в *receive-only type* `<-chan T` с помощью *assignment statement* ([link](../Lang.md#assignment)) или с помощью *explicit conversion* ([link](../Lang.md#conversion)). Но не наоборот.

```go
var a chan int

b := (<-chan int)(a)  // OK
c := (chan<- int)(a)  // OK

d := (chan int)(b)    // Error
e := (chan int)(c)    // Error
```

Для создания инициализированного *channel value* необходимо использовать функцию [`make`](../Lang.md#make) :

```go
make(t Type, capacity IntegerType) Type
make (chan int, 100)
```

У каждого *channel value* есть *capacity*. Он задает вторым аргументом функции `make()`. Второй аргумент является необязательным, и его значение по умолчанию равно 0.

Различают:

- *unbuffered channel* – *channel*, у которого *capacity* равен 0 или не указан (в это случае, также равен 0). Коммуникация будет успешной (??? без блокировки) только тогда, когда и *sender*, и *receiver* готовы.

* *buffered channel* – *channel*, у которого указан ненулевой *capacity*. Коммуникация пройдет без блокировки, если *buffer* не полный (для [send statement](../Lang.md#send-statement)) или не пустой (для [receive operator](../Lang.md#receive-operator)).
* `nil`  – *zero value* для *channel type*, *channel* никогда не готов к коммуникации.

Один и тот же *channel* может использоваться в [send statement](../Lang.md#send-statement), [receive operator](../Lang.md#receive-operator) и вызовах *built-in* функций `cap()`и `len()` любым количеством *goroutine*'s без синхронизации. 

*Channel* действуют как FIFO (*first-in-first-out*) *queue*. Т.е., если одна *goroutine* отправляет значения по *channel*, а вторая *goroutine* получает их, значения будут получены в том порядке, как они были отправлены.

## Операции над *channel*

Над *channel* можно выполнять пять операций.

Все эти операции синхронизированы, поэтому для безопасного выполнения этих операций не требуется никаких дополнительных синхронизаций.

### close()

```go
func close(c chan<- Type)
```

`close()`  закрывает *channel*, который должен быть *bidirectional* или *send-only*. После этого больше никакие *value*'s не могут быть *sent* по *channel*.

`close()` должен вызываться только *sender*'ом и не должен никогда вызываться *receiver*'ом, и имеет эффект отключения *channel*'а, после того как *last sent value* (*sender*'ом) – *received* (*receiver*'ом). 

*Send* и *close* на *closed channel* вызывает *panic*. *Close* для *nil channel* также вызывает *panic*. 

После *receive* любых ранее *sent value*'s из *closed channel* `c`, вызов *receive operator* из `c` будет успешным без *blocking*, возвращая *zero value* для *channel element type*. При использовании *closed channel* в *tuple assignment*:

```go
x, ok := <-c  // ok = false
```

получаем `ok = false`.

### Send statement

*Send statement* отправляет *value* в *channel*. 

```go
SendStmt = Channel "<-" Expression .
Channel  = Expression .
```

```go
ch <- v // отправляем значение v в канал ch
ch <- 3 // отправляем значение 3 в канал ch
```

- *channel expression* должно быть [channel type](types/channel.md)
- *channel* должен быть *bidirectional* или *send-only*
- тип *value*, которое будет sent (`Expression`), должно быть [assignable](#assignability) к *channel's element type*.

Коммуникация блокируется до тех пор пока *send* не может быть выполнено:

- *Send* в *unbuffered channel* может быть выполнен, если *receiver* готов
- *Send* в *buffered channel* может быть выполнен, если в *buffer*'е есть место. 
- *Send* в *closed channel* вызывает *panic*
- *Send* в `nil` *channel* приводит к *block forever*.

### Receive operator

*Receive operator* возвращает *value*, *received* из *channel* `ch`.

```go
<-ch
```

- `ch` должно быть [channel type](types/channel.md)
- *channel* должен быть *bidirectional* или *receive-only*
- тип возвращаемого значение – *channel's element type*.

Особенности:

- *Receive operator* блокируется до тех пор пока *value* не станет доступно
- *Receive* из `nil` *channel* приводит к *block forever*.
- *Receive* из *closed channel* будет всегда выполнено немедленно, возвращая вначале ранее *sent value*'s, а затем *element type's zero value*

```go
v1 := <-ch
v2 = <-ch
f(<-ch)
<-ch  // ждать, пока не будет получено value (???) и отбросить его
```

*Receive operator* может использоваться в *tuple assignment* и *declaration*:

```go
x, ok = <-ch
x, ok := <-ch
var x, ok = <-ch
var x, ok T = <-ch
```

- `ok` – *boolean value*, которое сообщает, была ли успешна коммуникация
  - `ok = true` – если *received value* было доставлено в результате успешного *send statement* 
  - `ok = false` – если это *zero value*, сгенерированное потому что *channel* – *closed* и *empty*.

Вместо *receive operator* можно использовать [цикл `for-range`](#цикл-for-range)

*Receive operator* может использоваться в *receive operation*. Например:

```go
x = <-c + 1
```



### `cap()`

```go
cap(ch)
```

Возвращает *capacity of channel buffer*. Для *nil channel* возвращает ноль. 

### `len()`

```go
len(ch)
```

Возвращает *length of channel buffer* – текущее количество элементов в *channel buffer*. Т.е. которые уже были успешно *sent* в *channel*, но еще не были *received*.

Для *nil channel* возвращает ноль. 

## Internals

Каждый *channel* состоит из 3 *queue*'s:

- *receiving goroutine queue* – очередь *goroutine*'s, все (!!!) из которых находятся в состоянии *block* и ожидают возможности *receive*. Обычно FIFO. Представляет собой *linked list* без ограничения размеров.
- *sending goroutine queue* – очередь *goroutine*'s, все (!!!) из которых находятся в состоянии *block* и ожидают возможности *send*. Обычно FIFO. Представляет собой *linked list* без ограничения размеров. *Value*, которое пытается *send* каждая *goroutine* также сохраняется в *queue*.
- *value buffer queue* – очередь буфера значений. Всегда FIFO. Это circular queue – кольцевая очередь, кольцевой буфер (в конец ее пишут, а из начала читают, последний элемент замкнут с первым). Его *size* равна *channel capacity*.

Каждый *channel* содержит *mutex* для исключения *data race* по всем операциям.

### Внутренняя реализация основных операций

Разделим *channel*'s на три типа:

- *nil channel*
- *non-nil, closed channel*
- *non-nil, not-closed channel*

Следующая таблица суммирует поведение для всех операций и типов *channel*'s.

| Operation   | Nil Channel   | Closed channel           | Non-nil, not-closed channel |
| ----------- | ------------- | ------------------------ | --------------------------- |
| **Close**   | panic         | panic                    | success (C)                 |
| **Send**    | block forever | panic                    | block or success (B)        |
| **Receive** | block forever | success, never block (D) | block or success (A)        |

Для пяти *case*'s (без символов в скобках) – поведение очевидно:

- *Close* для *nil channel* и уже *closed channel* приводит к *panic* в текущей *goroutine*.
- *Send* в *closed channel* также приводит к *panic* в текущей *goroutine*.
- *Send* и *receive* из *nil channel* приводит к *block forever*.

#### (A) *Receive* из *non-nil, not-closed channel*

*Receiving goroutine* `R` вначале *acquire mutex*, относящийся к этому *channel*. Затем выполняет следующие шаги, пока не будет выполнено одно из условий.

- Если *value buffer queue* – не пуст, это значит что *receiving goroutine queue* – пуст. Тогда *goroutine* `R` делает *receive value* из *value buffer queue* и продолжает находится в *running state* (*non-blocking operation*). Если при этом *sending goroutine queue* – не пуст, очередная *sending goroutine* будет взята из *sending goroutine queue*, ее *value* будет помещено в *value buffer queue*, она переходит в *running state* (выходит из *blocking state*). 
- Если это *unbuffered channel*, *value buffer queue* – пуст, и *sending goroutine queue* – не пуст. *Sending goroutine* извлекается из *sending goroutine queue*, *receiving goroutine* `R` получает *value* которое пыталась отправить *sending goroutine*. *Sending goroutine* переходит в *running state* (выходит из *blocking state*). *Receiving goroutine* `R` продолжает находится в *running state* (*non-blocking operation*).
- Если *value buffer queue* и *sending goroutine queue* – пусты, *goroutine* `R` будет помещена в *receiving goroutine queue* и перейдет в *blocking state* (*blocking operation*). Она вернется в *running state*, когда другая *goroutine* сделает *send* в *channel*.

#### (B) *Send* в *non-nil, not-closed channel*

*Sending goroutine* `S` вначале *acquire mutex*, относящийся к этому *channel*. Затем выполняет следующие шаги, пока не будет выполнено одно из условий.

- если *receiving goroutine queue* – не пуст, это значит что *value buffer queue* – пуст. Тогда *receiving goroutine* извлекается из *receiving goroutine queue*. *Sending goroutine* `S`  отправляет *value* этой *receiving goroutine*, которая переходит из *blocking* в *running state*. *Sending goroutine* `S` продолжит находиться в *running state* (*non-blocking operation*).
- Если *receiving goroutine queue* – пуст; и *value buffer queue* – не полон; в этом случае *sending goroutine queue* – так же пуст. *Sending goroutine* `S` отправляет *value* в *value buffer queue* и продолжает находиться в *running state* (*non-blocking operation*).

- Если *receiving goroutine queue* – пуст; и *value buffer queue* – полон. Тогда *sending goroutine* `S` будет помещена в *sending goroutine queue* и перейдет в *blocking state* (*blocking operation*). Она снова вернется в *running state*, когда другая *goroutine* сделает *receive* из *channel*.

#### (C) Close для Non-nil, not-closed channel

*Goroutine* вначале *acquire mutex*, относящийся к этому *channel*. Затем выполняет следующие шаги, пока не будет выполнено одно из условий.

- Если *receiving goroutine queue* – не пуст; в этом случае *value buffer queue* – пуст, из *receiving goroutine queue* извлекаются поочереди *receiving goroutine*'s, каждая получает *zero value* (и второе значение `false`) и переходит в *running state*.
- Если *sending goroutine queue* – не пуст, из *sending goroutine queue* извлекаются поочереди *sending goroutine*'s, каждая бросает *panic* из-за отправки в *closed channel*. Поэтому необходимо избегать конкурентных операций *send* и *close* на одном и том же *channel*. Когда включена [go command's data race detector option](https://golang.org/doc/articles/race_detector.html) (`-race`), конкурентные операции *send* и *close* могут быть определены в *run time* и выброшена *panic*.

После того как *channel* стал *closed*, значения из *value buffer queue* – продолжат там оставаться. Как они извлекаются описано в  (D).

#### (D) *Receive* из *Closed channel*

Значения могут быть *receive* из *value buffer queue* из *closed channel*. Значения извлекаются из *closed channel*, как обычно, с помощью *receive operator*. 

- если значения есть в *value buffer queue*: возвращаемые значения из *receive operator* – *value*, `true`. 
- если значения закончились в *value buffer queue*: возвращаемые значения из *receive operator* – *zero value*,  `false`.



### *Values* передаются по *channel* путем *copy by value*

Когда *value* передается из одной *goroutine* в другую, оно будет скопировано как минимум один раз. Если *value* помещается в *value buffer queue*, то в процессе передачи будет сделано две копии:

1. *value* будет скопировано из *sending goroutine* в *value buffer queue*
2. *value* будет скопировано из *value buffer queue* в *receiving goroutine queue*. 

Подобно *assignment* (*copy by value*) происходит копирование только [direct part](https://go101.org/article/value-part.html#about-value-copy).

### Размер *channel element type*

Размер *channel element type* должен быть меньше, чем `65536`. Однако, как правило, мы не должны создавать *channel* с большим размером *element type*, чтобы избежать слишком больших затрат на копирование в процессе передачи *value*'s между *goroutine*'s. Поэтому, если размером *element type* слишком велик, лучше использовать вместо него *pointer element type*, чтобы избежать затрат на копирование больших значений.



### *Garbage collector* для *channel* и *goroutine*

На *channel* ссылаются все *goroutine*'s в *sending* и *receiving goroutine queue*, поэтому, если ни одна из *queue*'s не пуста, *channel* не может быть собран *garbage collector*'ом. Также если *goroutine* находится в *blocking state* и в *sending* или *receiving goroutine queue*, то *goroutine* также не может быть собрана *garbage collector*'ом. Т.е. *goroutine* может быть собрана *garbage collector*'ом, только когда она уже завершилась.



## *Send statement* и *receive operator* как *simple statement*

*Send statement* и *receive operator* являются [*simple statement*](../Lang.md#statement). *Receive operator* также является *expression*. Поэтому они могут быть использованы в некоторых местах *control flow statement* (`if`, `for`, ...)

Пример, использования send statement и receive operator в заголовке цикла `for`:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c := make(chan uint64)  // unbuffered channel
	go func() {
		var x, y uint64 = 0, 1
		for ; y < 100; c <- y {  // send число фибоначи в channel 
			x, y = y, x+y        // суммируем числа фибоначи до 100
		}
		close(c)
	}()

	for x, ok := <-c; ok; x, ok = <-c { // receive число фибоначи из channel
		time.Sleep(time.Second)         // пока ok = true
		fmt.Println(x)
	}
}
```

## Цикл `for-range`

Цикл [`for-range`](../Lang.md#с-range-clause) можно использовать для итеративного *receive value*'s из *channel*. Цикл завершается, когда *channel* будет *close* и после этого его *value buffer queue* станет пустой. Как указано в документации к [`for-range`](../Lang.md#с-range-clause), можно использовать только одну *iteration variable* для записи *received value*.

```go
for v := range ch {
	// use v
}
```

это эквивалентно:

```go
for {
	v, ok = <-ch
	if !ok {
		break
	}
	// use v
}
```

-  *channel* должен быть *bidirectional* или *receive-only*
- для *nil channel* – цикл *block forever*



## `select` statement

`select` *statement* специально разработан для *channel*. `select` *statement* определяет, который из множества возможных [send statement](#send-statement) (`SendStmt`) и [receive statement](#receive-operator) (`RecvStmt`) будет выбран для исполнения. `select` *statement* похож на `switch` *statement*, но все блоки `case` могут содержать только *send statement* и *receive statement*.

---

<pre>
SelectStmt = "select" "{" { CommClause } "}" .
CommClause = CommCase ":" StatementList .
CommCase   = "case" ( <a href="#send-statement">SendStmt</a> | RecvStmt ) | "default" .
RecvStmt   = [ ExpressionList "=" | <a href="Lang.go#constant-declaration">IdentifierList</a> ":=" ] RecvExpr .
RecvExpr   = Expression .  
</pre>

---

`RecvStmt` и `SendStmt`, которые идут за `case` *keyword*, дальше будут называться *case operation*.

Смысл `ExpressionList`  и `IdentifierList`:

- `ExpressionList` – уже объявленные переменные
- `IdentifierList` – новые переменные, идентификаторы

Особенности:

- в отличии от `switch/case`, между `select` и `{` – никаких *expression* и *statement* быть не может

- В `RecvStmt` можно использовать 1 или 2 *variable*'s в присвоении.
- Можно использовать [short variable declaration](#Lang.md#short-variable-declaration).
- `RecvExpr` должен быть [*receive operation*](#receive-operator) (можно заключать в скобки `()`)

- может быть не более одного `default` *case*, и он может появляться в любом месте *case list*.
- внутри `case` *branch* нельзя использовать `fallthrough` *statement*



Примеры использования `select / case`:

```go
var a []int
var c, c1, c2, c3, c4 chan int
var i1, i2 int
select {
case i1 = <-c1:
	print("received ", i1, " from c1\n")
case c2 <- i2:
	print("sent ", i2, " to c2\n")
case i3, ok := (<-c3):  // то же самое что и: i3, ok := <-c3
	if ok {
		print("received ", i3, " from c3\n")
	} else {
		print("c3 is closed\n")
	}
case a[f()] = <-c4:
	// то же самое что и:
	// case t := <-c4
	//	a[f()] = t
default:
	print("no communication\n")
}

for {  // send random sequence of bits to c
	select {
    case c <- 0:  // обратите внимание: нет statement, не происходит fallthrough, (???) no folding of cases
	  case c <- 1:
	}
}

select {}  // block forever

```



### Порядок выполнения

Выполнение `select` *statement* происходит в следующем порядке:

1. Для всех *case operation*'s необходимо вычислить:

   - *channel* в *receive statement*
   - *channel* в *send statement*
   - правая часть *send statement* (*expression*, которое будет потом *send*)

   Каждое значение вычисляется один раз, в том порядке как они записаны в *select statement*. 

   Результатом вычислений соответственно будут:

   - *channel*'s для *receive*
   - *channel*'s для send
   - результаты вычисления *expression* (некоторое *value*) для *send*. 

   Поэтому в процессе этих вычислений могут произойти *side effect*'s (например, изменения значений переменных) , и они не зависят от того какая из *case branch*'s будет выбрана для исполнения (т.к. эти вычисления происходят до ее выбора).

   *Expression*'s в левой части `RecvStmt`  (т.е. в *short variable declaration* и *assignment*) на этом шаге не вычисляются (мое! но и как они могут быть вычислены, если еще непонятно какой из *case operation* будет выполняться).

2. Составить список из *case branch*'s в случайном порядке для выбора на (Шаг 5). *default branch* всегда помещается в конец списка. *Channel*'s могут повторяться в разных *case operation*'s.

3. Отсортировать все *channel*'s, задействованных в *case operation*'s, чтобы на следующем шаге избежать *deadlock* (с другими *goroutine*'s!!!) (мое: т.е. учесть порядок других *goroutine*'s в *sending goroutine queue* и *receiving goroutine queue* этих *channel*). Пусть `N`- количество задействованных *channel*'s в *case operation*'s. В первых `N` *channel*'s отсортированного результата не должно быть повторяющихся *channel*'s, их порядок назовем *channel lock order*.

4.  *acquire mutex* для всех задействованных *channel*'s в соответствии с *channel lock order*, полученном на предыдущем шаге

5. выбирать *branch*'s в случайном порядке, полученном на (Шаг 2):

   1. если это *case branch* и ее *case operation* – *send* в *closed channel*: *release mutex* для всех *channel*'s в соответствии с обратным *channel lock order* и бросить *panic*. Перейти к (Шаг 12).
   2. если это *case branch* и ее *channel operation* – *non-blocking*, выполнить *channel operation* и *release mutex* для всех *channel*'s в соответствии с обратным *channel lock order*, затем выполните соответствующую *case branch*. *Channel operation* может перевести другую *goroutine* из *blocking state* в *running state*. Перейти к (Шаг 12).
   3. если это *default branch*, то *release mutex* для всех *channel*'s в соответствии с обратным *channel lock order*, затем выполните *default branch*. Перейти к (Шаг 12).

   (Если дошли до сюда, то значит *default branch* отсутствует, и все *case operation*'s являются *blocking*).

6. поместить текущую *goroutine* (??? вместе с информацией соответствующей *case branch*) в *receiving* или *sending goroutine queue* каждого задействованного *channel* в каждой *case operation*. Текущая *goroutine* может быть помещена в *queue* одного и того же *channel* несколько раз, так как *channel* в *case operation*'s может повторяться.

7. перевести текущую *goroutine* в *blocking state* и *release mutex* для всех *channel*'s в соответствии с обратным *channel lock order*.

8. ждать в *blocking state*, пока другая *channel operation* в другой *goroutine* не разбудит текущую *goroutine*, ...

9. текущая *goroutine* пробуждается другой *channel operation* в другой *goroutine*. Это может быть:

   - *close operation* – всегда пробуждает текущую *goroutine*
   - *send/receive operation* – должна существовать *case operation*, взаимодействующая с ней путем передачи *value*. 

   В результате, текущая *goroutine* будет исключена из *receiving/sending goroutine queue* этого *channel*.

10. *acquire mutex* для всех задействованных *channel*'s в соответствии с *channel lock order*

11. исключить текущую *goroutine* из *receiving/sending goroutine queue* каждого задействованного *channel* в каждой *case operation*. Перейти к (Шаг 5).

12. Done



Общие выводы:

- `select` *statement* без каких-либо *case/default branch*'s (`select{}`) переводит текущую *goroutine* в *blocking state forever*.
- *goroutine* может находится сразу в *sending* и *receiving goroutine queue* нескольких *channel*'s одновременно. И даже *goroutine* может находится в *sending* и *receiving goroutine queue* одного *channel* одновременно (мое: при этом она не может получить *value* сама у себя, т.к. может выполнена только одна *channel operation*).









### Примеры

#### Пример 1

В программе обе *case operation*'s – *blocking operation*, выполняется `default` *branch*.

```go
package main

import "fmt"

func main() {
	var c chan struct{} // nil channel
	select {
	case <-c:             // blocking operation
	case c <- struct{}{}: // blocking operation
	default:
		fmt.Println("Печатает это")
	}
}
```

#### Пример 2

Программа с двумя функциями:

- `trySend()` – попытаться отправить в *channel*, в случае неуспеха – ничего не делать
- `tryReceive()` – попытаться получить из *channel*, в случае неуспеха вернуть `-`

```go
package main

import "fmt"

func main() {
	c := make(chan string, 2) // buffered channel
  
  // функция, которая делает попытку send в channel
  // если попытка неуспешна (channel полон) - ничего не делает
	trySend := func(v string) {
		select {
		case c <- v:
		default: // выполняет default, если channel - полон.
		}
	}
  
  // функция, которая делает попытку receive из channel
  // если попыта неуспешна (channel пуст) – возвращает "-"
	tryReceive := func() string {
		select {
		case v := <-c: return v
		default: return "-" // выполняет default, если channel - пуст.
		}
	}
  
	trySend("Hello!") // успешно send
	trySend("Hi!")    // успешно send
	trySend("Bye!")   // channel полон, неуспешно send, но не переходит в blocking state.

  fmt.Println(tryReceive()) // Hello! (успешно receive)
	fmt.Println(tryReceive()) // Hi!    (успешно receive)
  fmt.Println(tryReceive()) // -      (неуспешно receive)
}
```

#### Пример 3

В программе обе *case operation*'s – *non-blocking* (шаги 5.1. и 5.2. алгоритма). Поэтому 50/50 будет выбрана или одна или другая *case branch*.

```go
package main

func main() {
	c := make(chan struct{})
	close(c)
	select {
	case c <- struct{}{}:  // send в closed channel, non-blocking operation, шаг 5.1. алгоритма
		// будет брошена panic, если будет выбран этот case
	case <-c: // receive из closed channel, non-blocking operation, шаг 5.2. алгоритма
		// ничего не прозойдет, если будет выбран этот case
	}
}
```





# Примеры

## Пример 1

Одна *goroutine* отправляет другой значение 9 по *unbuffered channel* `c`. 

*Main goroutine* блокируется до получения по `done` *channel*.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c := make(chan int) // unbuffered bidirectional channel
	go func(ch chan<- int, x int) {
		// ch – send-only channel

		time.Sleep(time.Second)

		// Send 9 и блокироваться до тех пор пока другая goroutine не сделает receive
		ch <- x*x
	}(c, 3)

	done := make(chan struct{}) // unbuffered bidirectional channel

	go func(ch <-chan int) {
		// Блокируется до тех пор пока 9 не будет receive
		n := <-ch

		fmt.Println(n) // 9

		time.Sleep(time.Second)

		// struct{}, чтобы разблокировать main
		done <- struct{}{}
	}(c)

	// Блокируется до тех пор, пока не будет отправлено struct{}
	<-done

	fmt.Println("bye")
}
```

## Пример 2

Пример использования *buffered channel*.

```go
package main

import "fmt"

func main() {
	c := make(chan int, 2) // buffered channel
  
  // Send 3 и 5
	c <- 3
	c <- 5
  
  // Close channel
	close(c)
	fmt.Println(len(c), cap(c)) // 2 2

  // Receive 3
	x, ok := <-c
	fmt.Println(x, ok) // 3 true
	fmt.Println(len(c), cap(c)) // 1 2

  // Receive 5
	x, ok = <-c
	fmt.Println(x, ok) // 5 true
	fmt.Println(len(c), cap(c)) // 0 2

  // Receive из closed channel
	x, ok = <-c
	fmt.Println(x, ok) // 0 false

  // Receive из closed channel
	x, ok = <-c
	fmt.Println(x, ok) // 0 false

	fmt.Println(len(c), cap(c)) // 0 2

  // Close для closed channel
	close(c) // panic!

  // Send в closed channel
	c <- 7 // panic!
}
```



## Пример 3

Все *goroutine*'s ставятся в *receiving goroutine queue* и переходят в *blocking state*. Затем *main goroutine* делает `send "referee"` и одна из *goroutine* (первая в *queue*) переходит в *running state*, выводит строку `referee`, делает `send "<playerName>"` и разблокирует следующую *goroutine* в *receiving goroutine queue*, а сама становится в конец этой очереди.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	var ball = make(chan string) // unbuffered channel
	kickBall := func(playerName string) {
		for {
			// receive значение, которое отправляет goroutine с другим playerName
			fmt.Println(<-ball, "kicked the ball.")

			time.Sleep(time.Second)

			// переходит в blocking state при send своего playerName
			ball <- playerName
		}
	}
	go kickBall("John")
	go kickBall("Alice")
	go kickBall("Bob")
	go kickBall("Emily")

	// send "referee"
	ball <- "referee"

	// unbuffered nil channel для блокировки
	var c chan bool
	<-c               // block forever
}
```



## Future/Promises

Конструкции *future*, *promise* и *delay* в некоторых языках программирования формируют стратегию вычисления, применяемую для параллельных вычислений. С их помощью описывается объект (*promise*), к которому можно обратиться за результатом (*future*), вычисление которого может быть не завершено на данный момент. *Promise* — это функция, которая присваивает значение (*future*). *Future* — возвращаемое значение асинхронной функции *promise*. 

Это позволяет отделить значение (*future*) от процесса самого вычисления (*promise*), что позволяет запустить несколько процессов вычислений параллельно. 

### Реализация через возврат *receive-only channel* из функции



Значения двух аргументов функции `sum()` запрашиваются одновременно. Каждая из двух операций приема канала будет блокироваться до тех пор, пока операция отправки не будет выполнена на соответствующем канале. Для получения окончательного результата требуется около трех секунд вместо шести.

```go
package main

import (
	"time"
	"math/rand"
	"fmt"
)

// Promise -  функция выполняет долгую работу по генерации числа int32
// При этом она возвращает channel, в который будет возвращено вычисленное значение (future)
func longTimeRequest() <-chan int32 {
	r := make(chan int32)

	go func() {
		// Некоторая долгая обработка
		time.Sleep(time.Second * 3)
		r <- rand.Int31n(100)
	}()

	return r
}

// функция, которая использует вычисленные значения (futures)
func sum(a, b int32) int32 {
	return a + b
}

func main() {
  // Получение двух promises - channel's для получения значений
  // Процесс вычислений будет запущен параллельно, но самих значений еще нет
	a, b := longTimeRequest(), longTimeRequest()
  
  // Использование двух значений, которые будут вычислены в будущем (future)
  // receive operations будут заблокированы до тех пор пока send operation не будет выполнена для 
  // соответствующего channel (процесс вычислений идет параллельно)
	fmt.Println(sum(<-a, <-b)) 
}
```







TODO

https://go101.org/article/channel.html

https://habr.com/ru/post/490336/

https://medium.com/rungo/anatomy-of-channels-in-go-concurrency-in-go-1ec336086adb

https://www.velotio.com/engineering-blog/understanding-golang-channels



https://golang.org/doc/effective_go#channels

https://go101.org/article/channel-use-cases.html

https://habr.com/ru/post/308070/

https://golangbot.com/channels/

https://russianblogs.com/article/3267763547/













t.cJ9a8MTKrVYK-ZDYjyiPpbMe9w_sm7I4H5XGfD5Rrqh7VJYSQmasQd59LLdu1J1L6DVCvsDrnFXUGhIXIpo_fg