# Go/Java Concurrency

“Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once.” - Rob Pike

Concurrency is a property of the code; parallelism is a property of the running program.

> Concurrency is a _**semantic property of a program or system**_.  Concurrency is when multiple tasks are in progress for overlapping periods of time. Concurrency is a conceptual property of a program or a system, it’s more about how the program or system has been designed. Long story short, concurrency happens when you have context switching between sequential tasks.

Using the same example as Kirill Bobrov uses in [Grokking Concurrency](https://www.manning.com/books/grokking-concurrency), imagine that one cook is chopping salad while occasionally stirring the soup on the stove. He has to stop chopping, check the stove top, and then start chopping again, and repeat this process until everything is done.

As you can see, we only have one processing resource here, the chef, and his concurrency is mostly related to logistics; without concurrency, the chef has to wait until the soup on the stove is ready to chop the salad.

> Parallelism is an _**implementation property**_. It resides on the hardware layer.
>
> Parallelism is about multiple tasks or subtasks of the same task that literally run at the same time on a hardware with multiple computing resources like multi-core processor.

Back in the kitchen, now we have two chefs, one who can do stirring and one who can chop the salad. We’ve divided the work by having another processing resource, another chef.

{% hint style="info" %}
Concurrency can be parallelised but concurrency does not imply parallelism.\
e.g. In a single-core CPU, you can have concurrency but not parallelism.
{% endhint %}

\=> We don't write parallel code, only concurrent code that we hope might be ran in parallel.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

## Concurrency in Go vs Java

<table><thead><tr><th width="170.33333333333331">Concept</th><th>Go</th><th>Java</th></tr></thead><tbody><tr><td>Multithreading</td><td>through <strong>goroutines</strong></td><td>through threads via <strong>Thread</strong> class or <strong>Runnable</strong> interface</td></tr><tr><td>Memory Space</td><td>goroutines use only <mark style="color:yellow;"><strong>2 KB</strong></mark><strong> of memory space</strong>.</td><td>threads take <mark style="color:yellow;"><strong>2 MB</strong></mark><strong> of memory space</strong></td></tr><tr><td>Communication Coordination</td><td>through built in <mark style="color:yellow;"><strong>primivate channels</strong></mark> which are built to handle race conditions => safe and prevents explicit locking; <br><br>the data structure that is shared between goroutines doesn't have to be locked</td><td><ul><li>Threaded programming uses <mark style="color:yellow;"><strong>locks</strong></mark> in order to access a shared variable. These can to lead to deadlocks and race conditions which are difficult to detect.</li><li>Can only speak to one another through <mark style="color:yellow;"><strong>return values</strong></mark> or <mark style="color:yellow;"><strong>shared (volatile) variables</strong></mark> and are highly costly to build and manage.</li></ul></td></tr><tr><td>Scheduling </td><td>scheduling of goroutines is done by <mark style="color:yellow;"><strong>go runtime</strong></mark> and hence it is quite faster => context switching is faster</td><td>the scheduling of threads is done by <mark style="color:yellow;"><strong>OS runtime</strong></mark> => context switching is slower</td></tr><tr><td>Garbage Collection</td><td>not automatically garbage collected</td><td>Once the thread dies its native memory and stack are freed immediately without needing to be GC. However, the <code>Thread</code> object is like any other object and it lives until it the GC has decided it can be freed e.g. there is no strong reference to it.</td></tr><tr><td></td><td>thousands of goroutines are multiplexed on one or two OS threads.</td><td>if you launch 1000 threads in JAVA then it would consume lot of resources and these 1000 threads needs to be managed by OS. Moreover each of these threads will be more than 1 MB in size</td></tr></tbody></table>

## **Scheduling**

Go's mechanism for hosting goroutines is an implementation of what's called an <mark style="color:yellow;">**M:N scheduler**</mark>: which states that <mark style="color:yellow;">**M**</mark> number of goroutines can be distributed over <mark style="color:yellow;">**N**</mark> number of OS threads.

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

When a Go program starts => it is given a logical processor **P** for every virtual core => Every P is assigned an OS thread **M** => Every Go program is also given an initial G which is the path of execution for a Go program. OS threads are context-switched on and off a core, goroutines are context-switched on and off a M.

There are two run queues in the Go scheduler.

* **Global Run Queue (GRQ)**
* **Local Run Queue (LRQ)**

Each P is given given a LRQ that manages the goroutines assigned to be executed within the context of P. These goroutines take turn being context-switched on and off the M assigned to that P. GRQ is for goroutines that have not been assigned to a P yet.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

When a goroutine is performing an asynchronous system call, P can swap the G off M and put in a different G for execution. However, when a goroutine is performing a synchronous system call, the OS thread is effectively blocked. Go scheduler will create a new thread to continue servicing the existing goroutines in the LRQ.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Go follows a model of concurrency called the <mark style="color:yellow;">**fork-join model**</mark>:

* fork - at any point in the program, a _**child**_ branch of execution can be split off and run concurrently with its _**parent**_
* join - at some point in the future, the concurrent branches of execution will join back together

![](<../.gitbook/assets/image (5).png>)

## **Ways of declaring goroutines** <a href="#goroutines" id="goroutines"></a>

```go
func main() {
    go sayHello()
}

func sayHello() {
    fmt.Println("hello")
}
```

```go
go func() {
    fmt.Println("hello")
}() // <--- the anonymous function must be invoked immediately
```

```go
sayHello := func() {
    fmt.Println("hello")
}
go sayHello()
```

## **Synchronizing goroutines -** via sync package <a href="#goroutines" id="goroutines"></a>

To make sure your goroutines execute before the main goroutine we need <mark style="color:yellow;">**join points**</mark>. These can be created via:

### WaitGroup primitive

_for waiting for a set of concurrent operations to complete when you either don't care about the result of the concurrent operation, or you have other means of collecting their results_

<pre class="language-go" data-line-numbers><code class="lang-go">var wg sync.WaitGroup
sayHello := func() {
<strong>    defer wg.Done() // &#x3C;- before we exit the goroutine, we indicate to the WaitGroup that we have exited
</strong>    fmt.Println("hello")
}
<strong>wg.add(1) // &#x3C;- one goroutine is starting
</strong>go sayHello() 
<strong>wg.Wait() // &#x3C;---- join point
</strong></code></pre>

**Closures** = _a function value that references variables from outside its body._

With closures, we'd have **to pass a copy of the variable** into the closure so by the time a goroutine is run, it will be operating on the data from its iteration of the loop.

{% tabs %}
{% tab title="BAD" %}
{% code lineNumbers="true" %}
```go
var wg sync.WaitGroup
for _, salutation := range []string{"hello", "greetings", "good day"} {
    wg.Add(1)
    go func() {
        defer wg.Done()
        fmt.Println(salutation)
    }()
}
wg.Wait()

// good day
// good day
// good day
```
{% endcode %}
{% endtab %}

{% tab title="GOOD" %}
<pre class="language-go" data-line-numbers><code class="lang-go">var wg sync.WaitGroup
for _, salutation := range []string{"hello", "greetings", "good day"} {
    wg.Add(1)
<strong>    go func(salutation string) {
</strong>        defer wg.Done()
        fmt.Println(salutation)
<strong>    }(salutation) // &#x3C;-- we pass in the current iteration's variable to the closure. 
</strong>                  // a copy of the string struct is made
                  // when the goroutine is run, we'll be refering to the proper string
}
wg.Wait()

// good day
// hello
// greetings
</code></pre>
{% endtab %}
{% endtabs %}

### Mutex

_provides a concurrent-safe way to express exclusive access to these shared resources._

**`sync.Mutex`** interface with **`Lock()`** and **`Unlock()`** methods

{% hint style="warning" %}
Shares memory by creating a convention developers must follow to synchronise access to the memory.
{% endhint %}

<pre class="language-go" data-line-numbers><code class="lang-go">var count int
var lock sync.Mutex

increment := func() {
<strong>    lock.Lock() // &#x3C;--- locking section
</strong><strong>    defer lock.Unlock() // &#x3C;--- unlocking
</strong>    count++
}

decrement := func() {
<strong>    lock.Lock() 
</strong><strong>    defer lock.Unlock()
</strong>    count--
}

var arithmetic sync.WaitGroup
for i := 0; i&#x3C;=5; i++ {
    arithmetic.Add(1)
    go func() {
        defer arithmetic.Done()
        increment()
    }
}

