# Guidelines

## Interfaces

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

\
\
\
