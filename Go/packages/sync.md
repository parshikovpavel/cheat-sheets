## `sync`

Предоставляет базовые примитивы синхронизации, такие как *mutex*. За исключением типов `Once` и `WaitGroup`, большинство из них предназначены для использования низкоуровневыми библиотечными подпрограммами. Синхронизацию более высокого уровня лучше выполнять через *channel* и *communication* (что это???).

Значения, содержащие *type*'s, определенные в этом *package*, не должны копироваться.

### `type Mutex`

`Mutex` (*mutual exclusion*, взаимное исключение) – *lock* взаимного исключения. *Zero value* для `Mutex` - это *unlocked mutex*.

`Mutex` нельзя копировать после первого использования.

```go
type Mutex struct {
    // contains filtered or unexported fields
}
```

#### `Lock()`

```go
func (m *Mutex) Lock()
```

Блокирует (lock, захватывает?) *mutex* `m`. Если *mutex* `m` уже заблокирован, то вызывающая *goroutine* блокируется до тех пор, пока *mutex* не станет доступным.

#### `Unlock()`

```
func (m *Mutex) Unlock()
```

Разблокирует *mutex* `m`. Если `m` не заблокирован, то происходит *run-time error*.

Заблокированный *mutex* не связан с конкретной *goroutine*. Разрешена ситуация, когда *mutex* блокируется одной *goroutine* и разблокируется другой *goroutine*.



#### Пример

 Пример *mutex*'а в *goroutine* для синхронизации:

 ```go
  result := make(map[int]int)
  var wg sync.WaitGroup
	var mutex sync.Mutex

	for key, value := range values {
		
		wg.Add(1)
    // обязательно нужно передавать значение через аргументы в goroutine
		go func(val int) {
			defer wg.Done()

			mutex.Lock()
			result[key] = val
			mutex.Unlock()
		}(value)
	}

	wg.Wait()
 ```



### `type Once()`

```go
type Once struct {
	// contains filtered or unexported fields
}
```

`Once` – это объект, который будет выполнять точно одно действие (??? действие точно один раз).

`Once` нельзя копировать после первого использования.

#### `Do()`

```go
func (o *Once) Do(f func())
```

`Do()` вызывает функцию `f()` тогда и только тогда, когда `Do()` вызывается первый раз для этого экземпляра `Once()`. 

Например, если дано:

```go
var once Once
```

и `once.Do(f)` вызывается несколько раз, только первый вызов вызовет `f()`, даже если аргумент-функция `f` имеет разные значение в каждом вызове. 

`Do()` может использоваться для инициализации, которую нужно выполнить точно один раз. Поскольку `f()` не принимает аргументов, эти аргументы для функции необходимо получить через *closure*.



Пример:

```go
func main() {
	var once sync.Once
	onceBody := func() {
		fmt.Println("Only once")
	}
	done := make(chan bool)
	for i := 0; i < 10; i++ {
		go func() {
			once.Do(onceBody)
			done <- true
		}()
	}
	for i := 0; i < 10; i++ { // барьер
		<-done
	}
}
```





### `type RWMutex`

`RWMutex`  (*mutual exclusion*, взаимное исключение) - это *lock* взаимного исключения *reader/writer*. *Lock* может удерживать произвольное количество *reader*'s ИЛИ (!!!) одиночный *writer*. *Zero value* для `RWMutex` - это *unlocked mutex*.

`RWMutex` нельзя копировать после первого использования.

Если *goroutine* удерживает `RWMutex` для *reading* и другая горутина может вызвать `Lock()` (для *writing*), ни какая *goroutine* не сможет взять *read lock* до тех пор пока не будет снята начальная *read lock*. В частности, это запрещает рекурсивную *read locking*. Это необходимо для того, чтобы *lock* в конечном итоге (*eventually*) стала доступной; вызов `Lock()` исключает захват блокировки новыми *reader*'s.

```go
type RWMutex struct {
    // contains filtered or unexported fields
}
```



TODO!!!



### `type WaitGroup`

Ожидает завершения набора *goroutine*'s. *Main goroutine* вызывает `Add()`, чтобы установить количество ожидаемых *gotoutine*'s. Затем каждая из *goroutine* запускается и по завершении вызывает `Done()`. `Wait()` можно использовать, чтобы заблокироваться дло тех пор пока все *goroutine*'s не будут завершены.

`WaitGroup` не должен копироваться после первого использования. В функцию `WaitGroup` необходимо передавать *by pointer*. 

```
type WaitGroup struct {
     // contains filtered or unexported fields (???)
}
```

<u>Примеры:</u>


```go
wg := sync.WaitGroup{}


wg.Add(1)
go func() {
   defer wg.Done()
   // ...
}()


wg.Add(1)
go func() {
   defer wg.Done()

   // ...
}()

wg.Wait()
```

 ```go
var wg sync.WaitGroup
	var urls = []string{
		"http://www.golang.org/",
		"http://www.google.com/",
		"http://www.somestupidname.com/",
	}
	for _, url := range urls {
		// Increment the WaitGroup counter.
		wg.Add(1)
		// Launch a goroutine to fetch the URL.
		go func(url string) {
			// Decrement the counter when the goroutine completes.
			defer wg.Done()
			// Fetch the URL.
			http.Get(url)
		}(url)
	}
	// Wait for all HTTP fetches to complete.
	wg.Wait()
 ```

#### `sync.Add()`

```go
func (wg *WaitGroup) Add(delta int)
```

Добавляет значение `delta`, которое может быть отрицательным, к *counter*'у. Если *counter* становится равным 0, все *goroutine*'s, заблокированные на `Wait()`, освобождаются. Если *counter* станет отрицательным, `Add` вызывает *panic*.

Обычно `Add()` вызывается перед `go` *statement* или другим событием, которое должно ожидаться. 

#### `sync.Done()`

```go
func (wg *WaitGroup) Done()
```

Уменьшает *counter* на единицу.



#### `sync.Wait()`

```
func (wg *WaitGroup) Wait()
```

Блокирует, до тех пор пока *counter* не станет равным 0.



