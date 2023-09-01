# Functions

### Anonymous functions/Closures/Lambda Functions <a href="#first-class-functions" id="first-class-functions"></a>

\= functions that are defined and called in the same place, without a name. They retain bindings to variables defined outside the body of the closure.

{% code lineNumbers="true" %}
```go
package main

import "fmt"

func main() {

     sum := func(a, b, c int) int {
          return a + b + c
     }(3, 5, 7)

     fmt.Println("5+3+7 =", sum)
}
```
{% endcode %}

### Higher-order functions <a href="#first-class-functions" id="first-class-functions"></a>

\= functions that take one or more functions as arguments or return a function as a result.

<pre class="language-go" data-line-numbers><code class="lang-go">package main

import (
   "fmt"
)

<strong>func power(fn func(int) int) func(int) int { // &#x3C;--- high-order function
</strong>   return func(x int) int {
      return fn(x)
   }
}

func square(x int) int {
   return x * x
}

func cube(x int) int {
   return x * x * x
}

func main() {
   squareFunc := power(square)
   cubeFunc := power(cube)

   fmt.Println(squareFunc(2)) // Output: 4
   fmt.Println(cubeFunc(2))   // Output: 8
}
</code></pre>

### Monads <a href="#first-class-functions" id="first-class-functions"></a>

### Functors <a href="#first-class-functions" id="first-class-functions"></a>

### Monoids <a href="#first-class-functions" id="first-class-functions"></a>