for i := 0; i&#x3C;=5; i++ {
    arithmetic.Add(1)
    go func() {
        defer arithmetic.Done()
        decrement()
    }
}

</code></pre>

### RWMutex

_same as Mutex but it provides a read/write lock. We can have a multiple number of readers holding a reader lock as long as nobody is holding a writer lock._

**`sync.RWMutex`** interface with **`RLock()`** and **`RUnlock()`** methods

&#x20;**RWMutex can only be held by n readers at a time, or by a single writer**

<figure><img src="../.gitbook/assets/Microservice Communication (4).jpg" alt=""><figcaption></figcaption></figure>

### Cond

_a rendezvous point for goroutines waiting for or announcing the occurence of an event (=signal between 2 or more goroutines, has no info other than it happened)._

**`sync.NewCond(&sync.Mutex{})`** with 2 methods&#x20;

* **`Signal`** - notifies goroutines (runtime picks the one that has been waiting the longest) blocked on a `Wait` call that the condition has been triggered

{% code lineNumbers="true" %}
```go
c := sync.NewCond(&sync.Mutex{})
c.L.Lock()
for conditionTrue() == false {
    c.Wait() // <--- we wait to be notified that the condition has occurred
             // this is a blocking call and the goroutine will be suspended
             // allows other goroutines to run on the OS thread
}
c.L.Unlock()
```
{% endcode %}

