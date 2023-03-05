# Mutexes

### Zero-value Mutexes are Valid

The zero-value of `sync.Mutex` and `sync.RWMutex` is valid, so you almost never need a pointer to a mutex.

```go
// BAD
mu := new(sync.Mutex)
mu.Lock()
```

```go
// GOOD
var mu sync.Mutex
mu.Lock()
```

If you use a struct by pointer, then the mutex should be a non-pointer field on it. Do not embed the mutex on the struct, even if the struct is not exported.

```go
// BAD
type SMap struct {
  sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

The `Mutex` field, and the `Lock` and `Unlock` methods are unintentionally part of the exported API of `SMap`

```go
// GOOD
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

The mutex and its methods are implementation details of `SMap` hidden from its callers.

\
