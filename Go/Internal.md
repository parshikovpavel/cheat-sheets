# Компиляция программы на C++

```c++
#include <stdio.h>

int main() {
  printf("123");
  
  return 0;
}
```

Компиляция:

```bash
gcc hello.c -o hello
```

Количество ассемблерных инструкций:

```bash
objdump -d hello | wc -l
```







# select / poll / epoll

https://habr.com/ru/company/infopulse/blog/415259/





# Runtime

https://dave.cheney.net/

https://blog.cloudflare.com/how-stacks-are-handled-in-go/

https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html

https://morsmachine.dk/go-scheduler

https://rakyll.org/codegen/

https://www.youtube.com/watch?v=YHRO5WQGh0k

https://habr.com/ru/company/mailru/blog/358088/

http://m0sth8.github.io/runtime-1/#11



Runtime – это окружение, в котором выполняется наша программа. Код, который выполняется в нашей программе, но который мы сами не вызывали. Код, который включается в нашу программу и работает для нас прозрачно.

- планировщик
- сборщик мусора (TBD)
- аллокатор памяти (TBD)

## Планировщик

Сердце программы, маленькая операционная система, который управляет работой goroutine's. 

Т.е. по сути мы строим планировщик задач поверх планировщика задач ОС.

Основные концепции:

- у программиста есть только goroutines и нет threads ОС.
- запускается столько thread's сколько процессорных ядер (или можно специально настроить значение `GOMAXROCS`)
- распределеяют на эти thread's нужное количество goroutine's, которые запускает программист

Основные термины:

- Machine (M) – Thread ОС
- Processor (P) – процессор
- Goroutine (G)

Связь между ними:

![image-20211002123120699](../img/go/image-20211002123120699.png)





# Goroutine

Стандартные thread ОС – тяжелые и медленные, тяжело контролировать из программы. Goroutine также называются зелеными thread. Они как thread, только лучше.

Преимущества:

- контроль над выполнением асинхронных задач
- экономия ресурсов – goroutine занимает меньше 2Kb, а thread - около 2 Mb
- более эффективное переключение между задачами. Переключение между threads в ОС – очень сложный и медленный процесс.

goroutine может начать выполняться на одном системной thread, а продолжить на другом.