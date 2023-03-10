# Concurrency

**Mutex**

`Mutex` (MUTual EXclusion) provides a lock that can only be accessed by one goroutine at a time. This is used to synchronize data across multiple goroutines. Attempting to lock a `Mutex` will block (wait) until it is safe to do so. Once locked, the protected data can be operated upon since all other goroutines are forced to wait until the lock is available. Unlock the `Mutex` once work is completed, so other goroutines can access it.

```go
// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mu.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mu.Unlock()
	return c.v[key]
}
```

**Goroutines**

```go
package main

import (
	"fmt"
	"time"
)

func f(from string) {
	for i := 0; i < 3; i++ {
		fmt.Println(from, ":", i)
	}
}

func main() {
	f("direct")
	go f("goroutine")

	go func(msg string) {
		fmt.Println(msg)
	}("going")

	time.Sleep(time.Second)
	fmt.Println("done")
}
```

**WaitGroup**

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func w(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("%d starting\n", id)

	time.Sleep(time.Second)
	fmt.Printf("%d done\n", id)
}

func main() {
	var wg sync.WaitGroup
	for i := 1; i <= 5; i++ {
		wg.Add(1)
		go w(i, &wg)
	}
	wg.Wait()
}

```

**Closing channels**

```go
ch <- 1
ch <- 2
ch <- 3
close(ch) // Closes a channel

// Iterate the channel until closed
for i := range ch {
  ···
}

// Closed if `ok == false`
v, ok := <- ch
```

