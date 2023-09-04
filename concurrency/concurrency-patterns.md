# Concurrency Patterns

{% hint style="danger" %}
This page is a WIP
{% endhint %}

### Worker Pool

* to manage and reuse a fixed number of worker goroutines for performing tasks concurrently.
* useful when you have a large number of tasks to execute, and you want to limit the number of concurrent executions to control resource usage.
* Implementation:
  1. Create a pool of worker goroutines.
  2. Use a channel to send tasks to the worker goroutines.
  3. Each worker goroutine continuously listens for tasks on the channel.
  4. When a task arrives, a worker picks it up and executes it.
  5. After completing a task, the worker can pick up another task from the channel if available.
  6. The worker pool continues to execute tasks until there are no more tasks, after which it can be gracefully shut down.

{% code lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"sync"
)

// Job represents a task that can be executed by a worker.
type Job struct {
	ID int
}

func main() {
	// Number of worker goroutines in the pool.
	numWorkers := 3

	// Create a task channel to send jobs to workers.
	tasks := make(chan Job, 10)

	// Create a wait group to wait for all workers to finish.
	var wg sync.WaitGroup

	// Create and start worker goroutines.
	for i := 1; i <= numWorkers; i++ {
		wg.Add(1)
		go worker(i, tasks, &wg)
	}

	// Add jobs to the task channel.
	for i := 1; i <= 5; i++ {
		tasks <- Job{ID: i}
	}

	// Close the task channel to signal that no more jobs will be added.
	close(tasks)

	// Wait for all workers to finish.
	wg.Wait()
}

// worker is a function that represents a worker goroutine.
func worker(id int, tasks <-chan Job, wg *sync.WaitGroup) {
	defer wg.Done()

	for task := range tasks {
		fmt.Printf("Worker %d is processing job %d\n", id, task.ID)
		// Simulate some work by sleeping.
		// In a real application, you would perform the actual task here.
	}
}

```
{% endcode %}

### Fan-out/Fan-in

This pattern is about distributing work across multiple goroutines (fan-out) and then combining the results (fan-in). It allows you to parallelize the processing of multiple tasks and merge the outcomes efficiently. It's particularly useful when dealing with batch processing or aggregating results from multiple sources.





1. Pipelines: Pipelines involve chaining multiple stages of processing together using channels. Each stage performs a specific operation on the incoming data and passes it to the next stage through channels. This pattern enables efficient streaming and processing of data in a structured and modular manner.
2. Context-Aware Cancellation: The context package in Go provides a mechanism for propagating cancellation signals across goroutines and coordinating the termination of related operations. It's crucial for managing resources and terminating long-running operations gracefully when cancellation is requested.
3. Rate Limiting: Rate limiting is a technique to control the rate of execution of certain operations or limit resource usage. The golang.org/x/time/rate package provides an implementation of rate limiting in Go. It's beneficial in scenarios where you need to prevent excessive requests or throttle resource-intensive operations.
4. Circuit Breaking: Circuit breaking is a pattern that helps protect systems from cascading failures. It involves monitoring the state of remote services and temporarily breaking the circuit to prevent further requests when the service is experiencing failures or becoming unresponsive. The github.com/sony/gobreaker package provides an implementation of circuit breaking in Go.
5. Mutex-Free Data Structures: Go's sync package provides mutexes for protecting shared data in concurrent scenarios. However, sometimes you can achieve better performance by using mutex-free data structures such as atomic operations, lock-free queues, or read-copy-update (RCU) data structures. These alternatives eliminate the need for locking and can improve scalability in certain cases.
6. Barrier: The barrier pattern enables synchronization among a group of goroutines, ensuring that they all reach a particular point before proceeding. It's useful when you have a set of goroutines that need to coordinate their execution and synchronize their progress.
7. Context-Specific Behavior: By utilizing Go's context package, you can introduce context-specific behavior in concurrent operations. For example, you can associate deadlines, cancellation signals, or request-scoped data with a context, allowing goroutines to be aware of and respond to such context changes.
8. Non-blocking Algorithms: In some scenarios, non-blocking algorithms can be used to avoid locks and improve concurrency. Go's atomic package provides support for atomic operations and compare-and-swap (CAS) operations, which can be utilized to build non-blocking algorithms and data structures.
