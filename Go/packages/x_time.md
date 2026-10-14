# Rate

Package `rate` предоставляет *ratelimiter*. 

Он реализует алгоритм **Token Bucket** (ведро токенов) и входит в расширенный набор библиотек Go. 

Алгоритм работает так: в «ведро» (лимитер) добавляются токены с заданной скоростью (например, 1 токен в секунду). Каждый запрос «берёт» один токен. Если токены закончились, запрос либо отклоняется, либо откладывается до появления свободного токена. Параметр 

```
burst
```

 (размер «ведра») позволяет сделать кратковременный всплеск запросов перед стабилизацией лимита. [dev.to](https://dev.to/vivekalhat/rate-limiting-for-beginners-what-it-is-and-how-to-build-one-in-go-955)[compilenrun.com](https://www.compilenrun.com/docs/language/go/go-concurrency/go-rate-limiting/)[pkg.go.dev](https://pkg.go.dev/golang.org/x/time/rate)[slingacademy.com](https://www.slingacademy.com/article/rate-limiting-and-traffic-control-in-go-servers/)

## Основные компоненты

Ключевой объект — 

```
*rate.Limiter
```

. Создать его можно так:

```
limiter := rate.NewLimiter(rate.Limit(1), 3) // 1 запрос в секунду, всплеск до 3 запросов
```

Основные методы:

- ```
  Allow()
  ```

   — неблокирующая проверка: возвращает 

  ```
  true
  ```

  , если запрос разрешён, и 

  ```
  false
  ```

  , если лимит исчерпан.

- ```
  Wait(ctx context.Context)
  ```

   — блокирует выполнение, пока не появится доступный токен. Учитывает контекст (можно отменить ожидание).

- ```
  Reserve()
  ```

   — возвращает 

  ```
  Reservation
  ```

  , который позволяет отложить выполнение и узнать, сколько времени нужно подождать.

 [pkg.go.dev](https://pkg.go.dev/golang.org/x/time/rate)[pythonlib.ru](https://pythonlib.ru/library-theme943)

Также есть вариации с 

```
N
```

 вместо 

```
1
```

 (

```
AllowN
```

, 

```
ReserveN
```

, 

```
WaitN
```

) для пакетной обработки. [pkg.go.dev](https://pkg.go.dev/golang.org/x/time/rate)

Ограничитель **потокобезопасен** — его можно безопасно использовать из нескольких горутин одновременно. [pkg.go.dev](https://pkg.go.dev/golang.org/x/time/rate)

## Пример использования

Вот как можно применить лимитер в HTTP-middleware:

```
func rateLimitingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        ip := r.RemoteAddr
        limiter := getClientLimiter(ip) // Получаем лимитер для IP

        if !limiter.Allow() {
            http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```

