*Package* `os` предоставляет независимый от платформы интерфейс к функциям операционной системы. Использован Unix-like дизайн, но при этом обработка *error* – *Go-like*; неудачные вызовы возвращают значения типа `error`, а не номера ошибок (*error number*). Часто *error* содержит дополнительную информацию. Например, если функции `Open()` или `Stat()`, принимающие *file name*, завершаются с *error*, то *error* будет включать в себя ошибочное *file name* при печати и будет иметь тип `*PathError`, который можно распаковать для получения дополнительной информации.

Интерфейс `os` должен быть единым для всех операционных систем. *Feature*'s, которые обычно недоступны, появляются в специфичном для системы *package syscall* (??? что такое syscall).

Вот простой пример, открывающий файл:

```go
file, err := os.Open("file.go") // Для доступа на read.
if err != nil {
	log.Fatal(err)
}
```

Если произошла *error*, то *error string* будет содержать:

```
open file.go: no such file or directory
```

Затем данные файла могут быть считаны в *slice of byte*. `Read()` и `Write()` для определения количества байт используют *slice length*.

```go
data := make([]byte, 100)
count, err := file.Read(data)
if err != nil {
	log.Fatal(err)
}
fmt.Printf("read %d bytes: %q\n", count, data[:count])
```

Максимальное количество конкурентных операций над *file* может быть лимитировано ОС. Число должно быть высоким, но его превышение может привести к снижению производительности или вызвать другие проблемы.

# Примеры

Открыть файл для *read-write*, *create* если не существует.

```go
func main() {
	f, err := os.OpenFile("notes.txt", os.O_RDWR|os.O_CREATE, 0755)
	if err != nil {
		log.Fatal(err)
	}
	if err := f.Close(); err != nil {
		log.Fatal(err)
	}
}
```

Открыть файл для *write* в режиме *append*, *create* если не существует.

```go
func main() {
	// If the file doesn't exist, create it, or append to the file
	f, err := os.OpenFile("access.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		log.Fatal(err)
	}
	if _, err := f.Write([]byte("appended some data\n")); err != nil {
    f.Close() // Не сохраняет error для функции `Close()`. Пишет в log ошибку для `Write()`
		log.Fatal(err)
	}
	if err := f.Close(); err != nil {
		log.Fatal(err)
	}
}
```

Открыть файл в режиме read-only и прочитать из него всё содержимое



```go
f, err := os.OpenFile(file, os.O_RDONLY, 0)
if err != nil {
	log.Printf("Opening Document [%s] : ERROR : %v", doc, err)
	return
}
defer f.Close()

data, err := ioutil.ReadAll(f)
if err != nil {
	log.Printf("Reading Document [%s] : ERROR : %v", doc, err)
	return
}
```



# Constant

Флаги для `OpenFile()`, обертывающие флаги базовой системы. Не все флаги могут быть реализованы в данной системе.

```go
const (
  // Должен быть указан ровно один из O_RDONLY, O_WRONLY или O_RDWR. 
	O_RDONLY int = syscall.O_RDONLY // open the file read-only.
	O_WRONLY int = syscall.O_WRONLY // open the file write-only.
	O_RDWR   int = syscall.O_RDWR   // open the file read-write.
	// Остальные значения могут быть объединены через or для управления поведением
	O_APPEND int = syscall.O_APPEND // append data to the file when writing.
	O_CREATE int = syscall.O_CREAT  // create a new file if none exists.
	O_EXCL   int = syscall.O_EXCL   // used with O_CREATE, file must not exist.
	O_SYNC   int = syscall.O_SYNC   // open for synchronous I/O.
	O_TRUNC  int = syscall.O_TRUNC  // truncate regular writable file when opened.
)

```

# Type

## `type File`

```go
type File struct {
	// содержит filtered или unexported fields
}
```

`File` представляет собой открытый *file descriptor*.

### `Close()`

```go
func (f *File) Close() error
```

`Close()` закрывает `File`, делая его непригодным для I/O. 

В файлах, поддерживающих [SetDeadline](https://pkg.go.dev/os#File.SetDeadline), любые незавершенные операции I/O будут *cancel* и немедленно завершатся с `ErrClosed` *error*.

`Close()` вернет *error*, если она уже была вызвана.





### `Open()`

```go
func Open(name string) (*File, error)
```

`Open` открывает указанный файл (`name`) для *Read*. В случае успеха методы возвращенного `*File` можно использовать для *Read*; связанный *file descriptor* имеет режим `O_RDONLY`. Если возвращается `error`, то она будет иметь тип `*PathError`.

### `OpenFile()`

```go
func OpenFile(name string, flag int, perm FileMode) (*File, error)
```

`OpenFile()` — это универсальная функция для *file open*; чаще всего вместо нее используются `Open()` или `Create()`. 

`OpenFile()` открывает указанный файл (`name`) с указанным флагом `flag` (`O_RDONLY` и т.д.). Если файл не существует и передан флаг `O_CREATE`, он создается с *mode* `perm` (??? перед *umask*). В случае успеха методы возвращенного `*File` могут использоваться для I/O. Если возвращается `error`, то она будет иметь тип `*PathError`.



### `Read()`

```go
func (f *File) Read(b []byte) (n int, err error)
```

`Read()` читает до `len(b)` байт из `File` и сохраняет их в `b`. Он возвращает количество прочитанных байт `n` и `error`, если она произошла. При достижении *End of file*, `Read()` возвращает `0, io.EOF`.



### `Write()`

```go
func (f *File) Write(b []byte) (n int, err error)
```

`Write()` записывает `len(b)` байтов из `b` в файл. Он возвращает количество записанных байтов `n` и `error`, если она произошла. `Write()` возвращает *non-nil error*, когда `n != len(b)`.







TODO!!!

https://pkg.go.dev/os#pkg-types







