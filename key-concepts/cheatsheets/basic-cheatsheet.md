# Basic Cheatsheet

**Variables**

```go
var msg string
var msg = "Hello, world!"
var msg string = "Hello, world!"
var x, y int
var x, y int = 1, 2
var x, msg = 1, "Hello, world!"
msg = "Hello"

var (
  x int
  y = 20
  z int = 30
  d, e = 40, "Hello"
  f, g string
)
```

**Constants**

```go
const Phi = 1.618
const Size int64 = 1024
const x, y = 1, 2
const (
  Pi = 3.14
  E  = 2.718
)
const (
  Sunday = iota
  Monday
  Tuesday
  Wednesday
  Thursday
  Friday
  Saturday
)
```

**Basic Types**

```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32 ~= a character (Unicode code point) - very Viking

float32 float64

complex64 complex128
```

**Structs**

```go
package main

import (
	"fmt"
)

type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	v.X = 4
	fmt.Println(v.X, v.Y) // => 4 2
}
```

**Arrays**

```go
var a [5]int // fixed size
primes := [...]int{2, 3, 5, 7, 11, 13}
var a [2][3]int
pa := *[32]byte{}

// Same as [:3], Outputs: [2 3 5]
fmt.Println(primes[0:3])
```

```go
var a [2]string
a[0] = "Hello"
a[1] = "World"

fmt.Println(a[0], a[1]) //=> Hello World
fmt.Println(a)   // => [Hello World]
```

**Slices**

```go
var s []int // dynamic size
slice := []int{2, 3, 4}
slice := []byte("Hello")
s := make([]string, 3)

s[0] = "a"
s[1] = "b"
s = append(s, "d")
s = append(s, "e", "f")
```

**Maps**

```go
m := make(map[string]int)
m["k1"] = 7
m["k2"] = 13
fmt.Println(m) // => map[k1:7 k2:13]

v1 := m["k1"]
fmt.Println(v1)     // => 7
fmt.Println(len(m)) // => 2

delete(m, "k2")
fmt.Println(m) // => map[k1:7]

_, prs := m["k2"]
fmt.Println(prs) // => false

n := map[string]int{"foo": 1, "bar": 2}
fmt.Println(n) // => map[bar:2 foo:1]
```

**Loops**

```go
for i := 0; i < 10; i++ {/**/}
for i <= 3 { i = i + 1 }
for {/**/ continue /**/ break}
```

**Ranges**

```go
s := []string{"a", "b", "c"}
for idx, val := range s {/**/}
m := map[string]int{"a": 1}
for k, v := range m {/**/}
```
