# Interview Cheatsheet

### **Keywords**

<mark style="color:yellow;">**defer**</mark> = language feature that allows you to schedule a function call to be executed when the surrounding function completes, regardless of how it completes

<mark style="color:yellow;">**context**</mark> = provides a way to propagate request-scoped values, deadlines, and cancellations across Goroutines, allowing for better control and management of concurrent operations.



### Types

* basic: **numbers**, **booleans**, **strings**
* aggregate: **arrays**, **structs**
* reference: **pointers**, **slices**, **maps**, **functions**, **channels**
  * slices consists of 3 elements: pointers(used to store the memory address of another variable), length(no of elements present in array) and capacity(size to which an array can expand or shrink)
  * functions in a slice: copy, append
* **interface**

```go
// Remove from slice at index
func remove(slice []int, index int) []int {
    return append(slice[:index], slice[index+1:]...)
}
```



### **Memory Management**&#x20;

<mark style="color:yellow;">**escape analysis**</mark> = method used to determine which variables should be allocated on the stack of a goroutine and which variables should be allocated on the heap.

* The way a variable is used - not declared - determines whether it lives on the stack or the heap.
* **stack** - memory allocated for each thread - managed by the compiler/runtime - stores function call frames, local variables, fixed size variables
* **heap** - memory used to store data in your program - managed by the GC - used for large data, sending pointers or values containing pointers to channels, storing pointers or values containing pointers in a slice, calling methods on an interface type

<mark style="color:yellow;">**memory leaks**</mark> - happen when allocated memory is not properly released or deallocated by the program.&#x20;

* a string is allocated memory inside an infinite loop



### **Concurrency**

<mark style="color:yellow;">**goroutines**</mark> = fundamental concurrency feature in Go that allow concurrent execution of lightweight, independently scheduled functions or methods. They provide a simple and efficient way to achieve concurrency and parallelism in Go-based applications.

* managed by the Go runtime (unlike threads that are managed by the os's kernel)

<mark style="color:yellow;">**mutex**</mark>** =** a synchronization primitive used to protect shared resources from concurrent access. It provides a way to allow only one Goroutine to access the protected resource at a time, ensuring exclusive access and preventing data races.

<mark style="color:yellow;">**channels**</mark> = core language feature used for communication and synchronization between Goroutines. They provide a way for Goroutines to send and receive values concurrently, ensuring safe communication and coordination.

<mark style="color:yellow;">**deadlocks**</mark> = program in which all concurrent processes are waiting on one another.

<mark style="color:yellow;">**livelocks**</mark> = programs that are actively performing concurrent operations, but these operations do nothing to move the state of the program forward.

<mark style="color:yellow;">**starvation**</mark> = any situation where a concurrent process cannot get all the resources it needs to perform work.

<mark style="color:yellow;">**data race**</mark> - when one concurrent operation attempts to read a variable while at some undetermined time another concurrent operation is attempting to write to the same variable.

* occurs when two goroutines access the same variable concurrently and at least one of the accesses is a write

<mark style="color:yellow;">**race condition**</mark> - occurs when two or more operations must execute in the correct order, but the program has not been written so that this order is guaranteed to be maintained.

<mark style="color:yellow;">**goroutine pool**</mark> = design pattern that involves pre-creating a fixed number of Goroutines to handle tasks concurrently. Used for:

1. to limit the number of concurrent Goroutines and efficiently manage task execution
2. avoid the overhead of creating and destroying Goroutines for each task, resulting in better performance and resource management.



### Patterns

<mark style="color:yellow;">**dependency injection**</mark> = design pattern that allows the separation of concerns by providing dependencies to an object from the outside rather than creating them internally. Used for:

* to achieve loose coupling between components and improve testability, reusability, and maintainability of the codebase.
* via constructor injection = passing dependencies as parameters to a struct’s constructor function
* via method injection = passing dependencies as arguments to specific methods that require them.

<mark style="color:yellow;">**interface**</mark> = collection of method signatures that define a behavior. It specifies what methods a concrete type must implement to satisfy the interface contract. Interfaces provide a way to achieve polymorphism and decoupling in Go programs.

