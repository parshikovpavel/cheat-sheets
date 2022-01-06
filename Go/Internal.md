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



# Alignment / padding

*Data structure alignment* (выравнивание структур данных) - способ размещения данных в памяти особым образом для ускорения доступа.. Она состоит из трех отдельных, но взаимосвязанных проблем: *data alignment* (выравнивание данных), *data structure padding* (отступ структур данных) и *packing* (упаковка).

CPU выполняют операции чтения и записи в память наиболее эффективно, когда данные *naturally aligned* (естественно выровнены). Это означает , что *data's memory address* является кратным *data size*. Например, в *64-bit architecture* данные могут быть *aligned*, если данные хранятся в четырех последовательных *byte*'s, а первый *byte* находится на *4-byte boundary* (границе). Архитектура определяет размер *word* (машинного слова), его размер равен 2^k.

*Alignment* - это всего лишь методы микрооптимизации, современные процессоры очень умны, и они знают, как обрабатывать *unaligned memory*, но в какой-то плохой ситуации процессору требуется несколько дополнительных циклов для получения *unaligned memory*. 

*Data alignment*  - это выравнивание элементов в соответствии с их *natural alignment*. Чтобы обеспечить *natural alignment*, может потребоваться вставить некоторый *padding* (отступ) между *structure element*'s или после последнего *structure element*. Например, на *32-bit machine* – *data structure*, содержащая *16-bit value*, за которым следует *32-bit value*, может иметь *16 bits of padding* между *16-bit value* и *32-bit value* для выравнивания *32-bit value* по *32-bit boundary*. В качестве альтернативы можно *pack* (упаковать) *structure*, опуская *padding*, что может привести к более медленному доступу, но сокращает потребление памяти на четверть.

При сохранении какого-то объекта в памяти может случиться, что некое поле, состоящее из нескольких байтов, пересечёт *natural boundary* для *word*'s в памяти. Некоторые модели процессоров не могут обращаться к данным в памяти, нарушающим границы *word*'s. Некоторые CPU могут обращаться к *невыровненным* данным дольше, нежели к данным, находящимся внутри целого *word* в памяти.

Хотя *data structure alignment* является фундаментальной проблемой для всех современных компьютеров, многие компьютерные языки и реализации компьютерных языков обрабатывают выравнивание данных автоматически. 

## Определение

*Memory address* `a` называется *n-byte aligned* (n-байтово выровнен), когда `a` кратно *n* байт (где *n* является степенью 2). *n-byte aligned address* имеет минимум log2(n) нулей в младших разрядах, когда выражен в двоичном виде.

Поэтому *alignment* всегда является числом-степенью 2 (1, 2, 4, 8, ...)

*Memory access* называется *aligned*, когда данные, к которым осуществляется доступ, имеют длину *n* *byte*'s и адрес элемента данных *n-byte aligned*. Когда *memory access* – *not aligned*, он называется *misaligned*.

В Go используется термин *required alignment*. Исходя из предыдущего определения, для элемент данных *required alignment* равен его размеру (но не для *struct*, смотреть ниже).

*Memory pointer*, который относится к примитивным данным, *n byte*'s длиной называется *aligned*, если он содержит только *address*'s, которые *n-byte aligned*, иначе он называется *unaligned*. *Memory pointer*, который ссылается на *data aggregate* (*data structure* или *array*) называется *aligned*, если каждый примитивный элемент данных в *aggregate* – *aligned*.

Alignment для разных type's и архитектур процессоров, которые чаще всего используются (возможно, в Go есть отличия). ??? Получается, что *alignment* не зависит от *architecture*, а зависит только от размера данных???

| **Data Type** | **32-bit (bytes)** | **64-bit (bytes)** |
| ------------- | ------------------ | ------------------ |
| char          | 1                  | 1                  |
| short         | 2                  | 2                  |
| int           | 4                  | 4                  |
| float         | 4                  | 4                  |
| double        | 8                  | 8                  |
| pointer       | 4                  | 8                  |



## Проблема

CPU обращается к *memory* по одному *memory word* за раз. Пока *memory word size* больше размера самого большого примитивного типа данных, поддерживаемого компьютером, *aligned access* всегда будет обращаться к одному *memory word*. Это может быть неверно для *misaligned data access*.

Пример *aligned data* для *32 bit system*:

![заполнение структуры в C](../img/go/aligned-data.png)

Пример доступа к *aligned data*:

![Выравнивание памяти](../img/go/aligned-access.png)

Пример доступа к *unaligned data*:



