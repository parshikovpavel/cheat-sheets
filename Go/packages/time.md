# `time`

*Package* `time` предоставляет функции для измерения и отображения времени.



## Internals

Принцип работы таймеров https://blog.gopheracademy.com/advent-2016/go-timers/



### Замер времени исполнения

```go
start := time.Now()
// код, время исполнения которого замеряется
log.Info(time.Since(start))
```



## Структура

### `After()`

```go
func After(d Duration) <-chan Time
```

`After()` ожидает истечения времени, а затем *send* текущее *time* по возвращенному *channel*. Это эквивалентно `NewTimer(d).C`. *Underlying* `Timer`  не вычищается *garbage collector*'ом, пока *timer* не сработает. Если эффективность важна, используйте `NewTimer()` вместо этого и вызовите `Timer.Stop()`, если *timer* больше не нужен.

Пример (аналогичный пример рассматривается при описании [channel](../types/channel.md#timeout)):

```go
package main

import (
	"fmt"
	"time"
)

var c chan int

func handle(int) {}

func main() {
	select {
	case m := <-c:
		handle(m)
	case <-time.After(10 * time.Second):
		fmt.Println("timed out")
	}
}
```

#### Timer leak

Важно!!! обратить внимание, что

> *Underlying* `Timer`  (созданный через `time.After()`) не вычищается *garbage collector*'ом, пока *timer* не сработает. Если эффективность важна, используйте `NewTimer()` вместо этого и вызовите `Timer.Stop()`, если *timer* больше не нужен.

##### Допустимое использование `After()`

```go
select {
  case <-time.After(time.Second):
     // сделать что-то после 1 секунды
}
```

После завершения блока `select`, *timer* гарантированно сработал, а значит ресурсы для него были очищены

##### Ошибочное использование `After()`

Часто используют `After()` таким очевидным способом:

```go
select {
  case <-time.After(time.Second):
     // сделать что-то после 1 секунды
  case <-ctx.Done():
     // сделать что-то, когда context будет cancel
     // ресурсы, выделенные функцией `time.After()` не будут собраны garbage collector'ом
  }
```

Или например так в цикле, для извлечения *value* из *channel* `in` в цикле `for`: 

```go
	for {
		select {
		case s := <-in:
			// обработка `s`
		case <-time.After(5 * time.Minute):
			// сделать что-то при простое в 5 минут
		}
	}
```

В этих примерах, если `ctx.Done()` или `s := <-in` сработают раньше чем *timer*, то *timer* не останавливается и ресурсы не освобождаются — возникает *memory leak*.

При 60k *value*'s в секунду в *channel*, получаем `60k * 60 * 5 = 18M` активных *timer*'ов и еще некоторое количество тех, которые должен собрать *garbage collector*. На практике, это привело к потреблению 9Гб(!) памяти под *timer*'ы.

##### Правильное использование `After()`

Правильное решение – использовать `NewTimer()` и вызывать `Timer.Stop()`, если *timer* больше не нужен.

```go
  delay := time.NewTimer(time.Second)
 
  select {
  case <-delay.C:
     // сделать что-то после 1 секунды
  case <-ctx.Done():
     // сделать что-то, когда context будет cancel
    // (конструкция ниже из документации по `Timer.Stop()`)
     if !delay.Stop() { // остановить timer
        // если timer был уже ранее остановлен, тогда очищаем channel.
        <-delay.C
     }    
  }
```

В этом случае, если `ctx.Done()` произойдет раньше чем сработает *timer*, то его ресурсы освобождаются с помощью функции `delay.Stop()`функции. 

Может случиться так, что сработает `<-ctx.Done()` и сразу после этого *timer* останавливается. В этом случае `delay.Stop() == false`, при этом мы не сделали *read* из `delay.C`, из *channel* нужно явно вычитать *value*.



### `Tick()`

```go
func Tick(d Duration) <-chan Time
```

`Tick()` – это обертка для `NewTicker()`, предоставляющая доступ только к *channel* внутри `Ticker`. 

`Tick()` необходимо использовать только в тех случаях, когда не нужно выключать `Ticker`. Без возможности выключения, *underlying* `Ticker` не может быть удален *garbage collector*'ом; он «leak». 

```go
package main

import (
	"fmt"
	"time"
)

func statusUpdate() string { return "" }

func main() {
	c := time.Tick(5 * time.Second)
	for next := range c {
		fmt.Printf("%v %s\n", next, statusUpdate())
	}
}
```

Для медленных *receiver*'ов отправляет время, которое *receiver* может принять позже (т.е. *receiver* принимает не актуальный *tick*, а старый). А потом отправляет уже актуальное время. Какая используется логика для реализации такого поведения можно посмотреть в кастомной реализации *ticker*'а в [channel](../types/channel.md#ticker)

```go
func main() {
	t := time.Now()
	c := time.Tick(5 * time.Second)
	fmt.Println((<-c).Sub(t), time.Since(t))      // 5s, 5s
	time.Sleep(12 * time.Second)
	fmt.Println((<-c).Sub(t), time.Since(t))      // 10s, 17s – receive старый tick, который произошел в 10s
	fmt.Println((<-c).Sub(t), time.Since(t))      // 20s, 20s
}
```







### `Now()`

```go
func Now() Time
```

Функция, возвращает текущее время.

### `Since()`

```go
func Since(t Time) Duration
```

Возвращает время, прошедшее с момента `t`. Сокращение для `time.Now().Sub(t)`.





### `Sleep()`

```go
func Sleep(d Duration)
```

Приостанавливает текущую *goroutine* как минимум на время `d`.

Пример:

```go
func main() {
	time.Sleep(100 * time.Millisecond)
}
```



### `type Duration`

```go
type Duration int64
```

`Duration` представляет собой время, прошедшее между двумя моментами, в виде числа наносекунд в формате `int64`. Представление ограничивает наибольшую представляемую *duration* приблизительно 290 годами.



#### `ParseDuration()`

```go
func ParseDuration(s string) (Duration, error)
```

`ParseDuration()` парсит *duration string*.  *Duration string* - это последовательность десятичных цифр. Можно использовать:

- знак (`-`)
- дробные числа
- суффикс единиц измерения

Примеры:

- `"300ms"`
- `"-1.5h"`
- `"2h45m"`

Допустимые единицы времени: 

- `"ns"`
- `"us"` (или `"µs"`)
- `"ms"`
- `"s"`
- `"m"`
- `"h"`

Пример:

```go
func main() {
	hours, _ := time.ParseDuration("10h")
	complex, _ := time.ParseDuration("1h10m10s")

	fmt.Println(hours)		// 10h0m0s
	fmt.Println(complex)	// 1h10m10s
}
```







### `type Time`

```go
type Time struct {
	// contains filtered or unexported fields
}
```

Тип, описывает момент времени с точностью до наносекунды.

*Zero value* для типа `Time` является `January 1, year 1, 00:00:00.000000000 UTC`. Поскольку на практике это время маловероятно, метод `IsZero()` дает простой способ определения времени, которое не было явно инициализировано.

#### `IsZero()`

```go
func (t Time) IsZero() bool
```

`IsZero()` сообщает, представляет ли `t` нулевой момент времени, January 1, year 1, 00:00:00 UTC.



### `type Ticker`

```go
type Ticker struct {
	C <-chan Time // channel, в который будут send "ticks"
	// contains filtered or unexported fields
}
```

`Ticker` содержит *channel* `С`, в который будут *send “ticks”* через определенные промежутки времени.

#### `NewTicker()`

```go
func NewTicker(d Duration) *Ticker
```

`NewTicker()` возвращает новый `Ticker`, содержащий *channel* `C`, который будет *send time* в *channel* после каждого *tick*'а. Период *tick*'ов указываются через аргумент *duration* `d`. *Ticker* будет настраивать (??? что значит настраивать?) *time interval*  или дропнет *tick*, если *receiver*'ы – медленные. Необходимо сделать `Ticker.Stop()`, чтобы освободить связанные ресурсы.

```go
import (
	"fmt"
	"time"
)

func main() {
	ticker := time.NewTicker(time.Second)
	defer ticker.Stop()
	done := make(chan bool)
	go func() {
		time.Sleep(10 * time.Second)
		done <- true
	}()
	for {
		select {
		case <-done:
			fmt.Println("Done!")
			return
		case t := <-ticker.C:
			fmt.Println("Current time: ", t)
		}
	}
}
```