{% code lineNumbers="true" %}
```go
c := sync.NewCond(&sync.Mutex{})
queue := make([]interface{}, 0, 10)

removeFromQueue := func(delay time.Duration) {
    time.Sleep(delay)
    c.L.Lock()
    queue = queue[1:]
    c.L.Unlock()
    c.Signal() // <- let a goroutine waiting for a condition to know that smth happened
}

for i:=0; i<10; i++ {
    c.L.Lock()
    for len(queue) == 2 {
        c.Wait()
    }
    queue = append(queue, struct{}{})
    go removeFromQueue(1*time.Second)
    c.L.Unlock()
}
```
{% endcode %}

**`Brodcast`** - sends signal to all waiting goroutines

<pre class="language-go" data-line-numbers><code class="lang-go">type Button struct { // contains a condition
<strong>    Clicked *sync.Cond
</strong>}

<strong>button := Button{Clicked: sync.NewCond(&#x26;sync.Mutex{})}
</strong>
subscribe := func(c *sync.Cond, fn func()) { // allows us to register functions
    var goroutineRunning sync.WaitGroup      // to handle signals from conditions
    goroutineRunning.Add(1)
    
    go func() {
        goroutineRunning.Done()
<strong>        c.L.Lock()
</strong><strong>        defer c.L.Unlock()
</strong>        c.Wait()
        fn()
    }()
    goroutineRunning.Wait()
}
var clickRegistered sync.WaitGroup
clickRegistered.Add(3)
<strong>subscribe(button.Clicked, func() {
</strong>    // do smth
    clickRegistered.Done()
})
subscribe(button.Clicked, func() {
    // do smth
    clickRegistered.Done()
})
subscribe(button.Clicked, func() {
    // do smth
    clickRegistered.Done()
})
    
<strong>button.Clicked.Broadcast()
</strong>clickRegistered.Wait()
</code></pre>

### Once

ensures that only **one call to `Do`** ever calls the function passed in

{% hint style="danger" %}
counts the **no of times `Do`** is called**, not how many unique functions passed into `Do`** are called
{% endhint %}

<pre class="language-go" data-line-numbers><code class="lang-go">var count int
increment := func() {count++}
decrement := func() {count--}

var once sync.Once
<strong>once.Do(increment)
</strong><strong>once.Do(decrement)
</strong>
fmt.Println(count) // 1
</code></pre>

### Pool

_= concurrent-safe implementation of the object pool pattern._

**`Get`** interface - checks wether the are any available instances within the pool to return to the caller, and if not, call its **`New`** member variable to create one.

**`Put`** interface - to put the instance they were working with back in the pool

{% code lineNumbers="true" %}
```go
myPool := &sync.Pool{
    New: func() interface{}{
        fmt.Println("Creating new instance.")
        return struct{}{}
    }
}

myPool.Get() // no instance available, calls New;
instance := myPool.Get() // no instance available, calls New;
myPool.Put(instance) // instances in pool = 1
myPool.Get() // get instance from pool

// Creating new instance.
// Creating new instance.
```
{% endcode %}

**Uses cases:**&#x20;

* **memory optimisations** as instantiated objects are garbage collected.

{% code lineNumbers="true" %}
```go
var numCalcsCreated int
calclPool := &sync.Pool{
    New: func() interface{}{
        numCalcsCreated += 1
        mem := make([]byte, 1024)
        return &mem
    },
}

// Seed the pool with 4KB
calclPool.Put(calclPool.New())
calclPool.Put(calclPool.New())
calclPool.Put(calclPool.New())
calclPool.Put(calclPool.New())

const numWorkers = 1024*1024
var wg sync.WaitGroup
wg.Add(numWorkers)

for i:=numWorkers; i>0; i-- {
    go func() {
        defer wg.Done()
        
        mem := calcPool.Get().(*[]byte)
        defer calcPool.Put(mem)
    }()
}

wg.Wait()
fmt.Printf(numCalcsCreated) // 8
```
{% endcode %}

* **warming up a cache of pre-allocated objects** for operations that must run as quickly as possible. (by trying to guard the host machine's memory front-loading the time it takes to get a reference to another object)

<pre class="language-go" data-line-numbers><code class="lang-go">func warmServiceConnCache() *sync.Pool {
    p := &#x26;sync.Pool {
        New: connectToService,
    }
    for i:=0; i&#x3C;10; i++ {
        p.Put(p.New())
    }
    return p
}

funct startNetworkDaemin() *sync.WaitGroup {
    var wg sync.WaitGroup
    wg.Add(1)
    go func() {
<strong>        connPool := warmServiceConnCache()
</strong>        
        server, err := net.Listen("tcp", "localhost:8080")
        if err != nil {
            log.Fatalf("cannot listem: %v", err)
        }
        defer server.Close()
        
        wg.Done()
        
        for {
            conn, err := server.Accept()
            if err != nil {
                log.Printf("cannot accept connection: %v", err)
                continue
            }
<strong>            svcConn := connPool.Get()
</strong>            fmt.Fprintln(conn, "")
<strong>            connPool.Put(svcConn)
</strong>            conn.Close()
        }
    }()
    return &#x26;wg
}
</code></pre>

### Channels

can be used to synchronise access of the memory and to communicate information between goroutines.&#x20;

#### **Instantiating**

{% code lineNumbers="true" %}
```go
var dataStream chan interface{}
dataStream = make(chan interface{})
```
{% endcode %}

support **unidirectional** flow of data:&#x20;

* channel that can only **read**

```go
var dataStream <-chan interface{}
dataStream = make(<-chan interface{})
```

* channel that can only **send**

```go
var dataStream chan<- interface{}
dataStream = make(chan<- interface{})
```

**Sending/Receiving**&#x20;

{% code lineNumbers="true" %}
```go
stringStream := make(chan string)
go func() {
    stringStream <- "hello" // pass literal onto channel
}()
fmt.Println(<-stringStreams) // read the literal from channel
```
{% endcode %}

{% hint style="danger" %}
* Channels are _**blocking**_.
* Any goroutine that attempts to write to a channel that is full will wait until the channel has been emptied.
* Any goroutine that attempts to read from a channel that is empty will wait until at least one item is placed on it.
{% endhint %}

#### Closing&#x20;

{% code lineNumbers="true" %}
```go
stringStream := make(chan string)
close(stringStream)
```
{% endcode %}

#### Ranging over a channel

{% code lineNumbers="true" %}
```go
intStream := make(chan int)
go func() {
    defer close(intStream)
    for i:=1; i<=5; i++ {
        intStream <- i
    }
}()

for integer := range intStream {
    fmt.Printf("%v ", integer)
}

// 1 2 3 4 5
```
{% endcode %}

#### Closing a channel signals to multiple goroutines

<pre class="language-go" data-line-numbers><code class="lang-go"><strong>begin := make(chan interface{})
</strong>var wg sync.WaitGroup
for i:=0; i&#x3C;5; i++ {
    wg.Add(1)
    go func(i int) {
        defer wg.Done()
<strong>        &#x3C;- begin
</strong>        fmt.Printf("%v has begun\n", i)
    }()
}