![Действия, выполняемые процессором для доступа к невыровненной памяти](/Users/paparshikov/repos/cheat-sheets/img/go/unaligned-access.png?1)



Если старший и младший байты в элементе данных не находятся в одном и том же *memory word*, компьютер должен разделить доступ к элементу данным на несколько *memory access*'s. Это требует множества сложных схем для выполнения *memory access*'s и их координации. 

Некоторые конструкции процессоров намеренно избегают такой сложности и вместо этого выполняют альтернативное поведение в случае *misaligned memory access*. 

Когда осуществляется доступ к одному *memory word*, операция является атомарной, то есть *memory word* целиком читается или записывается сразу, и другие устройства должны ждать завершения операции чтения или записи, прежде чем они смогут получить к нему доступ. Это может быть неверно для *unaligned access*'s к нескольким *memory word*'s, например, первое *word* может быть прочитано одним устройством, оба *word*'s записаны другим устройством, а затем второе *word* прочитано первым устройством, так что считанное значение не является ни исходным значением, ни обновленным значением. Хотя такие сбои случаются редко, их бывает очень сложно идентифицировать.

*Alignment* ускоряет доступ к памяти за счет генерации кода, который требует по одной инструкции для чтения и записи ячейки памяти. Без *alignment* мы можем столкнуться с ситуацией, когда процессору придется использовать две или более инструкции для доступа к данным, расположенным между адресами, кратными размеру *machine word*.

## Data structure padding (отступ структур данных)

Хотя компилятор обычно *allocate* отдельные элементы данных на *aligned boundary*'s, *data structure* часто содержат *member*'s с различными требованиями к *alignment*. Чтобы поддерживать правильное *alignment*, компилятор обычно вставляет дополнительные *unnamed data member*'s, чтобы каждый *member* был правильно *aligned*. Кроме того, *structure* в целом может быть *padded* последним *unnamed member*. Это позволяет каждой *structure* быть правильно *aligned* внутри *array*.

*Padding* вставляется только тогда, когда:

- за одним *member* следует другой *member* с более высокими требованиями к *alignment*
- или в конце структуры. 

Изменяя порядок *member*'s в *structure* можно изменить количество *padding*'s, необходимых для *alignment*. Например, если *member*'s сортируются по убыванию требований к *alignment*, потребуется минимальное количество *padding*'s. Требуемое минимальное количество *padding*'s всегда меньше максимального *alignment* в *structure*, т.е. размера максимального *member*. 

*Struct* сами по себе *aligned* и значение *alignment* для них равно размеру самого большого *member* в *structure*!!!! Или другими словами, *alignment* в *struct* должен быть таким же как *alignment*  для наиболее строгого (большого) *member*. В этом случае все *member* в *structure* могут быть *aligned* – от самого большого *member* до самого маленького. 

*Alignment* для *array type* равен *alignment* его *element type*.

Хотя C++ и Go не позволяет компилятору переупорядочивать *member*'s в *structure* для экономии места, другие языки могут это делать. 

### Вычисление *padding*

Ниже формулы для расчета *padding*, необходимого для *alignment* начала *data structure*:

```
padding = align - (offset mod align)
aligned = offset + padding
```

Например, имеем *4-byte aligned structure* и *offset* `0x59d`, тогда `padding = 3`. В этом случае *structure* будет начинаться с `0x5a0`, что кратно 4. 

## Struct in Go

Пусть имеем 64-битную архитектуру, размер *word* – *8 bytes*

Допустим объявлена *struct*:

```go
type myStruct struct {
  myBool   bool    // 1 byte
  myFloat float64  // 8 bytes
  myInt  int32     // 4 bytes
}
```

Посмотрим размер *instance* этой *struct* в памяти:

```go
a := myStruct{}
fmt.Println(unsafe.Sizeof(a)) // 24 bytes
```

Т.е. вместо 1 + 8 + 4 = 13 байт *struct* занимает в памяти 24 байта.

Т.к. в памяти она будет размещена следующим образом:

![img](../img/go/struct-in-memory.png)

Оптимизировать *struct* можно, упорядочив *field*'s по их размеру (а значит и требованию к *alignment*):

```go
type myStructOptimized struct {
 myFloat float64 // 8 bytes
 myBool  bool    // 1 byte
 myInt   int32   // 4 bytes
}
```

(только здесь не совсем упорядочили почему-то).

Размер *instance* этой *struct* в памяти:

```go
b := myStructOptimized{}
fmt.Println(unsafe.Sizeof(b)) // ordered 16 bytes
```

