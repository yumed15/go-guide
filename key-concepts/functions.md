# Functions

### Higher-order functions <a href="#first-class-functions" id="first-class-functions"></a>

_= functions that take one or more functions as arguments or return a function as a result._

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

### Anonymous functions/Closures/Lambda Functions <a href="#first-class-functions" id="first-class-functions"></a>

_= functions that are defined and called in the same place, without a name. They retain bindings to variables defined outside the body of the closure._

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

### Monads <a href="#first-class-functions" id="first-class-functions"></a>

_= design pattern that allows us to chain operations together in a pipeline, while abstracting away the details of how the operations are performed. Has 3 components:_

* **Type Constructor (`T`):** A type that represents a computational context. It wraps a value and provides a way to apply functions to that value while preserving the context.
* **Unit Function (`unit` or `return`):** A function that takes a value and wraps it in the monadic context, creating a new instance of the monad.
* **Bind Function (`flatMap`, `chain`, `>>=`):** A function that takes a monad and a function that maps a value to a new monad. It applies the function to the value inside the monad and returns a new monad.

{% code lineNumbers="true" %}
```go
package main

import (
	"errors"
	"fmt"
)

// Result represents a monad-like type that can either hold a value or an error.
type Result struct {
	value int
	err   error
}

// Unit function wraps a value in the Result monad.
func Unit(val int) Result {
	return Result{value: val}
}

// Bind function applies a function to the value inside the Result monad.
func (r Result) Bind(f func(int) Result) Result {
	if r.err != nil {
		return r
	}
	return f(r.value)
}

// Example functions that work with Result monads.
func addOne(x int) Result {
	return Unit(x + 1)
}

func divideBy(x int, y int) Result {
	if y == 0 {
		return Result{err: errors.New("division by zero")}
	}
	return Unit(x / y)
}

func main() {
	// Using Result monad to sequence computations.
	result := Unit(10).Bind(addOne).Bind(func(x int) Result {
		return divideBy(x, 5)
	})

	if result.err != nil {
		fmt.Println("Error:", result.err)
	} else {
		fmt.Println("Result:", result.value)
	}
}

```
{% endcode %}

### Functors

\= _objects or data structures that can be mapped over, meaning you can apply a function to each element within the functor._

\= superclass of a `monad`, which means that all `monads` are `functors` as well.

\= design pattern that represents a container or structure that can be mapped over. In functional programming, a functor is often used to apply a function to each element of a collection and return a new collection with the transformed elements.

{% code lineNumbers="true" %}
```go
package main

import "fmt"

// IntList is a custom functor-like type representing a list of integers.
type IntList []int

// Map applies a given function to each element in the IntList and returns a new IntList.
func (il IntList) Map(fn func(int) int) IntList {
	result := make(IntList, len(il))
	for i, v := range il {
		result[i] = fn(v)
	}
	return result
}

func main() {
	// Creating an IntList.
	list := IntList{1, 2, 3, 4, 5}

	// Define a function to square each element.
	square := func(x int) int {
		return x * x
	}

	// Using the Map method to apply the square function to each element.
	squaredList := list.Map(square)

	// Printing the squared list.
	fmt.Println(squaredList) // Output: [1 4 9 16 25]
}

```
{% endcode %}

### Monoids <a href="#first-class-functions" id="first-class-functions"></a>

\= data types that have two key properties: associativity and an identity element.

1. **Associativity:** For all values `a`, `b`, and `c` of the same type, `(a mappend b) mappend c` should be equal to `a mappend (b mappend c)`, where `mappend` represents the operation.
2. **Identity Element:** There exists an element (usually denoted as `mempty`) such that for any value `a` of the same type, `mempty mappend a` is equal to `a` and `a mappend mempty` is equal to `a`.

{% code lineNumbers="true" %}
```go
package main

import "fmt"

// IntMonoid is a custom monoid-like type representing integers with addition as the operation.
type IntMonoid int

// Mappend defines the addition operation for IntMonoid.
func (a IntMonoid) Mappend(b IntMonoid) IntMonoid {
	return a + b
}

// Mempty represents the identity element for IntMonoid, which is 0 for addition.
func (IntMonoid) Mempty() IntMonoid {
	return 0
}

func main() {
	// Create instances of IntMonoid.
	x := IntMonoid(5)
	y := IntMonoid(10)

	// Use the Mappend method to combine values.
	result := x.Mappend(y)

	// Print the result.
	fmt.Println(result) // Output: 15
}

```
{% endcode %}

