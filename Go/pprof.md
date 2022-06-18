# pprof

Инструмент профилирования

Можно посмотреть:

- память
- цепочки вызовов





# trace

Пример получения и визуализации трасировки работы программы в браузере https://www.ardanlabs.com/blog/2019/07/garbage-collection-in-go-part3-gcpacing.html

Для трассировки необходимо добавить в файл:

```go
03 import "runtime/trace"
04
05 func main() {
06     trace.Start(os.Stdout)
07     defer trace.Stop()
```

Программа выводит *trace* на `os.Stdout`. Его необходимо перенаправить в файл:

```bash
$ go build
$ time ./trace > t.out
Searching 4000 files, found president 28000 times.
./trace > t.out  2.67s user 0.13s system 106% cpu 2.626 total
```

Для просмотра trace необходимо выполнить:

```
$ go tool trace t.out
```

Выполнение этой команды запускает браузер Chrome с интерфейсом для визуальной трассировки:

![img](/Users/paparshikov/repos/cheat-sheets/img/go/pprof.png)