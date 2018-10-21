## Alignment as a mechanical sympathy can reduce memory footprint and increase performance

## Check your alignment with the help of this tool
* For propriertary structs, give the fields generic names when using this tool.
* http://golang-sizeof.tips/
* Sometimes all that is necessary is to define larger fields near the top with exceedingly smaller fields towards the bottom.

# Example 1
* https://play.golang.org/p/dNWspo2Dxv

```go
package main

import (
	"fmt"
	"unsafe"
)

type struct1 struct {
	a bool
	b uint64
	c bool
	d uint64
	e bool
	f uint16

}

type struct2 struct {
	b uint64
	d uint64
	a bool
	c bool
	f uint16
	e bool
}

func main() {
	var sV1 struct1
	fmt.Printf("struct1: %p\n", &sV1)
	fmt.Printf("\ta: %p, offset: %d, alignment: %d\n", &sV1.a, unsafe.Offsetof(sV1.a), unsafe.Alignof(sV1.a))
	fmt.Printf("\tb: %p, offset: %d, alignment: %d\n", &sV1.b, unsafe.Offsetof(sV1.b), unsafe.Alignof(sV1.b))
	fmt.Printf("\tc: %p, offset: %d, alignment: %d\n", &sV1.c, unsafe.Offsetof(sV1.c), unsafe.Alignof(sV1.c))
	fmt.Printf("\td: %p, offset: %d, alignment: %d\n", &sV1.d, unsafe.Offsetof(sV1.d), unsafe.Alignof(sV1.d))
	fmt.Printf("\te: %p, offset: %d, alignment: %d\n", &sV1.e, unsafe.Offsetof(sV1.e), unsafe.Alignof(sV1.e))
	fmt.Printf("\tf: %p, offset: %d, alignment: %d\n", &sV1.f, unsafe.Offsetof(sV1.f), unsafe.Alignof(sV1.f))
	fmt.Printf("\n\tsize: %d bytes\n", unsafe.Sizeof(sV1))

	fmt.Printf("\n---\n\n")

	var sV2 struct2
	fmt.Printf("struct2: %p\n", &sV2)
	fmt.Printf("\tb: %p, offset: %d, alignment: %d\n", &sV2.b, unsafe.Offsetof(sV2.b), unsafe.Alignof(sV2.b))
	fmt.Printf("\td: %p, offset: %d, alignment: %d\n", &sV2.d, unsafe.Offsetof(sV2.d), unsafe.Alignof(sV2.d))
	fmt.Printf("\ta: %p, offset: %d, alignment: %d\n", &sV2.a, unsafe.Offsetof(sV2.a), unsafe.Alignof(sV2.a))
	fmt.Printf("\tc: %p, offset: %d, alignment: %d\n", &sV2.c, unsafe.Offsetof(sV2.c), unsafe.Alignof(sV2.c))
	fmt.Printf("\tf: %p, offset: %d, alignment: %d\n", &sV2.f, unsafe.Offsetof(sV2.f), unsafe.Alignof(sV2.f))
	fmt.Printf("\te: %p, offset: %d, alignment: %d\n", &sV2.e, unsafe.Offsetof(sV2.e), unsafe.Alignof(sV2.e))
	fmt.Printf("\n\tsize: %d bytes\n", unsafe.Sizeof(sV2))
}
```

```sh
// Output
struct1: 0x44c030
	a: 0x44c030, offset: 0, alignment: 1
	b: 0x44c038, offset: 8, alignment: 8
	c: 0x44c040, offset: 16, alignment: 1
	d: 0x44c048, offset: 24, alignment: 8
	e: 0x44c050, offset: 32, alignment: 1
	f: 0x44c052, offset: 34, alignment: 2

	size: 40 bytes

---

struct2: 0x45a020
	b: 0x45a020, offset: 0, alignment: 8
	d: 0x45a028, offset: 8, alignment: 8
	a: 0x45a030, offset: 16, alignment: 1
	c: 0x45a031, offset: 17, alignment: 1
	f: 0x45a032, offset: 18, alignment: 2
	e: 0x45a034, offset: 20, alignment: 1

	size: 24 bytes
```

## Example 2
* https://play.golang.org/p/0qsgpuAHHp

```go
package main

import (
    "fmt"
    "unsafe"
)

type Compact struct {
    a, b                   uint64
    c, d, e, f, g, h, i, j byte
}

// Larger memory footprint than "Compact" - but less fields!
type Inefficient struct {
    a uint64
    b byte
    c uint64
    d byte
}

func main() {
    newCompact := new(Compact)
    fmt.Println(unsafe.Sizeof(*newCompact))
    newInefficient := new(Inefficient)
    fmt.Println(unsafe.Sizeof(*newInefficient))
}
```

```sh
# Output
24
32
```

## Example 3

* https://medium.com/@felipedutratine/how-to-organize-the-go-struct-in-order-to-save-memory-c78afcf59ec2
* https://play.golang.org/p/cUgB54yCpL

```go
package main

import (
	"fmt"
	"unsafe"
)

type myStruct struct {
	myInt   bool    // 1 byte
	myFloat float64 // 8 bytes
	myBool  int32   // 4 bytes
}

type myStructOptimized struct {
	myFloat float64 // 8 bytes
	myInt   int32   // 4 bytes
	myBool  bool    // 1 byte
}

func main() {
	a := myStruct{}
	b := myStructOptimized{}

	fmt.Println(unsafe.Sizeof(a)) // unordered 24 bytes
	fmt.Println(unsafe.Sizeof(b)) // ordered 16 bytes
}

```

## Alignment check linter
* https://github.com/opennota/check

```sh
# sample output
aligncheck net/http
net/http: /usr/lib/go/src/net/http/server.go:123:6: struct conn could have size 160 (currently 168)
net/http: /usr/lib/go/src/net/http/server.go:315:6: struct response could have size 152 (currently 176)
net/http: /usr/lib/go/src/net/http/transfer.go:37:6: struct transferWriter could have size 96 (currently 112)
net/http: /usr/lib/go/src/net/http/transport.go:49:6: struct Transport could have size 136 (currently 144)
net/http: /usr/lib/go/src/net/http/transport.go:811:6: struct persistConn could have size 160 (currently 176)
```