Т.к. в памяти она будет размещена следующим образом:

![img](../img/go/struct-in-memory-optimized.png)

### Struct alignment

Каждая *struct* целиком тоже *aligned* в памяти. Это означает, что *structure* в целом может быть *padded* последним *unnamed member*. Это позволяет каждой *structure* быть правильно *aligned* внутри *array* (дубль текста выше).

В Go используется термин *required alignment*. Значение *required alignment* (требуемого *alignment*) для *struct* равно размеру самого большого *member* в *structure*!!!! Или другими словами, *alignment* в *struct* должен быть таким же как *alignment*  для наиболее строгого (большого) *member*. В этом случае все *member* в *structure* могут быть *aligned* – от самого большого *member* до самого маленького, т.к. они все являются степень 2 (дубль выше).

Пример *struct* с *required alignment 8 byte*'s:

```go
type myStruct struct {
	myInt   int64   // 8 bytes
	myBool  bool    // 1 byte
}

func main() {
	a := myStruct{}
	fmt.Println(unsafe.Sizeof(a)) // 16 bytes
}

```

![image-20211105182348170](/Users/paparshikov/repos/cheat-sheets/img/go/struct-in-memory-8bytes.png)

Изменив размер самого большого *field* в *struct* на *4 byte*, меняем и *required alignment* для *struct* на *4 byte*:

```go
type myStruct struct {
	myInt   int32   // 4 bytes
	myBool  bool    // 1 byte
}

func main() {
	a := myStruct{}
	fmt.Println(unsafe.Sizeof(a)) // 8 bytes
}

```

![image-20211105183258511](/Users/paparshikov/repos/cheat-sheets/img/go/struct-in-memory-4bytes.png)

### 32-bit и 64-bit architecture 

Разница между *32-bit* и *64-bit architecture* следующая: в *32-bit architecture* – *word boundary*'s встречаются каждые 4 bytes, a в *64-bit architecture* – каждые 8 *bytes*.

Рассмотрим *struct*:

```go
type myStruct struct {
        a    uint32   // 4 bytes
        b    uint64   // 8 bytes
}

func main() {
	a := myStruct{}
	fmt.Println(unsafe.Sizeof(a))  // 16
}
```

Для *64 bit machine* адрес `uint64` *field* должен быть кратен 8 bytes. И поэтому *struct* будет выглядеть следующим образом и `size = 16 bytes`:

```go
type myStruct struct {
        a    uint32   // 4 bytes
        _    [4]byte  // padding added by compiler
        b    uint64   // 8 bytes
}
```

Для *32 bit machine* – *word boundary*'s встречаются каждые *4 bytes*, поэтому нет необходимости добавлять *padding*

```go
type myStruct struct {
        a    uint32   // 4 bytes
        b    uint64   // 8 bytes
}

func main() {
	a := myStruct{}
	fmt.Println(unsafe.Sizeof(a))  // 12
}
```

Таким образом, `uint64 ` *field* должны быть *aligned*: каждые *4 bytes* на *32 bit platform* и каждые *8 bytes* на *64 bit platform*.



## `unsafe` *package*

Функция [`unsafe.Alignof()`](packages/unsafe.md#alignof) возвращает *required alignment*:

- для *value* целиком (??? равен размеру, кроме составных типов, *struct*, *array*)
- для *field* внутри *struct*

```go
type myStruct struct {
	myInt   int64   // 8 bytes
	myBool  bool    // 1 byte
}

func main() {
	a := myStruct{}
	fmt.Println(unsafe.Alignof(a))        // 8
	fmt.Println(unsafe.Alignof(a.myInt))  // 8
	fmt.Println(unsafe.Alignof(a.myBool)) // 1
}
```

Функция [`unsafe.Offsetof()`](packages/unsafe.md#Offsetof) возвращает *offset* для *field* `x` внутри *struct*:

```go
type myStruct struct {
	myBool1  bool    // 1 byte
	myBool2  bool    // 1 byte
	myInt   int64    // 8 bytes
	myBool3  bool    // 1 byte
	myBool4  bool    // 1 byte
}

func main() {
	a := myStruct{}
	fmt.Println(unsafe.Offsetof(a.myBool1))  // 0
	fmt.Println(unsafe.Offsetof(a.myBool2))  // 1
	fmt.Println(unsafe.Offsetof(a.myInt))    // 8
	fmt.Println(unsafe.Offsetof(a.myBool3))  // 16
	fmt.Println(unsafe.Offsetof(a.myBool4))  // 17
}
```

