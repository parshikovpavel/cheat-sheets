# Graceful shutdown

https://www.youtube.com/watch?v=ASiF-AH7GFc&t=415s

Существует виды завершения приложения:

- hard (жесткое)
- graceful (изящное)

## Hard

Выходим из приложения при любой проблеме с помощью:

- `os.Exit()`
- `panic()`
- `log.Fatal()`

Преимущества:

- сделать очень просто



## Graceful

Сделать сложно. Нужно знать:

- какие операции произвести
- в каком порядке их произвести (закрыть соединение после того как будут завершены все *goroutine*'s, которые в него пишут

Преимущества:

- корректное завершение запущенных *goroutine*'s
- контроль – правильное закрытие коннекта к БД, файла, возвращение *message* в *queue*
- экономия ресурсов – возможность дождаться выполнение запроса, и в этом случае его не пришлось бы запускать повторно. В случае *hard shutdown* – например, *message* не был бы обработан, *message* вернулся в очередь и обрабатывался бы повторно.



Изящное завершение работы (Graceful Shutdown) важно для любого длительного процесса, особенно для того, который обрабатывает состояние. Например, если вы хотите завершить работу веб-сервера с тысячами соединений, но не дожидаетесь окончания запросов. Изящное завершение работы не только положительно сказывается на пользовательском опыте, но и облегчает внутренние операции, что приводит к более счастливым инженерам и менее напряженным SRE.

Изящное завершение работы заключается в том, что программа завершается после:

- Завершения всех незавершенных процессов (веб-запрос, циклы) - не должно запускаться никаких новых процессов и не должно приниматься никаких новых веб-запросов.
- Закрытия всех открытых соединений с внешними сервисами и базами данных.

Для изящного завершения работы необходимо выяснить несколько моментов:

- Когда следует завершить работу - все ли процессы завершены, и как это узнать? Что делать, если процесс застрял?
- Как мы можем сообщить процессам о необходимости завершения работы, а также узнать, когда они это сделают?



## Антипаттерны

### Искусственное блокирование

Первый антипаттерн - это идея блокировки основной горутины без фактического ожидания чего-либо. Приведем пример игрушечной реализации:

```go
func KeepProcessAlive() {
	var ch chan int
	<-ch
}

func main() {


	//...
	KeepProcessAlive()
}
```

### os.Exit()

Вызов `os.Exit(1)` в то время, когда другие горутины еще работают, по сути, равносилен SIGKILL - нет возможности закрыть открытые соединения и завершить обработку запросов в полете.

```go
go func() {
	<-ch
	os.Exit(1)
}()

go func () {

	for /*...*/ {

	}
}()
```

## Как сделать его изящнее в Go

Для того чтобы изящно завершить работу сервиса, необходимо понять две вещи:

1. Как дождаться выхода всех запущенных горутин
2. Как передать сигнал о завершении работы нескольким горутинам.

Go предоставляет все необходимые инструменты для правильной реализации. Давайте рассмотрим их более подробно.

### Ожидание завершения работы горутин

Go предоставляет достаточно способов управления параллелизмом. Посмотрим, какие варианты ожидания горутин доступны.

#### Использование канала

Самое простое решение - использование примитива *channel*.

1. Создаем пустой *struct channel* `make(chan struct{}, 1)` 
2. Каждая дочерняя горутина **должна публиковать в канале свои данные** (здесь может быть полезен принцип defer).
3. Родительская горутина **должна читать из канала столько раз, сколько ожидаемых горутин**.



```go
func run(ctx context.Context) {
	wait := make(chan struct{}, 1)

	go func() {
		defer func() {
			wait <- struct{}{}
		}()
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Break the loop")
				return
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

	go func() {
		defer func() {
			wait <- struct{}{}
		}()
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Break the loop")
				return
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

	// wait for two goroutines to finish
	<-wait
	<-wait

	fmt.Println("Main done")
}
```

В основном это полезно при ожидании выполнения одной горутины.

#### С WaitGroup

Решение с каналом может быть несколько некрасивым, особенно в случае нескольких горутин.

[sync.WaitGroup](https://pkg.go.dev/sync#WaitGroup/) - это пакет стандартной библиотеки, который может быть использован как более идиоматичный способ достижения вышеописанного.

```go
func run(ctx context.Context) {
	var wg sync.WaitGroup

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Break the loop")
				return
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Break the loop")
				return
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

	wg.Wait()
	fmt.Println("Main done")
}
```

#### С помощью errgroup

Пакет [sync/errgroup](https://pkg.go.dev/golang.org/x/sync/errgroup/) предоставляет более эффективный способ решения этой задачи.

- Два метода `errgroup` `.Go` и `.Wait` более читабельны и просты в сопровождении по сравнению с `WaitGroup`.
- Кроме того, как следует из названия, он выполняет распространение ошибки и отменяет контекст, чтобы в случае ошибки завершить другие горутины.

```go
func run(ctx context.Context) {
	g, gCtx := errgroup.WithContext(ctx)
	g.Go(func() error {
		for {
			select {
			case <-gCtx.Done():
				fmt.Println("Break the loop")
				return nil
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	})

	g.Go(func() error {
		for {
			select {
			case <-gCtx.Done():
				fmt.Println("Break the loop")
				return nil
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	})

	err := g.Wait()
	if err != nil {
		fmt.Println("Error group: ", err)
	}
	fmt.Println("Main done")
}
```

## Завершение процесса

Даже если мы разобрались с тем, как правильно передавать состояние процессов и ожидание, нам все равно необходимо реализовать завершение. Посмотрим, как это можно сделать на простом примере, введя все необходимые примитивы Go.

Начнем с очень простого примера "Hello in a loop":

```go
func main() {
	for {
		time.Sleep(1 * time.Second)
		fmt.Println("Hello in a loop")
	}
}
```

### Внедрение обработки сигналов

Прослушивание сигнала ОС для остановки выполнения:

```go
exit := make(chan os.Signal, 1) // we need to reserve to buffer size 1, so the notifier are not blocked
signal.Notify(exit, os.Interrupt, syscall.SIGTERM)
```

- Нам необходимо использовать `os.Interrupt` для изящного завершения работы по `Ctrl+C`, который является SIGINT
- `syscall.SIGTERM` - обычный сигнал завершения работы и сигнал по умолчанию (может быть [изменен](https://docs.docker.com/engine/reference/builder/#stopsignal)) для контейнеров [docker](https://docs.docker.com/engine/reference/commandline/stop/), который также используется в [kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination/).
- Подробнее о `signal` можно прочитать [в документации к пакету](https://pkg.go.dev/os/signal/) и [рассмотреть на примере](https://gobyexample.com/signals).

### Разрыв цикла

Теперь, когда у нас есть способ перехвата сигналов, необходимо найти способ прервать цикл.

#### Неблокирующий выбор канала

`select` дает возможность в каждом конкретном случае `case` потреблять сигнал из нескольких каналов.

Для лучшего понимания можно просмотреть следующие ресурсы:

- <https://gobyexample.com/non-blocking-channel-operations>
- <https://tour.golang.org/concurrency/5>
- <https://gobyexample.com/timeouts>

Наш простой цикл теперь останавливается по сигналу завершения:

```go
func main() {
	c := make(chan os.Signal, 1) // we need to reserve to buffer size 1, so the notifier are not blocked
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)

	for {
		select {
		case <-c:
			fmt.Println("Break the loop")
			return
		case <-time.After(1 * time.Second):
			fmt.Println("Hello in a loop")
		}
	}
}
```

*Примечание: нам пришлось заменить* `time.Sleep(1*time.Second)` на `time.After(1*time.Second)`

### Как это сделать с помощью Context

Context - это очень полезный интерфейс в go, который должен использоваться и распространяться во всех блокирующих функциях. Он позволяет распространять отмену по всей программе.

Считается хорошей практикой, чтобы `ctx context.Context` был первым аргументом в каждом методе или функции, которые прямо или косвенно используются для внешних зависимостей.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/e61/9ca/091/e619ca0918118d152d42c330a6399e24.jpg)

Очень подробная статья о контексте: <https://go.dev/blog/context>

### Проблема разделения каналов

Рассмотрим, как свойства контекста могут помочь в более сложной ситуации.

*Параллельное выполнение нескольких циклов с использованием каналов (контрпример):*

```go
// COUNTER EXAMPLE, DO NOT USE THIS CODE
func main() {
	exit := make(chan os.Signal, 1)
	signal.Notify(exit, os.Interrupt, syscall.SIGTERM)

	// This will not work as expected!!
	var wg sync.WaitGroup

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-exit: // Only one go routine will get the termination signal
				fmt.Println("Break the loop: hello")
				break
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-exit: // Only one go routine will get the termination signal
				fmt.Println("Break the loop: ciao")
				break
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

	wg.Wait()
	fmt.Println("Main done")
}
```

*Почему это не работает?*

Go-каналы не работают **широковещательно**, только одна горутина получит один `os.Signal`. Также нет никакой гарантии, какая именно горутина его получит.

`wait := make(chan struct{}, 2)`

Но контекст может помочь нам заставить работать вышеописанное, давайте посмотрим, как это сделать.

#### Использование контекста для завершения работы

Попробуем решить эту проблему, введя [context.WithCancel](https://pkg.go.dev/context#WithCancel/)

```go
func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go func() {
		exit := make(chan os.Signal, 1)
		signal.Notify(exit, os.Interrupt, syscall.SIGTERM)
		<-exit
		cancel()
	}()

	var wg sync.WaitGroup

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Break the loop")
				return
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Break the loop")
				return
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

	wg.Wait()
	fmt.Println("Main done")
}
```

По сути, функция `cancel()` транслируется всем горутинам, вызывающим `.Done()`.

*Канал Done возвращаемого контекста закрывается при вызове возвращаемой функции cancel или при закрытии канала Done родительского контекста, в зависимости от того, что произойдет раньше.*

### NotifyContext

В go 1.16 в пакете signal появился новый полезный метод [singal.NotifyContext](https://pkg.go.dev/os/signal#NotifyContext):

```go
func NotifyContext(parent context.Context, signals ...os.Signal) (ctx context.Context, stop context.CancelFunc)
```

*NotifyContext возвращает копию родительского контекста, которая помечается как выполненная (ее канал Done закрыт) при поступлении одного из перечисленных сигналов, при вызове возвращаемой функции stop или при закрытии канала Done родительского контекста, в зависимости от того, что произойдет раньше.*

Использование NotifyContext позволяет упростить приведенный выше пример до:

```go
func main() {
	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
	defer stop()

	var wg sync.WaitGroup

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Break the loop")
				return
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Break the loop")
				return
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

	wg.Wait()
	fmt.Println("Main done")
}
```

*Полный рабочий пример можно найти* [*в нашем репозитории примеров*](https://github.com/rudderlabs/graceful-shutdown-examples/tree/main/signal/)

## Общие библиотеки

### HTTP-сервер

В приведенных выше примерах для упрощения использовался цикл `for`, но давайте рассмотрим нечто более практичное.

Во время non-graceful shutdown, HTTP-запросы могут столкнуться со следующими проблемами:

- Они никогда не получают ответа, и поэтому у них наступает таймаут.
- Некоторый прогресс был достигнут, но он прерывается на полпути, что приводит к пустой трате ресурсов или несоответствию данных, если транзакции используются неправильно.
- Соединение с внешней зависимостью закрывается другой горутиной, и запрос не может продвигаться дальше.

*⚠️* ***Изящное завершение работы HTTP-сервера очень важно.*** *В облачной среде сервисы/подсистемы завершаются несколько раз в течение дня либо для автомасштабирования, либо для применения конфигурации, либо для развертывания новой версии сервиса. Таким образом, влияние прерывания или таймаута запросов может быть существенным для SLA сервиса.*

К счастью, go предоставляет возможность изящно завершить работу HTTP-сервера.

Давайте посмотрим, как это делается:

```go
func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go func() {
		c := make(chan os.Signal, 1) // we need to reserve to buffer size 1, so the notifier are not blocked
		signal.Notify(c, os.Interrupt, syscall.SIGTERM)

		<-c
		cancel()
	}()

	db, err := repo.SetupPostgresDB(ctx, getConfig("DB_DSN", "root@tcp(127.0.0.1:3306)/service"))
	if err != nil {
		panic(err)
	}

	httpServer := &http.Server{
		Addr:    ":8000",
	}

	g, gCtx := errgroup.WithContext(ctx)
	g.Go(func() error {
		return httpServer.ListenAndServe()
	})
	g.Go(func() error {
		<-gCtx.Done()
		return httpServer.Shutdown(context.Background())
	})

	if err := g.Wait(); err != nil {
		fmt.Printf("exit reason: %s \n", err)
	}
}
```

Мы используем две горутины:

1. выполнить `httpServer.ListenAndServe()` как обычно
2. дождаться `<-gCtx.Done()` и затем вызвать `httpServer.Shutdown(context.Background())`

Для того чтобы понять, как это работает, необходимо ознакомиться с документацией по пакету:

- Shutdown изящно завершает работу сервера, **не прерывая активных соединений**.
- Красиво, но как?
- Shutdown работает следующим образом: сначала закрываются все открытые слушатели, затем закрываются все простаивающие соединения, а затем неопределенное время ожидается, пока соединения вернутся в состояние простоя, после чего происходит отключение.
- Почему я должен предоставлять контекст?
- Если предоставленный контекст истекает до завершения выключения, Shutdown возвращает ошибку контекста, в противном случае возвращается любая ошибка, возникшая при закрытии базового слушателя (слушателей) сервера.
- В примере мы выбрали контекст `context.Background()`, который не имеет срока действия.

#### Отмена долго выполняющихся запросов

При вызове метода `.Shutdown` сервис перестает принимать новые соединения и ожидает завершения уже существующих, после чего может вернуться к функции `.ListenAndServe()`.

Бывают случаи, когда http-запросы требуют достаточно длительного времени для завершения. Это может быть, например, долго выполняющееся задание или соединение через websocket.

Как же лучше всего завершить их изящно и не зависнуть в ожидании завершения?

Ответ состоит из двух частей:

1. Во-первых, необходимо извлечь контекст из http.Request `ctx := req.Context()` и использовать этот контекст для завершения долго выполняющегося процесса.
2. Используйте [BaseContext](https://pkg.go.dev/net/http#Server) (представленный в go1.13), чтобы передавать ваш основной ctx в качестве контекста в каждом запросе.

`BaseContext` опционально задает функцию, которая возвращает базовый контекст для входящих запросов на данном сервере.

Предоставленный `Listener` - это конкретный `Listener`, который собирается начать принимать запросы.

Если `BaseContext` равен `nil`, то по умолчанию используется `context.Background()`.

Если не `nil`, то должен быть возвращен не `nil` контекст.

В приведенном ниже примере фиктивный http-обработчик продолжает печатать в stdout `Hello in a loop`, который остановится либо когда запрос будет отменен, либо когда экземпляр получит сигнал о завершении.

```go
func main() {
	mainCtx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
	defer stop()

	httpServer := &http.Server{
		Addr: ":8000",
		Handler: http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			ctx := r.Context()

			for {
				select {
				case <-ctx.Done():
					fmt.Println("Graceful handler exit")
					w.WriteHeader(http.StatusOK)
					return
				case <-time.After(1 * time.Second):
					fmt.Println("Hello in a loop")
				}
			}
		}),
		BaseContext: func(_ net.Listener) context.Context {
			return mainCtx
		},
	}
	g, gCtx := errgroup.WithContext(mainCtx)
	g.Go(func() error {
		return httpServer.ListenAndServe()
	})
	g.Go(func() error {
		<-gCtx.Done()
		return httpServer.Shutdown(context.Background())
	})

	if err := g.Wait(); err != nil {
		fmt.Printf("exit reason: %s \n", err)
	}
}
```

Полный рабочий пример можно найти [в нашем репозитории примеров](https://github.com/rudderlabs/graceful-shutdown-examples/tree/main/httpserver/), не стесняйтесь экспериментировать, комментируя `BaseContext` или `httpServer.Shutdown`.

### HTTP-клиент

В стандартных библиотеках Go предусмотрен способ передачи контекста при выполнении HTTP-запроса: [NewRequestWithContext](https://pkg.go.dev/net/http#NewRequestWithContext).

Посмотрим, как можно рефакторизовать следующий код для его использования:

```go
resp, err := netClient.Post(uri, "application/json; charset=utf-8",
					bytes.NewBuffer(payload))
//...
```

Эквивалент с передачей `ctx`:

```go
req, err := http.NewRequestWithContext(ctx, "POST", uri, bytes.NewBuffer(payload))
if err != nil {
	return err
}
req.Header.Set("Content-Type", "application/json; charset=utf-8")

resp, err := netClient.Do(req)
//...
```

Следующие приемы необходимы для более сложных случаев использования. Например, если вы используете пул воркеров или у вас есть цепочка зависимостей компонентов, которые должны отключаться по порядку.

### Слив каналов воркеров

При наличии горутин воркеров, которые потребляют/производят из/в канал, необходимо позаботиться о том, чтобы при завершении процесса в каналах не оставалось никаких элементов. Для этого необходимо использовать метод go `close` для канала. Вот отличный [обзор по закрытию каналов](https://gobyexample.com/closing-channels), а также более [подробная статья](https://go101.org/article/channel-closing.html) на эту тему.

О закрытии канала следует помнить две вещи:

- Запись в закрытый канал приведет к панике
- При чтении для канала можно использовать `value, ok <- ch`. При чтении из закрытого канала будут возвращены все буферизованные элементы. Как только элементы буфера будут "исчерпаны", канал вернет нулевое `value`, а `ok` будет равен `false`. *Примечание: Пока в канале еще есть элементы, ok будет истинным.*
- В качестве альтернативы можно сделать `range` на канале `for value := range ch {/*...*/}` . В этом случае цикл for остановится, когда в канале больше не останется элементов, и канал будет закрыт. Это гораздо красивее, чем описанный выше подход, но не всегда возможно.

Из сказанного выше следует вывод:

- Если **в канал пишет один воркер**, закрывайте канал сразу после завершения работы:

```go
go func() {
	defer close(ch) // close after write is no longer possible
	for {
		select {
			case <-ctx.Done():
				return
			//...
			case ch <- value: // write to the channel only happens inside the loop
		}
	}
}()
```

- Если **в один и тот же канал записывают несколько воркеров**, закройте канал, дождавшись окончания работы всех воркеров:

```go
g, gCtx := errgroup.WithContext(ctx)
ch := make( /*...*/ )     // channel will be written from multiple workers
for w := range workers { // create n number of workers
	g.Go(func() error {
		return w.Run(ctx, ch) // workers will publish
	})
}
g.Wait()  // we need to wait for all workers to stop
close(ch) // and then close the channel
```

- Если вы читаете из канала, выходите только тогда, когда в канале больше нет данных. По сути, ответственность за прекращение чтения лежит на писателе, который закрывает канал:

```go
for v := range ch {

}

// or
for {
	select {
	case v, ok := <-ch:
		if !ok { // nothing left to read
			return
		}
		foo(v) // process `v` normally
	case /*...*/ :
		//...
	}
}
```

- Если воркер одновременно читает и пишет, то он должен остановиться, когда в канале, из которого он читает, больше нет данных, и закрыть пишущий канал.

## Изящные методы

Мы уже рассмотрели несколько приемов, позволяющих изящно завершить долго выполняющийся код. Также полезно рассмотреть, как компоненты могут предоставлять экспортируемые методы, которые можно вызывать и затем облегчать их изящное завершение.

### Блокирование с помощью ctx

Это наиболее распространенный подход, который легче всего понять и реализовать.

- Вы вызываете метод
- Вы передаете ему контекст
- Метод блокируется
- Возвращается в случае ошибки или при отмене контекста / таймауте.

```go
// calling:
err := srv.Run(ctx, /*...*/)

// implementation
func (srv *Service) Run(ctx context.Context, /*...*/) error {
	//...
	for {
		//...
    select {
			case <- ctx.Done()
				return ctx.Err() // Depending on our business logic,
												 //   we may or may not want to return a ctx error:
												 //   https://pkg.go.dev/context#pkg-variables
		}
	}
}
```

### Настройка/выключение

Бывают случаи, когда блокировка с помощью ctx-кода не является лучшим подходом. Это тот случай, когда мы хотим получить больший контроль над тем, когда произойдет `.Shutdown`. Этот подход немного сложнее, и, кроме того, существует опасность, что люди забудут вызвать `.Shutdown`.

#### Пример использования

Приведенный ниже код демонстрирует, почему этот паттерн может быть полезен. Мы хотим убедиться, что `db Shutdown` произойдет только после того, как `Service` перестанет работать, поскольку работа службы зависит от работы базы данных.

Вызывая `db.Shutdown()` на `defer`, мы обеспечиваем его запуск после возврата `g.Wait`:

```go
// calling:
func () {
   err := db.Setup() // will not block
   defer db.Shutdown()

   svc := Service{
     DB: db
   }

   g.Run(/*...*/
     svc.Run(ctx, /*...*/)
   )
   g.Wait()
}
```

#### Пример реализации

```go
type Database struct {
	//...
	cancel func()
	wait   func() error
}

func (db *Database) Setup() error {
	//...

	ctx, cancel := context.WithCancel(context.Background())
	g, gCtx := errgroup.WithContext(ctx)

	db.cancel = cancel
	db.wait = g.Wait

	for {
		//...
		select {
		case <-ctx.Done():
			return ctx.Err() // Depending on our business logic,
											 //   we may or may not want to return a ctx error:
											 //   https://pkg.go.dev/context#pkg-variables
		}
	}
}

func (db *Database) Shutdown() error {
	db.cancel()
	return db.wait()
}
```

## Заключительные размышления

Изящное завершение долго работающих сервисов - важный паттерн, который рано или поздно придется реализовать. Это особенно актуально для таких систем, как RudderStack, которые выступают в роли промежуточного звена, где существует множество соединений с внешними сервисами и одновременно обрабатываются большие объемы данных.

Go предлагает все необходимые инструменты для реализации этого паттерна, и выбор подходящих из них во многом зависит от конкретного случая использования. Я хотел написать эту статью в качестве руководства, которое поможет выбрать инструменты, подходящие именно для вашего случая.

## Бонус-трек - ответы от ChatGPT

```go
func main() {
	c := make(chan os.Signal, 1) // we need to reserve to buffer size 1, so the notifier are not blocked
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)

	for {
		select {
		case <-c:
			fmt.Println("Break the loop")
			return
		case <-time.After(1 * time.Second):
			fmt.Println("Hello in a loop")
		}
	}
}
```

> А зачем тут буферированный канал? Вроде работает одинаково, если небуферированный.

Буферизированный канал используется в этом коде для обеспечения того, что сигналы операционной системы (os.Interrupt, syscall.SIGTERM) могут быть корректно обработаны, даже если главная горутина занята и не может немедленно принять сигнал.

Если бы канал был небуферизированным, функция signal.Notify могла бы заблокироваться, если главная горутина не готова принять сигнал. Это происходит потому, что отправка в небуферизированный канал блокируется до тех пор, пока другая горутина не будет готова принять данные из канала.

В случае буферизированного канала размером 1, функция signal.Notify может поместить сигнал в канал без блокировки, даже если главная горутина в данный момент не может принять сигнал. Это обеспечивает более надежную обработку сигналов операционной системы.

Однако вы правы в том, что в данном конкретном случае разница между буферизированным и небуферизированным каналами может быть несущественной, поскольку основная горутина регулярно проверяет канал на наличие новых сигналов. Но использование буферизированного канала является хорошей практикой для обработки сигналов операционной системы.

---

```go
	ch := make(chan os.Signal, 2)
	signal.Notify(ch, os.Interrupt, syscall.SIGTERM)
	go func() {
		<-ch
		log.Print("Stop...")
		os.Exit(1)
	}()
```

> А зачем тут буфер размером 2?

Буферизированный канал с размером буфера 2 используется в этом коде для обработки сигналов операционной системы. Размер буфера 2 позволяет каналу хранить два сигнала, если они приходят очень быстро один за другим, и главная горутина не успевает их обработать.

В данном случае, если придут два сигнала подряд (например, два сигнала прерывания), они оба будут помещены в буфер канала и затем обработаны горутиной. Если бы канал был небуферизированным или имел размер буфера 1, то второй сигнал мог бы быть потерян, если бы он пришел, пока горутина обрабатывала первый сигнал.

Однако в большинстве случаев размер буфера 1 будет достаточен, так как сигналы прерывания обычно не приходят так быстро. Размер буфера 2 может быть использован как дополнительная мера предосторожности.

---

> В GoLang cчитается хорошей практикой, чтобы `ctx context.Context` был первым аргументом в каждом методе или функции, которые прямо или косвенно используются для внешних зависимостей. Но в ReactJS контекст не маячит перед глазами. Есть ли подобные предложения для GoLang?

Да, в Go действительно считается хорошей практикой передавать `context.Context` как первый аргумент в функциях, которые взаимодействуют с внешними зависимостями. Это позволяет контролировать время выполнения этих функций и отменять их при необходимости.

Однако в отличие от ReactJS, где контекст часто используется для передачи данных между компонентами, в Go контекст обычно используется для передачи данных управления, таких как таймауты и отмены операций, а не для передачи состояния приложения.

В Go нет строгого требования к тому, чтобы `context.Context` всегда был виден. Вместо этого рекомендуется использовать `context.Context` только тогда, когда это действительно необходимо, чтобы избежать ненужной сложности и улучшить читаемость кода.

Важно помнить, что `context.Context` должен использоваться ответственно. Например, его не следует хранить в структурах или использовать для передачи опциональных параметров.

> Казалось бы, есть `context.Background()`, или же у `http.Server` есть `.BaseContext` - т.е. какой-то синглтон. Тогда зачем контекст таскать? А для обеспечения "dependency injection". Чтобы выполнять отмены в Graceful Shutdown обратно по графу зависимостей. Правильно я понимаю? Вот! У нас же есть всякие `google/wire` и `uber-go/fx`, а я хочу подобное для контекста.

---

> О закрытии канала следует помнить, что можно сделать range на канале `for value := range ch {/*...*/}`. В этом случае цикл for остановится, когда в канале больше не останется элементов, и канал будет закрыт. Но этот подход не всегда возможен. А почему?

Цикл `for range` по каналу в Go действительно остановится, когда канал будет закрыт и все его элементы будут прочитаны. Однако этот подход не всегда применим по нескольким причинам:

1. **Многопоточность**: Если вы работаете с несколькими горутинами, которые могут читать из одного и того же канала, использование `for range` может привести к состоянию гонки. В этом случае одна горутина может закрыть канал после того, как другая горутина проверила, что канал открыт, но перед тем, как она попыталась прочитать из него.
2. **Непрерывная обработка**: Если ваш код должен постоянно обрабатывать входящие данные (например, в сервере), то вы не захотите закрывать канал после обработки каждого сообщения. Вместо этого вы захотите оставить канал открытым и продолжить читать из него новые сообщения.
3. **Неизвестное количество сообщений**: Если вы не знаете заранее, сколько сообщений будет отправлено в канал, то использование `for range` может быть проблематичным. Если вы закроете канал слишком рано, вы можете пропустить некоторые сообщения.

В этих случаях вместо использования `for range` вы можете использовать `select` для чтения из канала с возможностью обработки других событий или таймаутов.

