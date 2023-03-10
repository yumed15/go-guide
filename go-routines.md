# Go Routines

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

### Defer to Clean Up

Use defer to clean up resources such as files and locks.

```go
// BAD
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// easy to miss unlocks due to multiple returns
```

```go
// GOOD
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// more readable
```

### Channel Size is One or None

Channels should usually have a size of one or be unbuffered. By default, channels are unbuffered and have a size of zero. Any other size must be subject to a high level of scrutiny. Consider how the size is determined, what prevents the channel from filling up under load and blocking writers, and what happens when this occurs.

```go
// BAD
// Ought to be enough for anybody!
c := make(chan int, 64)
```

```go
// GOOD
// Size of one
c := make(chan int, 1) // or
// Unbuffered channel, size of zero
c := make(chan int)
```

### Don't fire-and-forget goroutines

Goroutines are lightweight, but they're not free: at minimum, they cost memory for their stack and CPU to be scheduled. While these costs are small for typical uses of goroutines, they can cause significant performance issues when spawned in large numbers without controlled lifetimes. Goroutines with unmanaged lifetimes can also cause other issues like preventing unused objects from being garbage collected and holding onto resources that are otherwise no longer used.

\
In general, every goroutine:

* must have a predictable time at which it will stop running; or
* there must be a way to signal to the goroutine that it should stop

```go
// BAD
// There's no way to stop this goroutine. This will run until the application exits.
go func() {
  for {
    flush()
    time.Sleep(delay)
  }
}()
```

```go
// GOOD
// This goroutine can be stopped with close(stop), 
// and we can wait for it to exit with <-done.
var (
  stop = make(chan struct{}) // tells the goroutine to stop
  done = make(chan struct{}) // tells us that the goroutine exited
)
go func() {
  defer close(done)

  ticker := time.NewTicker(delay)
  defer ticker.Stop()
  for {
    select {
    case <-ticker.C:
      flush()
    case <-stop:
      return
    }
  }
}()

// Elsewhere...
close(stop)  // signal the goroutine to stop
<-done       // and wait for it to exit
```

### **Wait for goroutines to exit**

Given a goroutine spawned by the system, there must be a way to wait for the goroutine to exit. There are two popular ways to do this:

* Use a `sync.WaitGroup`. Do this if there are multiple goroutines that you want to wait for

```go
var wg sync.WaitGroup
for i := 0; i < N; i++ {
  wg.Add(1)
  go func() {
    defer wg.Done()
    // ...
  }()
}

// To wait for all to finish:
wg.Wait()
```

* Add another `chan struct{}` that the goroutine closes when it's done. Do this if there's only one goroutine.

```go
done := make(chan struct{})
go func() {
  defer close(done)
  // ...
}()

// To wait for the goroutine to finish:
<-done
```

\
\
\
