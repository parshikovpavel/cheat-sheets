# Deadlock

Deadlock — ситуация, когда все *goroutine*'s заблокированы навсегда. 

*Runtime Go* детектит только глобальный *deadlock* — когда все *goroutine*'s спят и продвинуться некому - возникает паника:

```
fatal error: all goroutines are asleep - deadlock!
```

Локальный *deadlock* (часть *goroutine*'s зависла, остальные работают) *runtime* не ловит — его видно только через профилировщик. 

Типичные источники: 

- захват двух *mutex*'ов в разном порядке
- отправка в небуферизованный *channel* без читателя
- `WaitGroup.Wait()` при недостающем `Done()` . 

# *Goroutine* leak

*Goroutine leak* — *goroutine*, которая никогда не завершится.

Примеры *goroutine leak*: 

- висит на *channel*'е без второй стороны
- забыт `Done()`
- бесконечный цикл без `ctx.Done()` . 

Каждая такая *goroutine* держит минимум ~2 КБ стека и ссылки на объекты — это постепенно приводит к деградации по памяти и числу *goroutine*'s. 

Способы детекции *goroutine leak*:

-  `runtime.NumGoroutine()`
- `pprof`, *goroutine*-профиль
- в тестах — `go.uber.org/goleak` . 

# Обработка задач

- создать под каждую задачу отдельную *goroutine*. При этом под стек отжираем по 2КБ на каждую *goroutine*. Но если задача CPU-bound все равно упираемся в количество процессоров (`GOMAXPROCS`)
- использовать некоторое количество параллельных операций через:
  - *worker pool*
  - *semaphore*



# Паттерны

## Rate limiter

### **Базовый *rate limiter*** 

Идея *limiter*'а - создать *ticker*, который будет отщелкивать моменты обработки *request*'ов. В единицу времени обрабатывается только один *request*:

```go
func rateLimiter(requests <-chan int, limit time.Duration) <-chan int {
    throttled := make(chan int)
    ticker := time.NewTicker(limit)
    
    go func() {
        defer close(throttled)
        for req := range requests {
            <-ticker.C  // Wait for tick
            throttled <- req
        }
        ticker.Stop()
    }()
    
    return throttled
}
```

Отправляем request'ы в *channel*:

```go
requests := make(chan int)
limited := rateLimiter(requests, 200*time.Millisecond)  // 5 requests per second

for i := 0; i < 10; i++ {
    requests <- i
}
```

Результат (запросы обрабатываются каждые 200мс):

```go
$ go run rate-limiting.go
request 1 2012-10-19 00:38:18.687438 +0000 UTC
request 2 2012-10-19 00:38:18.887471 +0000 UTC
request 3 2012-10-19 00:38:19.087238 +0000 UTC
request 4 2012-10-19 00:38:19.287338 +0000 UTC
request 5 2012-10-19 00:38:19.487331 +0000 UTC
```

[Пример ratelimiter с graceful shutdown в файле](https://github.com/parshikovpavel/platform-tasks/retry_with_limit)



### Алгоритм маркерной корзины

[Здесь описание](../System design/rate limiter.md).

Алгоритм позволяет обрабатывать короткие всплески *request*'ов. 

Заводим *channel* маркерную корзину, сразу наполняем ее некоторым количеством маркеров.

```go
func BurstRateLimitCall(ctx context.Context, client Client, payloads []*Payload, burstLimit int) {
  burstyLimiter := make(chan time.Time, burstLimit)

  for range 3 {
    burstyLimiter <- time.Now()
  }

```

Каждый период времени будем пытаться добавить новое значение в маркерную корзину.

```go
  ctx, cancel := context.WithCancel(ctx)
  defer cancel()

  go func() {
    ticker := time.NewTicker(rateLimit)
    defer ticker.Stop()
    for t := range ticker.C {
        select {
        case burstyLimiter <- t:
        case <-ctx.Done():
            return // exit goroutine when surrounding function returns
        }
    }
  }()
```

Смоделируем 5 входящих *request*'ов. Здесь они пытаются достать маркеры из маркерной корзины (`burstyLimiter`). 

```go
burstyRequests := make(chan int, 5)
for i := 1; i <= 5; i++ {
    burstyRequests <- i
}
close(burstyRequests)
for req := range burstyRequests {
    <-burstyLimiter
    fmt.Println("request", req, time.Now())
}
```

Первые 3 *request*'а обрабатываются немедленно :

```bash
$ go run rate-limiting.go
request 1 2012-10-19 00:38:20.487578 +0000 UTC
request 2 2012-10-19 00:38:20.487645 +0000 UTC
request 3 2012-10-19 00:38:20.487676 +0000 UTC
request 4 2012-10-19 00:38:20.687483 +0000 UTC
request 5 2012-10-19 00:38:20.887542 +0000 UTC
```

### Стандартная библиотека

В Go для реализации rate limiting (ограничения частоты запросов) чаще всего используют пакет `golang.org/x/time/rate`. ([ссылка](packages/x_time.md))
