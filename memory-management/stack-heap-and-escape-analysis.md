# Stack, Heap and Escape Analysis

The Go compiler uses escape analysis to determine _which variables should be allocated on the stack of a goroutine and which variables should be allocated on the heap._

The way a variable is used - not declared - determines whether it lives on the stack or the heap.

Sharing up (returning pointers) typically escapes the Heap

{% tabs %}
{% tab title="Go" %}
The compiler will try to allocate a variable to the local stack frame of the function in which it is declared. However, it is also able to perform _escape analysis_: if it cannot prove that a variable is not referenced after the function returns, then it allocates it on the _heap_ instead.

Output: `42`

{% code lineNumbers="true" %}
```go
package main

import (
    "fmt"
)

func f() *int {
    a := 42
    return &a
}

func main() {
    defer profile.Start(profile.TraceProfile, profile.ProfilePath(".")).Stop()
    fmt.Printf("%d", *f())
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

The heap footprint increases with time. At a certain point, the GC runs and cleans up the heap, before it starts growing again.

We can rewrite the program so that there is no reference to `a` outside of the stack frame, then the variable doesn’t escape to the heap.

{% code lineNumbers="true" %}
```go
package main

import (
    "github.com/pkg/profile"
)

func f() int {
    a := 42
    return a
}

func main() {
    defer profile.Start(profile.TraceProfile, profile.ProfilePath(".")).Stop()
    for i := 0; i < 1000000; i++ {
        f()
    }
}

// go build -o ./main -gcflags=-m ./main.go
// # command-line-arguments
// ./main.go:7:6: can inline f
// ./main.go:15:4: inlining call to f
// ./main.go:13:21: ... argument does not escape
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

&#x20;`a` does not escape to the heap, since it is not modified “above” `f`’s stack frame, (only “below”, in `g`’s).

{% code lineNumbers="true" %}
```go
package main

func g(a *int) {
    *a++
}

func f() int {
    a := 42
    g(&a)
    return a
}

func main() {
    for i := 0; i < 1000000; i++ {
        f()
    }
}

// go build -o ./main -gcflags=-m ./main.go
// # command-line-arguments
// ./main.go:9:6: can inline g
// ./main.go:13:6: can inline f
// ./main.go:15:3: inlining call to g
// ./main.go:22:4: inlining call to f
// ./main.go:22:4: inlining call to g
// ./main.go:9:8: a does not escape
```
{% endcode %}

If we execute `g` in a goroutine, we find that `a` escapes to the heap.

{% code lineNumbers="true" %}
```go
package main

func g(a *int) {
    *a++
}

func f() int {
    a := 42
    go g(&a) // yes, this is a race condition, but this is just a demo :)
    return a
}

func main() {
    for i := 0; i < 1000000; i++ {
        f()
    }
}

// go build -o ./main -gcflags=-m ./main.go
// # command-line-arguments
// ./main.go:3:6: can inline g
// ./main.go:3:8: a does not escape
// ./main.go:8:2: moved to heap: a
```
{% endcode %}

This is because each goroutine has its own stack. As a result, the compiler cannot guarantee that `f`’s stack hasn’t been popped (invalidating `a`) when `g` accesses it. Therefore, the variable must live on the heap.
{% endtab %}

{% tab title="C" %}
In C, the variable lives in the stack frame of `f()`. When the function returns, the stack is popped and the memory addresses of any variables in that frame become invalid. Accessing them leads to a segmentation fault.

Output: `main.c:6:12: warning: function returns address of local variable [-Wreturn-local-addr]`\
`Segmentation fault (core dumped)`

{% code lineNumbers="true" %}
```c
#include <stdio.h>
int* f() {
    int a;
    a = 42;
    return &a;
}
void main()
{
    printf("%d", *f());
}
```
{% endcode %}
{% endtab %}
{% endtabs %}



