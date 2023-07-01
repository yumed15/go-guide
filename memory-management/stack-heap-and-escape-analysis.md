# Stack, Heap and Escape Analysis

{% hint style="danger" %}
This page is a WIP
{% endhint %}

The Go compiler uses escape analysis to determine _which variables should be allocated on the stack of a goroutine and which variables should be allocated on the heap._

The way a variable is used - not declared - determines whether it lives on the stack or the heap.

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

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

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