fmt.Println("unblocking goroutines...")
<strong>close(begin)
</strong>wg.Wait()
</code></pre>

#### Buffered Channels

_channels that are given a capacity when they're instantiated._

{% code lineNumbers="true" %}
```go
var dataStream chan interface{}
dataStream = make(chan interface{}, 4)
```
{% endcode %}

{% hint style="warning" %}
Buffered channels are in-memory FIFO queue for concurrent processes to communicate over.
{% endhint %}

#### Result of channel operation given a channel's state

<table><thead><tr><th width="183.33333333333331">Operation</th><th>Channel state</th><th>Result</th></tr></thead><tbody><tr><td>Read</td><td>nil</td><td>block</td></tr><tr><td></td><td>open and non empty</td><td>value</td></tr><tr><td></td><td>open and empty</td><td>block</td></tr><tr><td></td><td>closed</td><td>&#x3C;default value>, false</td></tr><tr><td></td><td>write only</td><td>compilation error</td></tr><tr><td>Write</td><td>nil</td><td>block</td></tr><tr><td></td><td>open and non empty</td><td>block</td></tr><tr><td></td><td>open and empty</td><td>write value</td></tr><tr><td></td><td>closed</td><td>panic</td></tr><tr><td></td><td>receive only</td><td>compilation error</td></tr><tr><td>close</td><td>nil</td><td>panic</td></tr><tr><td></td><td>open and non empty</td><td>closes channel; reads suceed until channel is drained, then reads produce default value</td></tr><tr><td></td><td>open and empty</td><td>closes channel; reads produces default value</td></tr><tr><td></td><td>closed</td><td>panic</td></tr><tr><td></td><td>receive only</td><td>compilation error</td></tr></tbody></table>

