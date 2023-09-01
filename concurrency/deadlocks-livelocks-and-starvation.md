# Deadlocks, Livelocks, and Starvation

#### Deadlocks

_= program in which all concurrent processes are waiting on one another._

{% code lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	ch1 := make(chan int)
	ch2 := make(chan int)

	wg.Add(1)
	go func() {
		defer wg.Done()
		// Attempting to send data to ch2, but no one is receiving.
		ch2 <- 42
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		// Attempting to send data to ch1, but no one is receiving.
		ch1 <- 23
	}()

	wg.Wait()
}
```
{% endcode %}

#### Livelocks

_= programs that are actively performing concurrent operations, but these operations do nothing to move the state of the program forward._

{% code overflow="wrap" lineNumbers="true" %}
```go
/* Two goroutines are trying to access a shared resource (condition) protected by a mutex (mu). 
They repeatedly check the condition and take actions to resolve the conflict. However, their actions lead to an ongoing conflict without making progress, resulting in a livelock.*/

package main

import (
	"fmt"
	"sync"
)

func main() {
	var mu sync.Mutex
	condition := true

	go func() {
		for {
			mu.Lock()
			if condition {
				mu.Unlock()
				continue
			}
			fmt.Println("Goroutine 1")
			condition = true
			mu.Unlock()
		}
	}()

	go func() {
		for {
			mu.Lock()
			if !condition {
				mu.Unlock()
				continue
			}
			fmt.Println("Goroutine 2")
			condition = false
			mu.Unlock()
		}
	}()

	select {}
}
```
{% endcode %}

#### Starvation

_= any situation where a concurrent process cannot get all the resources it needs to perform work._

{% code overflow="wrap" lineNumbers="true" %}
```go
/* One goroutine continually acquires and releases a lock, preventing the starving goroutine from ever acquiring it. */

package main

import (
	"fmt"
	"sync"
)

func main() {
	var mu sync.Mutex

	// Starving goroutine
	go func() {
		mu.Lock()
		fmt.Println("Starving goroutine: acquired the lock")
		mu.Unlock()
	}()

	// Goroutine that keeps the lock
	go func() {
		for {
			mu.Lock()
			fmt.Println("Lock keeper goroutine: acquired the lock")
			mu.Unlock()
		}
	}()

	select {}
}
```
{% endcode %}
