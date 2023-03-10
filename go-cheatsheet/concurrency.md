# Concurrency

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

