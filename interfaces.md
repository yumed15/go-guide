# Interfaces

### No Pointers to Interfaces

You almost never need a pointer to an interface. You should be passing interfaces as values—the underlying data can still be a pointer.

An interface is two fields:

1. A pointer to some type-specific information. You can think of this as "type."
2. Data pointer. If the data stored is a pointer, it’s stored directly. If the data stored is a value, then a pointer to the value is stored.

If you want interface methods to modify the underlying data, you must use a pointer.

### Verify Interface Compliance

Verify interface compliance at compile time where appropriate. This includes:

* Exported types that are required to implement specific interfaces as part of their API contract
* Exported or unexported types that are part of a collection of types implementing the same interface
* Other cases where violating an interface would break users

```go
// BAD

type Handler struct {
  // ...
}


func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  ...
}

```

```go
// GOOD

type Handler struct {
  // ...
}

var _ http.Handler = (*Handler)(nil)

func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

The statement `var _ http.Handler = (*Handler)(nil)` will fail to compile if `*Handler` ever stops matching the `http.Handler` interface.

The right hand side of the assignment should be the zero value of the asserted type. This is `nil` for pointer types (like `*Handler`), slices, and maps, and an empty struct for struct types.

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}

var _ http.Handler = LogHandler{}

func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

### Receivers and Interfaces

Methods with value receivers **** can be called on pointers as well as values.

&#x20;Methods with pointer receivers can only be called on pointers or [addressable values](https://golang.org/ref/spec#Method\_values).

```go
type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

// We cannot get pointers to values stored in maps, because they are not
// addressable values.
sVals := map[int]S{1: {"A"}}

// We can call Read on values stored in the map because Read
// has a value receiver, which does not require the value to
// be addressable.
sVals[1].Read()

// We cannot call Write on values stored in the map because Write
// has a pointer receiver, and it's not possible to get a pointer
// to a value stored in a map.
//
//  sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// You can call both Read and Write if the map stores pointers,
// because pointers are intrinsically addressable.
sPtrs[1].Read()
sPtrs[1].Write("test")
```

Similarly, an interface can be satisfied by a pointer, even if the method has a value receiver.

```go
type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// The following doesn't compile, since s2Val is a value, and there is no value receiver for f.
//   i = s2Val
```

\
\
\
\