| Stack                                                                                                                                                                                                                                                                                                                                                                        | Heap                                                                                                                                                                                                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p>space for the execution of thread; <br><br>when a function in called, a <code>block</code> (<mark style="background-color:yellow;">stack frame</mark>) is reserved on top of the stack for local variables and some bookkeeping data; <br><br>when a function returns, the block becomes <strong>unused</strong> and can be used the next time any function is called</p> | <p>requires manual housekeeping of what memory is to be reserved and what is to be cleaned<br><br>the memory allocator will perform maintenance tasks such as <mark style="background-color:yellow;">defragmenting</mark> allocated memory or <mark style="background-color:yellow;">garbage collecting</mark> </p> |
| initial stack memory allocation is done by the OS at <mark style="background-color:yellow;">compile time</mark>                                                                                                                                                                                                                                                              | allocated at <mark style="background-color:yellow;">run time</mark>                                                                                                                                                                                                                                                 |



{% tabs %}
{% tab title="Variable allocation on stack " %}
The function `main` has its local variables n and n2. `main` function is assigned a stack frame in the stack. Same goes for the function `square`.

<div align="left">

<figure><img src="../.gitbook/assets/image (4).png" alt="" width="375"><figcaption></figcaption></figure>

</div>

Now as soon as the function `square` returns the value, the variable `n2` will become `16` and the memory function `square` is **NOT** cleaned up, it will still be there and marked as **invalid** (unused).

<div align="left">

<figure><img src="../.gitbook/assets/image (6).png" alt="" width="375"><figcaption></figcaption></figure>

</div>

Now when the `Println` function is called, the same `unused` memory on stack left behind by the square function is consumed by the `Println` function.&#x20;

<div align="left">

<figure><img src="../.gitbook/assets/image (8).png" alt="" width="375"><figcaption></figcaption></figure>

</div>
{% endtab %}

{% tab title="Pointers on stack" %}
Here we have a main function which passes the reference to the variable to a function that increments the value.

<div align="left">

<figure><img src="../.gitbook/assets/image (19).png" alt="" width="375"><figcaption></figcaption></figure>

</div>

Now as function **inc** `dereferences` the pointer and increments the value that it points to, and does its work, the stack frame of `inc` is again unused/invalid (freed for other functions allocation).

<div align="left">

<figure><img src="../.gitbook/assets/image (26).png" alt="" width="375"><figcaption></figcaption></figure>

</div>

As the function `Println` runs, it acquires the memory that was freed up by the `inc` function as shown in the below figure.

<div align="left">

<figure><img src="../.gitbook/assets/image (7).png" alt="" width="375"><figcaption></figcaption></figure>

</div>
{% endtab %}

{% tab title="Pointers on heap" %}
So we have a function `main` that has a variable `n` who's value is assigned by a function `answer` which returns a `pointer` to it's local variable. This is how the stack frame allocation is done initially for both the functions

<div align="left">

<figure><img src="../.gitbook/assets/image.png" alt="" width="375"><figcaption></figcaption></figure>

</div>

Now when the function `answer` executes and returns the pointer, the address for `x` is assigned to the variable `n` in the main function and the `answer` function's stack frame gets freed up (unused)

<div align="left">

<figure><img src="../.gitbook/assets/image (9).png" alt="" width="375"><figcaption></figcaption></figure>

</div>

`Println` function takes the space freed up by the `answer` function (notice the memory address) and takes reference to the returned value and divides it by 2 (making it 21).

<div align="left">

<figure><img src="../.gitbook/assets/image (15).png" alt="" width="375"><figcaption></figcaption></figure>

</div>

The value which `n` was pointing to (which was originally 42) has been overwritten by the Println call which made it 21. And since `Println` took over the `memory address` of `answer` function (after answer function freed up the space), now that memory has the value **21** (instead of **42**)

_Compiler knows it was NOT safe to leave the pointer variable on the stack._ So what it does is, it declares `x` (from the answer function) somewhere on the **Heap**

<div align="left">

<figure><img src="../.gitbook/assets/image (27).png" alt="" width="375"><figcaption></figcaption></figure>

</div>
{% endtab %}
{% endtabs %}

Resources:

* [https://dev.to/karankumarshreds/memory-allocations-in-go-1bpa](https://dev.to/karankumarshreds/memory-allocations-in-go-1bpa)