#### Channel Owners\&Consumers

A channel owner should:

* instantiate the channel.
* perform writes, or pass ownership to another goroutine.
* close the channel.
* encapsulate the previous three things in this list and expose them via a reader channel.

A channel consumer should:&#x20;

* knowing when a channel is closed.
* be responsible for handling blocking for any reason.

<pre class="language-go" data-line-numbers><code class="lang-go">chanOwner := func() &#x3C;- chan int {
<strong>    resultStream := make(chan int, 5) // create buffered channel
</strong>    go func() {
<strong>        defer close(resultStream) // close the channel
</strong>        for i:=0; i&#x3C;=5; i++ {
<strong>            resultStream &#x3C;- i // write to it
</strong>        }()
<strong>        return resultStream // return the read-only channel
</strong>    }
}

resultStream := chanOwner()
for result := range resultStream {
    fmt.Printf("received: %d\n", result)
}
fmt.Printf("done receiving")
</code></pre>

## Deadlocks, Livelocks, and Starvation

#### Deadlocks

_= program in which all concurrent processes are waiting on one another._

#### Livelocks

_= programs that are actively performing concurrent operations, but these operations do nothing to move the state of the program forward._

#### Starvation

_= any situation where a concurrent process cannot get all the resources it needs to perform work._

## Race Condition and Data Race

**Data race** _is when one concurrent operation attempts to read a variable while at some undetermined time another concurrent operation is attempting to write to the same variable._&#x20;

* occurs when two goroutines access the same variable concurrently and at least one of the accesses is a write

{% tabs %}
{% tab title="Data Race Condition" %}
```go
package main
import "fmt"

func main() {
    number := 0;
    
    go func(){
      number++ //reading and modifying the value of 'number'
    }()

    fmt.Println(number) //reading the value of 'number'
}
```
{% endtab %}
{% endtabs %}

**Race condition** _occur when two or more operations must execute in the correct order, but the program has not been written so that this order is guaranteed to be maintained._

{% tabs %}
{% tab title="Race Confition" %}
```go
package main
import "fmt"

func deposit(balance *int,amount int){
    *balance += amount //add amount to balance
}

func withdraw(balance *int, amount int){
    *balance -= amount //subtract amount from balance
}

func main() {
    balance := 100 
    
    go deposit(&balance,10) //depositing 10

    withdraw(&balance, 50) //withdrawing 50

    fmt.Println(balance) 
}
```
{% endtab %}
{% endtabs %}

\=> Can use the _Go race detector_ to detect potential issues.

## Patterns

#### Confinement



