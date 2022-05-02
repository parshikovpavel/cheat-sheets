# `net`

*Package* `net` предоставляет переносимый интерфейс (т.е. между ОС) для  network I/O, включая TCP/IP, UDP, *domain name resolution* и *Unix domain sockets*.



## Function

### `SplitHostPort()`

```go
func SplitHostPort(hostport string) (host, port string, err error)
```

`SplitHostPort()` разбивает *network address* вида:

- `host:port`
- `host%zone:port`
- `[host]:port`
- `[host%zone]:port`

на:

- `host` или `host%zone`
- и `port`

(не важно)



## Type

### `type IP`

```go
type IP []byte
```

IP — это один IP-адрес, *slice of byte*'s. *Function*'s в этом *package* принимают на вход *4-byte* (IPv4) или *16-byte* (IPv6) *slice*

(не важно)

<u>Пример:</u>

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	ipv6 := net.IP{0xfc, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}
	ipv4 := net.IPv4(10, 255, 0, 0)

  fmt.Println(ipv6.To4())  // <nil>  (??? почему)
	fmt.Println(ipv4.To4())  // 10.255.0.0

}
```



#### `ParseIP()`

```go
func ParseIP(s string) IP
```

`ParseIP()` парсит `s` как *IP address* и возвращает результат. Строка `s` может быть представлена в виде:

- десятичного числа с точками IPv4 (`192.0.2.1`)
- IPv6 (`2001:db8::68`)
- или *IPv4-mapped IPv6* формы (`::ffff:192.0.2.1`). 

Если `s` не является валидным текстовым представлением *IP address*'а, `ParseIP()` возвращает `nil`.

```go
fmt.Println(net.ParseIP("192.0.2.1"))			// 192.0.2.1
fmt.Println(net.ParseIP("2001:db8::68"))  // 2001:db8::68
fmt.Println(net.ParseIP("192.0.2"))       // <nil>
```

