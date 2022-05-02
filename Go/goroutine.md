## 

В Go было выбрано название *goroutine*, т.к. существующие термины (*thread*, *coroutine*, *process* и др.) – наполнены неточным смыслом. 

*Goroutine* – это *function*, которая выполняется в независимом *concurent thread of control*, в том же самом *address space* (вместе с другими *goroutine*'s). 

*Goroutine* –  *lightweight* (легковесная), и стоит немного больше, чем *allocation* в *stack space* (??? почему *stack*). *Stack* изначально маленький по размеру, поэтому он дешевый, и растет путем *allocating* (и *freeing*) в *heap storage* (почему *heap*???) по мере необходимости.

*Goroutine*'s мультиплексируются в множество *OS thread*'s, поэтому если одна из них блокируется (например, по I/O), другие продолжают выполняться. *Goroutine design* скрывает многие сложности создания и управления *thread*'ами.

Выполнение программы не дожидается завершения вызванной *function* (*goroutine*). Вызванная *function* начинает выполняться независимо в новой горутине. Когда функция завершается, ее горутина также тихо (???) завершается. Если функция имеет какие-либо возвращаемые значения, они отбрасываются по завершении функции.

У *goroutine* нет *parent* или *child*'s. Когда вы запускаете *goroutine*, она просто выполняется вместе со всеми другими запущенными *goroutine*. Каждая горутина завершается только тогда, когда происходит `return` в ее функции. Единственное исключение – все *goroutine*'s завершаются, когда завершается *goroutine*'s, которая исполняет функцию `main()`). Чтобы реализовать *cancel* для *goroutine* – необходимо использовать `Context`.

# `go` *statement*

`go` *statement* начинает выполнение *goroutine* (1) в в том же самом *address space*. 

<pre>
GoStmt = "go" <a href="#operator">Expression</a> .   
</pre>

*Expression* должно быть *function call* или *method call*, оно не может быть заключено в круглые скобки `(...)`. Использовать *built-in function* нельзя. 

```go
go foo()                    // использование function declaration
go func() {                 // использование function literal
        time.Sleep(delay)
        fmt.Println(message)
}()                         // Важно, что ставяться скобки () - необходимо вызвать function
```

*Function literal* – *closure*, поэтому *variable*'s, которые используются внутри *function*, не будут удалены в вызывающей *goroutine* и будут существовать до завершения выполнения *function* (что это значит???)

Для ожидания несколько *goroutine*'s необходимо использовать `sync.WaitGroup ` ([1](#sync-waitgroup)).



# Из собесов

Горутинами управляет runtime scheduler. Горутина занимает около 2 кб. Можно управлять кол-вом тредов с помощью GOMAXPROCS. Есть дорогостоящие переключения регистров при шедулинге thread'ов.

*Goroutine* начинают с очень маленького *stack*'а — всего 2 Кб, — который увеличивается по мере необходимости. В начале вызова функции в Go выполняется проверка, достаточно ли *stack space*  для следующего вызова. И если нет, то перед продолжением вызова *goroutine stack* *перемещается* в более крупную область *memory*, с перезаписью *pointer*'ов в случае необходимости.



TODO:

https://github.com/golang/go/wiki/CommonMistakes

https://stackoverflow.com/questions/36121984/how-to-use-a-method-as-a-goroutine-function

https://stackoverflow.com/questions/30183669/passing-parameters-to-function-closure