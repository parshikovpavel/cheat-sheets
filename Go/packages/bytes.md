`bytes` *package* реализует функции для манипулирования *byte slice*. Это аналогично возможностям `strings` *package*.



# Структура

## `type Reader`

```go
type Reader struct {
	// contains filtered or unexported fields
}
```

`type Reader` реализует *interface*'s `io.Reader`, `io.ReaderAt`, `io.WriterTo`, `io.Seeker`, `io.ByteScanner` и `io.RuneScanner` путем чтения из *byte slice*. В отличие от `type Buffer`, `type Reader` – *read-only* и поддерживает *seek*. 

*Zero value* для `Reader` действует как `Reader` для пустого *slice*.

```go
NewReader(make([]byte, 0))
```

### `NewReader()`

```go
func NewReader(b []byte) *Reader
```

`NewReader()` возвращает новый `Reader`, читающий из `b`.