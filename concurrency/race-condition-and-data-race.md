# Race Condition and Data Race

**Data race** _is when one concurrent operation attempts to read a variable while at some undetermined time another concurrent operation is attempting to write to the same variable._&#x20;

* occurs when two goroutines access the same variable concurrently and at least one of the accesses is a write

{% tabs %}
{% tab title="Data Race Condition" %}
```go
package main
import "fmt"

func main() {
   wait := make(chan struct{})
   n := 0
   go func() {
      n++ // read, increment, write
      close(wait)
   }()
   n++ // conflicting access
   <-wait
   fmt.Println(n) // Output: <unspecified>
}
```
{% endtab %}

{% tab title="Fix" %}
{% code lineNumbers="true" %}
```go
func main() {
   ch := make(chan int)
   go func() {
      n := 0 // A local variable is only visible to one goroutine.
      n++
      ch <- n // The data leaves one goroutine...
   }()
   
   n := <-ch // ...and arrives safely in another.
   n++
   fmt.Println(n) // Output: 2
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

**Race condition** _occurs when two or more operations must execute in the correct order, but the program has not been written so that this order is guaranteed to be maintained._

{% tabs %}
{% tab title="Race Confition" %}
```go
package main
import "fmt"

func deposit(balance *int, amount int){
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
